---
tags:
  - python
  - venv
  - setting
create: 2024-08-10 02:52:22
---
```sh
sudo rm /usr/lib/python3.11/EXTERNALLY-MANAGED
```

이 방법이 제일 깔금함


---

이 에러 메시지는 우분투에서 Python 패키지 설치를 시스템 전역으로 하는 대신, 가상환경을 사용하도록 권장하는 PEP 668에 따른 거야. 도커 컨테이너에서는 가상환경 없이 설치하는 게 더 편할 수 있어, 그래서 이 경고 메시지를 비활성화하고 패키지를 설치할 수 있는 방법을 알려줄게.

## 1. --break-system-packages 옵션 사용하기
가장 간단한 방법은 `--break-system-packages` 옵션을 추가해서 경고 메시지를 무시하고 패키지를 설치하는 거야. 예를 들어 `pip`을 사용할 때 이렇게 하면 돼:

```bash
pip install <패키지명> --break-system-packages
```

이 방법은 도커 컨테이너 내부에서 패키지를 설치할 때 유용해.

---

## 2. PEP 668 비활성화
PEP 668 기능을 완전히 비활성화하려면 `externally-managed` 파일을 삭제해도 돼. 이 파일이 `/usr/lib/python<버전>/EXTERNALLY-MANAGED` 경로에 있어. 

파일을 삭제해서 비활성화하려면 아래 명령어를 사용하면 돼:

```bash
sudo rm /usr/lib/python3.11/EXTERNALLY-MANAGED
```

이렇게 하면 PEP 668의 제한 없이 패키지를 설치할 수 있어. 이 방법은 도커 환경처럼 특정한 컨테이너 내부에서만 사용할 것을 권장해, 왜냐하면 시스템에 설치된 파이썬 환경을 깨뜨릴 수 있기 때문이야.

---
## 3. pipx 사용하기
만약 특정 패키지만 독립적으로 설치하고 싶다면,`pipx`를 사용하는 것도 방법이야. `pipx`는 각각의 패키지를 자체적인 가상환경에 설치해서 충돌을 피할 수 있게 해줘.

```bash
pipx install <패키지명>
```

하지만 도커 컨테이너에서 굳이 이렇게까지 할 필요는 없을 수도 있어.

도커 내부에서 자유롭게 패키지를 설치할 수 있도록 `--break-system-packages` 옵션이나 PEP 668 비활성화 방법을 선택하면 돼. 필요하면 더 물어봐!