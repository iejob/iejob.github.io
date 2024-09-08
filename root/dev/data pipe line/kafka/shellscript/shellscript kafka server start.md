---
tags:
  - kafka
  - code
  - shellscript
create: 2024-08-04 00:41:12
---
## 생성

```sh
mkdir ~/run/kafka
vi ~/run/kafka/start_server.sh
```

---
## 붙여 넣기

```sh
#!/bin/bash

cd /home/ubuntu/app/kafka/kafka_2.13-3.6.2/

nohup ./bin/kafka-server-start.sh config/server.properties >/dev/null 2>&1
```