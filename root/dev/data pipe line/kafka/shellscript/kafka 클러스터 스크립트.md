---
tags:
  - kafka
  - code
  - shellscript
create: 2024-08-03 22:10:23
---


## 생성


```sh
mkdir ~/run/kafka
vi ~/run/kafka/start_cluster.sh
```

---

## 붙여 넣기

### 리팩토링

```sh
#!/bin/bash

ips=("server1" "server2" "server3")

start_service() {
	local service_name=$1
	local script_path=$2
	local sleep_time=${3:-1} # 기본값을 1로 설정

	for ip in "${ips[@]}"; do
		echo ""
		echo "============== $service_name 시작 시도 중: $ip =============="
		echo ""
		ssh -o StrictHostKeyChecking=no -o ConnectTimeout=5 $ip 'bash -s' < "$script_path"
		sleep $sleep_time
		if [ $? -eq 0 ]; then
			echo "$ip 에서 $service_name 가 성공적으로 시작되었어"
		else
			echo "$ip 에서 $service_name 시작 실패"
			exit 1
		fi
	done
}

# Zookeeper 시작 및 상태 확인
start_service "Zookeeper" "./run/kafka/start_zookeeper.sh"

# 대기
sleep 3

# Kafka 서버 시작
start_service "Kafka 서버" "./run/kafka/start_server.sh" 2

# Kafka check
start_service "Kafka check" "./run/kafka/check_conn.sh" 2
```


### 원본
```sh
#!/bin/bash

ips=("server1" "server2" "server3")

# Zookeeper 시작 및 상태 확인
for ip in "${ips[@]}"; do
	echo "Zookeeper 시작 시도 중: $ip"
	ssh -o StrictHostKeyChecking=no -o ConnectTimeout=5 $ip 'bash -s' < ./run/kafka_zookeeper_start.sh
	sleep 1
	if [ $? -eq 0 ]; then
			echo "$ip 에서 Zookeeper가 성공적으로 시작되었어"
	else
			echo "$ip 에서 Zookeeper 시작 실패"
			exit 1
	fi
done

# 대기
sleep 3

# Kafka 서버 시작
for ip in "${ips[@]}"; do
	echo "Kafka 서버 시작 시도 중: $ip"
	ssh -o StrictHostKeyChecking=no -o ConnectTimeout=5 $ip 'bash -s' < ./run/kafka_server_start.sh
	sleep 1
	if [ $? -eq 0 ]; then
			echo "$ip 에서 Kafka 서버가 성공적으로 시작되었어"
	else
			echo "$ip 에서 Kafka 서버 시작 실패"
			exit 1
	fi
done
```

---
## Kafka 클러스터 종료 스크립트

```sh
mkdir ./run/kafka
vi ./run/kafka/stop_cluster.sh
```

```sh
#!/bin/bash

ips=("server1" "server2" "server3")

# Kafka 서버 종료
for ip in "${ips[@]}"; do
	echo "Kafka 서버 종료 시도 중: $ip"
	ssh -o StrictHostKeyChecking=no -o ConnectTimeout=5 $ip 'bash -s' <<-'EOF'
			cd /home/ubuntu/app/kafka/kafka_2.13-3.6.2/
			./bin/kafka-server-stop.sh
EOF
	if [ $? -eq 0 ]; then
			echo "$ip 에서 Kafka 서버가 성공적으로 종료되었어"
	else
			echo "$ip 에서 Kafka 서버 종료 실패"
			exit 1
	fi
done

# Zookeeper 서버 종료
for ip in "${ips[@]}"; do
	echo "Zookeeper 서버 종료 시도 중: $ip"
	ssh -o StrictHostKeyChecking=no -o ConnectTimeout=5 $ip 'bash -s' <<-'EOF'
			cd /home/ubuntu/app/kafka/kafka_2.13-3.6.2/
			./bin/zookeeper-server-stop.sh
EOF
	if [ $? -eq 0 ]; then
			echo "$ip 에서 Zookeeper 서버가 성공적으로 종료되었어"
	else
			echo "$ip 에서 Zookeeper 서버 종료 실패"
			exit 1
	fi
done
```


---

## 사용 방법
1. **종료 스크립트 사용 방법**:
- 스크립트에 실행 권한을 부여해:
```sh
chmod +x stop_kafka_cluster.sh
```
- 스크립트를 실행해:
```sh
./stop_kafka_cluster.sh
```

2. **시작 스크립트 사용 방법**:
- 이미 실행 권한을 부여한 상태라면:
```sh
./start_kafka_cluster.sh
```

## 스크립트 설명
- `stop_kafka_cluster.sh`: 각 서버에 SSH로 접속하여 Kafka 서버를 먼저 종료한 후, Zookeeper 서버를 종료해. 순차적으로 종료되도록 설계했어.
- `start_kafka_cluster.sh`: 각 서버에 SSH로 접속하여 Zookeeper 서버를 먼저 시작하고, 상태를 확인한 후 Kafka 서버를 시작해. Zookeeper가 제대로 시작되었는지 확인하는 과정이 포함되어 있어.

이 스크립트들을 사용하면 안전하게 Kafka 클러스터를 종료하고 시작할 수 있어. 문제가 발생할 경우 스크립트가 중단되기 때문에 안정성을 높일 수 있어.