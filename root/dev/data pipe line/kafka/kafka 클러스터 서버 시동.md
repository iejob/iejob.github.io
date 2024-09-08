---
tags:
  - kafka
  - server
create: 2024-07-31 01:51:53
---

## ssh 연결

```ad-attention
title:확인
- server1 
- server2 
- server3
```

```sh
~/run/ssh_check.sh
```

---
## zookeeper

```ad-attention
title:시동
- server1 
- server2 
- server3
```

```sh
~/run/kafka_zookeeper_start.sh
```

---
## kafka

```ad-attention
title:시동
- server1 
- server2 
- server3
```

```sh
~/run/kafka_server_start.sh
```

---

## 토픽 생성

```sh
cd ~/app/kafka/kafka_2.13-3.6.2
./bin/kafka-topics.sh --create --bootstrap-server 172.17.0.2:9092 --replication-factor 3 --partitions 3 --topic testtopic
```


## 토픽 삭제

```sh
cd ~/app/kafka/kafka_2.13-3.6.2
./bin/kafka-topics.sh --delete --topic testtopic --bootstrap-server 172.17.0.2:9092
```

---

## 워커 확인
```sh
cd ~/app/kafka/kafka_2.13-3.6.2
./bin/kafka-topics.sh --bootstrap-server 172.17.0.2:9092 --describe --topic testtopic
```

---

## 데이터 흘려보기

```sh
cd ~/app/kafka/kafka_2.13-3.6.2
./bin/kafka-console-producer.sh --bootstrap-server 172.17.0.2:9092 --topic testtopic
```

---
## 모니터링
### server1
```sh
cd ~/app/kafka/kafka_2.13-3.6.2
./bin/kafka-console-consumer.sh --bootstrap-server 172.17.0.2:9092 --topic testtopic
```
### server2
```sh
cd ~/app/kafka/kafka_2.13-3.6.2
./bin/kafka-console-consumer.sh --bootstrap-server 172.17.0.3:9092 --topic testtopic
```
### server3
```sh
cd ~/app/kafka/kafka_2.13-3.6.2
./bin/kafka-console-consumer.sh --bootstrap-server 172.17.0.4:9092 --topic testtopic
```


---


```sh
cd ~/app/kafka/kafka_2.13-3.6.2
./bin/kafka-topics.sh --bootstrap-server 172.18.0.10:9092 --describe --topic testtopic

cd ~/app/kafka/kafka_2.13-3.6.2
./bin/kafka-console-producer.sh --bootstrap-server 172.18.0.10:9092 --topic testtopic


cd ~/app/kafka/kafka_2.13-3.6.2
./bin/kafka-console-consumer.sh --bootstrap-server 172.18.0.10:9092 --topic testtopic

cd ~/app/kafka/kafka_2.13-3.6.2
./bin/kafka-console-consumer.sh --bootstrap-server 172.18.0.11:9092 --topic testtopic

cd ~/app/kafka/kafka_2.13-3.6.2
./bin/kafka-console-consumer.sh --bootstrap-server 172.18.0.12:9092 --topic testtopic
```


```sh
ssh server1
cd ~/app/kafka/kafka_2.13-3.6.2
./bin/kafka-console-consumer.sh --bootstrap-server 172.20.0.10:9092 --topic my_data

ssh server2
cd ~/app/kafka/kafka_2.13-3.6.2
./bin/kafka-console-consumer.sh --bootstrap-server 172.20.0.11:9092 --topic my_data

ssh server3
cd ~/app/kafka/kafka_2.13-3.6.2
./bin/kafka-console-consumer.sh --bootstrap-server 172.20.0.12:9092 --topic my_data
```