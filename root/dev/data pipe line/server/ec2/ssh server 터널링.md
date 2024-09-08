---
tags:
  - ssh
  - setting
create: 2024-08-04 23:01:36
---

## 확인
```sh
nc -zv localhost 2181
nc -zv localhost 9092
```

---
## 설정
```sh
# ZooKeeper 포트 포워딩
ssh -L 2181:localhost:2181 -N -f serverNumber

# Kafka 포트 포워딩
ssh -L 9092:localhost:9092 -N -f serverNumber
```

### server1
```sh
ssh -L 2181:localhost:2181 -L 9092:localhost:9092 -N -f ubuntu@172.31.14.186

# ZooKeeper 포트 포워딩
ssh -L 2181:localhost:2181 -N -f server1

# Kafka 포트 포워딩
ssh -L 9092:localhost:9092 -N -f server1
```

### server2
```sh

ssh -L 2181:localhost:2181 -L 9092:localhost:9092 -N -f ubuntu@172.31.10.99 


# ZooKeeper 포트 포워딩
ssh -L 2181:localhost:2181 -N -f server2

# Kafka 포트 포워딩
ssh -L 9092:localhost:9092 -N -f server2
```

### server3
```sh
ssh -L 2181:localhost:2181 -L 9092:localhost:9092 -N -f ubuntu@172.31.1.229


# ZooKeeper 포트 포워딩
ssh -L 2181:localhost:2181 -N -f server3

# Kafka 포트 포워딩
ssh -L 9092:localhost:9092 -N -f server3
```