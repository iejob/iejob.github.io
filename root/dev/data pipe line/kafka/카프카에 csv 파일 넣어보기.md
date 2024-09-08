---
tags:
  - kafka
  - code
create: 2024-07-30 00:22:15
---

```sh
pip install kafka-python
```


```python
from kafka import KafkaProducer
import pandas as pd
import json

def produce_csv_to_kafka(file_path, topic):
    # CSV 데이터 읽기
    data = pd.read_csv(file_path)

    # Kafka 프로듀서 설정 (클러스터 환경)
    producer = KafkaProducer(
        bootstrap_servers=['172.17.0.2:9092', '172.17.0.3:9092', '172.17.0.4:9092'],
        value_serializer=lambda v: json.dumps(v).encode('utf-8')
    )

    # 각 행을 Kafka 메시지로 게시
    for _, row in data.iterrows():
        message = row.to_dict()
        producer.send(topic, value=message)

    producer.flush()
    print("Kafka에 CSV 데이터 게시 완료.")

# CSV 파일 경로와 Kafka 토픽 설정
csv_file_path = 'data.csv'
produce_csv_to_kafka(csv_file_path, 'my_topic')
```



