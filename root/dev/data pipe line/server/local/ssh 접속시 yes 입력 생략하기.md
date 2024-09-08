---
tags:
  - ssh
  - setting
create: 2024-08-01 21:31:43
---

## 기본 사용법

SSH 서버에 처음 접속할 때 "yes"를 입력하는 과정은 서버의 호스트 키를 확인하고 저장하는 과정이야. 이 과정을 자동으로 생략하려면 `StrictHostKeyChecking` 옵션을 사용하면 돼.

다음과 같은 방법을 사용해봐:

1. **명령어에 직접 추가하기**:
```sh
ssh -o StrictHostKeyChecking=no user@hostname
```

2. **SSH 설정 파일 수정하기**:
SSH 설정 파일인 `~/.ssh/config`에 설정을 추가하면 매번 옵션을 입력하지 않아도 돼.
- 설정 파일을 열고 (없으면 새로 만들면 돼):
```sh
vi ~/.ssh/config
```
- 다음 내용을 추가해:
```sh
Host 172.*.*.*
 StrictHostKeyChecking no
 UserKnownHostsFile=/dev/null
```

이렇게 하면 모든 호스트에 대해 호스트 키 확인을 하지 않게 돼.

**주의**: `StrictHostKeyChecking=no` 옵션을 사용하면 보안적으로 위험할 수 있어. 신뢰할 수 없는 서버에 접속할 때는 주의가 필요해. 가능한 경우, 초기 연결 시만 "yes"를 입력하고 나중에 설정 파일을 통해 관리하는 방법을 추천해.

---


## 호스트

맞아, SSH 설정 파일에 다음 내용을 추가하면 모든 호스트에 대해 StrictHostKeyChecking을 비활성화하고, 호스트 키를 known_hosts 파일에 저장하지 않게 설정할 수 있어. 이렇게 하면 SSH 접속 시 "yes"를 입력할 필요가 없어져.

### SSH 설정 파일 수정

1. SSH 설정 파일 열기:
   ```sh
   nano ~/.ssh/config
   ```

2. 다음 내용을 추가:
   ```sh
   Host *
     StrictHostKeyChecking no
     UserKnownHostsFile=/dev/null
   ```

   위 설정은 모든 SSH 접속에 대해 적용되며, 특정 IP 범위에 대해서만 적용하고 싶다면 `Host *` 대신 특정 IP 패턴을 사용할 수 있어.

### 특정 IP 패턴에 적용
만약 특정 IP 범위(예: `172.*.*.*`)에만 적용하고 싶다면 다음과 같이 설정할 수 있어:

```sh
Host 172.*
  StrictHostKeyChecking no
  UserKnownHostsFile=/dev/null
```


### 호스트 네임 패턴

```sh
Host server*
  StrictHostKeyChecking no
  UserKnownHostsFile=/dev/null
```

이렇게 하면 

```ssh
ssh server1
ssh server2
ssh server3
```

이렇게 모두 생략할 수 있다.