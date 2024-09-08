---
tags:
  - ubuntu
  - ssh
  - hosts
create: 2024-07-27 16:10:39
---

## hosts

```sh
sudo vi /etc/hosts
```

```vim
ip_주소 사용하고_싶은_server_이름
```

```vim
172.17.0.2 server1
172.17.0.3 server2
172.17.0.4 server3
```


---


## ssh

### key 생성
```sh
ssh-keygen -t rsa
```

```ad-attention
title: 경고야!
비번을 입력하지 않고 조건 없이 엔터
```

### 등록

```sh
cd ~/.ssh/id_rsa.pub
```
접속자의 pc에서 생성된 id_rsa.pub 를


```sh
vi ~/.ssh/authorized_keys
```

```sh
touch ~/.ssh/authorized_keys
```

접속할 pc의 authorized_keys안에 입력 한다.