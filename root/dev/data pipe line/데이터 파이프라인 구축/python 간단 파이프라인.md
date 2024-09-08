---
tags:
  - python
  - mysql
  - pandas
create: 2024-07-25 02:06:51
---

간단한 데이터 파이프라인 예제를 통해 데이터 추출, 변환, 적재를 보여줄게. 여기서는 CSV 파일에서 데이터를 추출하고, 변환한 후 MySQL 데이터베이스에 적재하는 과정을 포함할 거야.

**예제: CSV 파일에서 데이터를 추출하고, 간단한 변환을 한 후 MySQL 데이터베이스에 적재하기**

1. **환경 설정:**

   - `pandas` 라이브러리: 데이터 처리
   - `sqlalchemy` 라이브러리: 데이터베이스 연결 및 데이터 적재
   - `pymysql` 라이브러리: MySQL 연결

   필요한 라이브러리를 설치해:

   ```sh
   pip install pandas sqlalchemy pymysql
   ```

2. **CSV 파일 준비:**

   예를 들어, `data.csv` 파일이 다음과 같은 형식을 가질 수 있어:

   ```
   name,age
   Alice,30
   Bob,24
   Charlie,29
   ```

3. **파이썬 스크립트 작성:**

   아래는 CSV 파일에서 데이터를 읽고, 변환한 후 MySQL 데이터베이스에 적재하는 스크립트야:

```python
import pandas as pd
from sqlalchemy import create_engine, text
import pymysql

# 데이터베이스 연결 설정
DATABASE_USER = 'ubuntu'
DATABASE_PASSWORD = '1234'
DATABASE_HOST = 'localhost'
DATABASE_NAME = 'my_data'

# 데이터 추출
data = pd.read_csv('data.csv')

# MySQL 서버에 연결 (데이터베이스 생성용)
connection = pymysql.connect(
	host=DATABASE_HOST,
	user=DATABASE_USER,
	password=DATABASE_PASSWORD
)

# 데이터베이스 생성
with connection.cursor() as cursor:
	cursor.execute(f"CREATE DATABASE IF NOT EXISTS {DATABASE_NAME}")
connection.close()

# 데이터베이스에 연결
engine = create_engine(f'mysql+pymysql://{DATABASE_USER}:{DATABASE_PASSWORD}@{DATABASE_HOST}/{DATABASE_NAME}')

# 테이블 존재 여부 확인 및 생성
with engine.connect() as conn:
	result = conn.execute(text("SHOW TABLES LIKE 'test_a'"))
	table_exists = result.fetchone() is not None

if not table_exists:
	# 테이블이 없으면 생성
	with engine.connect() as conn:
		conn.execute(text('''
			CREATE TABLE test_a (
				column1 VARCHAR(255),
				column2 INT,
				column3 DATE
			)
		'''))

# 데이터 적재 (테이블 이름은 'test_a')
data.to_sql('test_a', engine, if_exists='replace', index=False)

print("데이터가 성공적으로 적재되었습니다.")

```

4. **스크립트 실행:**

   위의 파이썬 스크립트를 저장하고 실행해보자:

   ```sh
   python your_script_name.py
   ```

**스크립트 설명:**

1. **데이터 추출:** `pandas`를 사용해 CSV 파일을 읽어 `data` 데이터프레임으로 저장.
2. **데이터 변환:** 나이 컬럼의 값을 1.5배로 변환. (변환은 예시일 뿐이야, 필요에 따라 다른 변환을 할 수 있어.)
3. **데이터베이스 연결:** `sqlalchemy`의 `create_engine`을 사용해 MySQL 데이터베이스에 연결.
4. **데이터 적재:** `to_sql` 메서드를 사용해 데이터프레임을 MySQL 데이터베이스에 적재. `if_exists='replace'`는 테이블이 이미 존재하면 교체하도록 설정해.

이렇게 하면, CSV 파일의 데이터를 MySQL 데이터베이스에 적재하는 간단한 데이터 파이프라인을 구현할 수 있어. 필요에 따라 이 구조를 확장하거나, 추가적인 데이터 처리 단계를 넣어볼 수 있어.