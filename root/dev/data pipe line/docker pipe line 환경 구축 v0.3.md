---
tags:
  - docker
  - pipeline
  - setup
create: 2024-08-12 01:22:41
---

```ad-check
title: 더 자세한 메뉴얼은
[여기](https://github.com/namugach/pipeline_base)에서 확인!
```
## 설치

### 저장소 복제
```sh
git clone https://github.com/namugach/pipeline_base
```

### 이동
```sh
cd pipeline_base
```

### 실행
```sh
./install
```


---

## docker
### 컨테이너 진입
```ad-note
title: 창을
2개 띄워서 나중에 모니터링 할 것도 미리서 준비 해놓자
```

```sh
docker container exec -it server1 bash -c "cd /root && bash"
```


### 카프카 클러스터 서버 시작
```sh
./run/kafka/start_cluster.sh
```

### 모니터링
```sh
./test/monitor.sh
```

### 데이터 삽입
```sh
./test/main.sh
```



### 확인
#### 접속
```sh
mysql -u ubuntu -p1234
```

#### 조회
```sh

USE my_data;
SELECT * FROM test_a;
```

