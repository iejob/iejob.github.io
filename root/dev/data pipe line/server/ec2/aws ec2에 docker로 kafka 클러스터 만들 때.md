---
tags:
  - aws
  - ec2
  - docker
  - kafka
  - ubuntu
create: 2024-08-05 04:23:18
---

```python
import sys, time
from pathlib import Path

base_dir = Path('/home/ubuntu/run')
sys.path.insert(0, str(base_dir))
from config.config import server_infos
from util.util import send_ssh_comm

ssh_key_dir = f"{base_dir}/config/keygen"
id_rsa_path = Path(ssh_key_dir) / "id_rsa"
id_rsa_pub_path = Path(ssh_key_dir) / "id_rsa.pub"

# ~/.ssh/config

ssh_config = '''
Host server1
	HostName 172.31.14.186
	Port 2222
	User ubuntu

Host server2
	HostName 172.31.10.99
	Port 2222
	User ubuntu

Host server3
	HostName 172.31.1.229
	Port 2222
	User ubuntu
'''

def get_create_container_comm(id, name, port=2222, server_infos=server_infos):
	with open(id_rsa_path, 'r') as id_rsa_file:
		id_rsa = id_rsa_file.read()
	
	with open(id_rsa_pub_path, 'r') as id_rsa_pub_file:
		id_rsa_pub = id_rsa_pub_file.read()
	print(id)
	return f'''
	docker network create --subnet=172.18.0.0/16 pipeline
	docker container run -itd \\
		--name {name} \\
		--hostname {name} \\
		--add-host {server_infos[0].name}:{server_infos[0].ip} \\
		--add-host {server_infos[1].name}:{server_infos[1].ip} \\
		--add-host {server_infos[2].name}:{server_infos[2].ip} \\
		-p {port}:{port} \\
		-p 2181:2181 \\
		-p 2888:2888 \\
		-p 3888:3888 \\
		-p 9092:9092 \\
		--net pipeline \\
		--ip 172.18.0.{id+10} \\
		namugach/ubuntu-pipeline:24.04-kafka-test \\
		/bin/bash -c "echo '{id_rsa}' > /home/ubuntu/.ssh/id_rsa && \\
				echo '{id_rsa_pub}' > /home/ubuntu/.ssh/id_rsa.pub && \\
				echo '{id_rsa_pub}' > /home/ubuntu/.ssh/authorized_keys && \\
				echo '{ssh_config}' >> /home/ubuntu/.ssh/config && \\
				chmod 600 /home/ubuntu/.ssh/id_rsa && \\
				chmod 644 /home/ubuntu/.ssh/id_rsa.pub && \\
				sudo service ssh start && \\
				sudo service mysql start && \\
				tail -f /dev/null"
	'''

for info in server_infos:
	send_ssh_comm(info.name, info.name, get_create_container_comm(info.id, info.name))
	time.sleep(1)
```


## 이거!!!!

```python
docker network create --subnet=172.18.0.0/16 pipeline
--ip 172.18.0.{id+10} \\
```

이거 알아내려고.. 그랬다..
