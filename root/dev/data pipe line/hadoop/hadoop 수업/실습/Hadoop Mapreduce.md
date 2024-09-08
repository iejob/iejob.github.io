---
tags:
  - hadoop
  - mapreduce
  - code
create: 2024-06-25 23:06:57
---

## World Count

[[vscode maven 프로젝트 생성]]

- 프로젝트 이름: mapreduce


### 디렉토리 구조

```
.
`-- mapreduce
    |-- data
    |   `-- word.txt
    |-- pom.xml
    `-- src
        `-- main
            `-- java
                `-- org
                    `-- example
                        |-- WordCountDriver.java
                        |-- WordCountMapper.java
                        `-- WordCountReducer.java
```

### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.example</groupId>
  <artifactId>mapreduce</artifactId>
  <version>1.0-SNAPSHOT</version>

  <name>mapreduce</name>
  <!-- FIXME change it to the project's website -->
  <url>http://www.example.com</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.7</maven.compiler.source>
    <maven.compiler.target>1.7</maven.compiler.target>
    <hadoop.version>3.3.6</hadoop.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
      <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-common</artifactId>
        <version>${hadoop.version}</version>
      </dependency>
      <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-client</artifactId>
        <version>${hadoop.version}</version>
      </dependency>
      <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-mapreduce-client-core</artifactId>
        <version>${hadoop.version}</version>
      </dependency>
  </dependencies>

</project>

```


### WordCountMapper.java

```java
package org.example;

import org.apache.hadoop.io.IntWritable; // Hadoop의 IntWritable 클래스 임포트
import org.apache.hadoop.io.LongWritable; // Hadoop의 LongWritable 클래스 임포트
import org.apache.hadoop.io.Text; // Hadoop의 Text 클래스 임포트
import org.apache.hadoop.mapreduce.Mapper; // Hadoop의 Mapper 클래스 임포트

import java.io.IOException; // 입출력 예외 처리를 위한 IOException 클래스 임포트

public class WordCountMapper extends Mapper<LongWritable, Text, Text, IntWritable> {

	private Text outputKey = new Text(); // 출력 키를 저장할 Text 객체 생성
	private IntWritable outputValue = new IntWritable(1); // 출력 값을 저장할 IntWritable 객체 생성 (항상 1로 설정)

	@Override
	protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
		// 한 줄의 텍스트를 String으로 변환
		String line = value.toString();
		// 공백을 기준으로 단어를 분리하여 배열에 저장
		String[] words = line.split(" ");
		// 분리된 각 단어에 대해
		for (String word : words) {
			// 단어를 outputKey에 설정
			outputKey.set(word);
			// (단어, 1) 쌍을 컨텍스트에 기록
			context.write(outputKey, outputValue);
		}
	}

}
```


### WordCountReducer.java

```java
package org.example; // 패키지 선언

import org.apache.hadoop.io.IntWritable; // Hadoop의 IntWritable 클래스 임포트
import org.apache.hadoop.io.Text; // Hadoop의 Text 클래스 임포트
import org.apache.hadoop.mapreduce.Reducer; // Hadoop의 Reducer 클래스 임포트
import java.io.IOException; // 입출력 예외 처리를 위한 IOException 클래스 임포트

public class WordCountReducer extends Reducer<Text, IntWritable, Text, IntWritable> {

    private IntWritable result = new IntWritable(); // 최종 결과 값을 저장할 IntWritable 객체 생성

    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        // 값들을 합산할 변수 초기화
        int sum = 0;
        // 각 값에 대해
        for (IntWritable value : values) {
            // 현재 값을 sum에 더함
            sum += value.get();
        }
        // 합산된 결과를 result에 설정
        result.set(sum);
        // (단어, 합계) 쌍을 컨텍스트에 기록
        context.write(key, result);
    }

}
```


### WordCountDriver.java
```java
package org.example;

import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

public class WordCountDriver extends Configured implements Tool {

    // Tool 인터페이스의 run 메서드를 오버라이드합니다.
    @Override
    public int run(String[] args) throws Exception {
        // 잡 인스턴스를 생성합니다. 현재 설정을 기반으로 "Word Count"라는 이름을 사용합니다.
        Job job = Job.getInstance(getConf(), "Word Count");

        // 잡의 JAR 파일을 이 클래스의 JAR 파일로 설정합니다.
        job.setJarByClass(getClass());

        // 맵퍼 클래스를 설정합니다.
        job.setMapperClass(WordCountMapper.class);

        // 리듀서 클래스를 설정합니다.
        job.setReducerClass(WordCountReducer.class);

        // 컴바이너 클래스를 설정합니다. (여기서는 리듀서와 동일하게 설정)
        job.setCombinerClass(WordCountReducer.class);

        // 출력 키 클래스를 설정합니다.
        job.setOutputKeyClass(Text.class);

        // 출력 값 클래스를 설정합니다.
        job.setOutputValueClass(IntWritable.class);

        // 입력 경로를 설정합니다.
        FileInputFormat.addInputPath(job, new Path(args[0]));

        // 출력 경로를 설정합니다.
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        // 잡이 완료될 때까지 기다리고, 성공하면 0을 반환하고 실패하면 1을 반환합니다.
        return job.waitForCompletion(true) ? 0 : 1;
    }

    public static void main(String[] args) {
        if (args.length != 2) {
            System.err.println("Usage: WordCountDriver <input path> <output path>");
            System.exit(-1);
        }
        try {
            int exitCode = ToolRunner.run(new WordCountDriver(), args);
            System.exit(exitCode);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```


### word.txt
```txt
Deer Bear River
Car Car River
Deer Car Bear
```


### 전처리

```sh
hadoop fs -mkdir -p /user/ubuntu/input
hadoop fs -put ./word.txt /user/ubuntu/input/
hadoop fs -ls /user/ubuntu/input/
```

### 빌드

![[Pasted image 20240625155903.png]]

1. 클릭: 탐색기
2. 우클릭: maven
3. 클릭: Run Maven commands
4. 클릭: install
5. 확인


### 실행

![[Pasted image 20240626145053.png]]

```sh
yarn jar 1 org.example.WordCountDriver 2 /user/ubuntu/output/
```

```ad-attention
이미지 1, 2를 shell의 1, 2 위치로 붙여넣기
```

```sh
yarn jar /home/ubuntu/work/hadoop/mapreduce/mapreduce/target/mapreduce-1.0-SNAPSHOT.jar org.example.WordCountDriver /user/ubuntu/input/word.txt /user/ubuntu/output/
```

### 출력

```sh
2024-06-26 14:39:02,936 INFO client.RMProxy: Connecting to ResourceManager at /0.0.0.0:8032
2024-06-26 14:39:03,181 INFO mapreduce.JobResourceUploader: Disabling Erasure Coding for path: /tmp/hadoop-yarn/staging/ubuntu/.staging/job_1719361466459_0001
2024-06-26 14:39:03,317 INFO input.FileInputFormat: Total input files to process : 1
2024-06-26 14:39:03,368 INFO mapreduce.JobSubmitter: number of splits:1
2024-06-26 14:39:03,454 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1719361466459_0001
2024-06-26 14:39:03,456 INFO mapreduce.JobSubmitter: Executing with tokens: []
2024-06-26 14:39:03,561 INFO conf.Configuration: resource-types.xml not found
2024-06-26 14:39:03,561 INFO resource.ResourceUtils: Unable to find 'resource-types.xml'.
2024-06-26 14:39:03,705 INFO impl.YarnClientImpl: Submitted application application_1719361466459_0001
2024-06-26 14:39:03,733 INFO mapreduce.Job: The url to track the job: http://66a2aa90a7fa:8088/proxy/application_1719361466459_0001/
2024-06-26 14:39:03,733 INFO mapreduce.Job: Running job: job_1719361466459_0001
2024-06-26 14:39:08,795 INFO mapreduce.Job: Job job_1719361466459_0001 running in uber mode : false
2024-06-26 14:39:08,795 INFO mapreduce.Job:  map 0% reduce 0%
2024-06-26 14:39:11,838 INFO mapreduce.Job:  map 100% reduce 0%
2024-06-26 14:39:15,854 INFO mapreduce.Job:  map 100% reduce 100%
2024-06-26 14:39:15,860 INFO mapreduce.Job: Job job_1719361466459_0001 completed successfully
2024-06-26 14:39:15,925 INFO mapreduce.Job: Counters: 54
        File System Counters
                FILE: Number of bytes read=50
                FILE: Number of bytes written=472597
                FILE: Number of read operations=0
                FILE: Number of large read operations=0
                FILE: Number of write operations=0
                HDFS: Number of bytes read=157
                HDFS: Number of bytes written=28
                HDFS: Number of read operations=8
                HDFS: Number of large read operations=0
                HDFS: Number of write operations=2
                HDFS: Number of bytes read erasure-coded=0
        Job Counters 
                Launched map tasks=1
                Launched reduce tasks=1
                Data-local map tasks=1
                Total time spent by all maps in occupied slots (ms)=1414
                Total time spent by all reduces in occupied slots (ms)=1521
                Total time spent by all map tasks (ms)=1414
                Total time spent by all reduce tasks (ms)=1521
                Total vcore-milliseconds taken by all map tasks=1414
                Total vcore-milliseconds taken by all reduce tasks=1521
                Total megabyte-milliseconds taken by all map tasks=1447936
                Total megabyte-milliseconds taken by all reduce tasks=1557504
        Map-Reduce Framework
                Map input records=3
                Map output records=9
                Map output bytes=80
                Map output materialized bytes=50
                Input split bytes=113
                Combine input records=9
                Combine output records=4
                Reduce input groups=4
                Reduce shuffle bytes=50
                Reduce input records=4
                Reduce output records=4
                Spilled Records=8
                Shuffled Maps =1
                Failed Shuffles=0
                Merged Map outputs=1
                GC time elapsed (ms)=68
                CPU time spent (ms)=590
                Physical memory (bytes) snapshot=495579136
                Virtual memory (bytes) snapshot=5120688128
                Total committed heap usage (bytes)=458752000
                Peak Map Physical memory (bytes)=296083456
                Peak Map Virtual memory (bytes)=2556694528
                Peak Reduce Physical memory (bytes)=199495680
                Peak Reduce Virtual memory (bytes)=2563993600
        Shuffle Errors
                BAD_ID=0
                CONNECTION=0
                IO_ERROR=0
                WRONG_LENGTH=0
                WRONG_MAP=0
                WRONG_REDUCE=0
        File Input Format Counters 
                Bytes Read=44
        File Output Format Counters 
                Bytes Written=28
```


### 확인
#### output
```sh
hadoop dfs -ls /user/ubuntu/output
```

```sh
WARNING: Use of this script to execute dfs is deprecated.
WARNING: Attempting to execute replacement "hdfs dfs" instead.

Found 2 items
-rw-r--r--   1 ubuntu supergroup          0 2024-06-26 14:39 /user/ubuntu/output/_SUCCESS
-rw-r--r--   1 ubuntu supergroup         28 2024-06-26 14:39 /user/ubuntu/output/part-r-00000
```


#### 출력
```sh
hadoop dfs -text /user/ubuntu/output/part-r-00000
```

```sh
WARNING: Use of this script to execute dfs is deprecated.
WARNING: Attempting to execute replacement "hdfs dfs" instead.

Bear    2
Car     3
Deer    2
River   2
```

---
## 브라우저에서 확인

![[Pasted image 20240626150226.png]]

1. 입력: localhost:8088
2. 확인: FINISHED, SUCCEEDED
3. 클릭: application_1719361466459_0001

![[Pasted image 20240626150325.png]]