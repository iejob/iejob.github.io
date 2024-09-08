---
tags:
  - kafka
  - prometheus
  - grafana
create: 2024-08-03 11:27:11
---

## 바로 실행

```sh

# Prometheus 실행
cd ~/app/prometheus-2.32.1.linux-amd64
./prometheus --config.file=prometheus.yml

# Kafka Exporter 실행
cd ~/app/kafka_exporter-1.4.2.linux-amd64
./kafka_exporter --kafka.server=172.17.0.2:9092

# grafana 실행
sudo grafana-server --homepath=/usr/share/grafana &

```

---
## 방법 1

웹 브라우저에서 Kafka 클러스터를 모니터링하려면 Prometheus와 Grafana를 설치하고 설정한 다음, 웹 브라우저를 통해 Grafana 대시보드를 확인하면 돼. 여기서 그 과정을 단계별로 설명할게.

### 1. Prometheus 설정

**1.1. Prometheus 설치**

```sh
# Prometheus 다운로드
wget https://github.com/prometheus/prometheus/releases/download/v2.32.1/prometheus-2.32.1.linux-amd64.tar.gz

# 압축 해제
tar -xvf prometheus-2.32.1.linux-amd64.tar.gz
cd ~/app/prometheus-2.32.1.linux-amd64

# Prometheus 실행
./prometheus --config.file=prometheus.yml
```

**1.2. Prometheus 설정 파일(prometheus.yml)**

`prometheus.yml` 파일을 다음과 같이 설정해:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'kafka'
    static_configs:
      - targets: ['localhost:9308'] # Kafka Exporter가 실행되는 호스트와 포트
```

### 2. Kafka Exporter 설정

**2.1. Kafka Exporter 설치**

```sh
# Kafka Exporter 다운로드
wget https://github.com/danielqsj/kafka_exporter/releases/download/v1.4.2/kafka_exporter-1.4.2.linux-amd64.tar.gz

# 압축 해제
tar -xvf kafka_exporter-1.4.2.linux-amd64.tar.gz
cd ~/app/kafka_exporter-1.4.2.linux-amd64

# Kafka Exporter 실행
./kafka_exporter --kafka.server=172.17.0.2:9092
```

### 3. Grafana 설정

**3.1. Grafana 설치**

```sh
# Grafana 다운로드 및 설치 (Ubuntu 예시)
sudo apt-get install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
sudo apt-get update
sudo apt-get install -y grafana

# Grafana 실행
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

### 4. Grafana 대시보드 설정

1. 웹 브라우저에서 `http://localhost:3000`으로 이동해 Grafana에 로그인해. (기본 로그인: admin/admin)
2. 로그인을 완료한 후, 왼쪽 사이드바에서 "Add data source"를 클릭해.
3. Prometheus를 선택해.
4. Prometheus 설정 페이지에서 URL을 `http://localhost:9090`으로 설정하고 "Save & Test"를 클릭해 연결을 확인해.

### 5. Kafka Grafana 대시보드 설정

1. Grafana 홈 화면에서 "Create" -> "Import"를 클릭해.
2. Import 페이지에서, Grafana Labs의 Kafka Exporter 대시보드 ID인 `7589`를 입력하고 "Load"를 클릭해.
3. 데이터를 입력하라는 창이 나타나면, "Prometheus" 데이터 소스를 선택해.
4. "Import"를 클릭해 대시보드를 가져와.

### 결과 확인

이제 웹 브라우저에서 `http://localhost:3000`으로 Grafana 웹 UI에 접속하면, Kafka 클러스터에 대한 다양한 모니터링 지표를 확인할 수 있는 대시보드를 볼 수 있을 거야.

이 과정에서 문제가 발생하면, 로그 파일을 확인하거나 설정 파일을 다시 검토해서 올바르게 설정되었는지 확인해봐. Prometheus와 Kafka Exporter, 그리고 Grafana가 제대로 설치되고 실행되어야만 모든 모니터링이 정상적으로 이루어질 거야.

---



## 방법 2

네, 맞아. 컨테이너 내부에서 실행되는 서비스를 호스트 시스템에서 접근하려면 포트 포워딩이 필요해. 이걸 통해 컨테이너 내부의 포트를 호스트 시스템의 포트에 매핑할 수 있어. 그래서 호스트 시스템에서 `localhost:3000`으로 접근할 수 있게 돼.

포트 포워딩을 설정하지 않으면 컨테이너 내부의 서비스에 호스트 시스템에서 직접 접근할 수 없어. 예를 들어, Docker 컨테이너에서 Grafana를 실행하고 호스트 시스템에서 접근하려면 컨테이너를 시작할 때 `-p` 옵션을 사용해 포트를 매핑해야 해.

다시 정리해서 설명할게.

## Docker 컨테이너 설정

컨테이너를 시작할 때 포트 포워딩을 설정해서 다시 실행해 보자:

### 1. Docker Ubuntu 컨테이너 실행

먼저, Ubuntu 컨테이너를 실행할 때 포트를 매핑해:

```sh
docker run -d -it --name monitoring-ubuntu -p 3000:3000 -p 9090:9090 -p 9308:9308 ubuntu
```

이제 컨테이너에 접속해:

```sh
docker exec -it monitoring-ubuntu bash
```

### 2. 필수 패키지 설치

컨테이너 내부에서 필요한 패키지를 설치해:

```sh
apt-get update
apt-get install -y wget gnupg2 curl
```

### 3. Prometheus 설치 및 설정

```sh
# Prometheus 다운로드 및 압축 해제
wget https://github.com/prometheus/prometheus/releases/download/v2.32.1/prometheus-2.32.1.linux-amd64.tar.gz
tar -xvf prometheus-2.32.1.linux-amd64.tar.gz
mv prometheus-2.32.1.linux-amd64 /opt/prometheus

# Prometheus 설정 파일 작성
mkdir /opt/prometheus/config

cat <<EOF > /opt/prometheus/config/prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'kafka'
    static_configs:
      - targets: ['localhost:9308']
EOF

# Prometheus 실행
/opt/prometheus/prometheus --config.file=/opt/prometheus/config/prometheus.yml &
```

### 4. Kafka Exporter 설치 및 설정

```sh
# Kafka Exporter 다운로드 및 압축 해제
wget https://github.com/danielqsj/kafka_exporter/releases/download/v1.4.2/kafka_exporter-1.4.2.linux-amd64.tar.gz
tar -xvf kafka_exporter-1.4.2.linux-amd64.tar.gz
mv kafka_exporter-1.4.2.linux-amd64 /opt/kafka_exporter

# Kafka Exporter 실행
/opt/kafka_exporter/kafka_exporter --kafka.server=localhost:9092 &
```

### 5. Grafana 설치 및 설정

`````ad-attention
title:설치할 때!

```sh
su root
```
해서 root로 접속해서 설치해야함!
`````

**Grafana의 GPG 키를 추가하고 APT 소스 리스트에 Grafana 저장소를 추가해:**

```sh
# Grafana의 GPG 키 추가
wget -q -O - https://packages.grafana.com/gpg.key | apt-key add -

# Grafana 저장소 추가
add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"

# 패키지 리스트 업데이트 및 Grafana 설치
apt-get update
apt-get install -y grafana
```

**Grafana를 설치하고 서비스로 실행해:**

```sh
# Grafana 실행
grafana-server --homepath=/usr/share/grafana &
```

### 6. Grafana 대시보드 설정

컨테이너 내부에서 Grafana 서버가 실행되면, 호스트 시스템의 웹 브라우저에서 `http://localhost:3000`으로 접속할 수 있어. 이후 절차는 다음과 같아:

1. 웹 브라우저에서 `http://localhost:3000`으로 이동해 Grafana에 로그인해. (기본 로그인: admin/admin)
2. 로그인을 완료한 후, 왼쪽 사이드바에서 "Add data source"를 클릭해.
3. Prometheus를 선택해.
4. Prometheus 설정 페이지에서 URL을 `http://localhost:9090`으로 설정하고 "Save & Test"를 클릭해 연결을 확인해.
5. Grafana 홈 화면에서 "Create" -> "Import"를 클릭해.
6. Import 페이지에서, Grafana Labs의 Kafka Exporter 대시보드 ID인 `7589`를 입력하고 "Load"를 클릭해.
7. 데이터를 입력하라는 창이 나타나면, "Prometheus" 데이터 소스를 선택해.
8. "Import"를 클릭해 대시보드를 가져와.

이제 호스트 시스템의 웹 브라우저에서 `http://localhost:3000`으로 Grafana 웹 UI에 접속하면, Kafka 클러스터에 대한 다양한 모니터링 지표를 확인할 수 있어.