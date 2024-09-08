---
tags:
  - dockerHub
  - docker
create: 2024-06-28 15:06:58
---
hahoop, zookeeper maridb, hive

## pull

```sh
docker pull namugach/hadoop-study:latest
```

---

## init
- create volum dir
```sh
mkdir ~/work
```


- create run docker container
```sh
docker container run -it -d -p 9870:9870 -p 8088:8088 -p 8042:8042 -v /home/$USER/work:/home/ubuntu/work --name my-hadoop-study namugach/hadoop-study

```

---
## connect
```sh
docker container exec -it my-hadoop-study bash
```

---
## hadoop
- start
  - ssh에 관한 게 나오면 yes를 입력
  - hadoopStart.sh 이것은 컨테이너를 만들고 나서 처음만 하면 됨.
```sh
~/hadoopStart.sh
```


- check
```sh
jps
```


- output
```sh
400 NameNode
3875 NodeManager
3220 SecondaryNameNode
3540 ResourceManager
8454 Jps
542 DataNode
```
- web
  - HDFS : http://localhost:9870
  - YARN : http://localhost:8088
  - NodeManager : http://localhost:8042

---

## Hive

```ad-attention
컨테이너를 바로 부팅하고 시작 하면 안되고, 좀 기다렸다 해야함.
```

- connect

```sh
hive
```

- show database
```sh
show databases;
```

- output
```sh
OK
default
test
Time taken: 0.475 seconds, Fetched: 2 row(s)
```

---
## zookeeper
- start
```sh
~/zkServer.sh start
```
- stop
```sh
~/zkServer.sh stop
```
