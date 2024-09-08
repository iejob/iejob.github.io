---
tags:
  - hadoop
  - java
  - code
create: 2024-06-25 23:04:30
---

[[vscode maven 프로젝트 생성]]

## 전체 구조

```
.
|-- pom.xml
`-- src
    `-- main
        `-- java
            `-- FileSystemPrint.java
```


## pom.xml

```xml title:pom.xml
<dependencies>
	<dependency>
			<groupId>org.apache.hadoop</groupId>
			<artifactId>hadoop-client</artifactId>
			<version>3.3.6</version>
	</dependency>
</dependencies>
```

## FileSystemPrint.java

```java title:FileSystemPrint.java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IOUtils;


import java.io.IOException;
import java.io.InputStream;
import java.net.URI;


public class FileSystemPrint {
  public static void main(String[] args) throws IOException {
    String uri = args[0];
    Configuration conf = new Configuration();
    FileSystem fs = FileSystem.get(URI.create(uri), conf);
    try (InputStream in = fs.open(new Path(uri))) {
      IOUtils.copyBytes(in, System.out, 4096, false);
    }
  }
}
```

