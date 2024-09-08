---
tags:
  - kafka
  - mysql
  - code
create: 2024-07-30 00:30:24
---

## flow
```ad-info
csv -> python -> kafka -> mysql
```

---
## pip install
```sh
pip install kafka-python pandas sqlalchemy pymysql
```

---
## code

```sh
vi kafka_consumer_mysql.py
```

```python
from kafka import KafkaProducer, KafkaConsumer
import pandas as pd
import json
from sqlalchemy import create_engine, Table, Column, MetaData, inspect
from sqlalchemy.types import String, BigInteger, Float, DateTime
import pymysql

CSV_DATA = 'data_mini.csv'

# 데이터베이스 연결 설정
DATABASE_USER = 'ubuntu'
DATABASE_PASSWORD = '1234'
DATABASE_HOST = 'localhost'
DATABASE_NAME = 'my_data'

# Kafka 설정
KAFKA_BOOTSTRAP_SERVERS = ['172.17.0.2:9092', '172.17.0.3:9092', '172.17.0.4:9092']
KAFKA_TOPIC = 'testtopic'
KAFKA_GROUP_ID = 'my-group'  # group_id 설정

# 데이터베이스 연결 함수
def create_database_if_not_exists():
	connection = pymysql.connect(
		host=DATABASE_HOST,
		user=DATABASE_USER,
		password=DATABASE_PASSWORD
	)
	try:
		with connection.cursor() as cursor:
			cursor.execute(f"CREATE DATABASE IF NOT EXISTS {DATABASE_NAME}")
	finally:
		connection.close()

# SQLAlchemy 데이터 타입 매핑 함수
def get_sqlalchemy_type(dtype):
	if pd.api.types.is_integer_dtype(dtype):
		return BigInteger
	elif pd.api.types.is_float_dtype(dtype):
		return Float
	elif pd.api.types.is_datetime64_any_dtype(dtype):
		return DateTime
	else:
		return String(length=255)

# 데이터베이스 및 테이블 설정
def setup_database_and_table(csv_path):
	engine = create_engine(f'mysql+pymysql://{DATABASE_USER}:{DATABASE_PASSWORD}@{DATABASE_HOST}/{DATABASE_NAME}')
	
	df = pd.read_csv(csv_path)
	
	metadata = MetaData()
	
	columns = [Column(col, get_sqlalchemy_type(df[col].dtype)) for col in df.columns]
	table = Table('test_a', metadata, *columns)
	
	inspector = inspect(engine)
	if not inspector.has_table('test_a'):
		metadata.create_all(engine)
		print(f"Table 'test_a' created successfully from CSV file.")
	else:
		print(f"Table 'test_a' already exists.")

def produce_csv_to_kafka(file_path, topic):
	data = pd.read_csv(file_path)
	producer = KafkaProducer(
		bootstrap_servers=KAFKA_BOOTSTRAP_SERVERS,
		value_serializer=lambda v: json.dumps(v).encode('utf-8')
	)
	
	for _, row in data.iterrows():
		message = row.to_dict()
		producer.send(topic, value=message)
	producer.flush()
	producer.close()
	print("CSV 데이터를 Kafka에 게시 완료.")

def load_data_to_mysql(data):
	engine = create_engine(f'mysql+pymysql://{DATABASE_USER}:{DATABASE_PASSWORD}@{DATABASE_HOST}/{DATABASE_NAME}')
	df = pd.DataFrame(data)
	df.to_sql('test_a', engine, if_exists='append', index=False)

consumer = KafkaConsumer(
	KAFKA_TOPIC,
	bootstrap_servers=KAFKA_BOOTSTRAP_SERVERS,
	auto_offset_reset='earliest',
	enable_auto_commit=False,
	group_id=KAFKA_GROUP_ID,  # group_id 추가
	value_deserializer=lambda x: json.loads(x.decode('utf-8'))
)

create_database_if_not_exists()
setup_database_and_table(CSV_DATA)

produce_csv_to_kafka(CSV_DATA, KAFKA_TOPIC)

data_buffer = []
buffer_limit = 100
timeout_ms = 500  # 5초 동안 새로운 메시지가 없으면 타임아웃

while True:
	message_pack = consumer.poll(timeout_ms=timeout_ms)
	
	if not message_pack:
		break
	
	for tp, messages in message_pack.items():
		for message in messages:
			data_buffer.append(message.value)
			
			if len(data_buffer) >= buffer_limit:
				load_data_to_mysql(data_buffer)
				data_buffer = []
				consumer.commit()

if data_buffer:
	load_data_to_mysql(data_buffer)
	consumer.commit()

consumer.close()

print("Kafka에서 MySQL로 데이터 적재 완료.")

```

---
## 코드 설명

1. **Kafka Consumer 설정**:
   - `KafkaConsumer`를 사용하여 지정된 토픽(`my_topic`)에서 메시지를 소비합니다.
   - `bootstrap_servers`에는 Kafka 클러스터의 브로커 주소를 지정합니다.

2. **MySQL 데이터베이스와 테이블 설정**:
   - 데이터베이스와 테이블이 존재하지 않으면 새로 생성합니다.

3. **데이터 적재 함수 (`load_data_to_mysql`)**:
   - Kafka에서 받은 메시지를 `pandas` 데이터프레임으로 변환하고, `to_sql` 메서드를 사용하여 MySQL에 데이터를 적재합니다.
   - `if_exists='append'` 옵션을 사용하여 기존 테이블에 데이터를 추가합니다.

4. **Kafka에서 메시지 소비 및 MySQL에 적재**:
   - Kafka에서 메시지를 소비하여 `data_buffer`에 저장합니다.
   - 버퍼의 데이터 수가 일정 한도(`buffer_limit`)를 넘으면, `load_data_to_mysql` 함수를 호출하여 MySQL에 데이터를 적재합니다.
   - 버퍼가 비워진 후에도 데이터가 남아 있으면, 마지막으로 MySQL에 적재합니다.


### 주의 사항

- **데이터 변환**: Kafka에서 받은 데이터는 JSON 형식으로 변환되므로, MySQL의 테이블 구조에 맞게 데이터 변환이 필요할 수 있습니다. 실제 환경에서는 데이터를 필터링하거나 변환할 필요가 있을 수 있습니다.

- **에러 처리**: 실제 사용 시에는 예외 처리 및 로그 기록을 추가하여 오류를 모니터링하고 대응할 수 있습니다.

- **성능 고려**: 데이터 적재 성능을 높이기 위해, 데이터 배치 처리(batch processing)나 병렬 처리를 고려할 수 있습니다.

이 코드는 실시간으로 Kafka에서 데이터를 소비하고 MySQL에 적재할 수 있도록 설계되었으며, 데이터가 지속적으로 들어오는 환경에서 유용합니다.


---

## 비교 설명

```ad-info
데이터를 MySQL에 적재할 때, Kafka를 거치는 것과 직접 MySQL에 적재하는 것 사이에는 여러 가지 차이점과 고려할 사항이 있습니다. 아래에 각각의 방법이 데이터베이스에 미치는 영향과 안정성에 대해 비교하여 설명하겠습니다.
```

### 1. Kafka를 통한 데이터 적재

**장점:**

1. **내결함성 (Fault Tolerance)**:
   - Kafka는 메시지의 내구성을 보장하고 데이터 복제를 통해 장애에 대비할 수 있습니다. Kafka에 데이터가 게시되면, Kafka는 이를 복제하고 유지하여 MySQL 데이터베이스가 다운되더라도 데이터 손실을 방지할 수 있습니다.

2. **성능 향상 (Performance)**:
   - Kafka는 데이터를 버퍼링하고, 배치로 처리하여 대량의 데이터를 효율적으로 처리합니다. 이로 인해 MySQL 데이터베이스에 직접적인 부하를 줄일 수 있습니다.

3. **비동기 처리 (Asynchronous Processing)**:
   - Kafka는 비동기 처리 모델을 사용하여 데이터 전송과 처리를 분리합니다. 이를 통해 데이터 수집과 처리의 병목 현상을 줄일 수 있습니다.

4. **확장성 (Scalability)**:
   - Kafka는 클러스터 기반으로 설계되어 데이터의 양이 증가하더라도 확장 가능성을 제공합니다. Kafka의 파티셔닝 및 복제 기능을 활용하여 데이터 처리량을 높일 수 있습니다.

**단점:**

1. **복잡성 증가 (Increased Complexity)**:
   - Kafka를 추가하면 시스템의 복잡성이 증가합니다. Kafka 브로커의 설정, 모니터링, 유지 관리가 필요합니다.

2. **지연 (Latency)**:
   - Kafka를 경유하면 데이터 전송에 약간의 지연이 있을 수 있습니다. 하지만 이는 일반적으로 성능 향상과 데이터 안정성을 고려할 때 감수할 수 있는 부분입니다.


### 2. Kafka 없이 직접 MySQL에 데이터 적재

**장점:**

1. **간단한 아키텍처 (Simpler Architecture)**:
   - Kafka를 사용하지 않으면 데이터 전송 경로가 간단해지며 시스템이 간결해집니다. 설정 및 유지 관리가 단순해질 수 있습니다.

2. **즉각적인 데이터 적재 (Immediate Data Insertion)**:
   - 데이터가 직접적으로 MySQL에 적재되므로 지연이 적을 수 있습니다.

**단점:**

1. **부하 (Load)**:
   - MySQL에 직접적으로 데이터를 적재할 경우, 데이터베이스에 큰 부하를 줄 수 있습니다. 특히 대량의 데이터가 동시에 들어오면 MySQL의 성능이 저하될 수 있습니다.

2. **내결함성 부족 (Lack of Fault Tolerance)**:
   - MySQL에 직접 데이터를 적재하는 경우, 데이터베이스의 장애나 다운타임에 대한 대비책이 부족할 수 있습니다. 데이터 손실이나 서비스 중단의 위험이 존재합니다.

3. **확장성 문제 (Scalability Issues)**:
   - 데이터량이 증가할수록 MySQL의 성능이 저하될 수 있으며, 클러스터를 사용하지 않는 경우 데이터베이스의 확장에 한계가 있을 수 있습니다.

4. **동기 처리 (Synchronous Processing)**:
   - 데이터 적재가 동기식으로 처리되면 데이터베이스의 응답 시간과 부하에 영향을 받을 수 있습니다.

---
## 결론

- **Kafka를 사용하는 경우**: Kafka는 데이터의 내결함성과 확장성을 제공하며, 대량의 데이터를 처리할 때 안정성을 높입니다. MySQL에 대한 부하를 줄이고, 장애 발생 시 데이터 손실을 방지할 수 있습니다. Kafka는 비동기식 데이터 전송을 통해 성능을 향상시키지만, 시스템의 복잡성이 증가할 수 있습니다.

- **Kafka 없이 직접 MySQL에 적재하는 경우**: 시스템이 간단해지고 즉각적인 데이터 적재가 가능하지만, 데이터베이스에 부하가 증가하고 내결함성과 확장성에 한계가 있을 수 있습니다. 특히 대량의 데이터가 있을 때 안정성 문제를 겪을 수 있습니다.

따라서, 데이터의 양과 시스템의 요구 사항에 따라 적절한 접근 방식을 선택하는 것이 중요합니다. 대량의 데이터 처리와 높은 안정성이 필요하다면 Kafka를 사용하는 것이 좋습니다. 반면, 데이터량이 상대적으로 적고 시스템의 복잡성을 줄이려는 경우 직접 MySQL에 적재하는 것도 고려할 수 있습니다.


