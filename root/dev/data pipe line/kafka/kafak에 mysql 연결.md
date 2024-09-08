---
tags:
create: 2024-07-29 03:10:06
---

카프카를 데이터 파이프라인의 중심에 두고, 데이터 생산자는 카프카에 데이터를 전송하고, 카프카 커넥트를 사용해 데이터를 다양한 싱크(데이터베이스 등)로 전달하는 방식은 매우 일반적이고 안정적인 데이터 처리 방법이야.

카프카와 데이터베이스를 연결하는 것은 여러 가지 장점이 있어:

1. **확장성**: 카프카는 높은 처리량을 처리할 수 있어서 대규모 데이터 전송에도 적합해.
2. **신뢰성**: 카프카는 데이터의 내구성을 보장해, 데이터 손실 없이 안정적으로 전송할 수 있어.
3. **유연성**: 다양한 소스와 싱크를 연결할 수 있어 데이터 파이프라인을 유연하게 구성할 수 있어.
4. **비동기 처리**: 데이터 생산자와 소비자가 비동기적으로 동작할 수 있어서 시스템의 복잡도를 줄일 수 있어.

다시 한번 정리하면, 카프카와 데이터베이스를 연결하는 시나리오는 다음과 같아:

1. **데이터 생산자 (Python 코드)**: CSV 파일 데이터를 읽고, 카프카로 전송.
2. **카프카**: 데이터를 토픽으로 받음.
3. **카프카 커넥트**: 카프카 토픽 데이터를 MySQL 데이터베이스로 전송.

이 시나리오를 실제로 구현하는 방법을 구체적으로 설명할게.

### 1. Kafka Connect 설치 및 설정

**Kafka Connect**는 카프카의 생태계에 포함되어 있는 도구야. 여기서는 `io.confluent.connect.jdbc.JdbcSinkConnector`를 사용해 MySQL과 연결할 거야.

#### (1) Kafka Connect 설치

카프카를 이미 설치했다면 Kafka Connect도 함께 설치되어 있어.

#### (2) MySQL JDBC 드라이버 설치

MySQL JDBC 드라이버를 다운로드하고 Kafka Connect의 `plugins` 디렉토리에 복사해줘.

### 2. Kafka Connect 설정 파일 작성

MySQL 싱크 커넥터를 설정하는 JSON 파일을 작성해.

#### `mysql-sink-connector.json` 파일:
```json
{
  "name": "mysql-sink-connector",
  "config": {
    "connector.class": "io.confluent.connect.jdbc.JdbcSinkConnector",
    "tasks.max": "1",
    "topics": "my_topic",
    "connection.url": "jdbc:mysql://localhost:3306/my_data",
    "connection.user": "ubuntu",
    "connection.password": "1234",
    "auto.create": "true",
    "insert.mode": "insert",
    "pk.mode": "none",
    "table.name.format": "test_a"
  }
}
```

### 3. Kafka Connect 실행 및 커넥터 등록

Kafka Connect를 실행하고, MySQL 싱크 커넥터를 등록해.

#### (1) Kafka Connect 실행
Kafka Connect를 배포 모드로 실행해.

```bash
bin/connect-distributed.sh config/connect-distributed.properties
```

#### (2) MySQL 싱크 커넥터 등록
MySQL 싱크 커넥터를 Kafka Connect에 등록해.

```bash
curl -X POST -H "Content-Type: application/json" --data @mysql-sink-connector.json http://localhost:8083/connectors
```

### 4. Python 코드 (데이터를 카프카로 전송)

Python 코드를 작성해서 CSV 파일 데이터를 카프카 토픽으로 전송해.

```python
from confluent_kafka import Producer
import pandas as pd
import json

# 카프카 프로듀서 설정
KAFKA_SERVERS = '172.17.0.2:9092,172.17.0.3:9092,172.17.0.4:9092'
KAFKA_TOPIC = 'my_topic'

# CSV 파일에서 데이터 추출
data = pd.read_csv('data.csv')

# 카프카 프로듀서 설정
producer_conf = {
    'bootstrap.servers': KAFKA_SERVERS,
}
producer = Producer(producer_conf)

# 데이터를 카프카 토픽으로 전송
def delivery_report(err, msg):
    if err is not None:
        print(f"Message delivery failed: {err}")
    else:
        print(f"Message delivered to {msg.topic()} [{msg.partition()}]")

for index, row in data.iterrows():
    message = json.dumps({
        'column1': row['column1'],
        'column2': row['column2'],
        'column3': row['column3']  # assuming column3 is in a date format already
    })
    producer.produce(KAFKA_TOPIC, key=str(index), value=message, callback=delivery_report)

producer.flush()

print("데이터가 카프카로 성공적으로 전송되었습니다.")
```

이렇게 하면, 데이터는 다음과 같은 순서로 흐르게 돼:
1. **Python 코드**: CSV 파일 데이터를 카프카 토픽으로 전송.
2. **카프카**: 데이터를 토픽으로 받음.
3. **Kafka Connect**: 카프카 토픽 데이터를 MySQL 데이터베이스에 적재.

이 시나리오는 데이터 파이프라인의 확장성과 신뢰성을 모두 제공해, 데이터가 손실 없이 안정적으로 처리될 수 있어. 이 방식은 많은 대규모 데이터 처리 시스템에서 사용되는 검증된 방법이야.