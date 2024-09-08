---
tags:
  - hadoop
  - wsl2
  - port
  - issue
create: 2024-07-01 23:55:18
---

```ad-check
- port: [ 8042, 8088, 9870 ]
`sudo lsof -i :port` 했을 때
아래와 같은 결과가 나와야 한다.
```

## netstat -utln
```sh
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:9870            0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.54:53           0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:8042            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:8088            0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:32921         0.0.0.0:*               LISTEN     
tcp6       0      0 :::9870                 :::*                    LISTEN     
tcp6       0      0 :::8042                 :::*                    LISTEN     
tcp6       0      0 :::8088                 :::*                    LISTEN     
udp        0      0 127.0.0.54:53           0.0.0.0:*                          
udp        0      0 127.0.0.53:53           0.0.0.0:*                          
udp        0      0 127.0.0.1:323           0.0.0.0:*                          
udp6       0      0 ::1:323                 :::*                               
```

---
## sudo lsof -i 
### 8088
```sh
>  sudo lsof -i :8088

COMMAND     PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
docker-pr 25876 root    4u  IPv4 396225      0t0  TCP *:omniorb (LISTEN)
docker-pr 25885 root    3u  IPv6 443958      0t0  TCP ip6-localhost:omniorb->ip6-localhost:37532 (ESTABLISHED)
docker-pr 25885 root    4u  IPv6 401527      0t0  TCP *:omniorb (LISTEN)
docker-pr 25885 root    5u  IPv4 443960      0t0  TCP 172.17.0.1:39330->172.17.0.2:omniorb (ESTABLISHED)
```

### 8042
```sh
>  sudo lsof -i :8042

COMMAND     PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
docker-pr 25857 root    4u  IPv4 398819      0t0  TCP *:8042 (LISTEN)
docker-pr 25869 root    3u  IPv6 437943      0t0  TCP ip6-localhost:8042->ip6-localhost:38704 (ESTABLISHED)
docker-pr 25869 root    4u  IPv6 396220      0t0  TCP *:8042 (LISTEN)
docker-pr 25869 root    5u  IPv4 437945      0t0  TCP 172.17.0.1:55528->172.17.0.2:8042 (ESTABLISHED)
```

### 9870
```sh
>  sudo lsof -i :9870

COMMAND     PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
docker-pr 25892 root    4u  IPv4 400877      0t0  TCP *:9870 (LISTEN)
docker-pr 25899 root    4u  IPv6 391975      0t0  TCP *:9870 (LISTEN)
```