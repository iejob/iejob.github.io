---
tags:
  - kafka
  - code
create: 2024-07-31 02:23:35
---

## flow
```ad-info
csv -> python -> kafka
```

---
## pip install
```sh
pip install kafka-python pandas
```

---
## code
```sh
vi csv_to_kafka.py
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


---


## 코드 설명

1. **CSV 데이터 읽기**: `pandas` 라이브러리를 사용하여 CSV 파일을 읽습니다.
2. **Kafka 프로듀서 설정**: 
   - `KafkaProducer`를 사용하여 Kafka 클러스터에 데이터를 게시합니다.
   - `bootstrap_servers`에는 Kafka 클러스터의 브로커 주소를 지정합니다.
   - `value_serializer`를 사용하여 메시지 값을 JSON 형식으로 직렬화합니다.
3. **Kafka에 데이터 게시**: 
   - CSV 데이터프레임의 각 행을 `row.to_dict()`를 사용하여 딕셔너리 형태로 변환합니다.
   - 각 딕셔너리를 Kafka 메시지로 게시합니다.
   - `producer.flush()`를 호출하여 모든 메시지가 Kafka에 전송되도록 합니다.

## 주의 사항

- **데이터 형식**: CSV 파일의 데이터 형식이 Kafka의 데이터 형식과 일치해야 합니다. 필요에 따라 데이터 변환이 필요할 수 있습니다.
- **에러 처리**: 실제 사용 시에는 예외 처리 및 로그 기록을 추가하여 오류를 모니터링하고 대응할 수 있습니다.
- **성능 고려**: 대량의 데이터를 처리하는 경우, 데이터를 배치로 처리하거나 병렬 처리를 고려하여 성능을 향상시킬 수 있습니다.


## 추가 정보

이 코드는 CSV 파일을 읽어 Kafka에 데이터를 게시하는 기본적인 예시입니다. 실제 환경에서는 다음과 같은 추가 기능을 고려해야 할 수 있습니다.

- **데이터 필터링**: 특정 조건을 만족하는 데이터만 Kafka에 게시합니다.
- **데이터 변환**: CSV 파일의 데이터 형식을 Kafka에 적합하게 변환합니다.
- **데이터 압축**: 데이터 크기를 줄이기 위해 데이터를 압축합니다.
- **메시지 키 설정**: Kafka의 파티셔닝을 위해 메시지 키를 설정합니다.
- **메시지 타임스탬프 설정**: 메시지의 타임스탬프를 설정하여 시간순으로 데이터를 처리합니다.
- **데이터 유효성 검사**: 데이터의 유효성을 검사하여 오류를 방지합니다.

이 코드를 기반으로 필요한 추가 기능을 구현하여 CSV 파일에서 Kafka로 데이터를 효율적으로 전송할 수 있습니다. 
