---
tags:
  - kafka
  - note
create: 2024-07-23 10:44:52
---


```sh
bin/kafka-broker-api-versions.sh --bootstrap-server 172.31.6.227:9092
```

## 토픽 생성

### 서버 1

```sh
./bin/kafka-topics.sh --create --bootstrap-server 172.31.6.227:9092 --replication-factor 3 --partitions 3 --topic testtopic
```

```sh
./bin/kafka-topics.sh --bootstrap-server 172.31.6.227:9092 --describe --topic testtopic
```


### 서버 1 메시지 생성 -> 서버 2, 서버 3 메시지 받음

#### 서버 1

```sh
./bin/kafka-console-producer.sh --bootstrap-server 172.31.6.227:9092 --topic testtopic
```


#### 서버 2
```sh
./bin/kafkaconsole-consumer.sh --bootstrap-server 172.31.14.208:9092 --topic testtopic --from-beginning
```


#### 서버 3
```sh
./bin/kafkaconsole-consumer.sh --bootstrap-server 172.31.1.189:9092 --topic testtopic --from-beginning
```


---





## 데이터 조회


### 1. Kafka 명령줄 도구 사용

Kafka의 명령줄 도구를 사용하여 특정 토픽의 데이터를 확인할 수 있습니다. 

#### 1.1. **`kafka-console-consumer` 사용**

`kafka-console-consumer`는 Kafka 토픽에서 메시지를 소비하고 출력하는 명령줄 도구입니다. 이 도구를 사용하여 실시간으로 데이터가 올라와 있는지 확인할 수 있습니다.

```sh
kafka-console-consumer --bootstrap-server localhost:9092 --topic my_topic --from-beginning
```

```sh
./bin/kafka-console-consumer.sh --bootstrap-server 172.17.0.4:9092 --topic my_topi --from-beginning
```

- `--bootstrap-server`: Kafka 클러스터의 브로커 주소를 지정합니다.
- `--topic`: 데이터를 확인할 토픽의 이름을 지정합니다.
- `--from-beginning`: 토픽의 시작부터 메시지를 소비합니다. 이 옵션을 생략하면, 현재로부터 새로운 메시지만 소비합니다.

이 명령을 실행하면 해당 토픽에 올라와 있는 메시지가 콘솔에 출력됩니다.

#### 1.2. **`kafka-topics` 명령으로 토픽 정보 확인**

토픽의 메타데이터나 메시지 수를 확인할 수 있습니다.

```sh
kafka-topics --bootstrap-server.sh localhost:9092 --describe --topic my_topic
```

```sh
./bin/kafka-topics --bootstrap-server.sh 172.17.0.4:9092 --describe --topic my_topic
```

이 명령은 지정한 토픽의 파티션과 리더 상태, 오프셋 정보를 출력합니다. 이 정보를 통해 토픽의 상태를 확인할 수 있습니다.

### 2. Kafka 클라이언트 라이브러리 사용

Kafka 클라이언트 라이브러리를 사용하여 프로그래밍적으로 토픽의 데이터를 조회할 수도 있습니다. 다음은 Python을 사용한 예제입니다.

#### 2.1. **Kafka Python 클라이언트**

```python
from kafka import KafkaConsumer

# Kafka Consumer 설정
consumer = KafkaConsumer(
    'my_topic',
    bootstrap_servers=['localhost:9092'],
    auto_offset_reset='earliest',
    enable_auto_commit=True,
    value_deserializer=lambda x: x.decode('utf-8')
)

# 메시지 출력
for message in consumer:
    print(f"Key: {message.key}, Value: {message.value}")
    # 특정 조건에서 루프를 종료하려면 추가 로직을 구현
    break  # 이 예제에서는 첫 번째 메시지 하나만 출력 후 종료
```

이 코드는 `my_topic` 토픽에서 메시지를 소비하고 출력합니다. `auto_offset_reset='earliest'`를 설정하면 토픽의 처음부터 메시지를 읽습니다.

### 3. Kafka 웹 UI 도구 사용

Kafka의 웹 기반 UI 도구를 사용하여 토픽의 데이터와 상태를 시각적으로 확인할 수 있습니다. 이러한 도구로는 다음과 같은 것들이 있습니다:

- **[Confluent Control Center](https://docs.confluent.io/platform/current/control-center/index.html)**: Confluent Platform의 일부로 제공되며, Kafka 클러스터를 모니터링하고 관리할 수 있습니다.
- **[Kafdrop](https://github.com/obsidiandynamics/kafdrop)**: Kafka 브로커와 토픽을 시각적으로 확인할 수 있는 오픈소스 웹 UI 도구입니다.

이 도구들은 Kafka 클러스터의 상태를 모니터링하고, 토픽의 메시지를 조회하는 데 유용합니다.

### 결론

- **명령줄 도구**: `kafka-console-consumer`를 사용하여 실시간으로 토픽의 데이터를 확인할 수 있습니다.
- **클라이언트 라이브러리**: Python 등에서 Kafka 클라이언트를 사용하여 프로그램적으로 데이터를 조회할 수 있습니다.
- **웹 UI 도구**: Confluent Control Center나 Kafdrop 등의 도구를 사용하여 시각적으로 Kafka 클러스터와 토픽의 상태를 확인할 수 있습니다.

이 방법들을 사용하여 Kafka에 데이터가 제대로 올라와 있는지 확인할 수 있습니다.


---


## 실시간 메시지만 확인하기


1. **토픽의 데이터 유지**:
   - Kafka는 메시지를 토픽에 저장하고, 이 메시지는 기본적으로 삭제되지 않고 계속 유지됩니다. `--from-beginning` 옵션을 사용하면 토픽의 시작부터 모든 메시지를 읽게 됩니다.

2. **오프셋 관리**:
   - Kafka는 메시지의 위치를 `오프셋(offset)`이라는 숫자로 관리합니다. `--from-beginning`을 사용하면, 소비자가 현재 오프셋이 아닌, 토픽의 시작부터 모든 메시지를 읽기 시작합니다. 
   - 만약 소비자가 처음으로 메시지를 읽는 것이라면, 메시지의 시작부터 읽게 되며, 이전에 작성된 메시지도 모두 포함됩니다.

3. **소비자 그룹 상태**:
   - 만약 특정 소비자 그룹이 이미 존재하고 그 그룹이 저장된 오프셋이 있으면, `--from-beginning` 옵션을 사용하더라도 해당 소비자 그룹의 저장된 오프셋이 무시되고 시작부터 모든 메시지를 읽게 됩니다.
   - 새로운 소비자 그룹을 사용하거나, 기존 소비자 그룹의 오프셋이 초기화된 경우에도 동일한 결과를 얻을 수 있습니다.

**예시**:
- `kafka-console-consumer.sh --bootstrap-server 172.17.0.2:9092 --topic my_topic --from-beginning` 명령어를 실행하면, `my_topic`에서 생성된 모든 메시지를 토픽의 시작부터 현재까지 읽어오게 됩니다.

이 옵션을 사용하지 않으면, 소비자는 마지막으로 읽은 지점부터 새로운 메시지만 읽어오게 됩니다. 만약 이전 메시지들만 읽고 싶지 않다면 `--from-beginning` 옵션을 제거하고 명령어를 실행하세요:

```sh
kafka-console-consumer.sh --bootstrap-server 172.17.0.2:9092 --topic my_topic
```

이렇게 하면, 해당 토픽에서 새로 생성된 메시지만을 실시간으로 소비하게 됩니다.