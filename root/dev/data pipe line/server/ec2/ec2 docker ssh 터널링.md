---
tags:
  - ec2
  - docker
  - ssh
create: 2024-08-04 03:21:57
---


```ad-attention
보안에서 2222 포트 열어주면 됨
```

```sh
# docker -> ec2
ssh server1 -p 2222 -i ./ssh/id_rsa

# ec2 -> docker
ssh server2 localhost -p 2222 -i ./ssh/id_rsa
```


```
172.31.14.186 server1
172.31.10.99 server2
172.31.1.229 server3

docker run -d -p 22:22 --name my_ssh_container ubuntu:24.04


docker run -itd \
  --name docker_server \
  -p 22:22 \
  namugach/ubuntu-pipeline:24.04-kafka-test

docker run -itd \
  --name server2 \
  --hostname server2 \
  --add-host server1:172.31.14.186 \
  --add-host server2:172.31.10.99 \
  --add-host server3:172.31.1.229 \
  -p 2222:2222 \
  namugach/ubuntu-pipeline:24.04-kafka-test

docker run -itd \
  --name server3 \
  --hostname server3 \
  --add-host server1:172.31.14.186 \
  --add-host server2:172.31.10.99 \
  --add-host server3:172.31.1.229 \
  namugach/ubuntu-pipeline:24.04-kafka-test
```