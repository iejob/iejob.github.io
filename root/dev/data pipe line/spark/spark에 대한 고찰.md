---
tags:
  - spark
  - python
create: 2024-07-27 14:57:19
---

Spark는 빅 데이터 처리와 분석에 널리 사용되는 오픈 소스 분산 컴퓨팅 시스템입니다. Kafka와 마찬가지로 데이터 처리와 분석에 있어 강력한 도구이지만, Spark는 특히 **대규모 데이터 처리와 분석**에서 빛을 발합니다. Spark를 사용할 때와 사용하지 않을 때의 차이점과 이를 체험할 수 있는 실습 코드를 제공하겠습니다.

## Spark의 장점

1. **속도**:
   - Spark는 메모리 내에서 데이터를 처리하기 때문에, 디스크 기반의 Hadoop MapReduce에 비해 최대 100배 빠른 속도로 데이터를 처리할 수 있습니다.

2. **유연성**:
   - Spark는 SQL, 데이터 프레임, 스트리밍, 머신러닝, 그래프 처리를 지원하여 다양한 데이터 처리 요구사항을 충족할 수 있습니다.

3. **확장성**:
   - Spark는 클러스터 컴퓨팅 프레임워크로 설계되어 있으며, 대규모 데이터 처리와 분석 작업을 수천 대의 노드에서 동시에 수행할 수 있습니다.

4. **배치 및 실시간 스트리밍 처리**:
   - Spark는 배치 처리와 실시간 스트리밍 처리를 모두 지원하여, 실시간 데이터 분석 및 처리에도 적합합니다.

## Spark를 사용한 시나리오

### 시나리오 1: Spark를 사용하지 않고 MySQL로 직접 데이터 처리

이 시나리오에서는 데이터를 MySQL에 직접 로드하고, SQL 쿼리를 사용하여 분석하거나 처리합니다.

#### 장점:
- 설정이 간단하며, SQL을 사용해 손쉽게 데이터를 처리할 수 있습니다.

#### 단점:
- 대규모 데이터 처리에 있어 성능이 제한됩니다.
- 실시간 데이터 처리 및 복잡한 데이터 분석 작업에 있어 유연성이 떨어집니다.

### 시나리오 2: Spark를 사용한 데이터 처리

이 시나리오에서는 데이터를 Spark에 로드하여, 데이터 프레임 API, SQL, 또는 스트리밍 API를 사용하여 데이터를 처리하고 분석합니다.

#### 장점:
- 대규모 데이터 세트를 빠르게 처리할 수 있습니다.
- 실시간 데이터 스트리밍과 배치 처리 모두 지원합니다.
- 다양한 분석 작업을 효율적으로 수행할 수 있습니다.

#### 단점:
- 초기 설정과 학습 곡선이 있으며, 클러스터 관리와 같은 추가적인 운영 작업이 필요할 수 있습니다.

## 실습 예제: CSV 데이터를 Spark로 처리

### 사전 준비
- Apache Spark 설치
- PySpark 라이브러리 설치

### CSV 데이터를 Spark 데이터 프레임으로 처리

#### CSV 파일 예시 (`users.csv`)
```
name,age
Alice,30
Bob,24
Charlie,29
```

#### Spark를 사용하지 않고 MySQL로 데이터 처리

기본적으로 MySQL로 CSV 데이터를 읽고 처리하는 코드는 이전과 동일하므로 생략합니다.

#### Spark를 사용한 데이터 처리

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col

# SparkSession 생성
spark = SparkSession.builder \
    .appName("CSV to DataFrame") \
    .config("spark.master", "local") \
    .getOrCreate()

# CSV 파일을 Spark DataFrame으로 로드
df = spark.read.csv('users.csv', header=True, inferSchema=True)

# 데이터 출력
df.show()

# 데이터 필터링 (예: 나이가 25 이상인 사용자만 선택)
filtered_df = df.filter(col('age') >= 25)

# 필터링된 데이터 출력
filtered_df.show()

# 필터링된 데이터를 MySQL에 저장 (예제, 실제 동작 시 데이터베이스 연결 필요)
# filtered_df.write \
#     .format("jdbc") \
#     .option("url", "jdbc:mysql://localhost:3306/my_data") \
#     .option("dbtable", "users_filtered") \
#     .option("user", "root") \
#     .option("password", "password") \
#     .save()

# SparkSession 종료
spark.stop()
```

## 실습 방법

1. **Spark 없이 MySQL을 사용하는 경우**:
   - 데이터를 MySQL에 직접 로드하고, SQL을 사용하여 데이터를 분석하거나 처리합니다.

2. **Spark를 사용하는 경우**:
   - Spark 데이터 프레임 API를 사용하여 데이터를 로드, 처리, 분석합니다. 예를 들어, 특정 조건에 맞는 데이터를 필터링하거나, 데이터를 집계하고, 이를 다시 MySQL에 저장할 수 있습니다.

이 실습을 통해 Spark를 사용했을 때의 장점, 특히 **대규모 데이터 처리 속도**와 **유연성**을 직접 경험할 수 있습니다. Spark의 강력한 데이터 처리 및 분석 기능은 특히 대량의 데이터 처리 및 복잡한 분석 작업에서 그 진가를 발휘합니다.