---
tags:
  - ssh
  - setup
create: 2024-08-04 22:40:44
---

`/etc/hosts` 파일에는 호스트 이름과 IP 주소의 매핑만을 설정할 수 있습니다. 포트 번호를 직접 지정할 수 있는 기능은 없습니다. `ssh` 명령어에서 포트 번호를 지정하려면 `ssh` 구성 파일을 사용하는 것이 일반적인 방법입니다.

## SSH 구성 파일 (`~/.ssh/config`) 사용

SSH 구성 파일을 사용하여 특정 호스트에 대해 기본 포트 번호를 설정할 수 있습니다. `~/.ssh/config` 파일에 다음과 같이 설정을 추가하면 됩니다:

```bash
# ~/.ssh/config

Host server1
	HostName 172.31.14.186
	Port 2222
	User ubuntu

Host server2
	HostName 172.31.1.229
	Port 2222
	User ubuntu

Host server3
	HostName 172.31.10.99
	Port 2222
	User ubuntu
```

이렇게 설정하면 `ssh server1` 명령어를 실행할 때 자동으로 `-p 2222` 옵션이 적용됩니다.

## `send_ssh_comm` 함수에서 구성 파일 활용

구성 파일을 설정한 후, `send_ssh_comm` 함수를 다음과 같이 사용할 수 있습니다:

```python
import subprocess

def send_ssh_comm(host, name, comm):
	ssh_command = [
			'ssh',
			'-o', 'StrictHostKeyChecking=no',
			host, comm
	]
	print(f"=========={name} 실행...========")
	run = subprocess.run(ssh_command, capture_output=True, text=True)
	print(run.stdout)

# 사용 예시
send_ssh_comm('server1', '테스트 명령', 'echo Hello, World!')
```

이렇게 하면 `server1`에 대해 `~/.ssh/config` 파일에 정의된 설정이 자동으로 적용됩니다.

## `/etc/hosts` 파일의 내용

`/etc/hosts` 파일은 그대로 유지합니다:

```bash
172.31.14.186   server1
172.31.1.229    server2
172.31.10.99    server3
```

이렇게 하면 `server1`, `server2`, `server3`에 대해 정의된 IP 주소가 호스트 이름으로 매핑됩니다.

## 종합 예제

```python
import subprocess

def send_ssh_comm(host, name, comm):
	ssh_command = [
			'ssh',
			'-o', 'StrictHostKeyChecking=no',
			host, comm
	]
	print(f"=========={name} 실행...========")
	run = subprocess.run(ssh_command, capture_output=True, text=True)
	print(run.stdout)

# 사용 예시
send_ssh_comm('server1', '테스트 명령', 'echo Hello, World!')
send_ssh_comm('server2', '테스트 명령', 'echo Hello, World!')
send_ssh_comm('server3', '테스트 명령', 'echo Hello, World!')
```

이렇게 하면 `server1`, `server2`, `server3`에 대해 정의된 포트 번호가 자동으로 적용됩니다.