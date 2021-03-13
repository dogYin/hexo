---
title: mysql主从复制方案
date: 2021-02-05 17:44:11
tags:
    - mysql
categories:
    - mysql
top_img: https://cdn.jsdelivr.net/gh/dogYin/imgOSS/img/mysqllogo.jpg
cover: https://cdn.jsdelivr.net/gh/dogYin/imgOSS/img/mysqllogo.jpg
---
# 基于binlog的复制

## 源配置

1. 配置唯一的server_id
2. 开启binlog
3. 创建副本用于复制的用户（拥有Replication slave权限）
```mysql
mysql> CREATE USER 'repl'@'%.example.com' IDENTIFIED BY 'password';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%.example.com';
```
4. 获取binlog坐标
```mysql
#客户端A
mysql> FLUSH TABLES WITH READ LOCK;
#客户端B
mysql> SHOW MASTER STATUS
```
## 副本配置

1. 配置唯一的server_id（该值为0时会阻止副本连接到源）

## 执行复制
1. 启动副本数据库
2. 执行以下命令
```mysql
mysql> CHANGE MASTER TO
    ->     MASTER_HOST='source_host_name',
    ->     MASTER_USER='replication_user_name',
    ->     MASTER_PASSWORD='replication_password',
    ->     MASTER_LOG_FILE='recorded_log_file_name',
    ->     MASTER_LOG_POS=recorded_log_position;

#8.0.23:
mysql> CHANGE REPLICATION SOURCE TO
    ->     SOURCE_HOST='source_host_name',
    ->     SOURCE_USER='replication_user_name',
    ->     SOURCE_PASSWORD='replication_password',
    ->     SOURCE_LOG_FILE='recorded_log_file_name',
    ->     SOURCE_LOG_POS=recorded_log_position;
```

## 注意
* 源数据库有数据时，可以选择使用mysqldump创建转储，然后再副本数据库恢复
```mysql
# 如果不使用--master-data参数，则必须将数据库锁在某个会话中
shell> mysqldump --all-databases --master-data > dbdump.db
```
* 或者通过binlog直接在副本数据库恢复数据


# 基于GTID的复制
