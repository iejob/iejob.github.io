---
tags:
  - python
  - mysql
create: 2024-07-25 01:45:19
---

물론이지! 파이썬으로 데이터를 MySQL에 삽입하는 간단한 예제를 만들어볼게. 이를 위해 `pymysql` 라이브러리를 사용할 거야. 이 라이브러리를 사용하면 파이썬에서 MySQL 서버와 쉽게 통신할 수 있어.

먼저, `pymysql` 라이브러리를 설치해야 해:

```sh
pip install pymysql
```

그리고 아래는 파이썬 스크립트의 예제야:

```python
import pymysql

# MySQL 연결 설정
connection = pymysql.connect(
    host='localhost',  # MySQL 서버의 호스트 이름 또는 IP 주소
    user='ubuntu',     # MySQL 사용자 이름
    password='1234',   # MySQL 사용자 비밀번호
    database='testdb'  # 사용할 데이터베이스 이름
)

try:
    with connection.cursor() as cursor:
        # 테이블 생성 (존재하지 않는 경우)
        create_table_query = """
        CREATE TABLE IF NOT EXISTS users (
            id INT AUTO_INCREMENT PRIMARY KEY,
            name VARCHAR(100),
            age INT
        )
        """
        cursor.execute(create_table_query)

        # 데이터 삽입
        insert_query = "INSERT INTO users (name, age) VALUES (%s, %s)"
        data_to_insert = [
            ('Alice', 30),
            ('Bob', 24),
            ('Charlie', 29)
        ]
        cursor.executemany(insert_query, data_to_insert)

    # 변경사항 커밋
    connection.commit()

    # 데이터 확인
    with connection.cursor() as cursor:
        cursor.execute("SELECT * FROM users")
        result = cursor.fetchall()
        for row in result:
            print(row)

finally:
    connection.close()
```

이 스크립트는 다음 단계를 수행해:

1. MySQL 데이터베이스에 연결.
2. `users` 테이블을 생성 (테이블이 이미 존재하면 아무 작업도 하지 않음).
3. `users` 테이블에 데이터를 삽입.
4. 삽입된 데이터를 조회하고 출력.
5. MySQL 연결을 닫음.

스크립트를 실행하기 전에 MySQL 서버에 연결할 수 있는지 확인하고, `testdb` 데이터베이스가 존재하는지 확인해야 해. 데이터베이스가 없다면, MySQL에서 데이터베이스를 생성해:

```sql
CREATE DATABASE testdb;
```

이제 이 스크립트를 실행하면, 데이터가 `users` 테이블에 삽입되고 조회 결과가 출력될 거야. 이 예제를 기반으로 데이터 파이프라인을 만들어 나갈 수 있을 거야. 더 궁금한 점이 있으면 알려줘!