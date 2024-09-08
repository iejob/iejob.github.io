---
tags:
  - docker
  - setting
create: 2024-08-09 00:20:29
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
- (D): indocker에서 하는 작업
```


## 이미지 다운

### (L) image pull

```sh
docker pull ubuntu:24.04
```

### (L) tag 붙이기
```sh
docker tag ubuntu:24.04 ubuntu-in-docker:24.04
```



---

## 기본 환경

### (L) 컨테이너 생성 및 실행
```sh
docker container run -itd --privileged ubuntu-in-docker:24.04
```

```ad-attention
title: 그렇군?
--privileged 옵션을 넣어 줘야 docker를 사용할 수 있음
```

### (L) 컨테이너 접속
```sh
docker container exec -it 컨테이너_이름 bash
```

### (C) 설정 및 docker 설치

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
curl -sSL get.docker.com | sh
```


### (C) python 설치

[[pyenv 설치]]


### (C) 자동 시작

```sh
vi ~/.bashrc
```

```sh
if ! sudo service docker status > /dev/null 2>&1; then
  sudo service docker start
fi
```


### (L) 커밋
```sh
docker container commit 컨테이너_이름 ubuntu-in-docker:24.04-basic
```

### (L) 삭제
```sh
docker container rm -f 컨테이너_이름
```


---

## ssh 연결

### (L) 컨테이너 생성 및 실행
```sh
docker container run -itd \
  --privileged \
  ubuntu-in-docker:24.04-basic \
  /bin/bash -c "sudo service ssh start && tail -f /dev/null"
```

### (L) 컨테이너 접속
```sh
docker container exec -it 컨테이너_이름 bash
```

### (C) key 생성
```sh
ssh-keygen -t rsa
cd .ssh
cat id_rsa.pub > authorized_keys
```

### (C) ssh 최초 접속시 yes 생략

```sh
vi ~/.shh/config
```

```sh
Host server*
  StrictHostKeyChecking no
  UserKnownHostsFile=/dev/null
```


### (L) 커밋
```sh
docker container commit 컨테이너_이름 ubuntu-in-docker:24.04-ssh-key
```


---

## 연결 테스트

### (L) 네트워크 만들기
```sh
docker network create --gateway 172.20.0.1 --subnet 172.20.0.0/24 basic
```

### (L) 컨테이너 생성 및 실행
```sh
docker container run -itd \
  --privileged \
  --name server1 \
  --network basic \
  --ip 172.20.0.11 \
  --hostname server1 \
  --add-host server1:172.20.0.11 \
  --add-host server2:172.20.0.12 \
  --add-host server3:172.20.0.13 \
  ubuntu-in-docker:24.04-ssh-key


docker container run -itd \
  --privileged \
  --name server2 \
  --network basic \
  --ip 172.20.0.12 \
  --hostname server2 \
  --add-host server1:172.20.0.11 \
  --add-host server2:172.20.0.12 \
  --add-host server3:172.20.0.13 \
  ubuntu-in-docker:24.04-ssh-key


docker container run -itd \
  --privileged \
  --name server3 \
  --network basic \
  --ip 172.20.0.13 \
  --hostname server3 \
  --add-host server1:172.20.0.11 \
  --add-host server2:172.20.0.12 \
  --add-host server3:172.20.0.13 \
  ubuntu-in-docker:24.04-ssh-key
```

### (L) 컨테이너 접속
```sh
docker container exec -it server1 bash
```

### (C) 작성

```sh
vi ssh_check.sh
```


 [[shell ssh 여러 개 접속 테스트]]


```sh
chmod +x ssh_check.sh

./ssh_check.sh
```


### (L) 삭제
```sh
docker container rm -f server1 server2 server3
```

---


## 내부 도커 만들기

### (L) 컨테이너 생성 및 실행
```sh
docker container run -itd \
  --privileged \
  --name server1 \
  --network basic \
  --ip 172.20.0.11 \
  --hostname server1 \
  --add-host server1:172.20.0.11 \
  --add-host server2:172.20.0.12 \
  --add-host server3:172.20.0.13 \
  ubuntu-in-docker:24.04-ssh-key


docker container run -itd \
  --privileged \
  --name server2 \
  --network basic \
  --ip 172.20.0.12 \
  --hostname server2 \
  --add-host server1:172.20.0.11 \
  --add-host server2:172.20.0.12 \
  --add-host server3:172.20.0.13 \
  ubuntu-in-docker:24.04-ssh-key


docker container run -itd \
  --privileged \
  --name server3 \
  --network basic \
  --ip 172.20.0.13 \
  --hostname server3 \
  --add-host server1:172.20.0.11 \
  --add-host server2:172.20.0.12 \
  --add-host server3:172.20.0.13 \
  ubuntu-in-docker:24.04-ssh-key
```

```sh
git clone https://github.com/namugach/pipelineShell
```

```sh
mv pipelineShell run
```

```sh
./run/ssh/check.sh
```

```sh
./run/docker/pull.sh namugach/ubuntu-pipeline:24.04-kafka
```

