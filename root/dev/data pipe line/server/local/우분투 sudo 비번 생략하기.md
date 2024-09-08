---
tags:
  - ubuntu
  - setting
  - linux
create: 2024-07-28 22:59:10
---

```ad-attention
root 계정으로 작업 해야함
```

```sh
echo '유저_이름 ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
```

```sh
echo 'ubuntu ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
```

