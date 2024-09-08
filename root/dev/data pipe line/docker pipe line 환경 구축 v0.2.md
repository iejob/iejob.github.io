---
tags:
  - docker
  - pipeline
  - setup
create: 2024-08-12 01:22:41
---
docker pipe line 환경 구축 v0.2

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


## (L) 컨테이너 만들기
```sh
docker container run -it -d ubuntu-pipeline:24.04
```

---
## (C) bash 설정


### 편집
```sh
vi .bashrc
```

내용을 전부 지운 뒤

[[우분투 root 계정일 때 bash 일반 계정처럼 만들기]]를 붙여 넣는다.

### 적용
```sh
. ~/.bashrc
```

---
## (C) 패스워드

```sh
passwd
```

---

## (C) 기본 프로그램 설치


```sh
apt-get update
apt-get install -y sudo
apt-get install -y curl
apt-get install -y net-tools
apt-get install -y git
apt-get install -y wget
apt-get install -y tree
apt-get install -y vim
apt-get install -y openssh-server
apt-get install -y mysql-server
apt-get install -y openjdk-17-jre-headless
```

중간에 

```sh
Please select the geographic area in which you live. Subsequent configuration questions
will narrow this down by presenting a list of cities, representing the time zones in
which they are located.

  1. Africa   3. Antarctica  5. Asia     
  2. America  4. Arctic      6. Atlantic 
Geographic area: 5

Please select the city or region corresponding to your time zone.

  49. Macau         67. Samarkand    
  50. Magadan       68. Seoul        
  51. Makassar      69. Shanghai     
Time zone: 68
```

time zones 선택하라고 나온다.
참고로 위의 보기는 많이 생략한 내용임으로 위처럼 나오지 않는다.

```ad-check
- 첫번째 에서
	5. Asia

- 두번 째 에서
	68. Seoul
```

---
## (C) 파이썬

```ad-check
title:wow!
openssh-server 설치하면서 같이 설치됨.. ㅎ...
```

### 링크 만들기
```sh
ln -s /bin/python3 /bin/python
```

이렇게 링크를 만들어 줘야
python shell에서 명령어를 입력 했을 때, python이 호출 된다.

### 가상환경 끄기
[[python pip venv 가상환경 사용하지 않기]]
```sh
sudo rm /usr/lib/python3.12/EXTERNALLY-MANAGED
```

이걸 끄지 않으면

```ad-error
error: externally-managed-environment

× This environment is externally managed
╰─> To install Python packages system-wide, try apt install
    python3-xyz, where xyz is the package you are trying to
    install.
    
    If you wish to install a non-Debian-packaged Python package,
    create a virtual environment using python3 -m venv path/to/venv.
    Then use path/to/venv/bin/python and path/to/venv/bin/pip. Make
    sure you have python3-full installed.
    
    If you wish to install a non-Debian packaged Python application,
    it may be easiest to use pipx install xyz, which will manage a
    virtual environment for you. Make sure you have pipx installed.
    
    See /usr/share/doc/python3.12/README.venv for more information.
```

이런 게 나옴.

참고로 python3.12 이 부분은 python -V  로 버전을 확인 한 후.
각자의 버전에 알맞게 수정하는 게 이로움.

그리고 설정을 끄고 나서 pip install로 패키지를 설치하고 나면

```ad-warning
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
```

이런 게 쓰는데, 신경 쓰지 않아도 괜찮음.
똬카(docker)니까

### pip
```sh
apt install python3-pip
```

---
## (C) mysql

[[docker 우분투 mysql 설치#시작]]

이미 기본 설치는 해버렸기 때문에 시작 부 터 설정만 해주면 된다.


---
## (C) ssh

### 시작
```sh
service ssh start
```

### 설정
#### 열기
```sh
vi /etc/ssh/sshd_config
```

#### 편집
```vim
Port 22
ListenAddress 0.0.0.0
PermitRootLogin yes
```


### key 생성
```sh
ssh-keygen -t rsa
cd .ssh
cat id_rsa.pub > authorized_keys
```


### ssh 최초 접속시 yes 생략

#### 열기
```sh
vi ~/.ssh/config
```

#### 작성
```sh
Host server*
  StrictHostKeyChecking no
  UserKnownHostsFile=/dev/null
```

[[ssh 접속시 yes 입력 생략하기]]


---

## (C) 서비스 자동 시작

### 설정
#### 열기
```sh
vi ~/.bashrc
```

#### 작성
```sh
if ! sudo service ssh status > /dev/null 2>&1; then
  sudo service ssh start
fi

if ! sudo service mysql status > /dev/null 2>&1; then
  sudo service mysql start
fi
```

### 적용

```sh
. ~/.bashrc
```

### 확인
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


---

## (C) JAVA

### 설정

#### 열기
```sh
vi ~/.bashrc
```

#### 작성

`````ad-attention
둘 중 하나만 적용한다.
amd는 intel 혹은 amd용이고
arm은 m1 시리즈를 사용하고 있는 mac이다.
맨 끝에를 보면 mac용은 arm64라고 적혀져 있는 걸 볼수 있다.
```sh
dpkg -s libc6 | grep Arch
```
위의 명령어로 자신의 아키텍쳐를 확인 할 수 있다.
`````


##### amd
```sh title:.bashrc
export JAVA_HOME=/usr/lib/jvm/java-1.17.0-openjdk-amd64
```

##### arm (mac)
```sh title:.bashrc
export JAVA_HOME=/usr/lib/jvm/java-1.17.0-openjdk-arm64
```


### 적용
```sh
. .bashrc
```

### 확인
```sh
java --version
```

---

## base 이미지 만들기

### 컨테이너

#### 이름 확인
```sh
docker container ls
```

#### 커밋
```sh
docker container commit 컨테이너_이름 ubuntu-pipeline:24.04-base
```


#### 삭제
```sh
docker container rm -f 컨테이너_이름
```


### 이미지 확인
#### 기본
```sh
docker image ls
```


#### 전체
```sh
docker image ls -a
```


#### 지정
```sh
docker image ls | grep base
```


```sh
REPOSITORY                 TAG            
ubuntu-pipeline            24.04-base     
```

이런 식으로 나온다면 성공



---

## ssh 연결

### (L) 컨테이너 생성
```sh
docker run -itd \
  --name server1 \
  --hostname server1 \
  --add-host server1:172.17.0.2 \
  --add-host server2:172.17.0.3 \
  --add-host server3:172.17.0.4 \
  ubuntu-pipeline:24.04-base

docker run -itd \
  --name server2 \
  --hostname server2 \
  --add-host server1:172.17.0.2 \
  --add-host server2:172.17.0.3 \
  --add-host server3:172.17.0.4 \
  ubuntu-pipeline:24.04-base

docker run -itd \
  --name server3 \
  --hostname server3 \
  --add-host server1:172.17.0.2 \
  --add-host server2:172.17.0.3 \
  --add-host server3:172.17.0.4 \
  ubuntu-pipeline:24.04-base
```

### (C) 확인
```sh
./run/ssh_check.sh
```

[[shell ssh 여러 개 접속 테스트]]


### (L) 컨테이너 제거
```sh
docker container rm server1 server2 server3
```

---

## kafka

### (L) 컨테이너 생성

```sh
docker container run -itd ubuntu-pipeline:24.04-base
```

### (C) 설치
[[kafka 설치#install]]
```sh
cd

mkdir app
cd app

mkdir kafka
cd kafka

wget https://downloads.apache.org/kafka/3.6.2/kafka_2.13-3.6.2.tgz

tar -xzf kafka_2.13-3.6.2.tgz
```


### (C) 설정
[[kafka 설정 및 실행]]

#### 아이디 만들기
```sh
cd
mkdir zkdata
cd zkdata
echo 1 > ./myid
```


#### server.properties 설정
```sh
vi /root/app/kafka/kafka_2.13-3.6.2/config/server.properties
```
[[kafka server.properties]]  내용을 전체 복사 붙여 넣기

#### zookeeper.properties 설정
```sh
vi /root/app/kafka/kafka_2.13-3.6.2/config/zookeeper.properties
```

```properties

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
```
위의 내용을 전체 복사 붙여 넣기


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

