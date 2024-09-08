---
tags:
  - kafka
  - install
create: 2024-07-26 02:49:04
---


## JAVA

### 설치

```ad-attention
title: docker 설치시
멈추면 터미널 모두 끄고 설정 해주면 됨
```

```sh
sudo apt-get update
sudo apt-get install -y openjdk-17-jre-headless
```

### 확인

```sh
java -version
# openjdk version "17.0.11" 2024-04-16
```


### 환경 변수

#### ~/.bashrc
- amd
```sh title:.bashrc
export JAVA_HOME=/usr/lib/jvm/java-1.17.0-openjdk-amd64
```

- arm (mac)
```sh title:.bashrc
export JAVA_HOME=/usr/lib/jvm/java-1.17.0-openjdk-arm64
```


#### 적용
```sh
source .bashrc
exec $SHELL
```



---
## install


### 디렉토리

```sh
cd

mkdir app
cd app

mkdir kafka
cd kafka
```

### 다운로드
```ad-attention
버전이 안 맞으면
https://downloads.apache.org/kafka
여기서 다운
```

```sh
wget https://downloads.apache.org/kafka/3.6.2/kafka_2.13-3.6.2.tgz
```


### 압출 풀기
```sh
tar -xzf kafka_2.13-3.6.2.tgz
```



[[kafka 설정 및 실행]]
