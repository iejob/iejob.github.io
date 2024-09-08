---
tags:
  - hive
  - docker
  - hadoop
create: 2024-06-26 20:51:49
---

## mysql 연결 안될 때

```sh
wget https://repo1.maven.org/maven2/mysql/mysql-connector-java/8.0.28/mysql-connector-java-8.0.28.jar
cp mysql-connector-java-8.0.28.jar $HIVE_HOME/lib/
vi $HIVE_HOME/conf/hive-env.sh
```


```vim
export HIVE_AUX_JARS_PATH=$HIVE_AUX_JARS_PATH:/path/to/mysql-connector-java-8.0.28.jar
```


```sh
$HIVE_HOME/bin/schematool -dbType mysql -initSchema
history 
```