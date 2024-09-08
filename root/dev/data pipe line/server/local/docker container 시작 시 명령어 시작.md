---
tags:
  - docker
  - command
create: 2024-07-28 15:24:42
---
```ad-attention
title:sudo 권한
으로 실행하는 것들은 앞에 sudo를 붙여 줘야 함.
그리고 계정이 sudo passwd 생략 할 수 있는 권한이 있어야 함

[[우분투 sudo 비번 생략하기]]
```


```sh
docker run -itd \
--name my_ssh_container \
root-ubuntu:24.01-mysql \
/bin/bash -c "sudo service ssh start && tail -f /dev/null"
```



```sh
docker run -itd --name my_ssh_container \
root-ubuntu:24.01-mysql \
/bin/bash -c "sudo service ssh start && sudo service mysql start && tail -f /dev/null"
```



컨테이너가 바로 종료되는 이유는 `-c "service ssh start && bash"` 명령어가 제대로 전달되지 않아서 `bash`가 실행되지 않기 때문이야. Docker에서는 `CMD` 명령어를 통해 전달되는 쉘 명령어를 실행할 때, `-c` 옵션을 사용하지 않아도 돼.

올바르게 실행하려면 다음과 같이 해봐:

```sh
docker run -itd --name my_ssh_container root-ubuntu:24.04-ssh-key /bin/bash -c "service ssh start && tail -f /dev/null"
```

여기서 `tail -f /dev/null`은 컨테이너가 백그라운드에서 계속 실행되도록 해주는 역할을 해. 

### 요약

1. `docker run -itd`: 백그라운드에서 인터랙티브 모드로 컨테이너 실행.
2. `--name my_ssh_container`: 컨테이너 이름을 `my_ssh_container`로 설정.
3. `root-ubuntu:24.04-ssh-key`: 사용할 Docker 이미지 이름.
4. `/bin/bash -c "service ssh start && tail -f /dev/null"`: SSH 서비스를 시작하고, `tail -f /dev/null` 명령어를 실행해서 컨테이너가 계속 실행 상태를 유지하도록 함.

이 명령어를 실행하면 SSH 서비스가 시작된 상태로 컨테이너가 계속 실행될 거야. 이제 컨테이너가 종료되지 않고, SSH 서버도 실행된 상태를 유지할 거야.