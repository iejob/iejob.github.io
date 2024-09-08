---
tags:
  - elasticsearch
  - install
create: 2024-07-19 14:27:23
---

https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html

## 설치
```ad-important
서버 3개에 설치
```

### 키 등록
```sh
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
```

### apt-transport-https 설치
```sh
sudo apt-get install apt-transport-https
```

```sh
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
```

```sh
sudo apt-get update
```

```sh
sudo apt-get -y install elasticsearch
```


### 확인
```sh
sudo systemctl status elasticsearch
```

```sh
elasticsearch.service - Elasticsearch
     Loaded: loaded (/usr/lib/systemd/system/elasticsearch.service; disabled; preset: enabled)
     Active: inactive (dead)
       Docs: https://www.elastic.co
```


---

## 설정

```ad-attention
- 3대의 서버 모두 일괄 적용
- node.name은 각자 서버에 맞게
```
### note list 만들기

```sh
["ip-172-31-6-227","ip-172-31-1-189","ip-172-31-14-208"]
```
1, 2, 3 node 넣는다.


### root 접속

```sh
sudo -i
```

### 이동

```sh
cd /etc/elasticsearch/
```

### 열기
```sh
vi elasticsearch.yml
```

### 편집
```yml title:elasticsearch.yml
# ---------------------------------- Cluster -----------------------------------
cluster.name: my-elk-stack

# ------------------------------------ Node ------------------------------------
node.name: ip-172-31-6-227 # 각자 노드 이름

# ---------------------------------- Network -----------------------------------
network.host: 0.0.0.0

http.port: 9200

# --------------------------------- Discovery ----------------------------------
discovery.seed_hosts: ["ip-172-31-6-227","ip-172-31-1-189","ip-172-31-14-208"]

----------------------- BEGIN SECURITY AUTO CONFIGURATION -----------------------
xpack.security.enabled: false
xpack.security.enrollment.enabled: false
xpack.security.http.ssl:
  enabled: false
xpack.security.transport.ssl:
  enabled: false

cluster.initial_master_nodes: ["ip-172-31-6-227","ip-172-31-1-189","ip-172-31-14-208"]
```



## 설치 확인
```ad-attention
3대 모두에서 확인 및 실행
```

### ssh 접속
```sh
ssh ip-172-31-6-227
ssh ip-172-31-1-189
ssh ip-172-31-14-208
```


### 실행
```sh
sudo /bin/systemctl daemon-reload

sudo /bin/systemctl enable elasticsearch.service

sudo systemctl start elasticsearch.service
```


### 확인
```sh
sudo systemctl status elasticsearch
```

```sh
curl localhost:9200
```

```shell title:out
{
  "name" : "ip-172-31-6-227",
  "cluster_name" : "my-elk-stack",
  "cluster_uuid" : "fw7VOLA4S96P24fZmtpRHQ",
  "version" : {
    "number" : "8.14.3",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "d55f984299e0e88dee72ebd8255f7ff130859ad0",
    "build_date" : "2024-07-07T22:04:49.882652950Z",
    "build_snapshot" : false,
    "lucene_version" : "9.10.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
```


### master

```sh
ubuntu@ip-172-31-6-227:~$ curl localhost:9200/_cat/health?v
epoch      timestamp cluster      status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1721373567 07:19:27  my-elk-stack green           3         3      0   0    0    0        0             0                  -                100.0%
ubuntu@ip-172-31-6-227:~$ curl localhost:9200/_cat/nodes?v
ip            heap.percent ram.percent cpu load_1m load_5m load_15m node.role   master name
172.31.1.189             1          73   1    0.00    0.01     0.01 cdfhilmrstw *      ip-172-31-1-189
172.31.14.208            4          73   0    0.00    0.01     0.00 cdfhilmrstw -      ip-172-31-14-208
172.31.6.227             4          73   0    0.00    0.00     0.00 cdfhilmrstw -      ip-172-31-6-227
```

별 붙어있음