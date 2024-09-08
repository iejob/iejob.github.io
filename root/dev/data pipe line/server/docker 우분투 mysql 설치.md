---
tags:
  - mysql
  - install
  - ubuntu
  - docker
create: 2024-07-25 01:04:07
---

https://jongsky.tistory.com/79

## 설치

```sh
sudo apt-get update
sudo apt-get -y install mysql-server
```

`````ad-attention
title:음...?
ubuntu 계정의 sudo 패스워드를 패스 하는 권한을 줘서 그런지
설치하고 나면 shell로 나와야 하는데
```sh
done!
update-alternatives: using /var/lib/mecab/dic/ipadic-utf8 to provide /var/lib/mecab/dic/debian (mecab-dictionary) in auto mode
Setting up libhtml-parser-perl:amd64 (3.81-1build3) ...
Setting up libhttp-message-perl (6.45-1ubuntu1) ...
Setting up mysql-server-8.0 (8.0.37-0ubuntu0.24.04.1) ...
debconf: unable to initialize frontend: Dialog
debconf: (No usable dialog-like program is installed, so the dialog based frontend cannot be used. at /usr/share/perl5/Debconf/FrontEnd/Dialog.pm line 79.)
debconf: falling back to frontend: Readline
invoke-rc.d: could not determine current runlevel
invoke-rc.d: policy-rc.d denied execution of stop.
update-alternatives: using /etc/mysql/mysql.cnf to provide /etc/mysql/my.cnf (my.cnf) in auto mode
Renaming removed key_buffer and myisam-recover options (if present)
mysqld will log errors to /var/log/mysql/error.log
mysqld is running as pid 2171
```
이렇게 뜨고, shell이 멈춤
ctrl + c 로 캔슬로 안되고..

나갔다 들어오면 잘 되긴 함.
`````

---

## 시작

```sh
sudo service mysql start
```

---

## 계정

[[mysql 계정 생성시 %의 의미]]

### 최조 접속

```sh
sudo mysql -u root
```


### 만들기
```mysql
CREATE USER 'ubuntu'@'%' IDENTIFIED BY '1234';
```

``````ad-info
title: 계정이 존재 하면 아래의 것으로 비번을 초기화 함

```sql
ALTER USER 'ubuntu'@'%' IDENTIFIED BY '1234';
```

``````

### 권한 부여
```mysql
GRANT ALL PRIVILEGES ON *.* TO 'ubuntu'@'%' WITH GRANT OPTION;
```
### 변경 사항 적용
```mysql
FLUSH PRIVILEGES;
```

### 나가기

```mysql
exit
```

---

## 설정

### 권한

```sh
sudo chown -R mysql:mysql /var/run/mysqld
sudo chmod 755 /var/run/mysqld
```


### 외부 접속 허용

#### 설정
- 열기
```sh
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
```

- 편집
```vim
port                    = 3306
bind-address            = 0.0.0.0
```


### 적용

```sh
sudo service mysql restart
```


### 확인
- 입력
```sh
sudo service --status-all
```

- 출력
```sh
 [ - ]  dbus
 [ + ]  mysql
 [ - ]  procps
 [ - ]  ssh
 [ - ]  x11-common
```


### 테스트
```sh
mysql -u ubuntu -p -h 자신의_아이피_주소
```

- ex
```sh
mysql -u ubuntu -p -h 172.17.0.3
```

```ad-important
테스트를 할 때, 설정을 한, 본 서버에서 접속이 된다면
타 서버에서도 접속 가능.
```
