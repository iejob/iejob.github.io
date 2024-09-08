---
tags:
  - elasticsearch
  - cerebro
create: 2024-07-19 16:23:30
---

https://github.com/lmenezes/cerebro/releases

## 설치

### 다운로드
```sh
wget https://github.com/lmenezes/cerebro/releases/download/v0.9.4/cerebro-0.9.4.tgz
```

### 압축 풀기
```sh
tar -xvzf cerebro-0.9.4.tgz
```


### 이동
```sh
ubuntu@ip-172-31-6-227:~/app/cerebro$ cd cerebro-0.9.4/

ubuntu@ip-172-31-6-227:~/app/cerebro/cerebro-0.9.4$ ls
README.md  bin  conf  lib

ubuntu@ip-172-31-6-227:~/app/cerebro/cerebro-0.9.4$ cd bin/

ubuntu@ip-172-31-6-227:~/app/cerebro/cerebro-0.9.4/bin$ ls
cerebro  cerebro.bat
# 리눅스용 # 윈도우용
```

---

## 설정

```sh
 vim cerebro
```

`````ad-attention 
title:들여쓰기 주의

```vim
  JAVA_OPTS="$JAVA_OPTS --add-opens java.base/java.lang=ALL-UNNAMED --add-opens java.base/java.util=ALL-UNNAMED --add-opens java.base/sun.net.www.protocol.file=ALL-UNNAMED"

  # Now we check to see if there are any java opts on the environment. These get listed first, with the script able to override them.
  if [[ "$JAVA_OPTS" != "" ]]; then
    java_opts="${JAVA_OPTS}"
  fi

```
246 번째 줄
`````

```vim title:~/app/cerebro/cerebro-0.9.4/bin/cerebro
JAVA_OPTS="$JAVA_OPTS --add-opens java.base/java.lang=ALL-UNNAMED --add-opens java.base/java.util=ALL-UNNAMED --add-opens java.base/sun.net.www.protocol.file=ALL-UNNAMED"
```


### 포트 열기

```ad-note
title: 인바운드 규칙
- 9000
```


### 접속

```sh
public_ip:9000
```


### 브라우저 접속해서
```sh
http://마스터_노드:9200

# http://ip-172-31-1-189:9200
```
