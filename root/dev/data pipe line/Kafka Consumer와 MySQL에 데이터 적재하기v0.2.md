---
tags:
create: 2024-08-22 02:09:05
---
### 수정된 코드
아래는 주어진 코드에 Kafka의 속도 조절 기능을 추가한 버전이야.

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
KAFKA_BOOTSTRAP_SERVERS = ['172.20.0.10:9092', '172.20.0.11:9092', '172.20.0.12:9092']
KAFKA_TOPIC = 'my_data'
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
        value_serializer=lambda v: json.dumps(v).encode('utf-8'),
        linger_ms=1000,  # 메시지를 1초 동안 모아서 배치로 보냄
        batch_size=32 * 1024  # 32KB의 배치를 구성하여 전송
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
    value_deserializer=lambda x: json.loads(x.decode('utf-8')),
    max_poll_records=10  # 한 번의 poll()에서 가져오는 메시지 수를 10개로 제한
)

create_database_if_not_exists()
setup_database_and_table(CSV_DATA)

produce_csv_to_kafka(CSV_DATA, KAFKA_TOPIC)

data_buffer = []
buffer_limit = 100
timeout_ms = 5000  # 5초 동안 새로운 메시지가 없으면 타임아웃

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

### 적용된 주요 변경 사항

1. 생산자 측 (`KafkaProducer`):
- `linger.ms=1000`: 메시지를 전송하기 전에 최대 1초 동안 데이터를 모아서 한 번에 보내도록 설정했어. 이를 통해 초당 메시지 전송 빈도를 조절할 수 있어.
- `batch_size=32 * 1024`: 32KB의 데이터가 모이면 한 번에 전송하게 설정했어. 이는 생산자가 작은 메시지를 자주 보내지 않고, 일정 크기만큼 모아서 보내도록 하여 네트워크 효율을 높일 수 있어.

2. 소비자 측 (`KafkaConsumer`):
- `max_poll_records=10`: 한 번의 `poll()` 호출로 가져올 수 있는 메시지 수를 10개로 제한했어. 이를 통해 소비자가 처리해야 할 데이터의 양을 조절하고, 초당 처리량을 낮출 수 있어.

3. 타임아웃 및 버퍼링:
- `timeout_ms=5000`: Kafka 소비자에서 데이터를 가져올 때, 5초 동안 새로운 메시지가 없으면 타임아웃되도록 설정했어. 이 값을 통해 소비자가 새로운 데이터를 기다리는 시간을 조절할 수 있어.
- 버퍼링: 소비자는 100개의 메시지를 버퍼에 모은 후 MySQL로 저장해. 이로 인해 데이터베이스에 대한 트래픽이 줄고, 효율적으로 데이터를 처리할 수 있어.

### 결론
이렇게 설정하면 Kafka를 사용해 데이터를 전송하고 처리하는 속도를 조절할 수 있어. 생산자와 소비자 양쪽에서 데이터를 모아 배치로 처리하거나, 일정 수량만큼 한 번에 가져와 처리할 수 있도록 설정한 것이 주요 포인트야. 이로 인해 시스템의 부하를 줄이면서도 안정적으로 데이터를 처리할 수 있게 돼.