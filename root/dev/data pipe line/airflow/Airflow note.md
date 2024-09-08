---
tags:
  - airflow
  - note
create: 2024-06-17 15:40:57
---

## 파이썬

### 가상환경
[[docker ubuntu 기본 환경#설치]]
[[docker ubuntu 기본 환경#가상환경]]

```sh
sudo apt update
sudo apt install -y pip
sudo apt-get install python3.12-venv
python3 -m venv airflow_venv 
source airflow_venv/bin/activate
```

### 라이브러리
```sh
pip install "apache-airflow[celery]==2.9.1" --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-2.9.1/constraints-3.12.txt"
```


## 3일차

https://github.com/hjkim-sun/airflow_2.9.1/blob/master/plugins/sensors/seoul_api_date_column_sensor.py  
https://github.com/hjkim-sun/airflow_2.9.1/blob/master/dags/dags_custom_sensor_2.py
