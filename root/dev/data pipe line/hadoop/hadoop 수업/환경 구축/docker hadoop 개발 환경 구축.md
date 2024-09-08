---
tags:
  - docker
  - hadoop
  - setting
create: 2024-06-25 10:54:52
---

## Docker

[[docker ubuntu 기본 환경]]

### 컨테이너 생성

```sh
docker container run -it -p 9870:9870 -p 8088:8088 -p 8042:8042 -v /home/wsl/work:/home/ubuntu/work --name hadoop-study2 -u ubuntu -w /home/ubuntu ubuntu
```

### ssh 실행

```sh
sudo apt-get update
sudo apt-get install -y openssh-server
sudo service ssh start
```

---
## 환경 설정
### 설치

```sh
ssh-keygen -t rsa
cat >> ~/.ssh/authorized_keys < ~/.ssh/id_rsa.pub

# 설치 가능한 리스트 업데이트
sudo apt-get -y update

# 업데이트한 패키지들을 최신 버전에 맞게 업그레이드
sudo apt-get -y upgrade

# 의존성까지 체크해서 업그레이드
sudo apt-get -y dist-upgrade

# 필요 라이브러리 설치
sudo apt-get install -y vim wget unzip ssh openssh-* net-tools tree

# Ubuntu 20.4 에는 native libray 인 snappy 가 설치되어 있지 않다.
# 아래 snappy 설치를 하지 않으면 하둡 설치 후 snappy 사용 시 에러가 발생한다.
sudo apt install libsnappy-dev -y


# EC2 Ubuntu terminal

# Java 8 설치
sudo apt-get install -y openjdk-8-jdk

# Java 버전 확인
java -version

# Java 경로 확인
sudo find / -name java-8-openjdk-amd64 2>/dev/null
# /usr/lib/jvm/java-8-openjdk-amd64

```

### JAVA 환경 변수
```sh
# EC2 Ubuntu terminal

# Java 시스템 환경변수 등록 및 활성화
sudo vim /etc/environment

# 아래 내용 추가 후 저장
PATH 뒤에 ":/usr/lib/jvm/java-8-openjdk-amd64/bin" 추가
JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64"

# 시스템 환경변수 활성화
source /etc/environment

# 사용자 환경변수 등록
sudo echo 'export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64' >> ~/.bashrc

# 사용자 환경변수 활성화
source ~/.bashrc
```


---

## Hadoop

### 다운 및 설치

```sh
# EC2 Ubuntu terminal

# 설치파일 관리용 다운로드 디렉토리 생성
mkdir ~/downloads && cd ~/downloads

# Hadoop 3.2.3 설치
wget https://archive.apache.org/dist/hadoop/common/hadoop-3.2.3/hadoop-3.2.3.tar.gz

# Hadoop 3.2.3 압축 해제
sudo tar -zxvf hadoop-3.2.3.tar.gz -C /usr/local

# 소유권 변경
sudo chown -R $USER:$USER /usr/local/hadoop-3.2.3

# hadoop 심볼릭 링크 생성
cd /usr/local && sudo ln -s hadoop-3.2.3 hadoop
```


### 환경설정

```sh
# EC2 Ubuntu terminal

# Hadoop 시스템 환경변수 설정
sudo vim /etc/environment

# 아래 내용 추가 후 저장
PATH 뒤에 ":/usr/local/hadoop/bin" 추가
PATH 뒤에 ":/usr/local/hadoop/sbin" 추가
HADOOP_HOME="/usr/local/hadoop"

# 시스템 환경변수 활성화
source /etc/environment

# Hadoop환 사용자 환경변수 설정
#sudo echo 'export YARN_CONF_DIR=$HADOOP_HOME/etc/hadoop' >> ~/.bashrc
sudo echo 'export HADOOP_HOME=/usr/local/hadoop' >> ~/.bashrc
sudo echo 'export HADOOP_COMMON_HOME=$HADOOP_HOME' >> ~/.bashrc
sudo echo 'export HADOOP_HDFS_HOME=$HADOOP_HOME' >> ~/.bashrc
sudo echo 'export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop' >> ~/.bashrc
sudo echo 'export HADOOP_YARN_HOME=$HADOOP_HOME' >> ~/.bashrc
sudo echo 'export HADOOP_MAPRED_HOME=$HADOOP_HOME' >> ~/.bashrc

# 이거 안하면 hadoop hash 안됨
sudo echo /etc/environment >> ~/.bashrc
# 사용자 환경변수 활성화
source ~/.bashrc
```


### 설정

#### hadoop-env.sh

```sh
vim $HADOOP_HOME/etc/hadoop/hadoop-env.sh
```

```sh title:hadoop-env.sh
# Java
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64

# Hadoop
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop
export HADOOP_COMMON_LIB_NATIVE_DIR=${HADOOP_HOME}/lib/native
export HADOOP_OPTS="${HADOOP_OPTS} -Djava.library.path=$HADOOP_HOME/lib/native"

# For PID
# hadoop-USER-datanode.pid (DataNode)
# hadoop-USER-journalnode.pid (JournalNode)
# hadoop-USER-namenode.pid (NameNode)
# hadoop-USER-zkfc.pid (DFSZKFailoverController)
export HADOOP_PID_DIR=${HADOOP_HOME}/pids
export HADOOP_SECURE_PID_DIR=${HADOOP_PID_DIR}
```


#### core-site.xml

```sh
vim $HADOOP_HOME/etc/hadoop/core-site.xml
```

```xml title:core-site.xml
<configuration>
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://localhost:9000</value>
	</property>
	<property>
		<name>hadoop.http.staticuser.user</name>
		<value>ubuntu</value>
	</property>
</configuration>
```

#### hdfs-site.xml


```sh
vim $HADOOP_HOME/etc/hadoop/hdfs-site.xml
```

```xml title:hdfs-site.xml
<configuration>
	<property>
		<name>dfs.replication</name>
		<value>1</value>
	</property>
	<property>
		<name>dfs.namenode.name.dir</name>
		<value>/usr/local/hadoop/data/namenode</value>
	</property>
	<property>
		<name>dfs.datanode.data.dir</name>
		<value>/usr/local/hadoop/data/datanode</value>
	</property>
	<property>
		<name>dfs.journalnode.edits.dir</name>
		<value>/usr/local/hadoop/data/dfs/journalnode</value>
	</property>
</configuration>
```


#### mapred-site.xml

```sh
vim $HADOOP_HOME/etc/hadoop/mapred-site.xml
```

```xml title:mapred-site.xml
<configuration>
	<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>
	<property>
		<name>mapreduce.application.classpath</name>   
		<value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
	</property>
</configuration>
```


#### yarn-site.xml

```sh
vim $HADOOP_HOME/etc/hadoop/yarn-site.xml
```

```xml title:yarn-site.xml
<configuration>
	<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
	</property>
	<property>
		<name>yarn.nodemanager.env-whitelist</name>
		<value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
	</property>
</configuration>
```


### 네임노드 포맷

```sh
hdfs namenode -format
```

### 실행

```sh
sudo service ssh start
cd $HADOOP_HOME/sbin
./start-all.sh
```


### 확인
```sh title:in
hadoop version
hdfs version
```

```sh title:out
Hadoop 3.2.3
Source code repository https://github.com/apache/hadoop -r abe5358143720085498613d399be3bbf01e0f131
Compiled by ubuntu on 2022-03-20T01:18Z
Compiled with protoc 2.5.0
From source with checksum 39bb14faec14b3aa25388a6d7c345fe8
This command was run using /usr/local/hadoop-3.2.3/share/hadoop/common/hadoop-common-3.2.3.jar
```

---
## 웹 데몬 확인

- HDFS : http://localhost:9870
- YARN : http://localhost:8088
- NodeManager : http://localhost:8042
