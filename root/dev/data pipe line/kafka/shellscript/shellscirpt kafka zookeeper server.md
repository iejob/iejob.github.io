---
tags:
  - kafka
  - code
  - shellscript
create: 2024-08-04 00:39:50
---

## 생성

```sh
mkdir ~/run/kafka
vi ~/run/kafka/start_zookeeper.sh
```

---
## 붙여 넣기

```sh
#!/bin/bash

cd /home/ubuntu/app/kafka/kafka_2.13-3.6.2/

nohup ./bin/zookeeper-server-start.sh config/zookeeper.properties >/dev/null 2>&1
```