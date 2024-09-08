---
tags:
  - docker
  - setup
  - pipeline
create: 2024-07-28 17:27:20
---

```ad-danger
title: 커밋할 때
절 때 hostname으로 이름 지은 것으로 하지 말아야함.
그렇지 않고, hostname이 지정된 상태에서 프로그램을 설치하고,
그 상태로 커밋을 한 다면, hostname과 일치하는 컨테이너는 문제가 없을 테지만,
그렇지 못한 컨테이너는 프로그램에 따라서 실행이 안된다.
```


```ad-check
title: 작업에 들어가기 앞서!
- (L): local에서 하는 작업
	- windows: wsl
	- mac: terminal
- (C): container에서 하는 작업
```

## 작업 환경 만들기


`````ad-attention
title:pull 받으려 하는데 로그인 하라고?
- http://dockerhub.com/ 에서
- 아이디 없으면 만들고
- 있으면 확인하고
- shell에서 아래의 명령어를 입력하여 로그인
```sh
docker login
```
`````

### (L) image pull

```sh
docker pull ubuntu:24.04
```

### (L) tag 붙이기
```sh
docker tag ubuntu:24.04 ubuntu-pipeline:24.04
```

### (L) 컨테이너 만들기
```sh
docker container run -it -d ubuntu-pipeline:24.04
```


---

## 기본 환경

### (C) 설정

```sh
apt-get update
apt-get install -y sudo
apt-get install -y curl
apt-get install -y net-tools
apt-get install -y git
apt-get install -y wget
apt-get install -y tree
apt-get install -y openssh-server
apt-get install -y vim
```




### (C) 패스워드 설정

```ad-attention
title: 매우 주의!
root 계정으로 작업 할 것!
```

#### root
```sh
passwd
```

#### 사용자 계정

```sh
passwd ubuntu
```


#### 사용자 계정 sudo 비번 생략

[[우분투 sudo 비번 생략하기]]

```sh
echo 'ubuntu ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
```

---
## 24.04-basic 이미지 만들기
### (L) 컨테이너

#### 이름 확인 
```sh
docker container ls
```

#### 커밋
```sh
docker container commit 컨테이너_이름 ubuntu-pipeline:24.04-basic
```


#### 삭제
```sh
docker container rm -f 컨테이너_이름
```




---

## 24.04-user-setting 이미지 만들기

### (L) 24.04-basic

#### 생성
```sh
docker container run -itd \
  -u ubuntu \
  -w /home/ubuntu \
  ubuntu-pipeline:24.04-basic
```


#### 이름 확인
```sh
docker container ls
```

#### 커밋
```sh
docker container commit 컨테이너_이름 ubuntu-pipeline:24.04-user-setting
```

#### 삭제
```sh
docker container rm -f 컨테이너_이름
```



---
  
## 24.04-pyenv 이미지 만들기
 

### (L) 24.04-user-setting 컨테이너 생성

```sh
docker container run -itd ubuntu-pipeline:24.04-user-setting 
```

### (C) 설치


[[pyenv 설치]]



### (L) 컨테이너

#### 이름 확인
```sh
docker container ls
```

#### 커밋
```sh
docker container commit 컨테이너_이름 ubuntu-pipeline:24.04-pyenv
```

#### 삭제
```sh
docker container rm -f 컨테이너_이름
```


---

## 24.04-mysql 이미지 만들기

### (L) 24.04-pyenv 컨테이너 생성

```sh
docker container run -itd ubuntu-pipeline:24.04-pyenv
```

### (C) mysql 설치

[[docker 우분투 mysql 설치]]


### (L) 컨테이너

#### 이름 확인
```sh
docker container ls
```

#### 커밋
```sh
docker container commit 컨테이너_이름 ubuntu-pipeline:24.04-mysql
```

#### 삭제
```sh
docker container rm -f 컨테이너_이름
```



---

## 24.04-service-auto-load 이미지 만들기

### (L) 24.04-mysql 컨테이너 생성

```sh
docker container run -itd ubuntu-pipeline:24.04-mysql
```

[[docker container 시작 시 명령어 시작]]

### (L) container run
```sh
docker container run -itd \
  ubuntu-pipeline:24.04-mysql\
  /bin/bash -c "sudo service ssh start && sudo service mysql start && tail -f /dev/null"
```

### (C) 자동 시작 확인
#### 입력
```sh
sudo service --status-all
```

#### 출력
```sh
 [ - ]  dbus
 [ + ]  mysql
 [ - ]  procps
 [ + ]  ssh
 [ - ]  x11-common
```


### (L) 컨테이너

#### 이름 확인
```sh
docker container ls
```

#### 커밋
```sh
docker container commit 컨테이너_이름 ubuntu-pipeline:24.04-service-auto-load
```

#### 삭제
```sh
docker container rm -f 컨테이너_이름
```

---


## ssh 연결

### (L) 24.04-servcie-auto-load 컨테이너 생성

```sh
docker container run -itd ubuntu-pipeline:24.04-service-auto-load
```

### (C) hosts  설정

[[우분투 hosts를 설정하여 ssh 편하게 이동하기]]

### (C) key 생성
```sh
ssh-keygen -t rsa
cd .ssh
cat id_rsa.pub > authorized_keys
```


### (C) ssh 최초 접속시 yes 생략

```sh
Host server*
  StrictHostKeyChecking no
  UserKnownHostsFile=/dev/null
```

[[ssh 접속시 yes 입력 생략하기]]

### (C) ssh_check 작성

```sh
cd
mkdir run
cd run
vi ssh_check.sh
```

[[shell ssh 여러 개 접속 테스트]] 를 작성

```sh
sudo chmod +x ssh_check.sh
```

### (L) 컨테이너

#### 이름 확인
```sh
docker container ls
```

#### 커밋
```sh
docker container commit 컨테이너_이름 ubuntu-pipeline:24.04-ssh-key
```

#### 삭제
```sh
docker container rm -f 컨테이너_이름
```


#### 생성

```ad-attention
title: 그리고
commit 된 이미로 3개의 컨테이너를 띄움
```

```sh
docker run -itd \
  --name server1 \
  --hostname server1 \
  --add-host server1:172.17.0.2 \
  --add-host server2:172.17.0.3 \
  --add-host server3:172.17.0.4 \
  ubuntu-pipeline:24.04-ssh-key

docker run -itd \
  --name server2 \
  --hostname server2 \
  --add-host server1:172.17.0.2 \
  --add-host server2:172.17.0.3 \
  --add-host server3:172.17.0.4 \
  ubuntu-pipeline:24.04-ssh-key

docker run -itd \
  --name server3 \
  --hostname server3 \
  --add-host server1:172.17.0.2 \
  --add-host server2:172.17.0.3 \
  --add-host server3:172.17.0.4 \
  ubuntu-pipeline:24.04-ssh-key
```

### (C) 확인

[[shell ssh 여러 개 접속 테스트]]

```sh
./run/ssh_check.sh
```



---

## java

### (L) 24.04-ssh-key 컨테이너 생성

```sh
docker container run -itd ubuntu-pipeline:24.04-ssh-key
```

[[kafka 설치#JAVA]]


### (L) 컨테이너

#### 이름 확인
```sh
docker container ls
```

#### 커밋
```sh
docker container commit 컨테이너_이름 ubuntu-pipeline:24.04-java
```

#### 삭제
```sh
docker container rm -f 컨테이너_이름
```


### java 컨테이너 확인

```ad-check
root 계정으로 java를 설치 했을 때는 
hostname이 java 설치 했을 때와 다르면
컨테이너가 죽는 현상이 발견 됐다.
하지만 이게 과연 root 계정이기 때문에 발생 된 것인지에 대한 건 모른다.
때문에 아래의 테스트를 꼭 거치고 다음 스텝을 밟아야 한다
```

#### 확인

```sh
docker run -itd \
  --name server1 \
  --hostname server1 \
  --add-host server1:172.17.0.2 \
  --add-host server2:172.17.0.3 \
  --add-host server3:172.17.0.4 \
  ubuntu-pipeline:24.04-java

docker run -itd \
  --name server2 \
  --hostname server2 \
  --add-host server1:172.17.0.2 \
  --add-host server2:172.17.0.3 \
  --add-host server3:172.17.0.4 \
  ubuntu-pipeline:24.04-java

docker run -itd \
  --name server3 \
  --hostname server3 \
  --add-host server1:172.17.0.2 \
  --add-host server2:172.17.0.3 \
  --add-host server3:172.17.0.4 \
  ubuntu-pipeline:24.04-java

```


#### 컨테이너 삭제

```sh
docker container rm -f server1 server2 server3 
```


---
## kafka

### (L) 24.04-java 컨테이너 생성

```sh
docker container run -itd ubuntu-pipeline:24.04-java
```

[[kafka 설치#install]]

[[kafka 설정 및 실행]]



### (L) 컨테이너

#### 이름 확인
```sh
docker container ls
```

#### 커밋
```sh
docker container commit 컨테이너_이름 ubuntu-pipeline:24.04-kafka
```

#### 삭제
```sh
docker container rm -f 컨테이너_이름
```


### 실행

```ad-danger
title:정말 위험해!
docker 컨테이너로 서버를 start 할 때 순서를 꼭 치켜야 한다.
그렇지 않으면 ip가 켜진 순서대로 할당 받기 때문에
kafka에서 정상 작동을 기대하기 어려워진다.
```

```sh
docker run -itd \
  --name server1 \
  --hostname server1 \
  --add-host server1:172.17.0.2 \
  --add-host server2:172.17.0.3 \
  --add-host server3:172.17.0.4 \
  ubuntu-pipeline:24.04-kafka

docker run -itd \
  --name server2 \
  --hostname server2 \
  --add-host server1:172.17.0.2 \
  --add-host server2:172.17.0.3 \
  --add-host server3:172.17.0.4 \
  ubuntu-pipeline:24.04-kafka

docker run -itd \
  --name server3 \
  --hostname server3 \
  --add-host server1:172.17.0.2 \
  --add-host server2:172.17.0.3 \
  --add-host server3:172.17.0.4 \
  ubuntu-pipeline:24.04-kafka
```

---

## kafka data pipe line 실습

[[kafka 클러스터 서버 시동]]

[[kafka에 csv로 데이터 흘리기]]

[[Kafka Consumer와 MySQL에 데이터 적재하기]]

