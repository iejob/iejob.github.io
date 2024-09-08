---
tags:
  - pyenv
  - install
  - public
create: 2024-05-10 16:21:22
---


## docker 초기 설정

### 설정

```sh
sudo apt-get update
sudo apt-get install locales
sudo locale-gen en_US.UTF-8
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
echo 'export LANG=en_US.UTF-8' >> ~/.bashrc
echo 'export LC_ALL=en_US.UTF-8' >> ~/.bashrc
source ~/.bashrc
```

```ad-attention
title:너무나 중요!!
초기 설정을 모두 끝낸 후 터미널 재시작!!
```

### 터미널 재시작 방법
```sh
exit
```


---
## 사전 준비
```sh
sudo apt-get update; sudo apt-get -y install make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
```
---
## 설치
```sh
curl https://pyenv.run | bash
```
---
## 초기 설정

```sh
vi ~/.bashrc
```

```sh title:.bashrc
export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

```ad-info
.bashrc 하단에 추가
```

```sh
. ~/.bashrc
```
---

## 버전 확인

```
pyenv install --list
```

---
## 설치

```sh
pyenv install 3.11.9
```

---



## 가상 환경

### 확인
```sh
pyenv versions
```

### 생성

```sh
pyenv virtualenv 3.11.9 py3_11_9
```


### 접속

```sh
pyenv activate py3_11_9
```


--------------------------------------



## shell 초기화

```sh
. ~/.bashrc
```
위와 아래는 같음
```sh
source ~/.bashrc
```