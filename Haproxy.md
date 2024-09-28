# HAProxy反向代理

# 1、haproxy介绍

![image-20240503140547043](Haproxy/image-20240503140547043.png)

负载均衡：Load Balance，简称 LB，是一种服务或基于硬件设备等实现的高可用反向代理技术，负载均衡将特定的业务(web服务、网络流量等)分担给指定的一个或多个后端特定的服务器或设备，从而提高了公司业务的并发处理能力、保证了业务的高可用性、方便了业务后期的水平动态扩展。阿里云SLB介绍 ：https://yq.aliyun.com/articles/1803

![image-20240503140638999](Haproxy/image-20240503140638999.png)

## **1.1** **为什么使用负载均衡**

```
增加业务并发访问及处理能力-->解决单服务器瓶颈问题

节约公网IP地址-->降低IT支出成本

隐藏内部服务器IP-->提高内部服务器安全性

Web服务器的动态水平扩展-->对用户无感知

负载均衡配置简单-->固定格式的配置文件

负载均衡功能丰富-->支持四层和七层，支持动态下线主机

负载均衡性能较强-->并发数万甚至百万
```

## **1.2** **负载均衡类型**

四层：

```
LVS：Linux Virtual Server
Nginx：1.9版之后   stream 
HAProxy：High Availability Proxy mode tcp
```

七层：

```
HAProxy mode http
Nginx http
```

硬件：

```
F5       https://f5.com/zh
Netscaler https://www.citrix.com.cn/products/citrix-adc/
Array     https://www.arraynetworks.com.cn/
深信服       http://www.sangfor.com.cn/
北京灵州     http://www.lingzhou.com.cn/cpzx/llfzjh/
```

## **1.3** **应用场景**

```
四层：Redis、Mysql、RabbitMQ、Memcached等
七层：Nginx、Tomcat、Apache、PHP、图片、动静分离、API等
```

# 2 安装部署

## 2.1、ubuntu安装部署

```
[root@ubuntu1804 ~]#apt-get install software-properties-common
[root@ubuntu1804 ~]#add-apt-repository ppa:vbernat/haproxy-2.0
[root@ubuntu1804 ~]#apt update
[root@ubuntu1804 ~]#apt-cache madison haproxy
#安装指定版本
[root@ubuntu1804 ~]#apt install haproxy=2.0.4-1ppa1~bionic
#或安装最新版
[root@ubuntu1804 ~]#apt-get install haproxy=2.0.\*
#验证haproxy版本
[root@ubuntu1804 ~]#haproxy -v
```

ubuntu2004

```sh
[root@ubuntu2004 ~]# apt-get install --no-install-recommends software-properties-common
[root@ubuntu2004 ~]# add-apt-repository ppa:vbernat/haproxy-2.8

[root@ubuntu2004 ~]# apt-get install haproxy=2.8.\*

[root@ubuntu2004 ~]# haproxy -v
HAProxy version 2.8.9-1ppa1~focal 2024/04/06 - https://haproxy.org/
Status: long-term supported branch - will stop receiving fixes around Q2 2028.
Known bugs: http://www.haproxy.org/bugs/bugs-2.8.9.html
Running on: Linux 5.4.0-177-generic #197-Ubuntu SMP Thu Mar 28 22:45:47 UTC 2024 x86_64
```

## 2.2、编译安装

编译安装HAProxy 2.0 LTS版本，更多源码包下载地址：http://www.haproxy.org/download/

### 2.2.1、解决lua环境问题

由于CentOS7 之前版本自带的lua版本比较低并不符合HAProxy要求的lua最低版本(5.3)的要求，因此需要编译安装较新版本的lua环境，然后才能编译安装HAProxy，过程如下：

#### **2.2.1.1 CentOS** **基础环境**

参考链接：http://www.lua.org/start.html

```sh
[root@centos7 ~]# lua -v
Lua 5.1.4  Copyright (C) 1994-2008 Lua.org, PUC-Rio

#安装基础命令及编译依赖环境
[root@centos7 ~]# yum install gcc readline-devel
[root@centos7 ~]# wget http://www.lua.org/ftp/lua-5.3.5.tar.gz
[root@centos7 ~]# tar xvf lua-5.3.5.tar.gz -C /usr/local/src
[root@centos7 ~]# cd /usr/local/src/lua-5.3.5
[root@centos7 lua-5.3.5]# make linux test

#查看编译安装的版本
[root@centos7 lua-5.3.5]#src/lua -v
Lua 5.3.5 Copyright (C) 1994-2018 Lua.org, PUC-Rio
```

ubuntu的Lua版本较新，也可以用包安装

```
#包安装Lua
[root@ubuntu2204 ~]#apt update && apt -y install liblua5.4-dev
```

#### **2.2.1.2 Ubuntu** **基础环境**

```sh
#安装基础命令及编译依赖环境
[root@ubuntu2204 ~]#apt update && apt -y install gcc make libssl-dev libpcre3 libpcre3-dev zlib1g-dev libreadline-dev libsystemd-dev
[root@ubuntu1804 ~]#apt update && apt -y install gcc make libssl-dev libpcre3 libpcre3-dev zlib1g-dev libreadline-dev libsystemd-dev

#安装Lua方法1：包安装Lua
[root@ubuntu2204 ~]#apt update && apt -y install liblua5.4-dev

#安装Lua方法2：编译安装Lua
[root@ubuntu1804 ~]# cd /usr/local/src
[root@ubuntu1804 ~]# wget http://www.lua.org/ftp/lua-5.3.5.tar.gz
[root@ubuntu1804 ~]# tar xvf lua-5.3.5.tar.gz
[root@ubuntu1804 ~]# cd lua-5.3.5
[root@ubuntu1804 ~]# make linux test ; make install
[root@ubuntu1804 ~]#pwd
/usr/local/src/lua-5.3.5
[root@ubuntu1804 ~]# ./src/lua -v
Lua 5.3.5 Copyright (C) 1994-2018 Lua.org, PUC-Rio

#或安装系统自带的lua
[root@ubuntu1804 ~]#apt install lua5.3=5.3.3-1ubuntu0.18.04.1
[root@ubuntu1804 ~]#lua5.3 -v
Lua 5.3.3 Copyright (C) 1994-2016 Lua.org, PUC-Rio
```

### 2.2.2、**编译安装** **HAProxy**

范例：ubuntu

```sh
#安装依赖包
[root@ubuntu2004 ~]#apt update && apt -y install gcc make libssl-dev libpcre3 libpcre3-dev zlib1g-dev libreadline-dev libsystemd-dev liblua5.4-dev

#下载二进制包
[root@ubuntu2004 ~]#wget https://www.haproxy.org/download/2.8/src/haproxy-2.8.9.tar.gz
[root@ubuntu2004 ~]#tar zxf haproxy-2.8.9.tar.gz && cd haproxy-2.8.9

#编译安装
[root@ubuntu2004 ~/haproxy-2.8.9]#make ARCH=x86_64 TARGET=linux-glibc USE_PCRE=1 USE_OPENSSL=1 USE_ZLIB=1 USE_SYSTEMD=1 USE_PROMEX=1 USE_LUA=1 && make install PREFIX=/apps/haproxy/

#查看安装目录
[root@ubuntu2004 ~/haproxy-2.8.9]#tree /apps/haproxy/
/apps/haproxy/
├── doc
│   └── haproxy
│       ├── 51Degrees-device-detection.txt
│       ├── architecture.txt
│       ├── configuration.txt
│       ├── cookie-options.txt
│       ├── DeviceAtlas-device-detection.txt
│       ├── intro.txt
│       ├── linux-syn-cookies.txt
│       ├── lua.txt
│       ├── management.txt
│       ├── netscaler-client-ip-insertion-protocol.txt
│       ├── network-namespaces.txt
│       ├── peers.txt
│       ├── peers-v2.0.txt
│       ├── proxy-protocol.txt
│       ├── regression-testing.txt
│       ├── seamless_reload.txt
│       ├── SOCKS4.protocol.txt
│       ├── SPOE.txt
│       └── WURFL-device-detection.txt
├── sbin
│   └── haproxy
└── share
    └── man
        └── man1
            └── haproxy.1

6 directories, 21 files

#查看版本
[root@ubuntu2004 ~/haproxy-2.8.9]#haproxy -v
HAProxy version 2.8.9-1842fd0 2024/04/05 - https://haproxy.org/
Status: long-term supported branch - will stop receiving fixes around Q2 2028.
Known bugs: http://www.haproxy.org/bugs/bugs-2.8.9.html
Running on: Linux 5.4.0-162-generic #179-Ubuntu SMP Mon Aug 14 08:51:31 UTC 2023 x86_64

[root@ubuntu2004 ~/haproxy-2.8.9]#haproxy -V
HAProxy version 2.8.9-1842fd0 2024/04/05 - https://haproxy.org/
Status: long-term supported branch - will stop receiving fixes around Q2 2028.
Known bugs: http://www.haproxy.org/bugs/bugs-2.8.9.html
Running on: Linux 5.4.0-162-generic #179-Ubuntu SMP Mon Aug 14 08:51:31 UTC 2023 x86_64
Usage : haproxy [-f <cfgfile|cfgdir>]* [ -vdVD ] [ -n <maxconn> ] [ -N <maxpconn> ]
        [ -p <pidfile> ] [ -m <max megs> ] [ -C <dir> ] [-- <cfgfile>*]
        -v displays version ; -vv shows known build options.
        -d enters debug mode ; -db only disables background mode.
        -dM[<byte>,help,...] debug memory (default: poison with <byte>/0x50)
        -V enters verbose mode (disables quiet mode)
        -D goes daemon ; -C changes to <dir> before loading files.
        -W master-worker mode.
        -Ws master-worker mode with systemd notify support.
        -q quiet mode : don't display messages
        -c check mode : only check config files and exit
        -cc check condition : evaluate a condition and exit
        -n sets the maximum total # of connections (uses ulimit -n)
        -m limits the usable amount of memory (in MB)
        -N sets the default, per-proxy maximum # of connections (0)
        -L set local peer name (default to hostname)
        -p writes pids of all children to this file
        -dC[[key],line] display the configuration file, if there is a key, the file will be anonymised
        -de disables epoll() usage even when available
        -dp disables poll() usage even when available
        -dS disables splice usage (broken on old kernels)
        -dG disables getaddrinfo() usage
        -dR disables SO_REUSEPORT usage
        -dL dumps loaded object files after config checks
        -dK{class[,...]} dump registered keywords (use 'help' for list)
        -dr ignores server address resolution failures
        -dV disables SSL verify on servers side
        -dW fails if any warning is emitted
        -dD diagnostic mode : warn about suspicious configuration statements
        -dF disable fast-forward
        -sf/-st [pid ]* finishes/terminates old pids.
        -x <unix_socket> get listening sockets from a unix socket
        -S <bind>[,<bind options>...] new master CLI
```

### 2.2.3、创建用户

```sh
[root@ubuntu2004 ~]#useradd -m -r -s /sbin/nologin -d /var/lib/haproxy haproxy
```

### **2.2.4** **配置文件**

haproxy.cfg文件中定义了chroot、pidfile、user、group等参数，如果系统没有相应的资源会导致haproxy无法启动，具体参考日志文件 "/var/log/syslog"或"/var/log/messages"

```sh
#准备配置文件
[root@ubuntu2004 ~]#mkdir /etc/haproxy/
[root@ubuntu2004 ~]:~# grep -Ev '^(#|$)' /etc/haproxy/haproxy.cfg 
global
maxconn 100000
stats socket /var/lib/haproxy/haproxy.sock mode 600 level admin
uid 99
gid 99
daemon
log 127.0.0.1 local2 info
defaults
option http-keep-alive
option forwardfor
maxconn 100000
mode http
timeout connect 300000ms
timeout client 300000ms
timeout server 300000ms
listen stats
    mode http
    bind 0.0.0.0:9999
    stats enable
    log global
    stats uri /haproxy-status
    stats auth admin:123456 
```

### 2.2.5、准备service启动文件

```sh
[root@ubuntu2004 ~]#cat /lib/systemd/system/haproxy.service 
[Unit]                                                                                                                                                                 
Description=HAProxy Load Balancer
After=network-online.target
Wants=network-online.target

[Service]
ExecStartPre=/usr/local/sbin/haproxy -f /etc/haproxy/haproxy.cfg -c -q
ExecStart=/usr/local/sbin/haproxy -Ws -f /etc/haproxy/haproxy.cfg -p /var/lib/haproxy/haproxyr.pid
ExecReload=/usr/local/sbin/haproxy -f $CONFIG -c -q $OPTIONS
ExecReload=/bin/kill -USR2 $MAINPID
SuccessExitStatus=143
KillMode=mixed
Type=notify

[Install]
WantedBy=multi-user.target

[root@ubuntu2004 ~]#systemctl daemon-reload
[root@ubuntu2004 ~]#systemctl status haproxy.service 
```

### 2.2.6、查看haproxy状态

![image-20240508213048553](Haproxy/image-20240508213048553.png)

![image-20240508213105388](Haproxy/image-20240508213105388.png)

# 3、**HAProxy** **基础配置**

官方帮助文档

```
http://cbonte.github.io/haproxy-dconv/
http://cbonte.github.io/haproxy-dconv/2.1/configuration.html
```

HAProxy 的配置文件haproxy.cfg由两大部分组成，分别是global和proxies部分

```
#global：全局配置段

进程及安全配置相关的参数
性能调整相关参数
Debug参数

#proxies：代理配置段

defaults：为frontend, backend, listen提供默认配置
listen：同时拥有前端和后端配置,配置简单,生产推荐使用
frontend：前端，相当于nginx中的server {}
backend：后端，相当于nginx中的upstream {}
```

## **3.1 Global配置**

### **3.1.1 Global** **配置参数说明**

官方文档：

```
http://cbonte.github.io/haproxy-dconv/2.6/configuration.html#3
http://cbonte.github.io/haproxy-dconv/2.4/configuration.html#3
```

**常见配置指令**

```SH
chroot /apps/haproxy #锁定运行目录
deamon               #以守护进程运行
stats socket /var/lib/haproxy/haproxy.sock mode 600 level admin process 1
#socket文件,并可以通过此文件管理
user, group, uid, gid  #运行haproxy对应worker进程的用户身份
#nbproc   n   #开启的haproxy worker 进程数，默认进程数是一个, nbproc从HAProxy 2.5开始不再支持
nbthread  1 #默认值auto，和多进程 nbproc配置互斥（版本有关,CentOS8的haproxy1.8无此问题）,指定每个haproxy进程开启的线程数，默认为每个进程一个线程
#如果同时启用nbproc和nbthread 会出现以下日志的错误，无法启动服务
Apr  7 14:46:23 haproxy haproxy: [ALERT] 097/144623 (1454) : config : cannot enable multiple processes if multiple threads are configured. Please use either nbproc or nbthread but not both.
#注意：下面方式不支持线程绑定
#cpu-map 1 0   #绑定haproxy worker 进程至指定CPU，将第1个worker进程绑定至0号CPU
#cpu-map 2 1     #绑定haproxy worker 进程至指定CPU，将第2个worker进程绑定至1号CPU
cpu-map auto:1/1-8 0-7 #haproxy2.4中启用nbthreads，在global配置中添加此选项，可以进行线程和CPU的绑定，nbproc选项2.5版本中将会删除,每个进程中1-8个线程分别绑定0-7号CPU，注意：新版用此项进行线程绑定
maxconn n   #每个haproxy进程的最大并发连接数
maxsslconn n   #每个haproxy进程ssl最大连接数,用于haproxy 配置了证书的场景下
maxconnrate n   #每个进程每秒创建的最大连接数量

spread-checks n #后端server状态check随机提前或延迟百分比时间，建议2-5(20%-50%)之间，默认值0
pidfile #指定pid文件路径
log 127.0.0.1 local2 info #定义全局的syslog服务器；日志服务器需要开启UDP协议，最多可以定义两个
```

### **3.1.2** **多进程和线程**

从2.5以上新版中用多线程代替多worker进程

范例: 多线程和CPU绑定

```SH
#范例1
[root@haproxy ~]#vim /etc/haproxy/haproxy.cfg
global 
 nbthread  8  #线程数
 cpu-map auto:1/1-8 0-7   #线程与cpu核数的绑定关系
......

#语法检查
[root@haproxy ~]#haproxy -c -f /etc/haproxy/haproxy.cfg
[root@haproxy ~]#systemctl restart haproxy.service

#查看CPU和线程之间的绑定关系
[root@ubuntu2004 ~]# ps axo pid,cmd,psr -L |grep haproxy
   6902 /usr/sbin/haproxy -Ws -f /e   5
   6904 /usr/sbin/haproxy -Ws -f /e   0
   6904 /usr/sbin/haproxy -Ws -f /e   1
   6904 /usr/sbin/haproxy -Ws -f /e   2
   6904 /usr/sbin/haproxy -Ws -f /e   3
   6904 /usr/sbin/haproxy -Ws -f /e   4
   6904 /usr/sbin/haproxy -Ws -f /e   5
   6904 /usr/sbin/haproxy -Ws -f /e   6
   6904 /usr/sbin/haproxy -Ws -f /e   7
   8045 grep --color=auto haproxy     7

#持续监控状态
[root@ubuntu2004 ~]# watch -n 1 'ps axo pid,cmd,psr -L |grep haproxy'

Every 1.0s: ps axo pid,cmd,psr -L |grep haproxy                                                                                    ubuntu2004: Wed May 22 09:49:11 2024

   6902 /usr/sbin/haproxy -Ws -f /e   5
   6904 /usr/sbin/haproxy -Ws -f /e   0
   6904 /usr/sbin/haproxy -Ws -f /e   1
   6904 /usr/sbin/haproxy -Ws -f /e   2
   6904 /usr/sbin/haproxy -Ws -f /e   3
   6904 /usr/sbin/haproxy -Ws -f /e   4
   6904 /usr/sbin/haproxy -Ws -f /e   5
   6904 /usr/sbin/haproxy -Ws -f /e   6
   6904 /usr/sbin/haproxy -Ws -f /e   7
   7342 grep haproxy                  7

#安装sysstats工具
[root@ubuntu2004 ~]#apt -y install sysstat
[root@ubuntu2004 ~]# pidstat -p 6904 -t
Linux 5.4.0-182-generic (ubuntu2004) 	05/22/2024 	_x86_64_	(8 CPU)

09:51:28 AM   UID      TGID       TID    %usr %system  %guest   %wait    %CPU   CPU  Command
09:51:28 AM    99      6904         -    0.00    0.00    0.00    0.00    0.00     0  haproxy
09:51:28 AM    99         -      6904    0.00    0.00    0.00    0.00    0.00     0  |__haproxy
09:51:28 AM    99         -      6905    0.00    0.00    0.00    0.00    0.00     1  |__haproxy
09:51:28 AM    99         -      6906    0.00    0.00    0.00    0.00    0.00     2  |__haproxy
09:51:28 AM    99         -      6907    0.00    0.00    0.00    0.00    0.00     3  |__haproxy
09:51:28 AM    99         -      6908    0.00    0.00    0.00    0.00    0.00     4  |__haproxy
09:51:28 AM    99         -      6909    0.00    0.00    0.00    0.00    0.00     5  |__haproxy
09:51:28 AM    99         -      6910    0.00    0.00    0.00    0.00    0.00     6  |__haproxy
09:51:28 AM    99         -      6911    0.00    0.00    0.00    0.00    0.00     7  |__haproxy
```

**范例：多进程和socket文件**

```sh
[root@haproxy ~]#vim /etc/haproxy/haproxy.cfg
global
maxconn 100000
chroot /apps/haproxy
nbproc 4
stats socket /var/lib/haproxy/haproxy1.sock mode 600 level admin process 1       
      
stats socket /var/lib/haproxy/haproxy2.sock mode 600 level admin process 2
stats socket /var/lib/haproxy/haproxy3.sock mode 600 level admin process 3
stats socket /var/lib/haproxy/haproxy4.sock mode 600 level admin process 4
uid 99
gid 99
daemon
[root@haproxy ~]##systemctl restart haproxy
[root@haproxy ~]#pstree -p |grep haproxy
           |-haproxy(1933)-+-haproxy(1935)
           |               |-haproxy(1936)
           |               |-haproxy(1937)
           |               `-haproxy(1938)
[root@haproxy ~]#ll /var/lib/haproxy/ 
total 12
drwxr-xr-x  2 root root 4096 Sep  4 10:02 ./
drwxr-xr-x 38 root root 4096 Sep  2 22:37 ../
srw-------  1 root root    0 Sep  4 10:02 haproxy1.sock=
srw-------  1 root root    0 Sep  4 10:02 haproxy2.sock=
srw-------  1 root root    0 Sep  4 10:02 haproxy3.sock=
srw-------  1 root root    0 Sep  4 10:02 haproxy4.sock=
[root@haproxy ~]#ps axo pid,cmd,psr|grep haproxy
  1933 /usr/sbin/haproxy -Ws -f /e   5
  1935 /usr/sbin/haproxy -Ws -f /e   0
  1936 /usr/sbin/haproxy -Ws -f /e   1
  1937 /usr/sbin/haproxy -Ws -f /e   2
  1938 /usr/sbin/haproxy -Ws -f /e   3
  1965 grep --color=auto haproxy     1
```

### **3.1.3 HAProxy**日志配置

HAproxy本身不记录客户端的访问日志.此外为减少服务器负载,一般生产中HAProxy不记录日志，也可以配置HAProxy利用rsyslog服务记录日志到指定日志文件中

#### **3.1.3.1 HAProxy配置**

```sh
#在global配置项定义：
log 127.0.0.1 local{0-7} info #基于syslog记录日志到指定设备，级别包括:err、warning、
info、debug
listen web_port
 bind 127.0.0.1:80
 mode http
 log global #开启当前web_port的日志功能，默认不记录日志
 server web1  127.0.0.1:8080 check inter 3000 fall 2 rise 5
  
# systemctl restart haproxy
```

#### 3.1.3.2 Rsyslog配置

```sh
[root@ubuntu2004 ~]# vim /etc/rsyslog.conf 
$ModLoad imudp
$UDPServerRun 514
......
local2.*   /var/log/haproxy.log
......
[root@ubuntu2004 ~]#systemctl restart haproxy.service rsyslog.service 
```

#### **3.1.3.3** **验证HAProxy日志**

**重启syslog服务并访问 Web 页面，然后验证是否生成日志**

```sh
[root@ubuntu2004 ~]#tail -f /var/log/haproxy.log 
May  8 22:44:22 ubuntu2004 haproxy[5196]: 11.0.1.99:56297 [08/May/2024:22:44:22.408] stats stats/<NOSRV> 0/-1/-1/-1/0 503 216 - - SC-- 2/2/0/0/0 0/0 "GET /favicon.ico HTTP/1.1"
May  8 22:44:43 ubuntu2004 haproxy[5192]: [NOTICE]   (5192) : haproxy version is 2.8.9-1842fd0
May  8 22:44:43 ubuntu2004 haproxy[5192]: [NOTICE]   (5192) : path to executable is /usr/local/sbin/haproxy
May  8 22:44:43 ubuntu2004 haproxy[5192]: [WARNING]  (5192) : Exiting Master process...
May  8 22:44:43 ubuntu2004 haproxy[5192]: [ALERT]    (5192) : Current worker (5196) exited with code 143 (Terminated)
May  8 22:44:43 ubuntu2004 haproxy[5192]: [WARNING]  (5192) : All workers exited. Exiting... (0)
May  8 22:44:43 ubuntu2004 haproxy[5291]: [NOTICE]   (5291) : New worker (5293) forked
May  8 22:44:43 ubuntu2004 haproxy[5291]: [NOTICE]   (5291) : Loading success.
```

#### **3.1.3.4** **实战案例：启动本地和远程日志**

```sh
#本地主机haproxy配置
[root@ubuntu2004 ~]# vim /etc/haproxy/haproxy.cfg 

global

......

log 127.0.0.1 local2 info
log 11.0.1.102 local2 info

......

[root@ubuntu2004 ~]# vim /etc/rsyslog.conf 

......

# provides UDP syslog reception
module(load="imudp")
input(type="imudp" port="514")

local2.* /var/log/haproxy.log
......

[root@ubuntu2004 ~]# systemctl restart rsyslog.service haproxy.service
[root@ubuntu2004 ~]# cat /var/log/haproxy.log 
May 22 10:48:35 localhost haproxy[13299]: Connect from 11.0.1.99:60638 to 11.0.1.101:9999 (stats/HTTP)
May 22 10:48:35 localhost haproxy[13299]: Connect from 11.0.1.99:60638 to 11.0.1.101:9999 (stats/HTTP)

#远程主机配置.102
[root@ubuntu2004 ~]# vim /etc/rsyslog.conf 

......

# provides UDP syslog reception
module(load="imudp")
input(type="imudp" port="514")


local2.* /var/log/haproxy.log
......
[root@ubuntu2004 ~]# systemctl restart rsyslog.service haproxy.service

[root@ubuntu2004 ~]#cat /var/log/haproxy.log 
May 22 10:47:37 11.0.1.101 haproxy[13299]: Connect from 11.0.1.99:60638 to 11.0.1.101:9999 (stats/HTTP)

```

## **3.2 Proxies** **配置**

官方文档：[HAProxy version 2.8.9-19 - Configuration Manual](https://docs.haproxy.org/2.8/configuration.html)

```sh
defaults [<name>] #默认配置项，针对以下的frontend、backend和listen生效，可以多个name也可以没有name
frontend <name>   #前端servername，类似于Nginx的一个虚拟主机 server和LVS服务集群。
backend <name>   #后端服务器组，等于nginx的upstream和LVS中的RS服务器
listen   <name>   #将frontend和backend合并在一起配置，相对于frontend和backend配置更简洁，生产常用
```

注意：name字段只能使用大小写字母，数字，‘-’(dash)，'_‘(underscore)，'.' (dot)和 ':'(colon)，并且严格区分大小写

### 3.2.1 **Proxies配置** **defaults**

**defaults 配置参数：**

```
option redispatch     #当server Id对应的服务器挂掉后，强制定向到其他健康的服务器，重新派发
option abortonclose   #当服务器负载很高时，自动结束掉当前队列处理比较久的连接，针对业务情况选择开启
option http-keep-alive #开启与客户端的会话保持
option forwardfor     #透传客户端真实IP至后端web服务器
mode http|tcp #设置默认工作类型,使用TCP服务器性能更好，减少压力
timeout http-keep-alive 120s #session 会话保持超时时间，此时间段内会转发到相同的后端服务器
timeout connect 120s #客户端请求从haproxy到后端server最长连接等待时间(TCP连接之前)，默认单位ms
timeout server 600s #客户端请求从haproxy到后端服务端的请求处理超时时长(TCP连接之后)，默认单位ms，如果超时，会出现502错误，此值建议设置较大些，防止出现502错误
timeout client 600s #设置haproxy与客户端的最长非活动时间，默认单位ms，建议和timeout server相同
timeout check   5s   #对后端服务器的默认检测超时时间
default-server inter 1000 weight 3   #指定后端服务器的默认设置
```

### 3.2.2  **Proxies**配置listen

使用listen替换 frontend和backend的配置方式，可以简化设置，常用于TCP协议的应用

```
#官网业务访问入口
listen WEB_PORT_80 
   bind 192.168.10.100:80  
   mode http
   option forwardfor
   server web1   10.0.0.17:8080   check inter 3000 fall 3 rise 5
   server web2   10.0.0.27:8080   check inter 3000 fall 3 rise 5
    
#检查语法
[root@ubuntu2004 ~]#haproxy -c -f /etc/haproxy/haproxy.cfg
Configuration file is valid
```

**范例：实现简单的服务代理**

```sh
#代理服务器配置
[root@ubuntu2004 ~]# vim /etc/haproxy/haproxy.cfg 
......

listen nginx-web
        bind 0.0.0.0:80
        mode http
        server web01 11.0.1.102:80
        server web02 11.0.1.103:80

......


#准备后端服务器
[root@web01 ~]#apt update && apt install -y nginx
[root@web01~]#echo `hostname -I` > /var/www/html/index.nginx-debian.html 

[root@web02 ~]#apt update && apt install -y nginx
[root@web02~]#echo `hostname -I` > /var/www/html/index.nginx-debian.html

#访问代理服务器
[root@centos7 ~]# curl 11.0.1.101
11.0.1.103
[root@centos7 ~]# curl 11.0.1.101
11.0.1.102
[root@centos7 ~]# curl 11.0.1.101
11.0.1.103
[root@centos7 ~]# curl 11.0.1.101
11.0.1.102
```

注：前端用户无法知晓后端真实服务器，代理服务器同时和前端用户和后端服务器建立连接

```sh
#用户模拟访问
[root@centos7 ~]# wget --limit-rate=1k http://11.0.1.101/m.img
--2024-05-09 14:23:48--  http://11.0.1.101/m.img
Connecting to 11.0.1.101:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 104857600 (100M) [application/octet-stream]
Saving to: ‘m.img’

 0% [                                                              ] 206,984     1.00KB/s  eta 28h 23m

#查看连接状况
[root@ubuntu2004 ~]# ss -nt
State            Recv-Q             Send-Q                             Local Address:Port                               Peer Address:Port            Process           
ESTAB            0                  25920                                 11.0.1.101:80                                     11.0.1.7:34002                             
ESTAB            0                  0                                     11.0.1.101:22                                    11.0.1.99:63374                             
ESTAB            0                  36                                    11.0.1.101:22                                    11.0.1.99:63372                             
ESTAB            1517472            0                                     11.0.1.101:49740                                11.0.1.102:80                                
ESTAB            0                  0                                     11.0.1.101:35856                                11.0.1.101:2377                              
ESTAB            0                  0                            [::ffff:11.0.1.101]:2377                        [::ffff:11.0.1.101]:35856  
```

### 3.2.3 Proxies配置frondend

**frontend** **配置参数：**

```
bind： #指定HAProxy的监听地址，可以是IPV4或IPV6，可以同时监听多个IP或端口，可同时用于listen字段中
#格式：
bind [<address>]:<port_range> [, ...] [param*]
#注意：如果需要绑定在非本机的IP，需要开启内核参数：net.ipv4.ip_nonlocal_bind=1
backlog <backlog> #针对所有server配置,当前端服务器的连接数达到上限后的后援队列长度，注意：不支持backend
```

范例：

```
frontend http_proxy #监听http的多个IP的多个端口和sock文件
   bind :80,:443,:8801-8810
   bind 10.0.0.1:10080,10.0.0.1:10443
   bind /var/run/ssl-frontend.sock user root mode 600 accept-proxy
frontend http_https_proxy #https监听
   bind :80
   bind :443 ssl crt /etc/haproxy/site.pem #公钥和私钥公共文件
frontend http_https_proxy_explicit #监听ipv6、ipv4和unix sock文件
   bind ipv6@:80
   bind ipv4@public_ssl:443 ssl crt /etc/haproxy/site.pem
   bind unix@ssl-frontend.sock user root mode 600 accept-proxy
listen external_bind_app1 #监听file descriptor
   bind "fd@${FD_APP1}"
```

```
frontend dinginx_web   #建议采用后面形式命名：业务-服务-端口号
   bind :80,:8080
   bind 11.0.1.101:10080,:8801-8810,11.0.1.11:9001-9010
   mode http|tcp     #指定负载协议类型
   use_backend <backend_name>  #调用的后端服务器组名称
```

### 3.2.4 Proxies配置backend

定义一组后端服务器，backend服务器将被frontend进行调用。

注意: backend 的名称必须唯一,并且必须在listen或frontend中事先定义才可以使用,否则服务无法启动

```
mode http|tcp     #指定负载协议类型,和对应的frontend必须一致
option #配置选项
server   #定义后端real server,必须指定IP和端口
```

注意：option后面加 httpchk，smtpchk,mysql-check,pgsql-check，ssl-hello-chk方法，可用于实现更多应用层检测功能。

**server** **配置**

```sh
#针对一个server配置
check #对指定real进行健康状态检查，如果不加此设置，默认不开启检查,只有check后面没有其它配置也可以启用检查功能
 	  #默认对相应的后端服务器IP和端口,利用TCP连接进行周期性健康性检查,注意必须指定端口才能实现健康性检查
 addr <IP>   #可指定的健康状态监测IP，可以是专门的数据网段，减少业务网络的流量
 port <num> #指定的健康状态监测端口
 inter <num> #健康状态检查间隔时间，默认2000 ms
 fall <num>   #后端服务器从线上转为线下的检查的连续失效次数，默认为3
 rise <num>   #后端服务器从下线恢复上线的检查的连续有效次数，默认为2
weight <weight> #默认为1，最大值为256，0(状态为蓝色)表示不参与负载均衡，但仍接受持久连接
backup #将后端服务器标记为备份状态,只在所有非备份主机down机时提供服务，类似Sorry Server
disabled #将后端服务器标记为不可用状态，即维护状态，除了持久模式，将不再接受连接,状态为深黄色,支持优雅平滑下线,继续处理旧请求，不接受新请求
maxconn <maxconn> #当前后端server的最大并发连接数,放在,放在server 指令后面
redir http://www.baidu.com       #将请求临时(302)重定向至其它URL，只适用于http模式,放在server 指令后面
```

**redirect 配置**

```sh
#注意:此指令和redir功能相似,但不属于server指令后面,是独立存放在listen,frontend,backend语句块
redirect prefix http://www.baidu.com/ #将请求临时(302)重定向至其它URL，只适用于http模式
```

### **3.2.5 frontend和backend** **组合配置案例**

**范例：实现7层http服务nginx代理**

```sh
#代理服务器配置
[root@ubuntu2004 ~]# vim /etc/haproxy/haproxy.cfg 
......

frontend www.dingbh.top_80
        bind 192.168.188.101:80
        use_backend web_servers

backend web_servers
        server web01 11.0.1.102:80
        server web02 11.0.1.103:80

......

[root@ubuntu2004 ~]# haproxy -c -f /etc/haproxy/haproxy.cfg 
Configuration file is valid
[root@ubuntu2004 ~]# systemctl restart haproxy.service 
```

**范例：实现4层MySQL服务代理**

```sh
#代理服务器配置
[root@ubuntu2004 ~]# vim /etc/haproxy/haproxy.cfg 

......

frontend  mysql_access
        bind 192.168.188.101:3306
        mode tcp
        use_backend mysql_server
backend mysql_server
        mode tcp
        server mysql01 11.0.1.102:3306
        server mysql02 11.0.1.103:3306
        
......

[root@ubuntu2004 ~]# haproxy -c -f /etc/haproxy/haproxy.cfg   #有警告可忽略
[NOTICE]   (40059) : haproxy version is 2.8.9-1842fd0
[NOTICE]   (40059) : path to executable is /usr/sbin/haproxy
[WARNING]  (40059) : config : 'option forwardfor' ignored for frontend 'mysql_access' as it requires HTTP mode.
[WARNING]  (40059) : config : 'option forwardfor' ignored for backend 'mysql_server' as it requires HTTP mode.
Warnings were found.
Configuration file is valid
[root@ubuntu2004 ~]# systemctl restart haproxy.service 

#测试
[root@intnet ~]# mysql -uweb -p123456 -h 192.168.188.101 -e "select @@server_id"
mysql: [Warning] Using a password on the command line interface can be insecure.
+-------------+
| @@server_id |
+-------------+
|         102 |
+-------------+
[root@intnet ~]# mysql -uweb -p123456 -h 192.168.188.101 -e "select @@server_id"
mysql: [Warning] Using a password on the command line interface can be insecure.
+-------------+
| @@server_id |
+-------------+
|         103 |
+-------------+
```







```
11.0.1.102
11.0.1.103
11.0.1.102
11.0.1.102
<html><body><h1>503 Service Unavailable</h1>
No server is available to handle this request.
</body></html>
11.0.1.103
11.0.1.103
11.0.1.103
11.0.1.103
11.0.1.103
11.0.1.103
11.0.1.103
11.0.1.103
11.0.1.103
11.0.1.103
11.0.1.103
11.0.1.103
<html><body><h1>503 Service Unavailable</h1>
No server is available to handle this request.
</body></html>
Sorry server
Sorry server
Sorry server
Sorry server
Sorry server
Sorry server
Sorry server

```

## 3.3 使用子配置文件

当业务众多时，将所有配置都放在一个配置文件中，会造成维护困难。

可以考虑按业务分类，将配置信息拆分，放在不同的子配置文件中，从而达到方便维护的目的。

**注意: 子配置文件的文件后缀必须为.cfg**

```sh
#主配置文件
[root@ubuntu2004 ~]# cat /etc/haproxy/haproxy.cfg 
global
maxconn 100000
stats socket /var/lib/haproxy/haproxy.sock mode 600 level admin
uid 99
gid 99
daemon
nbthread 8
cpu-map auto:1/1-8 0-7

#pidfile /var/lib/haproxy/haproxy.pid
log 127.0.0.1 local2 info
log 11.0.1.102 local2 info
spread-checks 3

defaults
option http-keep-alive
option forwardfor
maxconn 100000
mode http
timeout connect 300000ms
timeout client 300000ms
timeout server 300000ms

listen stats
    bind 0.0.0.0:9999
    stats enable
    log global
    stats uri /haproxy-status
    stats auth admin:123456

#listen nginx-web
#	bind 192.168.188.101:80
#	server web01 11.0.1.102:80
#	server web02 11.0.1.103:80

#listen kubernetes-6443
#    bind 11.0.1.100:6443
#    mode tcp
#    log global
#    server 11.0.1.101 11.0.1.101:6443 check inter 3000 fall 2 rise 5
#    server 11.0.1.102 11.0.1.102:6443 check inter 3000 fall 2 rise 5
#    server 11.0.1.103 11.0.1.102:6443 check inter 3000 fall 2 rise 5
frontend www.dingbh.top_80
	bind 192.168.188.101:80
	use_backend web_servers

backend web_servers
#	redirect prefix http://www.baidu.com/
	server web01 11.0.1.102:80 check weight 3 
	server web02 11.0.1.103:80 check inter 3000 fall 3 rise 2 weight 3
	server backup 127.0.0.1:8080 check backup

#listen mysql_server
#	bind 192.168.188.101:3306
#	mode tcp
#	server mysql02 11.0.1.103:3306
#	server mysql01 11.0.1.102:3306

#frontend  mysql_access
#	bind 192.168.188.101:3306
#	mode tcp
#	use_backend mysql_server
#backend mysql_server
#	mode tcp
#	server mysql01 11.0.1.102:3306 check
#	server mysql02 11.0.1.103:3306 check

#创建子配置文件
[root@ubuntu2004 ~]# vim /etc/haproxy/conf.d/mysql.cfg 

frontend  mysql_access
        bind 192.168.188.101:3306
        mode tcp
        use_backend mysql_server
backend mysql_server
        mode tcp
        server mysql01 11.0.1.102:3306 check
        server mysql02 11.0.1.103:3306 check backup
        
#修改启动文件，添加对配置文件的检查  -f /etc/haproxy/haproxy.cfg    
[root@ubuntu2004 ~]# vim /lib/systemd/system/haproxy.service

[Unit]
Description=HAProxy Load Balancer
After=syslog.target network.target

[Service]
ExecStartPre=/usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg  -f /etc/haproxy/conf.d/ -c -q
ExecStart=/usr/sbin/haproxy -Ws -f /etc/haproxy/haproxy.cfg -f /etc/haproxy/conf.d/ -p /var/lib/haproxy/haproxy.pid
ExecReload=/bin/kill -USR2 $MAINPID

[Install]
WantedBy=multi-user.target

[root@ubuntu2004 ~]# systemctl daemon-reload
[root@ubuntu2004 ~]# systemctl restart haproxy.service 
[root@ubuntu2004 ~]# 

#测试
[root@internet ~]# while true;do mysql -h192.168.188.101 -uweb -p123456 -e 'select @@server_id;' && sleep 2;done
```

# 4 HAProxy 调度算法

HAProxy通过固定参数 balance 指明对后端服务器的调度算法，该参数可以配置在listen或backend选项中。

HAProxy的调度算法分为静态和动态调度算法，但是有些算法可以根据不同的参数实现静态和动态算法相互转换。

官方文档：http://cbonte.github.io/haproxy-dconv/2.4/configuration.html#4-balance

## 4.1 Socat 工具

对服务器动态权重和其它状态可以利用 socat 工具进行调整，Socat 是 Linux 下的一个多功能的网络工具，名字来由是Socket CAT，相当于netCAT的增强版.Socat 的主要特点就是在两个数据流之间建立双向通道，且支持众多协议和链接方式。如 IP、TCP、 UDP、IPv6、Socket文件等

**范例：利用工具socat 对服务器动态权重调整**

```sh
#安装socat
[root@ubuntu2004 ~]#apt update && apt -y install socat
[root@centos ~]#yum -y install socat

#查看帮助，stdio表示输入
[root@ubuntu2004 ~]# socat -h
[root@ubuntu2004 ~]# echo "help" |socat stdio /var/lib/haproxy/haproxy.sock
The following commands are valid at this level:
  abort ssl ca-file <cafile>              : abort a transaction for a CA file
  abort ssl cert <certfile>               : abort a transaction for a certificate file
  abort ssl crl-file <crlfile>            : abort a transaction for a CRL file
  add acl [@<ver>] <acl> <pattern>        : add an acl entry
  add map [@<ver>] <map> <key> <val>      : add a map entry (payload supported instead of key/val)
  add server <bk>/<srv>                   : create a new server
  add ssl ca-file <cafile> <payload>      : add a certificate into the CA file
  add ssl crt-list <list> <cert> [opts]*  : add to crt-list file <list> a line <cert> or a payload
  clear acl [@<ver>] <acl>                : clear the contents of this acl
  clear counters [all]                    : clear max statistics counters (or all counters)
  clear map [@<ver>] <map>                : clear the contents of this map
  clear table <table> [<filter>]*         : remove an entry from a table (filter: data/key)
  commit acl @<ver> <acl>                 : commit the ACL at this version
  commit map @<ver> <map>                 : commit the map at this version
  commit ssl ca-file <cafile>             : commit a CA file
  commit ssl cert <certfile>              : commit a certificate file
  commit ssl crl-file <crlfile>           : commit a CRL file
  debug dev hash   [msg]                  : return msg hashed if anon is set
  del acl <acl> [<key>|#<ref>]            : delete acl entries matching <key>
  del map <map> [<key>|#<ref>]            : delete map entries matching <key>
  del server <bk>/<srv>                   : remove a dynamically added server
  del ssl ca-file <cafile>                : delete an unused CA file
  del ssl cert <certfile>                 : delete an unused certificate file
  del ssl crl-file <crlfile>              : delete an unused CRL file
  del ssl crt-list <list> <cert[:line]>   : delete a line <cert> from crt-list file <list>
  disable agent                           : disable agent checks
  disable dynamic-cookie backend <bk>     : disable dynamic cookies on a specific backend
  disable frontend <frontend>             : temporarily disable specific frontend
  disable health                          : disable health checks
  disable server (DEPRECATED)             : disable a server for maintenance (use 'set server' instead)
  enable agent                            : enable agent checks
  enable dynamic-cookie backend <bk>      : enable dynamic cookies on a specific backend
  enable frontend <frontend>              : re-enable specific frontend
  enable health                           : enable health checks
  enable server  (DEPRECATED)             : enable a disabled server (use 'set server' instead)
  get acl <acl> <value>                   : report the patterns matching a sample for an ACL
  get map <acl> <value>                   : report the keys and values matching a sample for a map
  get var <name>                          : retrieve contents of a process-wide variable
  get weight <bk>/<srv>                   : report a server's current weight
  new ssl ca-file <cafile>                : create a new CA file to be used in a crt-list
  new ssl cert <certfile>                 : create a new certificate file to be used in a crt-list or a directory
  new ssl crl-file <crlfile>               : create a new CRL file to be used in a crt-list
  operator                                : lower the level of the current CLI session to operator
  prepare acl <acl>                       : prepare a new version for atomic ACL replacement
  prepare map <acl>                       : prepare a new version for atomic map replacement
  set anon global-key <value>             : change the global anonymizing key
  set anon off                            : deactivate the anonymized mode
  set anon on [value]                     : activate the anonymized mode
  set dynamic-cookie-key backend <bk> <k> : change a backend secret key for dynamic cookies
  set map <map> [<key>|#<ref>] <value>    : modify a map entry
  set maxconn frontend <frontend> <value> : change a frontend's maxconn setting
  set maxconn global <value>              : change the per-process maxconn setting
  set maxconn server <bk>/<srv>           : change a server's maxconn setting
  set profiling <what> {auto|on|off}      : enable/disable resource profiling (tasks,memory)
  set rate-limit <setting> <value>        : change a rate limiting value
  set server <bk>/<srv> [opts]            : change a server's state, weight, address or ssl
  set severity-output [none|number|string]: set presence of severity level in feedback information
  set ssl ca-file <cafile> <payload>      : replace a CA file
  set ssl cert <certfile> <payload>       : replace a certificate file
  set ssl crl-file <crlfile> <payload>    : replace a CRL file
  set ssl ocsp-response <resp|payload>       : update a certificate's OCSP Response from a base64-encode DER
  set ssl tls-key [id|file] <key>         : set the next TLS key for the <id> or <file> listener to <key>
  set table <table> key <k> [data.* <v>]* : update or create a table entry's data
  set timeout [cli] <delay>               : change a timeout setting
  set weight <bk>/<srv>  (DEPRECATED)     : change a server's weight (use 'set server' instead)
  show acl [@<ver>] <acl>]                : report available acls or dump an acl's contents
  show activity [-1|0|thread_num]         : show per-thread activity stats (for support/developers)
  show anon                               : display the current state of anonymized mode
  show backend                            : list backends in the current running config
  show cache                              : show cache status
  show cli level                          : display the level of the current CLI session
  show cli sockets                        : dump list of cli sockets
  show env [var]                          : dump environment variables known to the process
  show errors [<px>] [request|response]   : report last request and/or response errors for each proxy
  show events [<sink>] [-w] [-n]          : show event sink state
  show fd [-!plcfbsd]* [num]              : dump list of file descriptors in use or a specific one
  show info [desc|json|typed|float]*      : report information about the running process
  show libs                               : show loaded object files and libraries
  show map [@ver] [map]                   : report available maps or dump a map's contents
  show peers [dict|-] [section]           : dump some information about all the peers or this peers section
  show pools [by*] [match <pfx>] [nb]     : report information about the memory pools usage
  show profiling [<what>|<#lines>|<opts>]*: show profiling state (all,status,tasks,memory)
  show resolvers [id]                     : dumps counters from all resolvers section and associated name servers
  show schema json                        : report schema used for stats
  show servers conn [<backend>]           : dump server connections status (all or for a single backend)
  show servers state [<backend>]          : dump volatile server information (all or for a single backend)
  show sess [id]                          : report the list of current sessions or dump this exact session
  show ssl ca-file [<cafile>[:<index>]]   : display the SSL CA files used in memory, or the details of a <cafile>, or a single certificate of index <index> of a CA file <cafile>
  show ssl cert [<certfile>]              : display the SSL certificates used in memory, or the details of a file
  show ssl crl-file [<crlfile[:<index>>]] : display the SSL CRL files used in memory, or the details of a <crlfile>, or a single CRL of index <index> of CRL file <crlfile>
  show ssl crt-list [-n] [<list>]         : show the list of crt-lists or the content of a crt-list file <list>
  show ssl ocsp-response [[text|base64] id]  : display the IDs of the OCSP responses used in memory, or the details of a single OCSP response (in text or base64 format)
  show ssl ocsp-updates                      : display information about the next 'nb' ocsp responses that will be updated automatically
  show startup-logs                       : report logs emitted during HAProxy startup
  show stat [desc|json|no-maint|typed|up]*: report counters for each proxy and server
  show table <table> [<filter>]*          : report table usage stats or dump this table's contents (filter: data/key)
  show tasks                              : show running tasks
  show threads                            : show some threads debugging information
  show tls-keys [id|*]                    : show tls keys references or dump tls ticket keys when id specified
  show trace [<module>]                   : show live tracing state
  show version                            : show version of the current process
  shutdown frontend <frontend>            : stop a specific frontend
  shutdown session [id]                   : kill a specific session
  shutdown sessions server <bk>/<srv>     : kill sessions on a server
  trace [<module>|0] [cmd [args...]]      : manage live tracing (empty to list, 0 to stop all)
  update ssl ocsp-response <certfile>     : send ocsp request and update stored ocsp response
  user                                    : lower the level of the current CLI session to user
  help [<command>]                        : list matching or all commands
  prompt [timed]                          : toggle interactive mode with prompt
  quit                                    : disconnect

#查看状态信息，实现后期zabbix监控
[root@ubuntu2004 ~]# echo "show info" |socat stdio /var/lib/haproxy/haproxy.sock 
Name: HAProxy
Version: 2.8.9-1842fd0
Release_date: 2024/04/05
Nbthread: 8
Nbproc: 1
Process_num: 1
Pid: 135877
Uptime: 0d 5h46m03s
Uptime_sec: 20763
Memmax_MB: 0
PoolAlloc_MB: 0
PoolUsed_MB: 0
PoolFailed: 0
Ulimit-n: 200054
Maxsock: 200054
Maxconn: 100000
Hard_maxconn: 100000
CurrConns: 0
CumConns: 48352
CumReq: 68
MaxSslConns: 0
CurrSslConns: 0
CumSslConns: 0
Maxpipes: 0
PipesUsed: 0
PipesFree: 0
ConnRate: 0
ConnRateLimit: 0
MaxConnRate: 2
SessRate: 0
SessRateLimit: 0
MaxSessRate: 2
SslRate: 0
SslRateLimit: 0
MaxSslRate: 0
SslFrontendKeyRate: 0
SslFrontendMaxKeyRate: 0
SslFrontendSessionReuse_pct: 0
SslBackendKeyRate: 0
SslBackendMaxKeyRate: 0
SslCacheLookups: 0
SslCacheMisses: 0
CompressBpsIn: 0
CompressBpsOut: 0
CompressBpsRateLim: 0
ZlibMemUsage: 0
MaxZlibMemUsage: 0
Tasks: 34
Run_queue: 0
Idle_pct: 100
node: ubuntu2004
Stopping: 0
Jobs: 6
Unstoppable Jobs: 1
Listeners: 5
ActivePeers: 0
ConnectedPeers: 0
DroppedLogs: 0
BusyPolling: 0
FailedResolutions: 0
TotalBytesOut: 1280176
TotalSplicedBytesOut: 0
BytesOutRate: 0
DebugCommandsIssued: 0
CumRecvLogs: 0
Build info: 2.8.9-1842fd0
Memmax_bytes: 0
PoolAlloc_bytes: 480272
PoolUsed_bytes: 480272
Start_time_sec: 1716447931
Tainted: 0
TotalWarnings: 66
MaxconnReached: 0
BootTime_ms: 14

#查看后端服务器数
[root@ubuntu2004 ~]# echo "show backend" |socat stdio /var/lib/haproxy/haproxy.sock 
# name
stats
web_servers
mysql_server

#查看服务器信息
[root@ubuntu2004 ~]# echo "show servers state" |socat stdio /var/lib/haproxy/haproxy.sock 
1
# be_id be_name srv_id srv_name srv_addr srv_op_state srv_admin_state srv_uweight srv_iweight srv_time_since_last_change srv_check_status srv_check_result srv_check_health srv_check_state srv_agent_state bk_f_forced_id srv_f_forced_id srv_fqdn srv_port srvrecord srv_use_ssl srv_check_port srv_check_addr srv_agent_addr srv_agent_port
4 web_servers 1 web01 11.0.1.102 2 0 3 3 2844 6 3 4 6 0 0 0 - 80 - 0 0 - - 0
4 web_servers 2 web02 11.0.1.103 2 0 3 3 8836 6 3 4 6 0 0 0 - 80 - 0 0 - - 0
4 web_servers 3 backup 127.0.0.1 2 0 1 1 20888 6 3 4 6 0 0 0 - 8080 - 0 0 - - 0
6 mysql_server 1 mysql01 11.0.1.102 2 0 1 1 2844 6 3 4 6 0 0 0 - 3306 - 0 0 - - 0
6 mysql_server 2 mysql02 11.0.1.103 2 0 1 1 2843 6 3 4 6 0 0 0 - 3306 - 0 0 - - 0


#查看权重信息
[root@ubuntu2004 ~]# echo "get weight mysql_server/mysql01" |socat stdio /var/lib/haproxy/haproxy.sock 
1 (initial 1)

[root@ubuntu2004 ~]# echo "get weight mysql_server/mysql02" |socat stdio /var/lib/haproxy/haproxy.sock 
1 (initial 1)
#修改权重信息
[root@ubuntu2004 ~]# echo "set weight web_servers/web02 3" |socat stdio /var/lib/haproxy/haproxy.sock 


#获取当前连接数
[root@haproxy ~]#echo "show info" | socat stdio /var/lib/haproxy/haproxy.sock | 
awk '/CurrConns/{print $2}'


#实现服务的优雅上线和下线
[root@ubuntu2004 ~]# echo "disable server web_servers/web01" |socat stdio /var/lib/haproxy/haproxy.sock

[root@ubuntu2004 ~]# echo "enable  server web_servers/web02" |socat stdio /var/lib/haproxy/haproxy.sock

```

**范例：上线和下线后端服务器脚本**

```sh
#!/bin/bash
#
#********************************************************************
#Author:           dingbaohang
#QQ:               904748581
#Date:             2024-05-11
#FileName:         host_up_down.sh
#URL:              http://www.dingbh.top
#Description:      The test script
#Copyright (C):    2024 All rights reserved
#********************************************************************
WEB_SERVERS="
11.0.1.202
11.0.1.203
"
BACKEND=web_servers
APP=nginx
IMAGE=$APP:1.20
DELAY=10
for i in $WEB_SERVERS;do
    echo "set server  $BACKEND/$i state maint" | socat stdio 
/var/lib/haproxy/haproxy.sock
    ssh $i docker rm -f $APP
    ssh $i "echo DOCKER $i WEBSITE $1 v1.0 >   /data/www/index.html"
    ssh $i docker run -d -p 80:80 -v /data/www:/usr/share/nginx/html --name $APP
$IMAGE
    sleep $DELAY
    echo "set server $BACKEND/$i state ready" | socat stdio 
/var/lib/haproxy/haproxy.sock
done


#服务上下线实现过程
[root@ubuntu2004 ~]# echo "set server web_servers/11.0.1.102 state maint"|socat stdio /var/lib/haproxy/haproxy.sock

[root@ubuntu2004 ~]# ssh 11.0.1.102 docker rm -f nginx
[root@ubuntu2004 ~]# ssh 11.0.1.102 "echo DOCKER 11.0.1.102 WEBSITE  v2.0 > /data/www/index.html"
[root@ubuntu2004 ~]# ssh 11.0.1.102 docker run -d -p 80:80 -v /data/www/:/usr/share/nginx/html  --name nginx nginx:1.20
0b66f759cc5a34524a2ac80033642d3880278a0b85aa80ba37afac640b2a8e52

[root@ubuntu2004 ~]# echo "set server web_servers/11.0.1.102 state ready"|socat stdio /var/lib/haproxy/haproxy.sock


[root@ubuntu2004 ~]# echo "set server web_servers/11.0.1.103 state maint"|socat stdio /var/lib/haproxy/haproxy.sock

#滚动升级web02
[root@ubuntu2004 ~]# ssh 11.0.1.103 docker rm -f nginx
nginx
[root@ubuntu2004 ~]# ssh 11.0.1.103 "echo DOCKER 11.0.1.103 WEBSITE  v2.0 > /data/www/index.html"
[root@ubuntu2004 ~]# ssh 11.0.1.103 docker run -d -p 80:80 -v /data/www/:/usr/share/nginx/html  --name nginx nginx:1.20
fe8c07c5300af83f967ab921935bfd4cc5bb74eea05df5441eca85cbf5f1309c
[root@ubuntu2004 ~]# echo "set server web_servers/11.0.1.103 state ready"|socat stdio /var/lib/haproxy/haproxy.sock

```

## 4.2 静态算法

静态算法：按照事先定义好的规则轮询进行调度，不关心后端服务器的当前负载、连接数和响应速度等，且无法实时动态修改权重(只能为0和1,不支持其它值)或者修改后不生效，如果需要修改只能靠重启HAProxy生效。

### 5.2.1 static-rr算法

static-rr：基于权重的轮询调度，不支持运行时利用socat进行权重的动态调整(只支持0和1,不支持其它值)及后端服务器慢启动，其后端主机数量没有限制，相当于LVS中的 wrr

```sh
frontend www.dingbh.top_80
        bind 192.168.188.101:80
        use_backend web_servers

backend web_servers
        balance static-rr
        server 11.0.1.102 11.0.1.102:80 check inter 3000 fall 3 rise 2 weight 3
        server 11.0.1.103 11.0.1.103:80 check inter 3000 fall 3 rise 2
        server backup 127.0.0.1:8080 check backup
```



范例：调整权重

```
[root@ubuntu2004 ~]# echo "get weight web_servers/11.0.1.102"|socat stdio /var/lib/haproxy/haproxy.sock
1 (initial 3)

[root@ubuntu2004 ~]# echo "get weight web_servers/11.0.1.103"|socat stdio /var/lib/haproxy/haproxy.sock
1 (initial 1)

[root@ubuntu2004 ~]# echo "set weight web_servers/11.0.1.103 40"|socat stdio /var/lib/haproxy/haproxy.sock
Backend is using a static LB algorithm and only accepts weights '0%' and '100%'.

#只支持0或100%

```

### 4.2.2 first 算法

first：根据服务器在列表中的位置，自上而下进行调度，但是其只会当第一台服务器的连接数达到上限，新请求才会分配给下一台服务，因此会忽略服务器的权重设置，此方式使用较少

不支持用socat进行动态修改权重,可以设置0和1,可以设置其它值但无效

```sh
frontend www.dingbh.top_80
        bind 192.168.188.101:80
        use_backend web_servers

backend web_servers
#       redirect prefix http://www.baidu.com/
#       balance static-rr
        balance first
        server 11.0.1.102 11.0.1.102:80 check inter 3000 fall 3 rise 2 weight 3
        server 11.0.1.103 11.0.1.103:80 check inter 3000 fall 3 rise 2
        server backup 127.0.0.1:8080 check backup
```

**测试访问效果**

```sh
#同时运行下面命令，观察结果
#在后端服务器上限速
server {
   ....
   limit_rate 10;
   ....
}
    
#用wget下载文件测试才能看到效果
wget --limit-rate 1k http://192.168.10.100/test.img
#curl测试不成功

# while true ; do curl 192.168.188.101 && sleep 1;done
#动态修改权重，不报错，但不生效
[root@ubuntu2004 ~]# echo "set weight web_servers/11.0.1.103 60"|socat stdio /var/lib/haproxy/haproxy.sock
```

## 4.3 动态算法

动态算法：基于后端服务器状态进行调度适当调整，新请求将优先调度至当前负载较低的服务器，且权重可以在haproxy运行时动态调整无需重启。

### **4.3.1 roundrobin 算法**

roundrobin：基于权重的轮询动态调度算法，支持权重的运行时调整，不同于lvs中的rr轮训模式，HAProxy中的roundrobin支持慢启动(新加的服务器会逐渐增加转发数)，其每个后端backend中最多支持4095个real server，支持对real server权重动态调整，**roundrobin为默认调度算法**,此算法使用广泛

```
[root@ubuntu2004 ~]# vim /etc/haproxy/conf.d/web.cfg

frontend www.dingbh.top_80
        bind 192.168.188.101:80
        use_backend web_servers

backend web_servers
#静态算法
#       balance static-rr
#       balance first
#动态算法
        balance roundrobin
        server 11.0.1.102 11.0.1.102:80 check inter 3000 fall 3 rise 2 weight 3
        server 11.0.1.103 11.0.1.103:80 check inter 3000 fall 3 rise 2
        server backup 127.0.0.1:8080 check backup
```

**支持动态调整权重：**

```sh
[root@ubuntu2004 ~]# echo "get weight web_servers/11.0.1.102"|socat stdio /var/lib/haproxy/haproxy.sock
3 (initial 3)

[root@ubuntu2004 ~]# echo "set weight web_servers/11.0.1.102 2"|socat stdio /var/lib/haproxy/haproxy.sock

[root@ubuntu2004 ~]# echo "get weight web_servers/11.0.1.102"|socat stdio /var/lib/haproxy/haproxy.sock
2 (initial 3)
```

### 4.3.2 leastconn 算法

leastconn 加权的最少连接的动态，支持权重的运行时调整和慢启动，即根据当前连接最少的后端服务器而非权重进行优先调度(新客户端连接)，比较适合长连接的场景使用，比如：MySQL等场景。

相当于LVS中的WLC算法

```sh
frontend www.dingbh.top_80
        bind 192.168.188.101:80
        use_backend web_servers

backend web_servers
#静态算法
#       balance static-rr
#       balance first
#动态算法
#       balance roundrobin
        balance leastconn
        server 11.0.1.102 11.0.1.102:80 check inter 3000 fall 3 rise 2 weight 3
        server 11.0.1.103 11.0.1.103:80 check inter 3000 fall 3 rise 2
        server backup 127.0.0.1:8080 check backup
```

### **4.3.3 random** **算法**

在1.9版本开始增加 random的负载平衡算法，其基于随机数作为一致性hash的key，随机负载平衡对于大型服务器场或经常添加或删除服务器非常有用，**支持weight的动态调整**，weight较大的主机有更大概率获取新请求

**random配置实例**

```sh
frontend www.dingbh.top_80
        bind 192.168.188.101:80
        use_backend web_servers

backend web_servers
#静态算法
#       balance static-rr
#       balance first
#动态算法
#       balance roundrobin
#       balance leastconn
        balance random
        server 11.0.1.102 11.0.1.102:80 check inter 3000 fall 3 rise 2 weight 3
        server 11.0.1.103 11.0.1.103:80 check inter 3000 fall 3 rise 2
        server backup 127.0.0.1:8080 check backup
```

**测试访问**

```sh
[root@internet ~]# while true ; do curl 192.168.188.101 && sleep 1;done

......

DOCKER 11.0.1.102 WEBSITE v2.0
DOCKER 11.0.1.103 WEBSITE v2.0
DOCKER 11.0.1.103 WEBSITE v2.0
DOCKER 11.0.1.102 WEBSITE v2.0
DOCKER 11.0.1.102 WEBSITE v2.0
DOCKER 11.0.1.103 WEBSITE v2.0
DOCKER 11.0.1.102 WEBSITE v2.0
DOCKER 11.0.1.102 WEBSITE v2.0
DOCKER 11.0.1.103 WEBSITE v2.0
DOCKER 11.0.1.102 WEBSITE v2.0
```

## 4.4 其他算法

其它算法即可作为静态算法，又可以通过选项成为动态算法

### 4.4.1 source 算法

源地址hash，基于用户源地址hash并将请求转发到后端服务器，后续同一个源地址请求将被转发至同一个后端web服务器。此方式当后端服务器数据量发生变化时，会导致很多用户的请求转发至新的后端服务器，默认为静态方式，但是可以通过hash-type选项进行更改。

这个算法一般是在不插入Cookie的TCP模式下使用，也可给不支持会话cookie的客户提供会话粘性，适用于需要session会话保持但不支持cookie和缓存的场景

源地址有两种转发客户端请求到后端服务器的服务器选取计算方式，分别是 **取模法和一致性hash**



#### **4.4.1.1 map-base 取模法**

map-based：取模法，对source地址进行hash计算，再基于服务器总权重的取模，最终结果决定将此请求转发至对应的后端服务器。

此方法是静态的，即不支持在线调整权重，不支持慢启动，可实现对后端服务器均衡调度。缺点是当服务器的总权重发生变化时，即有服务器上线或下线，都会因总权重发生变化而导致调度结果整体改变，hash-type 指定的默认值为此算法.

```
所谓取模运算，就是计算两个数相除之后的余数，10%7=3, 7%4=3
map-based算法：基于权重取模，hash(source_ip)%所有后端服务器相加的总权重
```

#### **4.4.1.2** **一致性** **hash**

一致性哈希，当服务器的总权重发生变化时，对调度结果影响是局部的，不会引起大的变动，hash（o）mod n ，该hash算法是动态的，支持使用 socat等工具进行在线权重调整，支持慢启动算法：

```
1、key1=hash(source_ip)%(2^32) [0---4294967295]
2、keyA=hash(后端服务器虚拟ip)%(2^32)
3、将key1和keyA都放在hash环上，将用户请求调度到离key1最近的keyA对应的后端服务器
```

hash环偏斜问题

```sh
增加虚拟服务器IP数量，比如：一个后端服务器根据权重为1生成1000个虚拟IP，再hash。而后端服务器权重为2则生成2000的虚拟IP，再进行hash运算,最终在hash环上生成3000个节点，从而解决hash环偏斜问题
```



### **4.4.2 uri 算法**

基于对用户请求的URI的左半部分或整个uri做hash，再将hash结果对总权重进行取模后，根据最终结果将请求转发到后端指定服务器，适用于后端是缓存服务器场景，默认是静态算法，也可以通过hash-type指定map-based和consistent，来定义使用取模法还是一致性hash。

**注意：此算法基于应用层，所以只支持** **mode http** **，不支持** **mode tcp**

```sh
<scheme>://<user>:<password>@<host>:<port>/<path>;<params>?<query>#<frag>
左半部分：/<path>;<params>
整个uri：/<path>;<params>?<query>#<frag>
```

![image-20240527184033323](Haproxy/image-20240527184033323.png)





#### **4.4.2.1 uri** **取模法配置示例**

```
listen web_host
 bind 10.0.0.7:80,:8801-8810,10.0.0.7:9001-9010
 mode http
 log global
 balance uri
 server web1 10.0.0.17:80 weight 1 check inter 3000 fall 2 rise 5
 server web2 10.0.0.27:80 weight 1 check inter 3000 fall 2 rise 5
```

#### **4.4.2.2 uri** **一致性hash配置示例**

```sh
listen web_host
 bind 192.168.188.101:80,:8801-8810,192.168.188.101:9001-9010
 mode http
 log global
 balance uri
 hash-type consistent
 server web1 11.0.1.102:80 weight 1 check inter 3000 fall 2 rise 5
 server web2 11.0.1.103:80 weight 1 check inter 3000 fall 2 rise 5
```

#### **4.4.2.3** **访问测试**

```
[root@internet ~]# while true ; do curl 192.168.188.101/test{1..10}.html ;sleep 1;done
11.0.1.102 172.17.0.1 test1
11.0.1.102 172.17.0.1 test2
11.0.1.102 172.17.0.1 test3
11.0.1.102 172.17.0.1 test4
11.0.1.102 172.17.0.1 test5
11.0.1.102 172.17.0.1 test6
11.0.1.102 172.17.0.1 test7
11.0.1.102 172.17.0.1 test8
11.0.1.102 172.17.0.1 test9
11.0.1.102 172.17.0.1 test10
```

### 4.4.3 url_param 算法

url_param对用户请求的url中的 params 部分中的一个参数key对应的value值作hash计算，并由服务器总权重相除以后派发至某挑出的服务器；通常用于追踪用户，以确保来自同一个用户的请求始终发往同一个real server，如果无没key，将按roundrobin算法

```sh
#假设：
url = http://www.dingbh.com/foo/bar/index.php?key=value 
\#则：
host = "www.dingbh.com"
url_param = "key=value"
```



#### **4.4.3.1 url_param 取模法配置示例**

```sh
[root@ubuntu2004 ~]# cat /etc/haproxy/conf.d/web.cfg 
frontend www.dingbh.top_80
        bind 192.168.188.101:80
        use_backend web_servers

backend web_servers
#静态算法
#       redirect prefix http://www.baidu.com/
#       balance static-rr
#       balance first
#动态算法
#	balance roundrobin
#	balance leastconn
#	balance random
#其他
#	balance uri
#	balance source
#	hash-type map-based
	balance url_param userid
        server 11.0.1.102 11.0.1.102:80 check inter 3000 fall 3 rise 2 weight 3
        server 11.0.1.103 11.0.1.103:80 check inter 3000 fall 3 rise 2
        server backup 127.0.0.1:8080 check backup
```









**测试**

```
[root@internet ~]# while true ; do curl 192.168.188.101/test{1..10}.html?userid=a ;sleep 1;done
11.0.1.102 172.17.0.1 test1
11.0.1.102 172.17.0.1 test2
11.0.1.102 172.17.0.1 test3
11.0.1.102 172.17.0.1 test4
11.0.1.102 172.17.0.1 test5
11.0.1.102 172.17.0.1 test6
11.0.1.102 172.17.0.1 test7
11.0.1.102 172.17.0.1 test8
11.0.1.102 172.17.0.1 test9
11.0.1.102 172.17.0.1 test10

[root@internet ~]# while true ; do curl 192.168.188.101/test{1..10}.html?userid=b ;sleep 1;done
11.0.1.102 172.17.0.1 test1
11.0.1.102 172.17.0.1 test2
11.0.1.102 172.17.0.1 test3
11.0.1.102 172.17.0.1 test4
11.0.1.102 172.17.0.1 test5
11.0.1.102 172.17.0.1 test6
11.0.1.102 172.17.0.1 test7
11.0.1.102 172.17.0.1 test8
11.0.1.102 172.17.0.1 test9
11.0.1.102 172.17.0.1 test10

[root@internet ~]# while true ; do curl 192.168.188.101/test{1..10}.html?userid=c ;sleep 1;done
11.0.1.103 172.17.0.1 test1
11.0.1.103 172.17.0.1 test2
11.0.1.103 172.17.0.1 test3
11.0.1.103 172.17.0.1 test4
11.0.1.103 172.17.0.1 test5
11.0.1.103 172.17.0.1 test6
11.0.1.103 172.17.0.1 test7
11.0.1.103 172.17.0.1 test8
11.0.1.103 172.17.0.1 test9
11.0.1.103 172.17.0.1 test10

```

#### **4.4.3.2 url_param 一致性 hash 配置示例**

```sh
[root@ubuntu2004 ~]# cat /etc/haproxy/conf.d/web.cfg 
frontend www.dingbh.top_80
        bind 192.168.188.101:80
        use_backend web_servers

backend web_servers
#静态算法
#       redirect prefix http://www.baidu.com/
#       balance static-rr
#       balance first
#动态算法
#	balance roundrobin
#	balance leastconn
#	balance random
#其他
#	balance uri
#	balance source
#	hash-type map-based
	balance url_param userid
	hash-type consistent
        server 11.0.1.102 11.0.1.102:80 check inter 3000 fall 3 rise 2 weight 3
        server 11.0.1.103 11.0.1.103:80 check inter 3000 fall 3 rise 2
        server backup 127.0.0.1:8080 check backup
```

**测试**

```sh
[root@internet ~]# while true ; do curl 192.168.188.101/test{1..10}.html?userid={1..5} ;sleep 1;done
11.0.1.102 172.17.0.1 test1
11.0.1.103 172.17.0.1 test1
11.0.1.102 172.17.0.1 test1
11.0.1.102 172.17.0.1 test1
11.0.1.102 172.17.0.1 test1
11.0.1.102 172.17.0.1 test2
11.0.1.103 172.17.0.1 test2
11.0.1.102 172.17.0.1 test2
11.0.1.102 172.17.0.1 test2
11.0.1.102 172.17.0.1 test2
11.0.1.102 172.17.0.1 test3
11.0.1.103 172.17.0.1 test3
11.0.1.102 172.17.0.1 test3
11.0.1.102 172.17.0.1 test3
11.0.1.102 172.17.0.1 test3
11.0.1.102 172.17.0.1 test4
11.0.1.103 172.17.0.1 test4
11.0.1.102 172.17.0.1 test4
11.0.1.102 172.17.0.1 test4
11.0.1.102 172.17.0.1 test4
11.0.1.102 172.17.0.1 test5
11.0.1.103 172.17.0.1 test5
11.0.1.102 172.17.0.1 test5
11.0.1.102 172.17.0.1 test5
11.0.1.102 172.17.0.1 test5
11.0.1.102 172.17.0.1 test6
11.0.1.103 172.17.0.1 test6
11.0.1.102 172.17.0.1 test6
11.0.1.102 172.17.0.1 test6
11.0.1.102 172.17.0.1 test6
11.0.1.102 172.17.0.1 test7
11.0.1.103 172.17.0.1 test7
11.0.1.102 172.17.0.1 test7
11.0.1.102 172.17.0.1 test7
11.0.1.102 172.17.0.1 test7
11.0.1.102 172.17.0.1 test8
11.0.1.103 172.17.0.1 test8
11.0.1.102 172.17.0.1 test8
11.0.1.102 172.17.0.1 test8
11.0.1.102 172.17.0.1 test8
11.0.1.102 172.17.0.1 test9
11.0.1.103 172.17.0.1 test9
11.0.1.102 172.17.0.1 test9
11.0.1.102 172.17.0.1 test9
11.0.1.102 172.17.0.1 test9
11.0.1.102 172.17.0.1 test10
11.0.1.103 172.17.0.1 test10
11.0.1.102 172.17.0.1 test10
11.0.1.102 172.17.0.1 test10
11.0.1.102 172.17.0.1 test10
```

### **4.4.4 hdr** **算法**

针对用户每个http头部(header)请求中的指定信息做hash，此处由 name 指定的http首部将会被取出并做hash计算，然后由服务器总权重取模以后派发至某挑出的服务器，如果无有效值，则会使用默认的轮询调度。

#### **4.4.4.1 hdr 取模法配置示例**

针对不同的浏览器进行调度

```
[root@ubuntu2004 ~]# cat /etc/haproxy/conf.d/web.cfg 
frontend www.dingbh.top_80
        bind 192.168.188.101:80
        use_backend web_servers

backend web_servers
#静态算法
#       redirect prefix http://www.baidu.com/
#       balance static-rr
#       balance first
#动态算法
#	balance roundrobin
#	balance leastconn
#	balance random
#其他
#	balance uri
#	balance source
#	hash-type map-based

#	balance url_param userid
#	hash-type consistent

	balance hdr(User-Agent)
        server 11.0.1.102 11.0.1.102:80 check inter 3000 fall 3 rise 2 weight 3
        server 11.0.1.103 11.0.1.103:80 check inter 3000 fall 3 rise 2
        server backup 127.0.0.1:8080 check backup
```

#### **5.4.4.2** **一致性 **hash 配置示例

```
[root@ubuntu2004 ~]# cat /etc/haproxy/conf.d/web.cfg 
frontend www.dingbh.top_80
        bind 192.168.188.101:80
        use_backend web_servers

backend web_servers
#静态算法
#       redirect prefix http://www.baidu.com/
#       balance static-rr
#       balance first
#动态算法
#	balance roundrobin
#	balance leastconn
#	balance random
#其他
#	balance uri
#	balance source
#	hash-type map-based

#	balance url_param userid
#	hash-type consistent

	balance hdr(User-Agent)
	hash-type consistent
        server 11.0.1.102 11.0.1.102:80 check inter 3000 fall 3 rise 2 weight 3
        server 11.0.1.103 11.0.1.103:80 check inter 3000 fall 3 rise 2
        server backup 127.0.0.1:8080 check backup

```

#### **4.4.4.3**  测试访问

```sh
#
[root@internet ~]# for i in {1..10};do curl -A"IE$i" 192.168.188.101;done
DOCKER 11.0.1.102 WEBSITE v2.0
DOCKER 11.0.1.102 WEBSITE v2.0
DOCKER 11.0.1.103 WEBSITE v2.0
DOCKER 11.0.1.102 WEBSITE v2.0
DOCKER 11.0.1.102 WEBSITE v2.0
DOCKER 11.0.1.102 WEBSITE v2.0
DOCKER 11.0.1.103 WEBSITE v2.0
DOCKER 11.0.1.102 WEBSITE v2.0
DOCKER 11.0.1.102 WEBSITE v2.0
DOCKER 11.0.1.103 WEBSITE v2.0

#一致性
[root@internet ~]# for i in {11..20};do curl -A"IE$i" 192.168.188.101;done
DOCKER 11.0.1.102 WEBSITE v2.0
DOCKER 11.0.1.102 WEBSITE v2.0
DOCKER 11.0.1.103 WEBSITE v2.0
DOCKER 11.0.1.102 WEBSITE v2.0
DOCKER 11.0.1.102 WEBSITE v2.0
DOCKER 11.0.1.102 WEBSITE v2.0
DOCKER 11.0.1.102 WEBSITE v2.0
DOCKER 11.0.1.103 WEBSITE v2.0
DOCKER 11.0.1.102 WEBSITE v2.0
DOCKER 11.0.1.102 WEBSITE v2.0

#测试验证
[root@internet ~]# curl -v http://10.0.0.7/index.html
[root@internet ~]# curl -vA 'firefox' http://10.0.0.7/index.html
[root@internet ~]# curl -vA 'chrome' http://10.0.0.7/index.html
```

### **4.4.5 rdp-cookie** **算法**

rdp-cookie对远 Windows远程桌面的负载，使用cookie保持会话，默认是静态，也可以通过hash-type指定map-based和consistent，来定义使用取模法还是一致性hash。

### **4.4.6** **算法总结**

```sh
#静态
static-rr--------->tcp/http  
first------------->tcp/http  

#动态
roundrobin-------->tcp/http 
leastconn--------->tcp/http 
random------------>tcp/http

#以下静态和动态取决于hash_type是否consistent（一致性）
source------------>tcp/http
Uri--------------->http
url_param--------->http     
hdr--------------->http
rdp-cookie-------->tcp

#各种算法使用场景
first #使用较少
static-rr #做了session共享的 web 集群
roundrobin
random
leastconn #数据库
source #基于客户端公网 IP 的会话保持
Uri--------------->http  #缓存服务器，CDN服务商，蓝汛、百度、阿里云、腾讯
url_param--------->http  #可以实现session保持
hdr #基于客户端请求报文头部做下一步处理
rdp-cookie #基于Windows主机,很少使用
```

## 4.5 LVS、HAProxy、Nginx调度算法

| 调度器   | HAProxy                                                      | LVS                                                          | Nginx |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ----- |
| 静态算法 | 1、static-rr：基于权重的轮询调度，不支持运行时利用socat进行权重的动态调整(只支持0和1,不支持其它值)及后端服务器慢启动，其后端主机数量没有限制，相当于LVS中的wrr<br/>2、first：根据服务器在列表中的位置，自上而下进行调度，但是其只会当第一台服务器的连接数达到上限，新请求才会分配给下一台服务，因此会忽略服务器的权重设置，此方式使用较少 | 仅根据算法本身进行调度<br/>1、RR：roundrobin，轮询,较常用，雨露均沾，大锅饭<br/>2、WRR：Weighted RR，加权轮询,较常用<br/>3、SH：Source Hashing，实现session sticky，源IP地址hash；将来自于同一个IP地址的请求始终发往第一次挑中的RS，从而实现会话绑定<br/>4、DH：Destination Hashing；目标地址哈希，第一次轮询调度至RS，后续将发往同一个目标地址的请求始终转发至第一次挑中的RS，典型使用场景是正向代理缓存场景中的负载均衡,如: Web缓存 |       |
| 动态算法 | 1、roundrobin：基于权重的轮询动态调度算法，支持权重的运行时调整，不同于lvs中的rr轮训模式，HAProxy中的roundrobin支持慢启动(新加的服务器会逐渐增加转发数)，其每个后端backend中最多支持4095个real server，支持对real server权重动态调整,**roundrobin为默认调度算法**,此算法使用广泛<br/>2、leastconn 加权的最少连接的动态，支持权重的运行时调整和慢启动，即根据当前连接最少的后端服务器而非权重进行优先调度(新客户端连接)，比较适合长连接的场景使用，比如：MySQL等场景。<br/><br/>3、在1.9版本开始增加 random的负载平衡算法，其基于随机数作为一致性hash的key，随机负载平衡对于大型服务器场或经常添加或删除服务器非常有用，**支持weight的动态调整**，weight较大的主机有更大概率获取新请求 | 1、LC：least connections 适用于长连接应用<br/>Overhead=activeconns*256+inactiveconns<br/><br/>2、WLC：Weighted LC（带权重的最小连接），默认调度方法,较常用（默认）  注：默认工作模式DR<br/>Overhead=(activeconns*256+inactiveconns)/weight<br/><br/>[root@Rocky8 ~]#grep -i WLC /boot/config-4.18.0-425.3.1.el8.x86_64 CONFIG_IP_VS_WLC=m<br/><br>3、SED：Shortest Expection Delay，初始连接高权重优先,只检查活动连接,而不考虑非活动连接<br/>Overhead=(activeconns+1)*256/weight<br/><br/>4、NQ：Never Queue，第一轮均匀分配，后续SED<br/><br/>5、LBLC：Locality-Based LC，动态的DH算法，使用场景：根据负载状态实现正向代理,实现Web Cache等<br/><br/>6、LBLCR：LBLC with Replication，带复制功能的LBLC，解决LBLC负载不均衡问题，从负载重的复制到负载轻的RS,,实现Web Cache等 |       |
| 其他     |                                                              |                                                              |       |

# **5 HAProxy** **高级功能**

介绍 HAProxy 高级配置及实用案例

## **5.1** **基于** **Cookie** **的会话保持**

cookie value：为当前server指定cookie值，实现基于cookie的会话黏性，相对于基于 source 地址hash 调度算法对客户端的粒度更精准，但同时也加重了haproxy负载，目前此模式使用较少， 已经被session共享服务器代替

注意：不支持 tcp mode，使用 http mode

**5.1.1** **配置选项**

```sh
cookie name [ rewrite | insert | prefix ][ indirect ] [ nocache ][ postonly ] [ preserve ][ httponly ] [ secure ][ domain ]* [ maxidle <idle> ][ maxlife ]

name： #cookie 的 key名称，用于实现持久连接
insert： #插入新的cookie,默认不插入cookie
indirect： #如果客户端已经有cookie,则不会再发送cookie信息
nocache： #当client和hapoxy之间有缓存服务器（如：CDN）时，不允许中间缓存器缓存cookie，因为这会导致很多经过同一个CDN的请求都发送到同一台后端服务器
```

**5.1.2** **配置示例**

```sh
[root@ubuntu2004 ~]# cat /etc/haproxy/conf.d/web.cfg 
frontend www.dingbh.top_80
        bind 192.168.188.101:80
        use_backend web_servers

backend web_servers
#静态算法
#       redirect prefix http://www.baidu.com/
#       balance static-rr
#       balance first
#动态算法
#	balance roundrobin
#	balance leastconn
#	balance random
#其他
#	balance uri

#	balance source
#	hash-type map-based

#	balance url_param userid
#	hash-type consistent

#	balance hdr(User-Agent)
#	hash-type consistent

#cookie绑定
	cookie WEBSRV insert nocache indirect
        server 11.0.1.102 11.0.1.102:80 check inter 3000 fall 3 rise 2 weight 3 cookie web01
        server 11.0.1.103 11.0.1.103:80 check inter 3000 fall 3 rise 2	cookie web02
        server backup 127.0.0.1:8080 check backup
```

**测试访问**

```sh
#命令行验证
#保存cookie
[root@internet ~]# curl -b cookie.txt -v 192.168.188.101
* Rebuilt URL to: 192.168.188.101/
*   Trying 192.168.188.101...
* TCP_NODELAY set
* Connected to 192.168.188.101 (192.168.188.101) port 80 (#0)
> GET / HTTP/1.1
> Host: 192.168.188.101
> User-Agent: curl/7.61.1
> Accept: */*
> Cookie: WEBSRV=web01
> 
< HTTP/1.1 200 OK
< server: nginx/1.20.2
< date: Fri, 31 May 2024 05:18:55 GMT
< content-type: text/html
< content-length: 31
< last-modified: Fri, 24 May 2024 13:48:59 GMT
< etag: "66509acb-1f"
< accept-ranges: bytes
< 
DOCKER 11.0.1.102 WEBSITE v2.0
* Connection #0 to host 192.168.188.101 left intact

#查看cookie信息
[root@internet ~]# cat cookie.txt 
# Netscape HTTP Cookie File
# https://curl.haxx.se/docs/http-cookies.html
# This file was generated by libcurl! Edit at your own risk.

192.168.188.101	FALSE	/	FALSE	0	WEBSRV	web01

#携带cookie信息访问主机
[root@internet ~]# curl -b cookie.txt -v 192.168.188.101
* Rebuilt URL to: 192.168.188.101/
*   Trying 192.168.188.101...
* TCP_NODELAY set
* Connected to 192.168.188.101 (192.168.188.101) port 80 (#0)
> GET / HTTP/1.1
> Host: 192.168.188.101
> User-Agent: curl/7.61.1
> Accept: */*
> Cookie: WEBSRV=web01
> 
< HTTP/1.1 200 OK
< server: nginx/1.20.2
< date: Fri, 31 May 2024 05:18:55 GMT
< content-type: text/html
< content-length: 31
< last-modified: Fri, 24 May 2024 13:48:59 GMT
< etag: "66509acb-1f"
< accept-ranges: bytes
< 
DOCKER 11.0.1.102 WEBSITE v2.0
* Connection #0 to host 192.168.188.101 left intact
```

![image-20240527192748213](Haproxy/image-20240527192748213.png)



## **5.2 HAProxy** **状态页**

通过web界面，显示当前HAProxy的运行状态

```
http://cbonte.github.io/haproxy-dconv/2.4/configuration.html#4-stats%20admin
http://cbonte.github.io/haproxy-dconv/2.0/configuration.html#4-stats%20admin
```

### **5.2.1 配置项**

```sh
stats enable   #基于默认的参数启用stats page
stats hide-version   #将状态页中haproxy版本隐藏
stats refresh <delay> #设定自动刷新时间间隔，默认不自动刷新,以秒为单位
stats uri <prefix> #自定义stats page uri，默认值：/haproxy?stats 
stats realm <realm> #账户认证时的提示信息，示例：stats realm   HAProxy\ Statistics
stats auth <user>:<passwd> #认证时的账号和密码，可定义多个用户,每行指定一个用户.默认：no authentication
stats admin { if | unless } <cond> #启用stats page中的管理功能

#状态页说明
listen haproxy-status
 bind :9999
 stats enable
  #stats hide-version 
 stats uri /haproxy-status           #自定义stats page uri
 stats realm HAProxy\ Stats\ Page     #账户认证时的提示信息
 stats auth haadmin:123456   #支持多个用户
 stats auth admin:123456
  #stats refresh 30
 stats admin if TRUE   #开启管理功能，基于安全原因，不建议开启

#状态配置项
[root@ubuntu2004 ~]# cat /etc/haproxy/conf.d/status.cfg 
listen status
	bind 11.0.1.101:88
	stats enable
	stats uri /haproxy-status
	stats auth admin:123456
	stats refresh 10
	stats admin if TRUE		#启用stats page中的管理功能
	
#验证访问
[root@haproxy ~]# curl http://admin:123456@11.0.1.101:88/haproxy-status
```

![image-20240531132545227](Haproxy/image-20240531132545227.png)

### 5.2.2 登录状态页说明

```sh
pid = 27134 (process #1, nbproc = 1, nbthread = 1) #pid为当前pid号，process为当前进程号，nbproc和nbthread为一共多少进程和每个进程多少个线程
uptime = 0d 0h00m04s #启动了多长时间
system limits: memmax = unlimited; ulimit-n = 200029 #系统资源限制：内存/最大打开文件数/
maxsock = 200029; maxconn = 100000; maxpipes = 0 #最大socket连接数/单进程最大连接数/最大管道数maxpipes
current conns = 2; current pipes = 0/0; conn rate = 2/sec; bit rate = 0.000 kbps #当前连接数/当前管道数/当前连接速率Running tasks: 1/14; idle = 100 % #运行的任务/当前空闲率
active UP： #在线服务器
backup UP： #标记为backup的服务器
active UP, going down： #监测未通过正在进入down过程
backup UP, going down： #备份服务器正在进入down过程
active DOWN, going up： #down的服务器正在进入up过程
backup DOWN, going up： #备份服务器正在进入up过程
active or backup DOWN： #在线的服务器或者是backup的服务器已经转换成了down状态
not checked： #标记为不监测的服务器
active or backup DOWN for maintenance (MAINT) #active或者backup服务器人为下线的
active or backup SOFT STOPPED for maintenance #active或者backup被人为软下线(人为将weight改成0)
```

## 5.3 IP透传

web服务器中需要记录客户端的真实IP地址，用于做访问统计、安全防护、行为分析、区域排行等场景。

### 5.3.1 **Layer 4** **与** **Layer 7**

四层：IP+PORT转发

七层：协议+内容交换

![image-20240531134910635](Haproxy/image-20240531134910635.png)

#### 5.3.1.1 四层负载

在LVS 传统的四层负载设备中，把client发送的报文目标地址(原来是负载均衡设备的IP地址)，根据均衡设备设置的选择web服务器的规则选择对应的web服务器IP地址，这样client就可以直接跟此服务器建立TCP连接并发送数据，而四层负载自身不参与建立连接

而和LVS不同，haproxy是伪四层负载均衡，因为haproxy 需要分别和前端客户端及后端服务器建立连接

#### **5.3.1.2** **七层代理**

七层负载均衡服务器起了一个反向代理服务器的作用，服务器建立一次TCP连接要三次握手，而client要访问Web Server要先与七层负载设备进行三次握手后建立TCP连接，把要访问的报文信息发送给七层负载均衡；然后七层负载均衡再根据设置的均衡规则选择特定的 Web Server，然后通过三次握手与此台Web Server建立TCP连接，然后Web Server把需要的数据发送给七层负载均衡设备，负载均衡设备再把数据发送给client；所以，七层负载均衡设备起到了代理服务器的作用，七层代理需要和Client和后端服务器分别建立连接

```sh
[root@haproxy ~]# tcpdump tcp -i eth0 -nn port ! 22 -w dump-tcp.pcap -v
[root@haproxy ~]# tcpdump tcp -i eth1 -nn port ! 22 -w dump-tcp4.pcap -v
```



![image-20240531170156603](Haproxy/image-20240531170156603.png)

### **5.3.2** 四层IP透传





### 5.3.3 七层IP透传

当haproxy工作在七层的时候，也可以透传客户端真实IP至后端服务器

#### 5.3.3.1 HAProxy配置

在由haproxy发往后端主机的请求报文中添加“X-Forwarded-For"首部，其值为前端客户端的地址；用于向后端主发送真实的客户端IP

```sh
option forwardfor [ except <network> ] [ header <name> ] [ if-none ]

[ except <network> ]：请求报请来自此处指定的网络时不予添加此首部，如haproxy自身所在网络
[ header <name> ]：使用自定义的首部名称，而非“X-Forwarded-For"，示例：X-client
[ if-none ] 如果没有首部才添加首部，如果有使用默认值
```

**范例：**

```sh
#haproxy代理服务器配置

defaults
#此为默认值,首部字段默认为：X-Forwarded-For
option forwardfor   
#或者自定义首部,如:dinginx
option forwardfor except 127.0.0.0/8 header dinginx


#listen配置
[root@haproxy ~]# vim /etc/haproxy/conf.d/web.cfg 

frontend www.dingbh.top_80			#一定是http模式,tcp 模式不会传递客户端IP
        bind 192.168.188.101:80	
        use_backend web_servers

backend web_servers
        option httpchk HEAD /index.html
        
        server 11.0.1.102 11.0.1.102:80 check
        server 11.0.1.103 11.0.1.103:80 check
        server backup 127.0.0.1:8080 check backu
```

#### 5.3.3.2 后端服务器日志格式配置

配置web服务器，记录负载均衡透传的客户端IP地址

```sh
#nginx 日志格式：默认就支持
$proxy_add_x_forwarded_for：包括客户端IP和中间经过的所有代理的IP
$http_x_forwarded_For：只有客户端IP
$http_dinginx:自定义首部字段

[root@ubuntu2004 ~]#vim /etc/nginx/nginx.conf
......
		##
        # Logging Settings
        ##
        log_format dinginxlog '$remote_addr - $remote_user [$time_local] '
                       '"$request" $status $bytes_sent '
                       '"$http_referer" "$http_user_agent" "$gzip_ratio"'
                       '"$http_x_forwarded_for" "$http_dinginx"' ;

        access_log /var/log/nginx/access.log dinginxlog;
        error_log /var/log/nginx/error.log;

        ##


#apache 配置：需要修改才支持
LogFormat "%{X-Forwarded-For}i %a %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%
{User-Agent}i\"" combined

#tomcat 配置：conf目录下的server.xml
<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
   prefix="localhost_access_log" suffix=".txt"
   pattern="%{X-Forwarded-For}i %h %l %u %t &quot;%r&quot; %s %b" />    
```

#### **5.3.3.3** 验证客户端IP地址

**测试**

```sh
[root@ubuntu2004 ~]#tail -f /var/log/nginx/access.log

......

11.0.1.101 - - [01/Jun/2024:22:09:39 +0800] "HEAD /index.html HTTP/1.0" 200 0 "-" "-"--""
11.0.1.101 - - [01/Jun/2024:22:09:41 +0800] "GET / HTTP/1.1" 200 27 "-" "curl/7.61.1"-192.168.188.88""
11.0.1.101 - - [01/Jun/2024:22:09:41 +0800] "HEAD /index.html HTTP/1.0" 200 0 "-" "-"--""
11.0.1.101 - - [01/Jun/2024:22:09:43 +0800] "GET / HTTP/1.1" 200 27 "-" "curl/7.61.1"-192.168.188.88""
11.0.1.101 - - [01/Jun/2024:22:09:43 +0800] "HEAD /index.html HTTP/1.0" 200 0 "-" "-"--""
11.0.1.101 - - [01/Jun/2024:22:09:45 +0800] "GET / HTTP/1.1" 200 27 "-" "curl/7.61.1"-192.168.188.88""
11.0.1.101 - - [01/Jun/2024:22:09:45 +0800] "HEAD /index.html HTTP/1.0" 200 0 "-" "-"--""
11.0.1.101 - - [01/Jun/2024:22:09:47 +0800] "GET / HTTP/1.1" 200 27 "-" "curl/7.61.1"-192.168.188.88""
11.0.1.101 - - [01/Jun/2024:22:09:47 +0800] "HEAD /index.html HTTP/1.0" 200 0 "-" "-"--""
11.0.1.101 - - [01/Jun/2024:22:09:49 +0800] "GET / HTTP/1.1" 200 27 "-" "curl/7.61.1"-192.168.188.88""
11.0.1.101 - - [01/Jun/2024:22:09:49 +0800] "HEAD /index.html HTTP/1.0" 200 0 "-" "-"--""
11.0.1.101 - - [01/Jun/2024:22:09:51 +0800] "GET / HTTP/1.1" 200 27 "-" "curl/7.61.1"-192.168.188.88""
11.0.1.101 - - [01/Jun/2024:22:09:51 +0800] "HEAD /index.html HTTP/1.0" 200 0 "-" "-"--""
11.0.1.101 - - [01/Jun/2024:22:09:53 +0800] "GET / HTTP/1.1" 200 27 "-" "curl/7.61.1"-192.168.188.88""
11.0.1.101 - - [01/Jun/2024:22:09:53 +0800] "HEAD /index.html HTTP/1.0" 200 0 "-" "-"--""
11.0.1.101 - - [01/Jun/2024:22:09:55 +0800] "GET / HTTP/1.1" 200 27 "-" "curl/7.61.1"-192.168.188.88""
11.0.1.101 - - [01/Jun/2024:22:09:55 +0800] "HEAD /index.html HTTP/1.0" 200 0 "-" "-"--""
11.0.1.101 - - [01/Jun/2024:22:09:57 +0800] "GET / HTTP/1.1" 200 27 "-" "curl/7.61.1"-192.168.188.88""
11.0.1.101 - - [01/Jun/2024:22:09:57 +0800] "HEAD /index.html HTTP/1.0" 200 0 "-" "-"--""
11.0.1.101 - - [01/Jun/2024:22:09:59 +0800] "GET / HTTP/1.1" 200 27 "-" "curl/7.61.1"-192.168.188.88""
11.0.1.101 - - [01/Jun/2024:22:09:59 +0800] "HEAD /index.html HTTP/1.0" 200 0 "-" "-"--""
11.0.1.101 - - [01/Jun/2024:22:10:01 +0800] "GET / HTTP/1.1" 200 27 "-" "curl/7.61.1"-192.168.188.88""
11.0.1.101 - - [01/Jun/2024:22:10:01 +0800] "HEAD /index.html HTTP/1.0" 200 0 "-" "-"--""
```

**apache日志**

```sh
#apache日志
[root@centos7 ~]#vim /etc/httpd/conf/httpd.conf 
LogFormat "%{X-Forwarded-For}i %h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%
{User-Agent}i\"" combined
[root@centos7 ~]#systemctl restart httpd
[root@centos6 ~]#hostname -I
10.0.0.6 
[root@centos6 ~]#curl   http://10.0.0.7
10.0.0.17
[root@centos7 ~]#tail -f /var/log/httpd/access_log 
10.0.0.6 10.0.0.7 - - [01/Apr/2020:01:08:31 +0800] "GET / HTTP/1.1" 200 10 "-"
"curl/7.19.7 (x86_64-redhat-linux-gnu) libcurl/7.19.7 NSS/3.27.1 zlib/1.2.3 
libidn/1.18 libssh2/1.4.2"
10.0.0.6 10.0.0.7 - - [01/Apr/2020:01:08:33 +0800] "GET / HTTP/1.1" 200 10 "-"
"curl/7.19.7 (x86_64-redhat-linux-gnu) libcurl/7.19.7 NSS/3.27.1 zlib/1.2.3 
libidn/1.18 libssh2/1.4.2
```

## 5.4 报文修改

### 5.4.1 新版

**2.1版本以上用下面指令http-request和http-response代替**

官方文档：参看2.4的帮助文档

```
http://cbonte.github.io/haproxy-dconv/2.4/configuration.html#4-http-request
http://cbonte.github.io/haproxy-dconv/2.4/configuration.html#4-http-response
```

**配置说明：**

```sh
#修改请求host首部,默认host首部会保留客户端原首部haproxy不会修改
http-request set-header host www.wangxiaochun.com

#添加向后端服务器发送的请求报文首部
http-request add-header <name> <fmt> [ { if | unless } <condition> ]
#示例：http-request add-header X-Haproxy-Current-Date %T

#删除向后端服务器发送的请求报文首部
http-request del-header <name> [ { if | unless } <condition> ]

#修改响应首部
http-response set-header server wangserver

#添加向客户端发送的响应报文首部
http-response add-header <name> <fmt> [ { if | unless } <condition> ]

#删除向客户端发送的响应报文首部
http-response del-header <name>
#示例：http-response del-header Server
```

**范例: 添加向后端服务器发送的请求报文首部**

```sh
#haproxy配置
[root@haproxy ~]# vim /etc/haproxy/conf.d/web.cfg 
frontend www.dingbh.top_80
        bind 192.168.188.101:80
        use_backend web_servers
        http-request add-header X-Haproxy-Current-Date %T  #添加报文头部信息

backend web_servers        
        server 11.0.1.102 11.0.1.102:80 check
        server 11.0.1.103 11.0.1.103:80 check

[root@haproxy ~]# systemctl restart haproxy.service 
```

**抓包信息查看**

![image-20240601223043187](Haproxy/image-20240601223043187.png)

**修改host报文头字段**

```sh
#haproxy配置
[root@haproxy ~]# vim /etc/haproxy/conf.d/web.cfg 
frontend www.dingbh.top_80
        bind 192.168.188.101:80
        use_backend web_servers
        http-request add-header X-Haproxy-Current-Date %T  #添加报文头部信息
		http-request set-header host www.baidu.com
backend web_servers        
        server 11.0.1.102 11.0.1.102:80 check
        server 11.0.1.103 11.0.1.103:80 check

[root@haproxy ~]# systemctl restart haproxy.service 	
```

![image-20240601223559555](Haproxy/image-20240601223559555.png)

**范例: 修改向客户端发送的响应报文首部,实现自定义Server首部**

```sh
#修改nginx配置
[root@haproxy ~]# cat /etc/haproxy/conf.d/web.cfg 
frontend www.dingbh.top_80
    bind 192.168.188.101:80
    use_backend web_servers
	http-request add-header X-Haproxy-Current-Date %T
	http-request set-header host www.baidu.com
	http-response del-header server
	http-response add-header server dinginx-server
#注意和上面指令的前后顺序
#http-response set-header server dinginx-server #可以替代上面两条

backend web_servers
        server 11.0.1.102 11.0.1.102:80 check
        server 11.0.1.103 11.0.1.103:80 check
        server backup 127.0.0.1:8080 check backup

[root@haproxy ~]# systemctl restart haproxy.service

#测试
[root@internet ~]# curl 192.168.188.101 -I
HTTP/1.1 200 OK
date: Sat, 01 Jun 2024 14:45:29 GMT
content-type: text/html
content-length: 27
last-modified: Sat, 01 Jun 2024 14:01:25 GMT
etag: "665b29b5-1b"
accept-ranges: bytes
server: dinginx-server	#自定义服务器版本信息
```

### **4.4.2** **旧版**

**注意：以下相关指令在2.1以上版本中已经取消**

在http模式下，基于实际需求修改客户端的请求报文与响应报文，通过reqadd和reqdel在请求报文添加删除字段，通过rspadd与rspidel在响应报文中添加与删除字段。

官方文档：参看2.0的帮助文档

```
http://cbonte.github.io/haproxy-dconv/2.0/configuration.html#4-rspadd
```

```sh
#在向后端服务器转发的请求报文尾部添加指定首部
 reqadd <string> [{if | unless} <cond>]
 示例：reqadd X-Via:\ HAproxy   #注意只能有一个空格，并需要转义
 #在向后端服务器转发的请求报文中删除匹配正则表达式的首部
 reqdel <search> [{if | unless} <cond>]
 reqidel <search> [{if | unless} <cond>]   #忽略大小写
 示例：
 reqidel user-agent
 reqidel X-Forwarded-For #无法删除
  
#在向前端客户端转发的响应报文尾部添加指定首部
 rspadd <string> [{if | unless} <cond>] 
 示例：
 rspadd X-Via:\ HAproxy 
 rspadd Server:\ wanginx
#从向前端客户端转发的响应报文中删除匹配正则表达式的首部
 rspdel <search> [{if | unless} <cond>]
 rspidel <search> [{if | unless} <cond>]   #忽略大小写
 示例：
 rspidel ^server:.* #从响应报文删除server信息
 rspidel X-Powered-By:.* #从响应报文删除X-Powered-By信息,一般此首部字段保存php版本信息 #
```

**范例：旧版才支持**

```sh
#添加向后端报务器发起的请求报文首部
vim haproxy.cfg
frontend main *:80
#       bind *:80
       default_backend websrvs
       reqadd testheader:\ haproxyserver   #加此行,注意只能有一个空格，并需要转义
#在后端httpd服务器
vim /etc/httpd/conf/httpd.conf
LogFormat "%{testheader}i %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
#查看日志
tail –f /var/log/httpd/acesss_log
```

**范例：旧版才支持**

```sh
#添加响应报文首部
vim haproxy.cfg
frontend main *:80
#       bind *:80
       default_backend websrvs
       rspadd X-Via:\ HAproxy-1   #加此行
       maxconn         5000
#客户端访问调试模式,查看reponse headers，看到
Server: Apache/2.4.37 (centos)  #系统自带显示
X-Via: HAproxy-1
```

**范例：旧版才支持**

```
#删除响应报文中的server首部
vim haproxy.cfg
frontend main *:80
#       bind *:80
       default_backend websrvs
       rspadd X-Via:\ HAproxy-1
       rspdel Server 或者 rspidel server    #加此行 ,忽略大小写
       rspidel X-Powered-By:.* #删除Php版本
       maxconn         5000
#客户端访问调试模式,查看reponse headers，看到Server: Apache/2.4.37 (centos)  #此行消失
X-Via: HAproxy-1
```

**范例：旧版才支持**

```
#增加响应报文的首部，实现伪装Server首部
vim haproxy.cfg
frontend main *:80
#       bind *:80
       default_backend websrvs
       rspadd X-Via:\ HAproxy-1
       rspdel Server  #或者用 rspidel server 
 rspadd Server:\   wanginx   #增加此行
[root@internet ~]#curl -i   172.16.0.100
HTTP/1.1 200 OK
date: Thu, 09 Apr 2020 08:32:10 GMT
last-modified: Thu, 09 Apr 2020 01:23:18 GMT
etag: "f-5a2d17630635b"
accept-ranges: bytes
content-length: 15
content-type: text/html; charset=UTF-8
server: wanginx
RS1 10.0.0.17
```

## **5.5** **自定义日志格式**

log global 开启日志功能，默认只会在记录下面格式的日志

```
[root@haproxy ~]# tail -f /var/log/haproxy.log 
May 24 20:43:58 localhost haproxy[169233]: Connect from 11.0.1.99:55087 to 11.0.1.101:9999 (stats/HTTP)
May 24 20:58:25 localhost haproxy[169233]: Proxy stats stopped (cumulated conns: FE: 10, BE: 0).
May 24 20:43:58 localhost haproxy[169233]: Connect from 11.0.1.99:55087 to 11.0.1.101:9999 (stats/HTTP)
May 24 20:58:25 localhost haproxy[169233]: Proxy stats stopped (cumulated conns: FE: 10, BE: 0).
May 24 21:57:39 localhost haproxy[179945]: Proxy stats stopped (cumulated conns: FE: 0, BE: 0).
May 24 21:57:39 localhost haproxy[179945]: Proxy stats stopped (cumulated conns: FE: 0, BE: 0).
May 24 22:05:26 localhost haproxy[183000]: Proxy stats stopped (cumulated conns: FE: 0, BE: 0).
```

option httplog 可以采用 http 格式记录下来，并且可以使用相关指令将特定信息记录在haproxy的日志中

但一般不建议开启，这会加重 HAProxy 负载

### 5.5.1 配置选项

```sh
log global 							#开启记录日志,默认不开启
option httplog                      #开启记录httplog日志格式选项
capture cookie <name> len <length> #捕获请求和响应报文中的 cookie及值的长度,将之记录到日志 
capture request header <name> len <length> #捕获请求报文中指定的首部内容和长度并记录日志
capture response header <name> len <length> #捕获响应报文中指定的内容和长度首部并记录日志

#示例：default模块下
default

log global 
option httplog
capture request header Host len  256
capture request header User-Agent len 512 
capture request header Referer len 15
capture request header X-Forwarded-For len 15
```

**同时开启日志功能log global和option httplog，记录日志格式如下**

### 5.5.2 配置范例

```sh

#haproxy配置
defaults
option http-keep-alive
option forwardfor
# except 127.0.0.1/8 header dinginx
maxconn 100000
mode http
timeout connect 300000ms
timeout client 300000ms
timeout server 300000ms
log global			#开启记录日志。默认不开启
option httplog		#开启记录http日志格式选项
```

**同时开启日志功能log global和option httplog，记录日志格式如下**

```sh
#验证格式内容
[root@haproxy ~]# tail -f /var/log/haproxy.log 

......

2 active and 0 backup servers left. 0 sessions active, 0 requeued, 0 remaining in queue.
Jun  1 23:05:09 localhost haproxy[32416]: 192.168.188.88:38998 [01/Jun/2024:23:05:09.019] www.dingbh.top_80 web_servers/11.0.1.102 0/0/0/1/1 200 208 - - ---- 1/1/0/0/0 0/0 "HEAD / HTTP/1.1"
Jun  1 23:05:09 localhost haproxy[32416]: 192.168.188.88:38998 [01/Jun/2024:23:05:09.019] www.dingbh.top_80 web_servers/11.0.1.102 0/0/0/1/1 200 208 - - ---- 1/1/0/0/0 0/0 "HEAD / HTTP/1.1"
```

## 5.6 压缩功能

对响应给客户端的报文进行压缩，以节省网络带宽，但是会占用部分CPU性能

建议在后端服务器开启压缩功能，而非在HAProxy上开启压缩

注意：默认Ubuntu的包安装nginx开启压缩功能

### 5.6.1 配置选项

```sh
compression algo <algorithm> ... #启用http协议中的压缩机制，常用算法有 gzip，deflate
#压缩算法<algorithm>支持下面类型：
 identity #debug调试使用的压缩方式
 gzip #常用的压缩方式，与各浏览器兼容较好
 deflate #有些浏览器不支持
 raw-deflate #新式的压缩方式
  
compression type <mime type> ... #要压缩的文件类型MIME

#示例：
compression algo gzip deflate
compression type text/html text/css text/plain
```

### 5.6.2 配置实例

```sh
[root@haproxy ~]# vim /etc/haproxy/conf.d/web.cfg 
[root@haproxy ~]# cat /etc/haproxy/conf.d/web.cfg 
frontend www.dingbh.top_80
        bind 192.168.188.101:80
        use_backend web_servers
#可以替代上面两条
		http-response set-header server dinginx-server

#启用压缩和指定算法
		compression algo gzip deflate 
#指定压缩文件类型
		compression type compression type text/plain text/html text/css text/xml text/javascript application/javascript
	
backend web_servers
        server 11.0.1.102 11.0.1.102:80 check
        server 11.0.1.103 11.0.1.103:80 check
        server backup 127.0.0.1:8080 check backup
[root@haproxy ~]# systemctl restart haproxy.service         
```

### 5.6.3 验证压缩功能



![image-20240601233839815](Haproxy/image-20240601233839815.png)

![image-20240601233603030](Haproxy/image-20240601233603030.png)

## 5.7 后端服务器健康性监测

### 5.7.1 三种状态监测方式

```
基于四层的传输端口做状态监测，此为默认方式
基于指定 URI 做状态监测,需要访问整个页面资源,占用更多带宽
基于指定 URI 的 request 请求头部内容做状态监测，占用较少带宽,建议使用此方式
```

### 5.7.2 基于应用层http协议进行健康性检测

基于应用层http协议，采有不同的监测方式，对后端**real server**进行状态监测

注意: 此方式会导致在后端服务器生成很多的HAProxy发起的访问日志

```sh
option httpchk   #支持Listen和backendf块，启用七层健康性检测，对tcp 和 http 模式都支持，
默认为：OPTIONS / HTTP/1.0，nginx默认不支持OTIONS
option httpchk <uri>
option httpchk <method> <uri>
option httpchk <method> <uri> <version>

#期望以上检查得到的响应码
http-check expect [!] <match> <pattern>
#示例：
http-check expect status 200
http-check expect ! rstatus ^5 #支持正则表达式

#关于HTTP/1.1的说明
<version> is the optional HTTP version string. It defaults to "HTTP/1.0" but some servers might behave incorrectly in HTTP 1.0, so turning it to HTTP/1.1 may sometimes help. Note that the Host field is         mandatory in HTTP/1.1, and as a trick, it is possible to pass it after "\r\n" following the version string.
```

### 5.7.3 配置实例

```sh
[root@haproxy ~]# cat /etc/haproxy/conf.d/web.cfg 
frontend www.dingbh.top_80
        bind 192.168.188.101:80
        use_backend web_servers
	
backend web_servers
	    #option httpchk GET /monitor/check.html #默认HTTP/1.0
	    #option httpchk GET /monitor/check.html HTTP/1.0
	    #option httpchk GET /monitor/check.html HTTP/1.1 #注意：HTTP/1.1强制要求必须有Host字段
	    option httpchk HEAD /monitor/check.html HTTP/1.1\r\nHost:\ www.wang.org #使用HEAD减少网络流量，新版不支持\r\nHost:\ www.wang.org
 		http-check send hdr Host www   #注意：新版要求：HTTP/1.1强制要求必须有Host字段
		#option httpchk GET /index.html	#GET消耗带宽大，HEAD只查看状态码，更合理
        server 11.0.1.102 11.0.1.102:80 check
        server 11.0.1.103 11.0.1.103:80 check
        server backup 127.0.0.1:8080 check backup
```

### 5.7.4 验证http健康检测

![image-20240604112042538](Haproxy/image-20240604112042538.png)

```sh
[root@ubuntu2004 ~]#tail -f /var/log/nginx/access.log
11.0.1.101 - - [04/Jun/2024:10:51:44 +0800] "HEAD /index.html HTTP/1.0" 200 0 "-" "-"--""
11.0.1.101 - - [04/Jun/2024:10:51:46 +0800] "HEAD /index.html HTTP/1.0" 200 0 "-" "-"--""
11.0.1.101 - - [04/Jun/2024:10:51:48 +0800] "HEAD /index.html HTTP/1.0" 200 0 "-" "-"--""
11.0.1.101 - - [04/Jun/2024:10:51:50 +0800] "HEAD /index.html HTTP/1.0" 200 0 "-" "-"--""
11.0.1.101 - - [04/Jun/2024:10:51:52 +0800] "HEAD /index.html HTTP/1.0" 200 0 "-" "-"--""
11.0.1.101 - - [04/Jun/2024:10:51:54 +0800] "HEAD /index.html HTTP/1.0" 200 0 "-" "-"--""
```

## **5.8 ACL**

访问控制列表（ACL，Access Control Lists）是一种基于包过滤的访问控制技术，它可以根据设定的条件对经过服务器传输的数据包进行过滤(条件匹配)，即对接收到的报文进行匹配和过滤，基于请求报文头部中的源地址、源端口、目标地址、目标端口、请求方法、URL、文件后缀等信息内容进行匹配并执行进一步操作，比如允许其通过或丢弃。

官方帮助：

```
http://cbonte.github.io/haproxy-dconv/2.1/configuration.html#7
http://cbonte.github.io/haproxy-dconv/2.0/configuration.html#7
```

### 5.8.1 定义ACL配置选项

```http
acl   <aclname>  <criterion>   [flags]     [operator]   [<value>]
acl      名称       匹配规范     匹配模式      具体操作符    操作对象类型
```

#### 5.8.1.1 ACL-name

```sh
acl   image_service hdr_dom(host)   -i   img.wang.com
#ACL名称，可以使用大字母A-Z、小写字母a-z、数字0-9、冒号：、点.、中横线和下划线，并且严格区分大小写，比如:my_acl和My_Acl就是两个完全不同的acl
```

#### **5.8.1.2 ACL-criterion**

**定义ACL匹配规范，即：判断条件**

```sh
hdr string，提取在一个HTTP请求报文的首部
hdr（[<name> [，<occ>]]）：完全匹配字符串,header的指定信息，<occ> 表示在多值中使用的值的出现次数
hdr_beg（[<name> [，<occ>]]）：前缀匹配，header中指定匹配内容的begin
hdr_end（[<name> [，<occ>]]）：后缀匹配，header中指定匹配内容end
hdr_dom（[<name> [，<occ>]]）：域匹配，header中的domain name
hdr_dir（[<name> [，<occ>]]）：路径匹配，header的uri路径
hdr_len（[<name> [，<occ>]]）：长度匹配，header的长度匹配
hdr_reg（[<name> [，<occ>]]）：正则表达式匹配，自定义表达式(regex)模糊匹配
hdr_sub（[<name> [，<occ>]]）：子串匹配，header中的uri模糊匹配

#示例：
hdr(<string>) 用于测试请求头部首部指定内容
hdr_dom(host) 请求的host名称，如 www.wang.com,m.wang.com
hdr_beg(host) 请求的host开头，如 www.   img.   video.   download.   ftp.
hdr_end(host) 请求的host结尾，如 .com   .net   .cn 

#示例：
acl bad_agent hdr_sub(User-Agent) -i curl wget
http-request deny  if bad_agent

#block if bad_agent 2.1版本后不再支持，用上面替代
hdr(host) ==>www.wang.org:8080
hdr_domain(host) ==> www.wang.org

#有些功能是类似的，比如以下几个都是匹配用户请求报文中host的开头是不是www
acl short_form hdr_beg(host)       www.
acl alternate1 hdr_beg(host) -m beg www.
acl alternate2 hdr_dom(host) -m beg www.
acl alternate3 hdr(host)     -m beg www.
base : string

#返回第一个主机头和请求的路径部分的连接，该请求从主机名开始，并在问号之前结束,对虚拟主机有用,下面的例子中是两个#中间的内容,实际#是没有的
<scheme>://<user>:<password>@#<host>:<port>/<path>;<params>#?<query>#<frag>
 base     : exact string match
 base_beg : prefix match
 base_dir : subdir match
 base_dom : domain match
 base_end : suffix match
 base_len : length match
 base_reg : regex match
 base_sub : substring match
 
path : string
#提取请求的URL路径，该路径从第一个斜杠开始，并在问号之前结束（无主机部分）
<scheme>://<user>:<password>@<host>:<port>#/<path>;<params>#?<query>#<frag>
 path     : exact string match
 path_beg : prefix match  #请求的URL开头，如/static、/images、/img、/css
 path_end : suffix match  #请求的URL中资源的结尾，如 .gif .png .css .js .jpg 
.jpeg
 path_dom : domain match
 path_dir : subdir match
 path_len : length match
 path_reg : regex match
 path_sub : substring match
#示例：
 path_beg -i /haproxy-status/   
 path_end .jpg .jpeg .png .gif 
 path_reg ^/images.*\.jpeg$ 
 path_sub image  
 path_dir jpegs 
 path_dom www.wang.org
 
url : string
#提取请求中的整个URL。一个典型的应用是具有预取能力的缓存，以及需要从数据库聚合多个信息并将它们保存在缓存中的网页门户入口，推荐使用path
 url ：exact string match
 url_beg : prefix match
 url_dir : subdir match
 url_dom : domain match
 url_end : suffix match
 url_len : length match
 url_reg : regex match
 url_sub : substring match

src   #源IP
src_port #源PORT
dst #目标IP
dst_port #目标PORT

#示例：
acl invalid_src src 10.0.0.7 192.168.1.0/24
acl invalid_src src 172.16.0.0/24
http-request deny if ! invalid_src
acl invalid_port src_port 0:1023
status : integer  #返回在响应报文中的状态码

#七层协议
acl valid_method method GET HEAD
http-request deny if ! valid_method
```

#### **5.8.1.3 ACL-flags**

ACL匹配模式

```basic
-i 不区分大小写
-m 使用指定的pattern匹配方法，beg 开头，end 结尾
-n 不做DNS解析
-u 禁止acl重名，否则多个同名ACL匹配或关系
```

#### **5.8.1.4 ACL-operator**

ACL 操作符

```basic
整数比较：eq、ge、gt、le、lt
字符比较：
- exact match     (-m str) :字符串必须完全匹配模式
- substring match (-m sub) :在提取的字符串中查找模式，如果其中任何一个被发现，ACL将匹配
- prefix match   (-m beg) :在提取的字符串首部中查找模式，如果其中任何一个被发现，ACL将匹配
- suffix match   (-m end) :将模式与提取字符串的尾部进行比较，如果其中任何一个匹配，则ACL进行匹配
- subdir match   (-m dir) :查看提取出来的用斜线分隔（“/"）的字符串，如其中任一个匹配，则ACL进行匹配
- domain match   (-m dom) :查找提取的用点（“."）分隔字符串，如果其中任何一个匹配，则ACL进行匹配
```

#### 5.8.1.5 ACL-value

value的类型

```bash
The ACL engine can match these types against patterns of the following types :
- Boolean #布尔值
- integer or integer range #整数或整数范围，比如用于匹配端口范围
- IP address / network #IP地址或IP范围, 192.168.0.1 ,192.168.0.1/24
- string--> www.dingbh.com
 exact 		#精确比较
 substring 	#子串
 suffix 	#后缀比较
 prefix 	#前缀比较
 subdir 	#路径， /wp-includes/js/jquery/jquery.js
 domain 	#域名，www.wang.com
- regular expression 	#正则表达式
- hex block 	#16进制
```

### 5.8.2 多个ACL的组合调用方式

**多个ACL的逻辑处理**

```bash
与：隐式（默认）使用
或：使用“or" 或 “||"表示
否定：使用 "!" 表示
```

**多个ACL调用方式：**

```
#示例：
if valid_src valid_port #与关系，ACL中A和B都要满足为true，默认为与
if invalid_src || invalid_port  #或，ACL中A或者B满足一个为true
if ! invalid_src #非，取反，不满足ACL才为true
```

### 5.8.3 ACL示例：域名匹配

范例：

```sh
#haproxy配置
[root@haproxy ~]# vim /etc/haproxy/conf.d/app.cfg 

frontend app_server_9000
        bind 192.168.188.101:9000
        acl pc_domain hdr_dom(host) -i www.dingbh.top
        acl mobile_domain hdr_dom(host) -i mobile.dingbh.top
#       use_backend pc_servers if pc_domain
        use_backend mobile_servers if mobile_domain
        default_backend pc_servers
        http-response set-header server dinginx-server

backend pc_servers
        server 11.0.1.102 11.0.1.102:80 check
        server backup 127.0.0.1:8080 check backup

backend mobile_servers
        server 11.0.1.103 11.0.1.103:80 check
        server backup 127.0.0.1:8080 check backup

```

测试

```bash
[root@internet ~]# curl www.dingbh.top:9000
11.0.1.102 pc.dingbh.top
[root@internet ~]# curl mobile.dingbh.top:9000
11.0.1.103 mobile.dingbh.top

```

### 5.8.4 ACL示例：基于源IP或子网调度访问

范例：

```sh
#haproxy配置
[root@haproxy ~]# vim /etc/haproxy/conf.d/app.cfg 

frontend app_server_9000
        bind 192.168.188.101:9000
        acl lan src 192.168.188.99   #基于源地址的ACL,定义多个ACL的顺序无关
        use_backend mobile_servers if lan	#放在前面的ACL规则优先生效,引用ACL时,严格的ACL应放在前面
        default_backend pc_servers
        http-response set-header server dinginx-server

backend pc_servers
        server 11.0.1.102 11.0.1.102:80 check
        server backup 127.0.0.1:8080 check backup

backend mobile_servers
        server 11.0.1.103 11.0.1.103:80 check
        server backup 127.0.0.1:8080 check backup

[root@haproxy ~]# systemctl restart haproxy.service
```

测试访问

```sh
[root@internet ~]# curl 192.168.188.101:9000
11.0.1.102 pc.dingbh.top
[root@internet ~]# curl 192.168.188.101:9000
11.0.1.102 pc.dingbh.top

```

![image-20240604191511582](Haproxy/image-20240604191511582.png)

### 5.8.5 ACL示例：基于源地址的访问控制

```
[root@haproxy ~]# vim /etc/haproxy/conf.d/app.cfg 

frontend app_server_9000
        bind 192.168.188.101:9000
#       acl pc_domain hdr_dom(host) -i www.dingbh.top
#       acl mobile_domain hdr_dom(host) -i mobile.dingbh.top
#       use_backend pc_servers if pc_domain
#       use_backend mobile_servers if mobile_domain
#       default_backend pc_servers
        acl lan src 192.168.188.88
#        use_backend mobile_servers if ! lan
        http-request deny if lan
        default_backend pc_servers
        http-response set-header server dinginx-server

backend pc_servers
        server 11.0.1.102 11.0.1.102:80 check
        server backup 127.0.0.1:8080 check backup

backend mobile_servers
        server 11.0.1.103 11.0.1.103:80 check
        server backup 127.0.0.1:8080 check backup

```

访问测试

```
[root@internet ~]# curl 192.168.188.101:9000
<html><body><h1>403 Forbidden</h1>
Request forbidden by administrative rules.
</body></html>
[root@internet ~]# curl 192.168.188.101:9000
<html><body><h1>403 Forbidden</h1>
Request forbidden by administrative rules.
</body></html>
[root@internet ~]# 
```

![image-20240604192510625](Haproxy/image-20240604192510625.png)

### 5.8.6 ACL示例：匹配浏览器类型

```sh
[root@haproxy ~]# vim /etc/haproxy/conf.d/app.cfg 

frontend app_server_9000
        bind 192.168.188.101:9000
        http-response set-header server dinginx-server
#ACL策略规则
        acl bad-agent hdr_sub(User-Agent) -i curl wget
        http-request deny if bad-agent
        default_backend pc_servers


backend pc_servers
        server 11.0.1.102 11.0.1.102:80 check
        server backup 127.0.0.1:8080 check backup

backend mobile_servers
        server 11.0.1.103 11.0.1.103:80 check
        server backup 127.0.0.1:8080 check back
```

测试访问

```html
[root@internet ~]# curl 192.168.188.101:9000
<html><body><h1>403 Forbidden</h1>
Request forbidden by administrative rules.
</body></html>
```

## 6.9 **自定义** **HAProxy** **错误界面**

对指定的报错进行重定向，进行优雅的显示错误页面

使用errorfile和errorloc指令的两种方法，可以实现自定义各种错误页面

默认情况下，所有后端服务器都down机后，会显示下面页面

![image-20240604193213866](Haproxy/image-20240604193213866.png)

### **5.9.1** **基于自定义的错误页面文件**

**注意：页面文件格式错误可能会导致Haproxy服务无法启动**

```html
#自定义错误页
errorfile <code> <file> 
<code> #HTTP status code.支持200, 400, 403, 405, 408, 425, 429, 500, 502，503,504
<file> #包含完整HTTP响应头的错误页文件的绝对路径。 建议后缀为".http"，以和一般的html文件相区分
```

**范例：403forbid错误界面实例**

```html
[root@haproxy ~]# vim /etc/haproxy/errorfile/403forbid.http 
HTTP/1.1 403 Forbidden
cache-control: no-cache
content-type: text/html

<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>精彩即将呈现</title>
<style>
    body {
        margin: 0;
        padding: 0;
        display: flex;
        justify-content: center; /* 水平居中 */
        align-items: center; /* 垂直居中 */
        height: 100vh; /* 让内容占据整个视口高度 */
    }

    .content {
        text-align: center; /* 水平居中 */
    }
</style>
</head>
<body>
<div class="content">
    <h1>精彩即将呈现 ！请稍后...... </h1>
    <p>请联系管理员或访问备用地址:www.dinginx.com</p>
    <tel>联系电话:400-888-8888</tel>
</div>
</body>
</html>
```

### 5.9.2 基于http重定向错误页面

```
#错误页面重定向
errorloc <code> <url>
#相当于errorloc302 <code> <url>，利用302重定向至指URL
#示例：
errorloc 503 http://www.dinginx.com/error_pages/503.html
errorloc 503 http://www.digninx.com/
```

范例：

```bash
#haproxy配置
[root@haproxy ~]# vim /etc/haproxy/conf.d/app.cfg 

frontend app_server_9000
        bind 192.168.188.101:9000
        acl lan src 192.168.188.99
        http-request deny if lan
        default_backend pc_servers
        errorfile 403 /etc/haproxy/errorfile/403forbid.http		#错误界面路径及配置

backend pc_servers
        server 11.0.1.102 11.0.1.102:80 check
        server backup 127.0.0.1:8080 check backup

backend mobile_servers
        server 11.0.1.103 11.0.1.103:80 check
        server backup 127.0.0.1:8080 check backup
```

**访问测试**

```sh
[root@internet ~]# echo `hostname -I`;curl 192.168.188.101:9000
192.168.188.88
11.0.1.102 pc.dingbh.top
```

**windos主机.99访问测试**

![image-20240607130241455](Haproxy/image-20240607130241455.png)

## 5.10 haproxy四层负载

针对除HTTP以外的TCP协议应用服务访问的应用场景

```
MySQL
Redis
Memcached
RabbitMQ
```

问题: 后端服务器到底是与haproxy还是和客户端建立三次握手呢?

### 5.10.1 四层负载示例

注意：如果使用frontend和backend，一定在 frontend 和 backend 段中都指定mode tcp 

```
listen redis-port
   bind 10.0.0.7:6379
   mode tcp
   balance leastconn
   server server1 10.0.0.17:6379 check
   server server2 10.0.0.27:6379 check backup
```

范例：对 MySQL 服务实现四层负载

```sh
[root@centos7 ~]#vim /etc/haproxy/haproxy.cfg
listen wang_mysql
 bind 10.0.0.7:3306
 mode tcp
 balance leastconn
 server mysql1 10.0.0.17:3306 check  
   server mysql2 10.0.0.27:3306 check           #如果不写端口号，可以转发，但无法
check状态
#或者使用frontend和backend实现
frontend mysql
       bind :3306
       mode tcp   #必须指定tcp模式
       default_backend mysqlsrvs
backend mysqlsrvs
       mode tcp #必须指定tcp模式
       balance leastconn
       server mysql1 10.0.0.17:3306
       server mysql2 10.0.0.27:3306
[root@centos7 ~]#systemctl restart haproxy
#在后端服务器安装和配置mariadb服务
[root@centos7 ~]#yum -y install mariadb-server
[root@centos7 ~]#mysql -e "grant all on *.* to test@'10.0.0.%' identified by 
'123456'"
[root@centos7 ~]#vim /etc/my.cnf
[mysqld]
server-id=17 #在另一台主机为27
[root@centos7 ~]#systemctl start mariadb
#测试
[root@centos6 ~]#mysql -utest -p123456 -e "show variables like 'hostname'"
+---------------+--------------------------+
| Variable_name | Value                   |
+---------------+--------------------------+
| hostname     | centos17.wangxiaochu.com |
+---------------+--------------------------+
[root@centos6 ~]#mysql -utest -p123456 -e "show variables like 'hostname'"
+---------------+--------------------------+
| Variable_name | Value                   |
+---------------+--------------------------+
| hostname     | centos27.wangxiaochu.com |
+---------------+--------------------------+
[root@centos6 ~]#mysql -utest -p123456 -h10.0.0.7 -e 'select @@server_id'
+-------------+
| @@server_id |
+-------------+
|          17 |
+-------------+
[root@centos6 ~]#mysql -utest -p123456 -h10.0.0.7 -e 'select @@server_id'
+-------------+
| @@server_id |
+-------------+
|          27 |
+-------------+
```

### 5.10.2 ACL示例-四层访问控制

```sh
frontend web_host
 bind 10.0.0.7:80
 mode http
 balance roundrobin
 log global
 option httplog
###################### acl setting ###############################  
 acl static_path path_beg  -i /static /images /javascript
 acl invalid_src src 192.168.1.0/24 10.0.0.8
###################### acl hosts #################################
 use_backend static_path_host if HTTP_1.1 TRUE static_path
 tcp-request connection reject if invalid_src     #四层ACL控制  
 default_backend default_web 
################### backend hosts ################################
backend php_server_host
 mode http
 server web1 10.0.0.17 check inter 2000 fall 3 rise 5
  
backend static_path_host
 mode http
 server web1 10.0.0.27 check inter 2000 fall 3 rise 5
  
backend default_web
 mode http
 server web1 10.0.0.37:80 check inter 2000 fall 3 rise 5
```

## 5.11 HAProxy Https 实现

haproxy支持https,基于性能考虑,证书是在后端服务器比如nginx上实现,即用户到haproxy利用tcp模式再到后端服务器

**范例: 基于tcp 模式实现**

### 5.11.1 生成证书

```sh
[root@haproxy /etc/haproxy/certs]# openssl genrsa -out haproxy.key 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
.......................................................+++++
.....................................+++++
e is 65537 (0x010001)
[root@haproxy /etc/haproxy/certs]# openssl req -new -x509 -key haproxy.key -out haproxy.crt -subj "/CN=www.dinginx.top" -days 365

[root@haproxy /etc/haproxy/certs]# cat haproxy.crt haproxy.key > haproxy.pem
[root@haproxy /etc/haproxy/certs]# cat haproxy.pem 
-----BEGIN CERTIFICATE-----
MIIDFTCCAf2gAwIBAgIURDkUgbjP2lQlsnw+ry6xt2FFG1wwDQYJKoZIhvcNAQEL
BQAwGjEYMBYGA1UEAwwPd3d3LmRpbmdpbngudG9wMB4XDTI0MDYwNzA1NDQyMFoX
DTI1MDYwNzA1NDQyMFowGjEYMBYGA1UEAwwPd3d3LmRpbmdpbngudG9wMIIBIjAN
BgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA6Bxz8pGIQkGtaBfF4V1nas8xB0UX
0tZ4MWES2BCbL2nAbV/FSOQGVv7afjZoDE9BDNh8npMcrw1/oe2YXrgw+fXtSAgS
LhONCVn0m1cuLdnxPCGOzllu8darMNGC3ZmxVlwo2VLVL3/ileDJB++hZ5EOjLCD
U4Cmh/DImUtPTHCz1l0f6DhkpA6Remn/dxwoK9Jt0OCmCtia4KzwcoCUEZxHZHDK
6P1fAgyZTr4TqtBtzWXSLopaDyL3CGmBU9UdRXeEo+04sRNScrfpLN0zAEHIY6LV
7IqsF7YvXO3a8L3UAb03sJf0ueKqFMOnyUvjbSOcEdIwMRDIRfdumidVoQIDAQAB
o1MwUTAdBgNVHQ4EFgQUI8cwAyUY92k5SySgWEyTvShTOUYwHwYDVR0jBBgwFoAU
I8cwAyUY92k5SySgWEyTvShTOUYwDwYDVR0TAQH/BAUwAwEB/zANBgkqhkiG9w0B
AQsFAAOCAQEAhD3x8K2+XY4BzP5BYrPNoY11OOKuiFc28xAu5QNxMU8sLUKOOHHk
RHPbraJYDdPvvvhlX45Ge2OiU9mMbjNy2PkB6YioZCTA74P/Bt/7ME9vwMQhadak
YHXG7WzWyWu5v2VwqncsuwF/6mLXk7VtNv6JlrzlHbMPz0MGb2bynkNoJMrDDtWW
DMpEowK4BVJm1LoiK4T9QmUMwO9ZKU98QummvNFv2upbvxajWTwNbP+DvEatDuYk
YYWho1DbzXeT0FF/yjNyWevxohjL3UkGnsV0B8xmGRQvQgkPhUnhap4R6u+qKfPW
vm9Ps8PiAz4PzG8noQIrSIBdrUbYxmy2HQ==
-----END CERTIFICATE-----
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEA6Bxz8pGIQkGtaBfF4V1nas8xB0UX0tZ4MWES2BCbL2nAbV/F
SOQGVv7afjZoDE9BDNh8npMcrw1/oe2YXrgw+fXtSAgSLhONCVn0m1cuLdnxPCGO
zllu8darMNGC3ZmxVlwo2VLVL3/ileDJB++hZ5EOjLCDU4Cmh/DImUtPTHCz1l0f
6DhkpA6Remn/dxwoK9Jt0OCmCtia4KzwcoCUEZxHZHDK6P1fAgyZTr4TqtBtzWXS
LopaDyL3CGmBU9UdRXeEo+04sRNScrfpLN0zAEHIY6LV7IqsF7YvXO3a8L3UAb03
sJf0ueKqFMOnyUvjbSOcEdIwMRDIRfdumidVoQIDAQABAoIBADH6BmEhQsR5Xbv+
+mVm9iCCUh0tnE6STTjcnPEWykMwMvWZ3GGwb0kGAMeSMV8NJYV9i3+dT1zFvEiC
Xv9uC2WWtSuASWHQZdqwmsFjyRaoSY/X7eayTPgXb67IPbQ3Jn7gww7gDfT3RTNe
PgrOZuNk0UwrqWsuVS0KBDc1hvJ/HMVgGgvvrfVqocS4ZSU4RhfWrQ3hOZNWKgOu
O2DNWEYlgVFYHP4jxhKlvhmeCzLOcOtrDgQWmMDik363GSxPoS1KEOFcesl6tYWO
b5RxbaEuQR2QpoeSq8ZkPo+XCJOLOVD08riN5Ir2Fj2tVV3beLOyOc+0TPWYtN6a
sULyOAECgYEA+vTlH47im7psSDTyNNVYRpooY3O2dAFSu/Cqr4koXpo3eSS1SxVW
O3GYbuik37NaJ9MjCh5hEL44cFePK8Oe2SKyTASuZym1EPJdAQVMBfG1uXF0Dj58
ibvZL8ls/DxuiyMcsAbpJCq6k2TOiOlKDDYeZmCZuSAMS7KzJeUjycECgYEA7Maa
WrspOvNiMAeVQaGmEX9vkjfldzz/BBOnMsvnw54A8cFdzv5KYCsryLwNUsrbkH0j
BnyaL6ecM2VtI+tzd84roQVSvE20TYGpfrdXX+pkd6oJZ/Dwz4WbfShBiJxUa9yu
XCGgNW5BAwtBUNDvKZy+P2ZBnV2npWzVARJFw+ECgYBSN69lImvNd0sIcQg1FPxh
zRmUU5Gf+dsAJb3V2oq2S7IG52bZ3Xit6u50z5VMkLbluz+Mjh1g3eEhtR37yZAe
NUbSrunwdHKUxcKAlhqtC95sjeXm0MP+X+Osy0IAdVAGF4/Lf8gcCD+uEGW15zBS
Rdy5z8YtJ3m/xyD4NlT+QQKBgA1oRY9Dr9b9tx6fH02DGMM+4xN1hNHWq14QutP+
hdYm2UjXOTeiYJXHfAg21c5NXFd3LoeMgLKYHBkw8hgpmfU7wG6yvXeaMs58vBtH
yHPO8vdvsm6nTwuBNkfFAsj4iTM+YMUoxkigQKAqf/1Sh2hkzsUKRkbh6b9NRVZ1
MWwhAoGAW9pnyPfLpZzEnTt5+fuOHaX5Jj7DV56IUlXGzL4McW/b9TmiCqOlAp1Z
0JIQkpVIQn3K0xvgrAQKHTxiaSDqm0p2pD00bW8SdrM2g91y9z1d5xdddDi35Ia+
974dRounhpvtQwyI42AYrsZL0jYj/pJpvKySA1mSewXGWoXNSCk=
-----END RSA PRIVATE KEY-----
[root@haproxy /etc/haproxy/certs]# 

#查看证书
[root@haproxy /etc/haproxy/certs]# openssl x509 -in haproxy.pem -noout -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            44:39:14:81:b8:cf:da:54:25:b2:7c:3e:af:2e:b1:b7:61:45:1b:5c
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = www.dinginx.top
        Validity
            Not Before: Jun  7 05:44:20 2024 GMT
            Not After : Jun  7 05:44:20 2025 GMT
        Subject: CN = www.dinginx.top
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:e8:1c:73:f2:91:88:42:41:ad:68:17:c5:e1:5d:
                    67:6a:cf:31:07:45:17:d2:d6:78:31:61:12:d8:10:
                    9b:2f:69:c0:6d:5f:c5:48:e4:06:56:fe:da:7e:36:
                    68:0c:4f:41:0c:d8:7c:9e:93:1c:af:0d:7f:a1:ed:
                    98:5e:b8:30:f9:f5:ed:48:08:12:2e:13:8d:09:59:
                    f4:9b:57:2e:2d:d9:f1:3c:21:8e:ce:59:6e:f1:d6:
                    ab:30:d1:82:dd:99:b1:56:5c:28:d9:52:d5:2f:7f:
                    e2:95:e0:c9:07:ef:a1:67:91:0e:8c:b0:83:53:80:
                    a6:87:f0:c8:99:4b:4f:4c:70:b3:d6:5d:1f:e8:38:
                    64:a4:0e:91:7a:69:ff:77:1c:28:2b:d2:6d:d0:e0:
                    a6:0a:d8:9a:e0:ac:f0:72:80:94:11:9c:47:64:70:
                    ca:e8:fd:5f:02:0c:99:4e:be:13:aa:d0:6d:cd:65:
                    d2:2e:8a:5a:0f:22:f7:08:69:81:53:d5:1d:45:77:
                    84:a3:ed:38:b1:13:52:72:b7:e9:2c:dd:33:00:41:
                    c8:63:a2:d5:ec:8a:ac:17:b6:2f:5c:ed:da:f0:bd:
                    d4:01:bd:37:b0:97:f4:b9:e2:aa:14:c3:a7:c9:4b:
                    e3:6d:23:9c:11:d2:30:31:10:c8:45:f7:6e:9a:27:
                    55:a1
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Key Identifier: 
                23:C7:30:03:25:18:F7:69:39:4B:24:A0:58:4C:93:BD:28:53:39:46
            X509v3 Authority Key Identifier: 
                keyid:23:C7:30:03:25:18:F7:69:39:4B:24:A0:58:4C:93:BD:28:53:39:46

            X509v3 Basic Constraints: critical
                CA:TRUE
    Signature Algorithm: sha256WithRSAEncryption
         84:3d:f1:f0:ad:be:5d:8e:01:cc:fe:41:62:b3:cd:a1:8d:75:
         38:e2:ae:88:57:36:f3:10:2e:e5:03:71:31:4f:2c:2d:42:8e:
         38:71:e4:44:73:db:ad:a2:58:0d:d3:ef:be:f8:65:5f:8e:46:
         7b:63:a2:53:d9:8c:6e:33:72:d8:f9:01:e9:88:a8:64:24:c0:
         ef:83:ff:06:df:fb:30:4f:6f:c0:c4:21:69:d6:a4:60:75:c6:
         ed:6c:d6:c9:6b:b9:bf:65:70:aa:77:2c:bb:01:7f:ea:62:d7:
         93:b5:6d:36:fe:89:96:bc:e5:1d:b3:0f:cf:43:06:6f:66:f2:
         9e:43:68:24:ca:c3:0e:d5:96:0c:ca:44:a3:02:b8:05:52:66:
         d4:ba:22:2b:84:fd:42:65:0c:c0:ef:59:29:4f:7c:42:e9:a6:
         bc:d1:6f:da:ea:5b:bf:16:a3:59:3c:0d:6c:ff:83:bc:46:ad:
         0e:e6:24:61:85:a1:a3:50:db:cd:77:93:d0:51:7f:ca:33:72:
         59:eb:f1:a2:18:cb:dd:49:06:9e:c5:74:07:cc:66:19:14:2f:
         42:09:0f:85:49:e1:6a:9e:11:ea:ef:aa:29:f3:d6:be:6f:4f:
         b3:c3:e2:03:3e:0f:cc:6f:27:a1:02:2b:48:80:5d:ad:46:d8:
         c6:6c:b6:1d

```

### 5.11.2 HAProxy配置

```
#haproxy配置
[root@haproxy /etc/haproxy/certs]# grep -Ev '^#|^$' /etc/haproxy/conf.d/web.cfg 
frontend www.dingbh.top_80
        bind 192.168.188.101:80
 ************************添加以下两行配置实现ssl*******************************************
        bind 192.168.188.101:443 ssl crt /etc/haproxy/certs/haproxy.pem
		redirect scheme https if !{ ssl_fc }
 ***************************************************************************************
        use_backend web_servers
		http-response set-header server dinginx-server
	
	
backend web_servers
		option httpchk HEAD /index.html HTTP/1.1
		http-check send hdr Host www
        acl bad_agent hdr_sub(User-Agent) -i curl wget
        http-request deny if bad_agent
        server 11.0.1.102 11.0.1.102:80 check
        server 11.0.1.103 11.0.1.103:80 check
        server backup 127.0.0.1:8080 check backup
```

### **5.11.3 访问测试**

![image-20240607140804413](Haproxy/image-20240607140804413.png)

# END

```
HAProxy调度算法
ACL使用与报文修改
动静分离(根据文件后缀或URL)
客户端源IP透传
服务器动态下线
编写shell脚本，实现能够基于参数传递 Real Server 服务器IP，并实现将其从多个HAProxy进程下
线与上线
比较LVS,haproxy,nginx三者的特性和调度算法区别(面试题)
```

