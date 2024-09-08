---
tags:
  - kafka
  - setting
create: 2024-07-23 10:03:39
---


[[kafka 설치]]


```ad-attention
server 1,2,3 모두 적용
```

그리고

```ad-attention
title:정말 중요!
docker 컨테이너 안에서 실행 할 때는 굳이
-p 옵션을 줘서 포트포워딩 할 필요가 없다.
그러니 이 부분에 있어서는 신경쓰지 말고 구현하자
```

## 설정

### 주키퍼 데이터 디렉토리 및 아이디 생성 

`````ad-attention
title: 서버에 맞게 설정!

- server1
```sh
cd
mkdir zkdata
cd zkdata
echo 1 > ./myid
```

- server2
```sh
cd
mkdir zkdata
cd zkdata
echo 2 > ./myid
```

- server3
```sh
cd
mkdir zkdata
cd zkdata
echo 3 > ./myid
```

`````



### 설정 디렉토리로 이동

```sh
cd /home/ubuntu/app/kafka/kafka_2.13-3.6.2/config
```

### server.properties


```ad-important
docker image를 만들기 위해서 [[kafka server.properties]]를 적용 시키고
서버에 맞게 설정 하는 것은 적용된 값에서 컨테이너마다 설정 해주자
```

`````ad-attention
title:서버에 맞게 설정!

```propertis
broker.id
listeners
advertised.listeners
```
`위의 프로퍼티는 각 서버에 맞게 수정해서 적용해야 함`

- server1
```propertis
broker.id = 1
listeners=PLAINTEXT://172.17.0.2:9092
advertised.listeners=PLAINTEXT://172.17.0.2:9092
```

- server2
```propertis
broker.id = 2
listeners=PLAINTEXT://172.17.0.3:9092
advertised.listeners=PLAINTEXT://172.17.0.3:9092
```

- server3
```propertis
broker.id = 3
listeners=PLAINTEXT://172.17.0.4:9092
advertised.listeners=PLAINTEXT://172.17.0.4:9092
```

`````



### zookeeper.properties

```properties
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
# 
#    http://www.apache.org/licenses/LICENSE-2.0
# 
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# the directory where the snapshot is stored.
dataDir=/home/ubuntu/zkdata
# the port at which the clients will connect
clientPort=2181
tickTime=2000
initLimit=10
syncLimit=5
server.1=172.17.0.2:2888:3888
server.2=172.17.0.3:2888:3888
server.3=172.17.0.4:2888:3888
# disable the per-ip limit on the number of connections since this is a non-production config
maxClientCnxns=0
# Disable the adminserver by default to avoid port conflicts.
# Set the port to something non-conflicting if choosing to enable this
admin.enableServer=false
# admin.serverPort=8080
```


---

## 자동 실행

### kafka zookeeper start

[[shellscirpt kafka zookeeper server]]

### kafka server start

[[shellscript kafka server start]]

### kafka check

[[shellscript kafka server conn check]]

### 권한 설정

```sh
sudo chmod +x start_zookeeper.sh start_server.sh check_conn.sh
```

---
## 수동 실행


### 디렉토리 이동

```sh
cd /home/ubuntu/app/kafka/kafka_2.13-3.6.2/
```


```ad-info
기본값은 백그라운드지만, 포그라운드를 만들어 놓은 이슈는
프로그램 동작 확인과 종료를 쉽게 하기 위함이다.
```

### 백그라운드

```sh
nohup ./bin/zookeeper-server-start.sh config/zookeeper.properties >/dev/null 2>&1 &
```

```sh
nohup ./bin/kafka-server-start.sh config/server.properties >/dev/null 2>&1 &
```


### 포그라운드

```sh
nohup ./bin/zookeeper-server-start.sh config/zookeeper.properties >/dev/null 2>&1
```

```sh
nohup ./bin/kafka-server-start.sh config/server.properties >/dev/null 2>&1
```



```sh
netstat -ntlp | grep 2181
```

```sh
netstat -ntlp | grep 9092
```

### 정상 실행이 안될 때

[[kafka error handling]]

---

## 실행 코드 분석

```sh
nohup ./bin/zookeeper-server-start.sh config/zookeeper.properties >/dev/null 2>&1 &
```

1. **`nohup`**: 
    - **목적**: 터미널을 닫아도 명령어가 계속 실행되게 해주는 명령어야.
    - **상세**: 'no hang up'의 줄임말로, 보통 터미널을 닫으면 그 터미널에서 실행 중인 프로세스도 종료되는데, `nohup`을 사용하면 백그라운드에서 계속 실행돼.

2. **`./bin/zookeeper-server-start.sh config/zookeeper.properties`**:
    - **목적**: 주키퍼 서버를 시작하는 스크립트야.
    - **상세**: `zookeeper-server-start.sh`는 주키퍼 서버를 시작하는 스크립트 파일이고, `config/zookeeper.properties`는 주키퍼의 설정 파일이야. 이 파일에 주키퍼 서버 설정들이 담겨 있어.

3. **`>/dev/null 2>&1`**:
    - **목적**: 출력과 에러 메시지를 무시하고 버려.
    - **상세**: 
        - `>/dev/null`: 표준 출력을 `/dev/null`로 보내서 출력 결과를 버려.
        - `2>&1`: 표준 에러를 표준 출력으로 리다이렉트해. 따라서 표준 출력과 표준 에러 모두 `/dev/null`로 가서 결과적으로 아무 출력도 보이지 않게 돼.

4. **`&`**:
    - **목적**: 명령어를 백그라운드에서 실행해.
    - **상세**: 이걸 붙이면 명령어가 백그라운드에서 실행돼서 터미널을 계속 사용할 수 있게 해줘.

종합해보면, 이 명령어는 주키퍼 서버를 설정 파일을 기반으로 백그라운드에서 실행하면서 출력 결과와 에러 메시지를 모두 무시하는 거야.

---

 
## 확인

### server1
```sh
cd ~/app/kafka/kafka_2.13-3.6.2

./bin/kafka-topics.sh --create --bootstrap-server 172.17.0.2:9092 --replication-factor 3 --partitions 3 --topic testtopic
```

```sh
cd ~/app/kafka/kafka_2.13-3.6.2

./bin/kafka-console-producer.sh --bootstrap-server 172.17.0.2:9092 --topic testtopic
```

### server2
```sh
cd ~/app/kafka/kafka_2.13-3.6.2

./bin/kafka-console-consumer.sh --bootstrap-server 172.17.0.3:9092 --topic testtopic --from-beginning
```


### server3
```sh
cd ~/app/kafka/kafka_2.13-3.6.2

./bin/kafka-console-consumer.sh --bootstrap-server 172.17.0.4:9092 --topic testtopic --from-beginning
```

---

## 종료

```sh
pkill -f kafka
```