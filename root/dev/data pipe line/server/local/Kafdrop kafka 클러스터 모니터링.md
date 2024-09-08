---
tags:
  - kafka
  - kafdrop
create: 2024-08-03 22:18:28
---

여러 브로커 서버가 있는 경우, `KAFKA_BROKERCONNECT` 환경 변수를 여러 브로커를 포함하도록 설정할 수 있습니다. Kafdrop은 이 변수를 통해 Kafka 클러스터의 브로커와 연결합니다. 

브로커 서버가 `server1`, `server2`, `server3`에 위치하고 각각의 IP 주소가 `172.17.0.2`, `172.17.0.3`, `172.17.0.4`라면, 이를 다음과 같이 설정할 수 있습니다.

## 1. Docker 명령어로 Kafdrop 실행

여러 브로커를 설정하려면 브로커 주소를 쉼표로 구분하여 `KAFKA_BROKERCONNECT` 환경 변수에 지정합니다.

```sh
docker run -d -p 9000:9000 \
  --add-host server1:172.17.0.2 \
  --add-host server2:172.17.0.3 \
  --add-host server3:172.17.0.4 \
  -e KAFKA_BROKERCONNECT=server1:9092,server2:9092,server3:9092 \
  obsidiandynamics/kafdrop
```

## 2. 브라우저로 Kafdrop 접속

컨테이너가 실행된 후 브라우저를 열고 다음 URL로 접속합니다.

```sh
http://localhost:9000
```



```sh
http://172.17.0.2:9000
```


이제 Kafdrop UI를 통해 Kafka 클러스터의 여러 브로커를 모니터링하고 관리할 수 있습니다.

## 추가 고려사항

### Docker 네트워크 설정

브로커 서버가 Docker 컨테이너 내에서 실행되고 있다면, Docker 네트워크 설정이 올바르게 되어 있는지 확인해야 합니다. 브로커 컨테이너와 Kafdrop 컨테이너가 동일한 네트워크에 있어야 통신이 가능합니다.

### 브로커 호스트와 포트

위 예시에서는 브로커 호스트가 `server1`, `server2`, `server3`이고, 포트가 `9092`로 동일하다고 가정했습니다. 실제 환경에서는 호스트명과 포트 번호가 다를 수 있으니 환경에 맞게 수정하세요.

## 요약

여러 Kafka 브로커를 사용하는 경우, Docker 명령어에서 `--add-host` 옵션을 사용해 브로커 호스트를 추가하고, `KAFKA_BROKERCONNECT` 환경 변수에 여러 브로커를 쉼표로 구분하여 설정합니다. 이를 통해 Kafdrop을 사용해 여러 브로커가 있는 Kafka 클러스터를 편리하게 관리할 수 있습니다.