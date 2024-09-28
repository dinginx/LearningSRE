# 堡垒机Jumpserver

# 1、**内容概述**

```
JumpServer 是全球首款完全开源的堡垒机, 使用 GNU GPL v2.0 开源协议, 是符合 4A 的专业运维审计系统。为互联网企业提供了认证，授权，审计，自动化运维等功能。
JumpServer 使用 Python / Django 进行开发, 遵循 Web 2.0 规范, 配备了业界领先的 Web Terminal 解决方案, 交互界面美观、用户体验好。
JumpServer 采纳分布式架构, 支持多机房跨区域部署, 中心节点提供 API, 各机房部署登录节点, 可横向扩展、无并发访问限制。
JumpServer 现已支持管理 SSH、 Telnet、 RDP、 VNC 协议资产。
```

## **JumpServer**的优势和功能

```
特色优势
开源: 零门槛，线上快速获取和安装
分布式: 轻松支持大规模并发访问；
无插件: 仅需浏览器，极致的 Web Terminal 使用体验
多云支持: 一套系统，同时管理不同云上面的资产
云端存储: 审计录像云端存储，永不丢失
多租户: 一套系统，多个子公司和部门同时使用
```

```
功能列表
堡垒机四个核心能力: 运维安全审计的4A规范：身份鉴别、授权控制、账号管理、安全审计
```

# 2、JumpServer安装

## 2.1 安装要求

```
JumpServer 环境要求:
硬件配置: 2个CPU核心, 4G 内存, 50G 硬盘（最低）
操作系统: Linux 发行版 x86_64
Python = 3.6.x
MySQL Server ≥ 5.6 或者 Mariadb Server ≥ 5.5.56 数据库编码要求 uft8,新版要求5.7以上
Redis: 新版要求6.0以上
```

## **2.2** **安装方法介绍**

```
官方提供了多种安装方法
	手动部署: 按组件逐个实现
	极速部署: 资产数量不多，或者测试体验的用户请使用本脚本快速部署
	容器部署: 基于 docker 实现
	分布式部署: 适用大型环境
```



## **2.3** **基于容器部署**

官方文档

```
https://github.com/jumpserver/Dockerfile/tree/master/allinone
https://docs.jumpserver.org/zh/master/install/docker_install/
```

### **2.3.1** **环境说明**

```
https://github.com/jumpserver/Dockerfile/tree/master/allinone
```

使用外置 MySQL 数据库和 Redis:

```
- 外置数据库要求 MySQL 版本大于等于 5.7
- 外置 Redis 要求 Redis 版本大于等于 6.0
# 自行部署 MySQL 可以参考
(https://docs.jumpserver.org/zh/master/install/setup_by_lb/#mysql)
# mysql 创建用户并赋予权限, 请自行替换 nu4x599Wq7u0Bn8EABh3J91G 为自己的密码
mysql -u root -p
create database jumpserver default charset 'utf8';
create user 'jumpserver'@'%' identified by 'nu4x599Wq7u0Bn8EABh3J91G';
grant all on jumpserver.* to 'jumpserver'@'%';
flush privileges;
# 自行部署 Redis 可以参考
(https://docs.jumpserver.org/zh/master/install/setup_by_lb/#redis)
```

**基于容器**,安装完毕后可以通过以下方式访问

```
浏览器访问: http://<容器所在服务器IP> 
默认管理员账户 admin 密码 admin
SSH 访问: ssh -p 2222 <容器所在服务器IP> 
XShell 等工具请添加 connection 连接, 默认 ssh 端口 2222
```

### **2.3.2** 安装docker 环境（ubuntu安装）

```
[root@ubuntu2004 ~]#apt update && apt list docker.io
Hit:1 http://mirrors.aliyun.com/ubuntu focal InRelease
Hit:2 http://mirrors.aliyun.com/ubuntu focal-updates InRelease
Hit:3 http://mirrors.aliyun.com/ubuntu focal-backports InRelease
Hit:4 http://mirrors.aliyun.com/ubuntu focal-security InRelease
Reading package lists... Done
Building dependency tree       
Reading state information... Done
86 packages can be upgraded. Run 'apt list --upgradable' to see them.
Listing... Done
docker.io/focal-updates,now 24.0.5-0ubuntu1~20.04.1 amd64 [installed]

[root@ubuntu2004 ~]#apt-cache madison docker.io
 docker.io | 24.0.5-0ubuntu1~20.04.1 | http://mirrors.aliyun.com/ubuntu focal-updates/universe amd64 Packages
 docker.io | 20.10.21-0ubuntu1~20.04.2 | http://mirrors.aliyun.com/ubuntu focal-security/universe amd64 Packages
 docker.io | 19.03.8-0ubuntu1 | http://mirrors.aliyun.com/ubuntu focal/universe amd64 Packages

[root@ubuntu2004 ~]#apt -y install docker.io

[root@ubuntu2004 ~]#systemctl enable --now docker
[root@ubuntu2004 ~]#systemctl status docker.service 
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2024-01-08 08:02:40 UTC; 58s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 34678 (dockerd)
      Tasks: 9
     Memory: 27.5M
     CGroup: /system.slice/docker.service
             └─34678 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

Jan 08 08:02:39 ubuntu2004 systemd[1]: Starting Docker Application Container Engine...
Jan 08 08:02:39 ubuntu2004 dockerd[34678]: time="2024-01-08T08:02:39.785169805Z" level=info msg="Starting up"
Jan 08 08:02:39 ubuntu2004 dockerd[34678]: time="2024-01-08T08:02:39.787348117Z" level=info msg="detected 127.0.0.53 nameserver, assuming systemd-resolved, so using re>
Jan 08 08:02:39 ubuntu2004 dockerd[34678]: time="2024-01-08T08:02:39.858901334Z" level=info msg="Loading containers: start."
Jan 08 08:02:40 ubuntu2004 dockerd[34678]: time="2024-01-08T08:02:40.002661124Z" level=info msg="Loading containers: done."
Jan 08 08:02:40 ubuntu2004 dockerd[34678]: time="2024-01-08T08:02:40.019768321Z" level=warning msg="WARNING: No swap limit support"
Jan 08 08:02:40 ubuntu2004 dockerd[34678]: time="2024-01-08T08:02:40.020022616Z" level=info msg="Docker daemon" commit="24.0.5-0ubuntu1~20.04.1" graphdriver=overlay2 v>
Jan 08 08:02:40 ubuntu2004 dockerd[34678]: time="2024-01-08T08:02:40.020227104Z" level=info msg="Daemon has completed initialization"
Jan 08 08:02:40 ubuntu2004 systemd[1]: Started Docker Application Container Engine.
Jan 08 08:02:40 ubuntu2004 dockerd[34678]: time="2024-01-08T08:02:40.040601743Z" level=info msg="API listen on /run/docker.sock"

[root@Jumpserver ~]#docker
docker        dockerd       docker-init   docker-proxy  
[root@Jumpserver ~]#docker version
Client:
 Version:           24.0.5
 API version:       1.43
 Go version:        go1.20.3
 Git commit:        24.0.5-0ubuntu1~20.04.1
 Built:             Mon Aug 21 19:50:14 2023
 OS/Arch:           linux/amd64
 Context:           default

Server:
 Engine:
  Version:          24.0.5
  API version:      1.43 (minimum version 1.12)
  Go version:       go1.20.3
  Git commit:       24.0.5-0ubuntu1~20.04.1
  Built:            Mon Aug 21 19:50:14 2023
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.7.2
  GitCommit:        
 runc:
  Version:          1.1.7-0ubuntu1~20.04.1
  GitCommit:        
 docker-init:
  Version:          0.19.0
  GitCommit:  
  
  
#镜像加速(可选)
[root@ubuntu2004 ~]#echo '{"registry-mirrors": 
["http://si7y70hh.mirror.aliyuncs.com"], "insecure-registries": 
["harbor.wang.org:80"]' > /etc/docker/daemon.json
[root@ubuntu2004 ~]#systemctl restart docker
```

### 2.3.3、docker部署MySQL

官方说明:

```
https://docs.jumpserver.org/zh/master/install/prod/distributed_03/
```

MySQL要求

```
create database jumpserver default charset 'utf8';
create user 'jumpserver'@'%' identified by 'nu4x599Wq7u0Bn8EABh3J91G';
grant all on jumpserver.* to 'jumpserver'@'%';
flush privileges;
```

注意：JumpServer-v2.28.7之前版本默认不支持MySQL8.0，选择MySQL5.7

#### **2.3.3.1** **下载** **MySQL** **镜像查看默认配置**

下载MySQL镜像并运行, 查看当前默认配置不符合JumpServer安装要求

```
docker run --rm --name mysql -e MYSQL_ROOT_PASSWORD=123456 -e MYSQL_DATABASE=jumpserver -e MYSQL_USER=jumpserver -e MYSQL_PASSWORD=123456 -d -p 3306:3306 mysql:5.7.30

#拉去mysql镜像
[root@Jumpserver ~]#docker run --rm --name mysql -e MYSQL_ROOT_PASSWORD=123456 -e MYSQL_DATABASE=jumpserver -e MYSQL_USER=jumpserver -e MYSQL_PASSWORD=123456 -d -p 3306:3306 mysql:5.7.30

#查看默认的MySQL容器配置不符合jumpserver要求
[root@Jumpserver ~]#docker exec -it mysql bash
root@9f35875388be:/# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.30 MySQL Community Server (GPL)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| jumpserver         |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> show create database jumpserver;
+------------+-----------------------------------------------------------------------+
| Database   | Create Database                                                       |
+------------+-----------------------------------------------------------------------+
| jumpserver | CREATE DATABASE `jumpserver` /*!40100 DEFAULT CHARACTER SET latin1 */ |
+------------+-----------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> select user,host from mysql.user;
+---------------+-----------+
| user          | host      |
+---------------+-----------+
| jumpserver    | %         |
| root          | %         |
| mysql.session | localhost |
| mysql.sys     | localhost |
| root          | localhost |
+---------------+-----------+
5 rows in set (0.00 sec)

mysql> exit

#查看配置文件路径
root@9f35875388be:/# cat /etc/mysql/my
......

!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/

#默认配置文件
root@9f35875388be:/# ls -R /etc/mysql
/etc/mysql:
conf.d	my.cnf	my.cnf.fallback  mysql.cnf  mysql.conf.d

/etc/mysql/conf.d:
docker.cnf  mysql.cnf  mysqldump.cnf

/etc/mysql/mysql.conf.d:
mysqld.cnf


#默认配置文件
root@9f35875388be:/#  grep '^[^#]' /etc/mysql/mysql.conf.d/mysqld.cnf
[mysqld]
pid-file	= /var/run/mysqld/mysqld.pid
socket		= /var/run/mysqld/mysqld.sock
datadir		= /var/lib/mysql
symbolic-links=0
#默认配置文件
root@9f35875388be:/# cat /etc/mysql/conf.d/mysql.cnf
[mysql]

root@9f35875388be:/# exit
exit
[root@Jumpserver ~]#

```

#### **2.3.3.2** 在宿主机准备MySQL配置文件

```
#准备相关目录
mkdir -p /etc/mysql/mysql.conf.d/
mkdir -p /etc/mysql/conf.d/

#生成服务器配置文件,指定字符集
tee /etc/mysql/mysql.conf.d/mysqld.cnf <<EOF
[mysqld]
pid-file= /var/run/mysqld/mysqld.pid
socket= /var/run/mysqld/mysqld.sock
datadir= /var/lib/mysql
symbolic-links=0
character-set-server=utf8    #添加此行,指定字符集
EOF

#生成客户端配置文件,指定字符集
tee /etc/mysql/conf.d/mysql.cnf <<EOF
[mysql]
default-character-set=utf8    #添加此行,指定字符集
EOF

#查看配置文件列表
[root@Jumpserver ~]#tree /etc/mysql/
/etc/mysql/
├── conf.d
│   └── mysql.cnf
└── mysql.conf.d
    └── mysqld.cnf

2 directories, 2 files

```

#### **2.3.3.3** 启动MySQL容器

将上面宿主机的设置好的配置文件挂载至MySQL容器



```
docker run --rm --name mysql -e MYSQL_ROOT_PASSWORD=123456 -e MYSQL_DATABASE=jumpserver -e MYSQL_USER=jumpserver -e MYSQL_PASSWORD=123456 -d -p 3306:3306 mysql:5.7.30

docker run -d -p 3306:3306 --name mysql --restart always \
-e MYSQL_ROOT_PASSWORD=123456 \
-e MYSQL_DATABASE=jumpserver \
-e MYSQL_USER=jumpserver \
-e MYSQL_PASSWORD=123456 \
-v /data/mysql:/var/lib/mysql \
-v /etc/mysql/mysql.conf.d/mysqld.cnf:/etc/mysql/mysql.conf.d/mysqld.cnf \
-v /etc/mysql/conf.d/mysql.cnf:/etc/mysql/conf.d/mysql.cnf mysql:5.7.30
```

#### **2.3.3.4** 验证MySQL

```
[root@Jumpserver ~]#docker exec -it mysql sh
# mysql -p123456 -e 'show variables like "character%"'
mysql: [Warning] Using a password on the command line interface can be insecure.
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
#  mysql -p123456 -e 'show variables like "collation%"'
mysql: [Warning] Using a password on the command line interface can be insecure.
+----------------------+-----------------+
| Variable_name        | Value           |
+----------------------+-----------------+
| collation_connection | utf8_general_ci |
| collation_database   | utf8_general_ci |
| collation_server     | utf8_general_ci |
+----------------------+-----------------+
#  cat /var/lib/mysql/jumpserver/db.opt
default-character-set=utf8
default-collation=utf8_general_ci
#  cat /etc/mysql/mysql.conf.d/mysqld.cnf
[mysqld]
pid-file= /var/run/mysqld/mysqld.pid
socket= /var/run/mysqld/mysqld.sock
datadir= /var/lib/mysql
symbolic-links=0
character-set-server=utf8
#  cat /etc/mysql/conf.d/mysql.cnf
[mysql]
default-character-set=utf8
#  mysql -p123456 -e 'select user,host from mysql.user'
mysql: [Warning] Using a password on the command line interface can be insecure.
+---------------+-----------+
| user          | host      |
+---------------+-----------+
| jumpserver    | %         |
| root          | %         |
| mysql.session | localhost |
| mysql.sys     | localhost |
| root          | localhost |
+---------------+-----------+
# mysql -p123456 -e 'select user,host from mysql.user'
mysql: [Warning] Using a password on the command line interface can be insecure.
+---------------+-----------+
| user          | host      |
+---------------+-----------+
| jumpserver    | %         |
| root          | %         |
| mysql.session | localhost |
| mysql.sys     | localhost |
| root          | localhost |
+---------------+-----------+
# ls /var/lib/mysql/ -l
total 188480
-rw-r----- 1 mysql mysql       56 Jan  8 08:41 auto.cnf
-rw------- 1 mysql mysql     1680 Jan  8 08:41 ca-key.pem
-rw-r--r-- 1 mysql mysql     1112 Jan  8 08:41 ca.pem
-rw-r--r-- 1 mysql mysql     1112 Jan  8 08:41 client-cert.pem
-rw------- 1 mysql mysql     1680 Jan  8 08:41 client-key.pem
-rw-r----- 1 mysql mysql     1346 Jan  8 08:41 ib_buffer_pool
-rw-r----- 1 mysql mysql 50331648 Jan  8 08:41 ib_logfile0
-rw-r----- 1 mysql mysql 50331648 Jan  8 08:41 ib_logfile1
-rw-r----- 1 mysql mysql 79691776 Jan  8 08:41 ibdata1
-rw-r----- 1 mysql mysql 12582912 Jan  8 08:41 ibtmp1
drwxr-x--- 2 mysql mysql     4096 Jan  8 08:41 jumpserver
drwxr-x--- 2 mysql mysql     4096 Jan  8 08:41 mysql
drwxr-x--- 2 mysql mysql     4096 Jan  8 08:41 performance_schema
-rw------- 1 mysql mysql     1676 Jan  8 08:41 private_key.pem
-rw-r--r-- 1 mysql mysql      452 Jan  8 08:41 public_key.pem
-rw-r--r-- 1 mysql mysql     1112 Jan  8 08:41 server-cert.pem
-rw------- 1 mysql mysql     1680 Jan  8 08:41 server-key.pem
drwxr-x--- 2 mysql mysql    12288 Jan  8 08:41 sys
# 


#或者执行下面

[root@Jumpserver ~]#docker exec mysql mysql -p123456 -e 'show variables like "character%"'
mysql: [Warning] Using a password on the command line interface can be insecure.
Variable_name	Value
character_set_client	utf8
character_set_connection	utf8
character_set_database	utf8
character_set_filesystem	binary
character_set_results	utf8
character_set_server	utf8
character_set_system	utf8
character_sets_dir	/usr/share/mysql/charsets/

[root@Jumpserver ~]#docker exec mysql mysql -p123456 -e 'show variables like "collation%"'
mysql: [Warning] Using a password on the command line interface can be insecure.
Variable_name	Value
collation_connection	utf8_general_ci
collation_database	utf8_general_ci
collation_server	utf8_general_ci

[root@Jumpserver ~]#docker exec mysql cat /var/lib/mysql/jumpserver/db.opt
default-character-set=utf8
default-collation=utf8_general_ci

[root@Jumpserver ~]#docker exec mysql cat /etc/mysql/conf.d/mysql.cnf
[mysql]
default-character-set=utf8

[root@Jumpserver ~]#docker exec mysql mysql -p123456 -e 'select user,host from mysql.user'
mysql: [Warning] Using a password on the command line interface can be insecure.
user	host
jumpserver	%
root	%
mysql.session	localhost
mysql.sys	localhost
root	localhost

[root@Jumpserver ~]#docker exec mysql ls /var/lib/mysql/ -l
total 188480
-rw-r----- 1 mysql mysql       56 Jan  8 08:41 auto.cnf
-rw------- 1 mysql mysql     1680 Jan  8 08:41 ca-key.pem
-rw-r--r-- 1 mysql mysql     1112 Jan  8 08:41 ca.pem
-rw-r--r-- 1 mysql mysql     1112 Jan  8 08:41 client-cert.pem
-rw------- 1 mysql mysql     1680 Jan  8 08:41 client-key.pem
-rw-r----- 1 mysql mysql     1346 Jan  8 08:41 ib_buffer_pool
-rw-r----- 1 mysql mysql 50331648 Jan  8 08:41 ib_logfile0
-rw-r----- 1 mysql mysql 50331648 Jan  8 08:41 ib_logfile1
-rw-r----- 1 mysql mysql 79691776 Jan  8 08:41 ibdata1
-rw-r----- 1 mysql mysql 12582912 Jan  8 08:41 ibtmp1
drwxr-x--- 2 mysql mysql     4096 Jan  8 08:41 jumpserver
drwxr-x--- 2 mysql mysql     4096 Jan  8 08:41 mysql
drwxr-x--- 2 mysql mysql     4096 Jan  8 08:41 performance_schema
-rw------- 1 mysql mysql     1676 Jan  8 08:41 private_key.pem
-rw-r--r-- 1 mysql mysql      452 Jan  8 08:41 public_key.pem
-rw-r--r-- 1 mysql mysql     1112 Jan  8 08:41 server-cert.pem
-rw------- 1 mysql mysql     1680 Jan  8 08:41 server-key.pem
drwxr-x--- 2 mysql mysql    12288 Jan  8 08:41 sys




#测试连接MySQL
[root@Rocky8 ~]# mysql -ujumpserver -p -h192.168.188.100
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 11
Server version: 5.7.30 MySQL Community Server (GPL)

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| jumpserver         |
+--------------------+
2 rows in set (0.00 sec)

```

### **2.3.4** 安装 Redis服务

官方说明

```
https://docs.jumpserver.org/zh/master/install/prod/distributed_04/
```

新版要求: 

```
外置 Redis 要求 Redis 版本大于等于 6.0
注意:不支持redis7.0
```

#### 2.3.4.1 启动Redis

```
docker run -d -p 6379:6379 --name redis --restart always  redis:6.2.7

[root@Jumpserver ~]#docker run -d -p 6379:6379 --name redis --restart always  redis:6.2.7
Unable to find image 'redis:6.2.7' locally
6.2.7: Pulling from library/redis
025c56f98b67: Pull complete 
060e65aed679: Pull complete 
b95291e865b7: Pull complete 
e3023c0b11d1: Pull complete 
143500497a02: Pull complete 
c38298c98c43: Pull complete 
Digest: sha256:a93c14584715ec5bd9d2648d58c3b27f89416242bee0bc9e5fb2edc1a4cbec1d
Status: Downloaded newer image for redis:6.2.7
8f920ef42abe2f86e5eb7c4b587fc97e13d4fa1ecd4f64df62c01a80529b0709

```

#### **2.3.4.2** 验证Redis连接

```
[root@Rocky8 ~]# yum -y install redis
[root@Rocky8 ~]# redis-cli -h 192.168.188.100
192.168.188.100:6379> info
# Server
redis_version:6.2.7
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:240c2fa521c5f553
redis_mode:standalone

.....

```

### 2.3.5 部署 JumpServer

#### **2.3.5.1** **生成** key 和 token

参考官方说明:

```
https://github.com/jumpserver/Dockerfile/tree/master/allinone
https://docs.jumpserver.org/zh/master/install/docker_install/
```

**需要先生成** **key** **和** **token**

![image-20240108170211750](堡垒机Jumpserver/image-20240108170211750.png)

```
[root@ubuntu2004 ~]#cat key.sh
#!/bin/bash
if [ ! "$SECRET_KEY" ]; then
  SECRET_KEY=`cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 50`;
  echo "SECRET_KEY=$SECRET_KEY" >> ~/.bashrc;
  echo SECRET_KEY=$SECRET_KEY;
else
  echo SECRET_KEY=$SECRET_KEY;
fi
if [ ! "$BOOTSTRAP_TOKEN" ]; then
  BOOTSTRAP_TOKEN=`cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 16`;
  echo "BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN" >> ~/.bashrc;
  echo BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN;
else
  echo BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN;
fi


[root@centos8 ~]#tail -n2 .bashrc
SECRET_KEY=hysjeaoleZISBTaNznz51HCYhm9YxPFERsykE6I8l73BzW17nS
BOOTSTRAP_TOKEN=8gxGTQUb2fhczzEb

ubuntu的key
[root@Jumpserver ~]#bash key.sh 
SECRET_KEY=UKTcXTULyYpHZHjzeJ8CnRoPbwQDaGGvUZhjxRBhjkF8JqK5b0
BOOTSTRAP_TOKEN=g1mSDon9REyb8Aew


```

#### **2.3.5.2** 运行容器jumpserver

```

docker run --name jms_all -d \
  -v /opt/jumpserver/core/data:/opt/jumpserver/data \
  -v /opt/jumpserver/koko/data:/opt/koko/data \
  -v /opt/jumpserver/lion/data:/opt/lion/data \
  -p 80:80 \
  -p 2222:2222 \
  -e SECRET_KEY=UKTcXTULyYpHZHjzeJ8CnRoPbwQDaGGvUZhjxRBhjkF8JqK5b0 \
  -e BOOTSTRAP_TOKEN=g1mSDon9REyb8Aew \
  -e LOG_LEVEL=ERROR \
  -e DB_HOST=192.168.188.100 \
  -e DB_PORT=3306 \
  -e DB_USER=jumpserver \
  -e DB_PASSWORD=123456 \
  -e DB_NAME=jumpserver \
  -e REDIS_HOST=192.168.188.100 \
  -e REDIS_PORT=6379 \
  -e REDIS_PASSWORD='' \
  --privileged=true \
  jumpserver/jms_all:v2.28.7
  
  #初次启动容器需要等待一会儿才能完成
```

#### **2.3.5.3** **验证是否成功**

##### **2.3.5.3.1** **查看日志**

```
范例：查看jumpserver 2.28.7日志确认成功
[root@Jumpserver ~]#docker logs -f jms_all


Time: 2024-01-08 17:12:22
The Installation is Complete.
    --------------------------------------------------
    | Documentation:    https://docs.jumpserver.org/ |
    | Official Website: https://www.jumpserver.org/  |
    --------------------------------------------------

       ██╗██╗   ██╗███╗   ███╗██████╗ ███████╗███████╗██████╗ ██╗   ██╗███████╗██████╗
       ██║██║   ██║████╗ ████║██╔══██╗██╔════╝██╔════╝██╔══██╗██║   ██║██╔════╝██╔══██╗
       ██║██║   ██║██╔████╔██║██████╔╝███████╗█████╗  ██████╔╝██║   ██║█████╗  ██████╔╝
  ██   ██║██║   ██║██║╚██╔╝██║██╔═══╝ ╚════██║██╔══╝  ██╔══██╗╚██╗ ██╔╝██╔══╝  ██╔══██╗
  ╚█████╔╝╚██████╔╝██║ ╚═╝ ██║██║     ███████║███████╗██║  ██║ ╚████╔╝ ███████╗██║  ██║
   ╚════╝  ╚═════╝ ╚═╝     ╚═╝╚═╝     ╚══════╝╚══════╝╚═╝  ╚═╝  ╚═══╝  ╚══════╝╚═╝  ╚═╝

                                                                   Version: v2.28.7

Default Access:
username: admin  password: admin

LOG_LEVEL: ERROR
JumpServer Logs:
Starting supervisor: 2024-01-08 17:12:22,627 CRIT Supervisor is running as root.  Privileges were not dropped because no user is specified in the config file.  If you intend to run as root, you can set user=root in the config file to avoid this message.
2024-01-08 17:12:22,627 INFO Included extra file "/etc/supervisor/conf.d/supervisord.conf" during parsing
2024-01-08 17:12:22,632 INFO RPC interface 'supervisor' initialized
2024-01-08 17:12:22,632 CRIT Server 'unix_http_server' running without any HTTP authentication checking
2024-01-08 17:12:22,632 INFO supervisord started with pid 28
2024-01-08 17:12:23,635 INFO spawned: 'core' with pid 30
2024-01-08 17:12:23,636 INFO spawned: 'guacd' with pid 31
2024-01-08 17:12:23,638 INFO spawned: 'koko' with pid 32
2024-01-08 17:12:23,639 INFO spawned: 'lion' with pid 33
2024-01-08 17:12:23,640 INFO spawned: 'magnus' with pid 34
Load config from env
{ComponentName:magnus Name:[Magnus]-55e8b274732a CoreHost:http://127.0.0.1:8080 BootstrapToken:g1mSDon9REyb8Aew BindHost:0.0.0.0 BindPort:9090 LogLevel:ERROR RootPath:/opt/magnus ExecuteProgram:/opt/magnus/magnus DataFolderPath:/opt/magnus/data LogFolderPath:/opt/magnus/data/logs KeyFolderPath:/opt/magnus/data/keys AccessKeyFilePath:/opt/magnus/data/keys/.access_key}
2024-01-08 17:12:23 [ERRO] Post "http://127.0.0.1:8080/api/v1/terminal/terminal-registrations/": dial tcp 127.0.0.1:8080: connect: connection refused
2024/01/08 17:12:23 Load config from env
2024/01/08 17:12:23 &{Root:/opt/lion DrivePath:/opt/lion/data/drive RecordPath:/opt/lion/data/replays LogDirPath:/opt/lion/data/logs AccessKeyFilePath:/opt/lion/data/keys/.access_key CertsFolderPath:/opt/lion/data/certs Name:[Lion]-55e8b274732a CoreHost:http://127.0.0.1:8080 BootstrapToken:g1mSDon9REyb8Aew BindHost:0.0.0.0 HTTPPort:8081 LogLevel:ERROR GuaHost:127.0.0.1 GuaPort:4822 DisableAllCopyPaste:false DisableAllUpDownload:false EnableRemoteAppUpDownLoad:false EnableRemoteAPPCopyPaste:false CleanDriveScheduleTime:1 ShareRoomType: RedisHost:192.168.188.100 RedisPort:6379 RedisPassword: RedisDBIndex:0 RedisSentinelPassword: RedisSentinelHosts: RedisUseSSL:false}
2024/01/08 17:12:23 Load config from env
2024/01/08 17:12:23 &{Name:[KoKo]-55e8b274732a CoreHost:http://127.0.0.1:8080 BootstrapToken:g1mSDon9REyb8Aew BindHost:0.0.0.0 SSHPort:2222 HTTPPort:5000 SSHTimeout:15 LogLevel:ERROR Comment:KOKO LanguageCode:zh UploadFailedReplay:true AssetLoadPolicy: ZipMaxSize:1024M ZipTmpPath:/tmp ClientAliveInterval:30 RetryAliveCountMax:3 ShowHiddenFile:false ReuseConnection:true ShareRoomType:local RedisHost:192.168.188.100 RedisPort:6379 RedisPassword: RedisDBIndex:0 RedisClusters:[] RedisSentinelPassword: RedisSentinelHosts: RedisUseSSL:false EnableLocalPortForward:false EnableVscodeSupport:false RootPath:/opt/koko DataFolderPath:/opt/koko/data LogDirPath:/opt/koko/data/logs KeyFolderPath:/opt/koko/data/keys AccessKeyFilePath:/opt/koko/data/keys/.access_key ReplayFolderPath:/opt/koko/data/replays CertsFolderPath:/opt/koko/data/certs}
2024-01-08 17:12:23 [ERRO] Post "http://127.0.0.1:8080/api/v1/terminal/terminal-registrations/": dial tcp 127.0.0.1:8080: connect: connection refused
2024-01-08 17:12:23 main main.go [ERROR] Post "http://127.0.0.1:8080/api/v1/terminal/terminal-registrations/": dial tcp 127.0.0.1:8080: connect: connection refused
2024-01-08 17:12:24,691 INFO success: core entered RUNNING state, process has stayed up for > than 1 seconds (startsecs)
2024-01-08 17:12:24,691 INFO success: guacd entered RUNNING state, process has stayed up for > than 1 seconds (startsecs)
2024-01-08 17:12:24,691 INFO success: koko entered RUNNING state, process has stayed up for > than 1 seconds (startsecs)
2024-01-08 17:12:24,691 INFO success: lion entered RUNNING state, process has stayed up for > than 1 seconds (startsecs)
2024-01-08 17:12:24,691 INFO success: magnus entered RUNNING state, process has stayed up for > than 1 seconds (startsecs)
2024-01-08 17:12:25 Check database connection: 0
System check identified no issues (0 silenced).
2024-01-08 17:12:26 Database connect success
2024-01-08 17:12:26 Collect static files
2024-01-08 17:12:26 Collect static files done
2024-01-08 17:12:26 Check database structure change ...
2024-01-08 17:12:26 Migrate model change to database ...
Operations to perform:
  Apply all migrations: acls, admin, applications, assets, audits, auth, authentication, captcha, common, contenttypes, django_cas_ng, django_celery_beat, jms_oidc_rp, notifications, ops, orgs, perms, rbac, sessions, settings, terminal, tickets, users
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0001_initial... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying users.0001_initial... OK
  Applying users.0002_auto_20171225_1157_squashed_0019_auto_20190304_1459... OK
  Applying authentication.0001_initial... OK
  Applying authentication.0002_auto_20190729_1423... OK
  Applying authentication.0003_loginconfirmsetting... OK
  Applying authentication.0004_ssotoken... OK
  Applying acls.0001_initial... OK
  Applying acls.0002_auto_20210926_1047... OK
  Applying acls.0003_auto_20211130_1037... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK

................................

2024-01-08 17:13:03 JumpServer version v2.28.7, more see https://www.jumpserver.org
2024-01-08 17:13:03 [ERRO] Post "http://127.0.0.1:8080/api/v1/terminal/terminal-registrations/": dial tcp 127.0.0.1:8080: connect: connection refused
2024-01-08 17:13:03 [ERRO] Post "http://127.0.0.1:8080/api/v1/terminal/terminal-registrations/": dial tcp 127.0.0.1:8080: connect: connection refused
2024-01-08 17:13:03 main main.go [ERROR] Post "http://127.0.0.1:8080/api/v1/terminal/terminal-registrations/": dial tcp 127.0.0.1:8080: connect: connection refused
Lion Version v2.28.7, more see https://www.jumpserver.org
time="2024-01-08T17:13:21+08:00" level=fatal msg="ssh: no key found"
2024-01-08 17:13:21,570 INFO exited: koko (exit status 1; not expected)
2024-01-08 17:13:22,585 INFO spawned: 'koko' with pid 160
2024/01/08 17:13:22 Load config from env
2024/01/08 17:13:22 &{Name:[KoKo]-55e8b274732a CoreHost:http://127.0.0.1:8080 BootstrapToken:g1mSDon9REyb8Aew BindHost:0.0.0.0 SSHPort:2222 HTTPPort:5000 SSHTimeout:15 LogLevel:ERROR Comment:KOKO LanguageCode:zh UploadFailedReplay:true AssetLoadPolicy: ZipMaxSize:1024M ZipTmpPath:/tmp ClientAliveInterval:30 RetryAliveCountMax:3 ShowHiddenFile:false ReuseConnection:true ShareRoomType:local RedisHost:192.168.188.100 RedisPort:6379 RedisPassword: RedisDBIndex:0 RedisClusters:[] RedisSentinelPassword: RedisSentinelHosts: RedisUseSSL:false EnableLocalPortForward:false EnableVscodeSupport:false RootPath:/opt/koko DataFolderPath:/opt/koko/data LogDirPath:/opt/koko/data/logs KeyFolderPath:/opt/koko/data/keys AccessKeyFilePath:/opt/koko/data/keys/.access_key ReplayFolderPath:/opt/koko/data/replays CertsFolderPath:/opt/koko/data/certs}
2024-01-08 17:13:22
KoKo Version v2.28.7, more see https://www.jumpserver.org
Quit the server with CONTROL-C.
2024-01-08 17:13:23,836 INFO success: koko entered RUNNING state, process has stayed up for > than 1 seconds (startsecs)

```

##### **2.3.5.3.2** **查看** **MySQL** **中生成相关表**

```
[root@Jumpserver ~]#docker exec -it mysql sh
# mysql -ujumpserver -p123456 jumpserver
mysql: [Warning] Using a password on the command line interface can be insecure.
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 127
Server version: 5.7.30 MySQL Community Server (GPL)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show tables;
+---------------------------------------------------+
| Tables_in_jumpserver                              |
+---------------------------------------------------+
| acls_loginacl                                     |
| acls_loginacl_reviewers                           |
| acls_loginassetacl                                |
| acls_loginassetacl_reviewers                      |
| applications_account                              |
| applications_application                          |
| applications_historicalaccount                    |
| assets_accountbackupplan                          |
| assets_accountbackupplan_recipients               |
| assets_accountbackupplanexecution                 |
| assets_adminuser                                  |
| assets_asset                                      |
| assets_asset_labels                               |
| assets_asset_nodes                                |
| assets_assetgroup                                 |
| assets_authbook                                   |
| assets_cluster                                    |
| assets_commandfilter                              |
| assets_commandfilter_applications                 |
| assets_commandfilter_assets                       |
| assets_commandfilter_nodes                        |
| assets_commandfilter_system_users                 |
| assets_commandfilter_user_groups                  |
| assets_commandfilter_users                        |
| assets_commandfilterrule                          |
| assets_commandfilterrule_reviewers                |
| assets_domain                                     |
| assets_favoriteasset                              |
| assets_gateway                                    |
| assets_gathereduser                               |
| assets_historicalauthbook                         |
| assets_label                                      |
| assets_node                                       |
| assets_platform                                   |
| assets_systemuser                                 |
| assets_systemuser_groups                          |
| assets_systemuser_nodes                           |
| assets_systemuser_users                           |
| audits_ftplog                                     |
| audits_operatelog                                 |
| audits_passwordchangelog                          |
| audits_userloginlog                               |
| auth_group                                        |
| auth_group_permissions                            |
| auth_permission                                   |
| authentication_accesskey                          |
| authentication_connectiontoken                    |
| authentication_privatetoken                       |
| authentication_ssotoken                           |
| authentication_temptoken                          |
| captcha_captchastore                              |
| django_admin_log                                  |
| django_cas_ng_proxygrantingticket                 |
| django_cas_ng_sessionticket                       |
| django_celery_beat_clockedschedule                |
| django_celery_beat_crontabschedule                |
| django_celery_beat_intervalschedule               |
| django_celery_beat_periodictask                   |
| django_celery_beat_periodictasks                  |
| django_celery_beat_solarschedule                  |
| django_content_type                               |
| django_migrations                                 |
| django_session                                    |
| jms_oidc_rp_oidcuser                              |
| notifications_sitemessage                         |
| notifications_sitemessage_groups                  |
| notifications_sitemessageusers                    |
| notifications_systemmsgsubscription               |
| notifications_systemmsgsubscription_groups        |
| notifications_systemmsgsubscription_users         |
| notifications_usermsgsubscription                 |
| ops_adhoc                                         |
| ops_adhoc_execution                               |
| ops_adhoc_hosts                                   |
| ops_celerytask                                    |
| ops_commandexecution                              |
| ops_commandexecution_hosts                        |
| ops_task                                          |
| orgs_organization                                 |
| orgs_organization_members                         |
| perms_applicationpermission                       |
| perms_applicationpermission_applications          |
| perms_applicationpermission_system_users          |
| perms_applicationpermission_user_groups           |
| perms_applicationpermission_users                 |
| perms_assetpermission                             |
| perms_assetpermission_assets                      |
| perms_assetpermission_nodes                       |
| perms_assetpermission_system_users                |
| perms_assetpermission_user_groups                 |
| perms_assetpermission_users                       |
| perms_userassetgrantedtreenoderelation            |
| rbac_menupermission                               |
| rbac_role                                         |
| rbac_role_permissions                             |
| rbac_rolebinding                                  |
| settings_setting                                  |
| terminal                                          |
| terminal_command                                  |
| terminal_commandstorage                           |
| terminal_endpoint                                 |
| terminal_endpointrule                             |
| terminal_replaystorage                            |
| terminal_session                                  |
| terminal_sessionjoinrecord                        |
| terminal_sessionreplay                            |
| terminal_sessionsharing                           |
| terminal_status                                   |
| terminal_task                                     |
| tickets_applyapplicationticket                    |
| tickets_applyapplicationticket_apply_applications |
| tickets_applyapplicationticket_apply_system_users |
| tickets_applyassetticket                          |
| tickets_applyassetticket_apply_assets             |
| tickets_applyassetticket_apply_nodes              |
| tickets_applyassetticket_apply_system_users       |
| tickets_applycommandticket                        |
| tickets_applyloginassetticket                     |
| tickets_applyloginticket                          |
| tickets_approvalrule                              |
| tickets_approvalrule_assignees                    |
| tickets_comment                                   |
| tickets_ticket                                    |
| tickets_ticketassignee                            |
| tickets_ticketflow                                |
| tickets_ticketflow_rules                          |
| tickets_ticketsession                             |
| tickets_ticketstep                                |
| users_user                                        |
| users_user_groups                                 |
| users_user_user_permissions                       |
| users_usergroup                                   |
| users_userpasswordhistory                         |
+---------------------------------------------------+
133 rows in set (0.00 sec)

mysql> 

```

##### **2.3.5.3.3** **查看端口**

```
[root@Jumpserver ~]#ss -ntl
State              Recv-Q              Send-Q                           Local Address:Port                            Peer Address:Port             Process             
LISTEN             0                   4096                                 127.0.0.1:39719                                0.0.0.0:*                                    
LISTEN             0                   4096                                   0.0.0.0:3306                                 0.0.0.0:*                                    
LISTEN             0                   4096                                   0.0.0.0:6379                                 0.0.0.0:*                                    
LISTEN             0                   4096                                   0.0.0.0:2222                                 0.0.0.0:*                                    
LISTEN             0                   4096                                   0.0.0.0:80                                   0.0.0.0:*                                    
LISTEN             0                   4096                             127.0.0.53%lo:53                                   0.0.0.0:*                                    
LISTEN             0                   128                                    0.0.0.0:22                                   0.0.0.0:*                                    
LISTEN             0                   128                                  127.0.0.1:6010                                 0.0.0.0:*                                    
LISTEN             0                   128                                  127.0.0.1:6011                                 0.0.0.0:*                                    
LISTEN             0                   128                                  127.0.0.1:6012                                 0.0.0.0:*                                    
LISTEN             0                   4096                                      [::]:3306                                    [::]:*                                    
LISTEN             0                   4096                                      [::]:6379                                    [::]:*                                    
LISTEN             0                   4096                                      [::]:2222                                    [::]:*                                    
LISTEN             0                   4096                                      [::]:80                                      [::]:*                                    
LISTEN             0                   128                                       [::]:22                                      [::]:*                                    
LISTEN             0                   128                                      [::1]:6010                                    [::]:*                                    
LISTEN             0                   128                                      [::1]:6011                                    [::]:*                                    
LISTEN             0                   128                                      [::1]:6012                                    [::]:*         
```

##### **2.3.5.3.4** 验证登录

\#安装完成后，直接访问即可，默认用户名和密码是admin/admin

![image-20240108172008093](堡垒机Jumpserver/image-20240108172008093.png)

```
不需要<style><style/>bi
```

# 3、**JumpServer** **常见功能**

官方文档:

```
https://docs.jumpserver.org/zh/master/
```

## **3.1** 登录并初始化配置

### **3.1.1** **首次登录**

#### **3.1.1.1** **浏览器访问**

```
http:<JumpServerIP>
```

登录 JumpServer 默认用户: admin 密码: admin，第一次登录要求重置密码

![image-20240108173012199](堡垒机Jumpserver/image-20240108173012199.png)

#### **3.1.1.2 ssh** **登录**

可以用admin用户和修改过的新密码, 连接jumpserver的ssh端口2222/tcp进行连接访问

注意:新版需要实现资产授权后才能连接登录

```
[root@Jumpserver ~]#ssh -p2222 admin@192.168.188.100
The authenticity of host '[192.168.188.100]:2222 ([192.168.188.100]:2222)' can't be established.
RSA key fingerprint is SHA256:M5JQqsC2eFbqarB6kIKPRt7yFdVr6+I9YGoUxYUWaDM.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[192.168.188.100]:2222' (RSA) to the list of known hosts.
admin@192.168.188.100's password: 
		Administrator,  JumpServer 开源堡垒机

	1) 输入 部分IP，主机名，备注 进行搜索登录(如果唯一).
	2) 输入 / + IP，主机名，备注 进行搜索，如：/192.168.
	3) 输入 p 进行显示您有权限的主机.
	4) 输入 g 进行显示您有权限的节点.
	5) 输入 d 进行显示您有权限的数据库.
	6) 输入 k 进行显示您有权限的Kubernetes.
	7) 输入 r 进行刷新最新的机器和节点信息.
	8) 输入 s 进行中英日语言切换.
	9) 输入 h 进行显示帮助.
	10) 输入 q 进行退出.
Opt> 

```

### **3.1.2** **修改** **admin** **用户的密码**





















用户组

jumpserever用户

特权账户

资产授权

系统用户的普通用户

