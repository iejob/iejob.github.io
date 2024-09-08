---
tags:
  - code
  - kafka
  - shellscript
create: 2024-08-04 00:42:25
---

## 생성

```sh
mkdir ~/run/kafka
vi ~/run/kafka/check_conn.sh
```

---
## 붙여 넣기

```sh
#!/bin/bash
netstat -ntlp | grep 2181

echo ""
echo "======================="
echo ""

netstat -ntlp | grep 9092
```