---
tags:
  - flask
  - kafka
create: 2024-07-23 11:37:27
---

```python
import json
from flask import Flask, request, jsonify
from kafka import KafkaProducer, KafkaConsumer
import psycopg2
from threading import Thread

app = Flask(__name__)
broker1 = "172.31.6.227:9092"
broker2 = "172.31.14.208:9092"
broker3 = "172.31.1.189:9092"

kafka_bootstrap_servers = [broker1, broker2, broker3]
kafka_topic = "testtopic"

db_host = "172.31.14.208"
db_port = 5432
db_name = "myuser"
db_user = "postgres"
db_password = "postgres"
db_table = "user_table"

def kafka_producer(data):
	producer = KafkaProducer(bootstrap_servers=kafka_bootstrap_servers)
	producer.send(kafka_topic, value=json.dumps(data).encode('utf-8'))
	producer.flush()
	pass

def kafka_consumer():
	consumer = KafkaConsumer(kafka_topic, bootstrap_servers=kafka_bootstrap_servers)
	for message in consumer:
		data = json.loads(message.value.decode("utf-8"))
		inser_data_into_postgres(data)
	pass

def inser_data_into_postgres(data):
	conn = psycopg2.connect(host=db_host, port=db_port, database=db_name, user=db_user, password=db_password)
	cursor = conn.cursor()

	query = f"INSERT INTO {db_table} (name, level) VALUES (%s, %s)"
	values = (data['name'], data['level'])

	cursor.execute(query, values)
	conn.commit()

	cursor.close()
	conn.close()
	pass


@app.route("/")
def hello():
	return "hello world"

@app.route("/sendinfo", methods=["POST"])
def receive_data():
	data = request.get_json()
	kafka_producer(data)
	return jsonify(message="success")

if __name__ == "__main__":
	kafka_consumer_thread = Thread(target=kafka_consumer)
	kafka_consumer_thread.start()
	app.run(host="0.0.0.0", port=5000)

	pass


```