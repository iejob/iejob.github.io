---
tags:
create: 2024-07-31 00:36:23
---


## `%`의 의미

- **`%`**: MySQL에서 `%`는 "모든 호스트"를 의미해. 즉, `%`를 사용하면 어떤 IP 주소에서든지 이 사용자 계정으로 MySQL 서버에 접근할 수 있도록 허용하는 거야.

## 사용자 계정 예시

1. **모든 호스트에서 접근 가능**:

   ```sql
   CREATE USER 'ubuntu'@'%' IDENTIFIED BY '1234';
   GRANT ALL PRIVILEGES ON *.* TO 'ubuntu'@'%' WITH GRANT OPTION;
   ```

   이 명령어는 `ubuntu`라는 사용자 계정이 어떤 IP 주소에서든지 접속할 수 있도록 설정해. 즉, `server3`에서뿐만 아니라 다른 모든 서버에서도 이 계정을 사용해 MySQL 서버에 접근할 수 있어.

2. **특정 IP 주소에서만 접근 가능**:

   만약 특정 IP 주소에서만 접근을 허용하려면 `%` 대신 그 IP 주소를 사용하면 돼. 예를 들어 `172.17.0.3`에서만 접근을 허용하려면:

   ```sql
   CREATE USER 'ubuntu'@'172.17.0.3' IDENTIFIED BY '1234';
   GRANT ALL PRIVILEGES ON *.* TO 'ubuntu'@'172.17.0.3' WITH GRANT OPTION;
   ```

   이 경우, `ubuntu` 사용자는 `172.17.0.3`에서만 MySQL 서버에 접근할 수 있어.

3. **특정 호스트 이름에서 접근 가능**:

   특정 호스트 이름을 사용해 접근을 제한할 수도 있어. 예를 들어 `server1.example.com`에서만 접근을 허용하려면:

   ```sql
   CREATE USER 'ubuntu'@'server1.example.com' IDENTIFIED BY '1234';
   GRANT ALL PRIVILEGES ON *.* TO 'ubuntu'@'server1.example.com' WITH GRANT OPTION;
   ```

   이 경우, `ubuntu` 사용자는 `server1.example.com`이라는 호스트 이름으로만 MySQL 서버에 접근할 수 있어.

## 요약

- **`%`**: 모든 호스트에서 접근 허용.
- **특정 IP 주소**: 특정 IP 주소에서만 접근 허용.
- **호스트 이름**: 특정 호스트 이름에서만 접근 허용.

이렇게 설정을 통해 MySQL 서버에 접근할 수 있는 사용자 계정과 그 계정이 접속할 수 있는 호스트를 세밀하게 제어할 수 있어. 추가로 궁금한 점이 있으면 언제든지 물어봐!
이렇게 만드는데, '계정명'@'% 이 부분에서 % 부분에 대해서 알려줘
