# 非关系性数据库NoSQL

# 1、redis介绍

## 1.1、特性

```
#键值对数据库redis

速度快: 10W QPS,基于内存,C语言实现
单线程
持久化
支持多种数据结构
支持多种编程语言
功能丰富: 支持Lua脚本,发布订阅,事务,pipeline等功能
简单: 代码短小精悍(单机核心代码只有23000行左右),单线程开发容易,不依赖外部库,使用简单
主从复制
支持高可用和分布式
```

### 1.1.1、面试：#单线程为何如此快?

```
#单线程为何如此快?
1、纯内存
2、非阻塞：非阻塞的I/O多路复用模型
3、避免线程切换和竞态消耗：通过单线程依次处理多个客户端连接，避免线程切换和竞态消耗
```

查看版本

```
[root@Rocky8 ~]# yum info redis

BaseOS                                                                                                                                      11 kB/s | 3.9 kB     00:00    
AppStream                                                                                                                                  9.9 kB/s | 4.3 kB     00:00    
EPEL                                                                                                                                       868  B/s | 4.7 kB     00:05    
EPEL                                                                                                                                       1.5 MB/s |  16 MB     00:10    
extras                                                                                                                                     3.8 kB/s | 1.5 kB     00:00    
centosplus                                                                                                                                 277  B/s | 1.5 kB     00:05    
Available Packages
Name         : redis
Version      : 5.0.3
Release      : 5.module_el8.4.0+955+7126e393
Architecture : x86_64
Size         : 927 k
Source       : redis-5.0.3-5.module_el8.4.0+955+7126e393.src.rpm
Repository   : AppStream
Summary      : A persistent key-value database
URL          : http://redis.io
License      : BSD and MIT
Description  : Redis is an advanced key-value store. It is often referred to as a data
             : structure server since keys can contain strings, hashes, lists, sets and
             : sorted sets.
             : 
             : You can run atomic operations on these types, like appending to a string;
             : incrementing the value in a hash; pushing to a list; computing set
             : intersection, union and difference; or getting the member with highest
             : ranking in a sorted set.
             : 
             : In order to achieve its outstanding performance, Redis works with an
             : in-memory dataset. Depending on your use case, you can persist it either
             : by dumping the dataset to disk every once in a while, or by appending
             : each command to a log.
             : 
             : Redis also supports trivial-to-setup master-slave replication, with very
             : fast non-blocking first synchronization, auto-reconnection on net split
             : and so forth.
             : 
             : Other features include Transactions, Pub/Sub, Lua scripting, Keys with a
             : limited time-to-live, and configuration settings to make Redis behave like
             : a cache.
             : 
             : You can use Redis from most programming languages also.

```

### 1.1.2、**Redis** **常见应用场景**

```
缓存：缓存RDBMS中数据,比如网站的查询结果、商品信息、微博、新闻、消息
Session 共享：实现Web集群中的多服务器间的session共享
计数器：商品访问排行榜、浏览数、粉丝数、关注、点赞、评论等和次数相关的数值统计场景
社交：朋友圈、共同好友、可能认识他们等
地理位置: 基于地理信息系统GIS（Geographic Information System)实现摇一摇、附近的人、外卖
等功能
消息队列：ELK等日志系统缓存、业务的订阅/发布系统
```

## 1.2、安装Redis

### 1.2.1、rocky8源码安装redis

```
[root@Rocky8 ~]# yum -y install redis
[root@Rocky8 ~]# rpm -ql redis
/etc/logrotate.d/redis
/etc/redis-sentinel.conf
/etc/redis.conf
/etc/systemd/system/redis-sentinel.service.d
/etc/systemd/system/redis-sentinel.service.d/limit.conf
/etc/systemd/system/redis.service.d
/etc/systemd/system/redis.service.d/limit.conf
/usr/bin/redis-benchmark
/usr/bin/redis-check-aof
/usr/bin/redis-check-rdb
/usr/bin/redis-cli
/usr/bin/redis-sentinel
/usr/bin/redis-server
/usr/lib/.build-id
/usr/lib/.build-id/19
/usr/lib/.build-id/19/e1e3e9c658ca7411675798da3adfb25b456626
/usr/lib/.build-id/46
/usr/lib/.build-id/46/bd1f92df23c67a27dfd80d946a0266614c818a
/usr/lib/.build-id/ac
/usr/lib/.build-id/ac/73b6f68fff8c23ad408fc848ff7ec5a954ef53
/usr/lib/systemd/system/redis-sentinel.service
/usr/lib/systemd/system/redis.service
/usr/lib64/redis
/usr/lib64/redis/modules
/usr/libexec/redis-shutdown
/usr/share/licenses/redis
/usr/share/licenses/redis/COPYING
/usr/share/licenses/redis/COPYING-hiredis
/usr/share/licenses/redis/COPYING-jemalloc
/usr/share/licenses/redis/COPYRIGHT-lua
/usr/share/man/man1/redis-benchmark.1.gz
/usr/share/man/man1/redis-check-aof.1.gz
/usr/share/man/man1/redis-check-rdb.1.gz
/usr/share/man/man1/redis-cli.1.gz
/usr/share/man/man1/redis-sentinel.1.gz
/usr/share/man/man1/redis-server.1.gz
/usr/share/man/man5/redis-sentinel.conf.5.gz
/usr/share/man/man5/redis.conf.5.gz
/var/lib/redis
/var/log/redis
/var/run/redis
[root@Rocky8 ~]# systemctl enable --now redis
Created symlink /etc/systemd/system/multi-user.target.wants/redis.service → /usr/lib/systemd/system/redis.service.

[root@Rocky8 ~]# systemctl status redis
● redis.service - Redis persistent key-value database
   Loaded: loaded (/usr/lib/systemd/system/redis.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/redis.service.d
           └─limit.conf
   Active: active (running) since Tue 2023-08-29 13:22:13 CST; 4s ago
 Main PID: 31455 (redis-server)
    Tasks: 4 (limit: 23016)
   Memory: 6.6M
   CGroup: /system.slice/redis.service
           └─31455 /usr/bin/redis-server 127.0.0.1:6379

Aug 29 13:22:13 Rocky8.8 systemd[1]: Starting Redis persistent key-value database...
Aug 29 13:22:13 Rocky8.8 systemd[1]: Started Redis persistent key-value database.

#默认端口6379
[root@Rocky8 ~]# ss -ntl
State               Recv-Q              Send-Q                           Local Address:Port                            Peer Address:Port              Process              
LISTEN              0                   511                                  127.0.0.1:6379                                 0.0.0.0:*                                      
LISTEN              0                   128                                    0.0.0.0:111                                  0.0.0.0:*                                      
LISTEN              0                   128                                    0.0.0.0:22                                   0.0.0.0:*                                      
LISTEN              0                   128                                       [::]:111                                     [::]:*                                      
LISTEN              0                   128                                       [::]:22                                      [::]:*   
```

### 1.2.2、查看进入数据库

```
[root@Rocky8 ~]# redis-cli 
127.0.0.1:6379> info
# Server
redis_version:5.0.3
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:9529b692c0384fb7
redis_mode:standalone
os:Linux 4.18.0-477.10.1.el8_8.x86_64 x86_64
arch_bits:64
multiplexing_api:epoll
atomicvar_api:atomic-builtin
gcc_version:8.4.1
process_id:31455
run_id:547130055aa5dd075e8ecfe803bf90204747b799
tcp_port:6379
uptime_in_seconds:99
uptime_in_days:0
hz:10
configured_hz:10
lru_clock:15565032
executable:/usr/bin/redis-server
config_file:/etc/redis.conf

# Clients
connected_clients:1
client_recent_max_input_buffer:2
client_recent_max_output_buffer:0
blocked_clients:0

# Memory
used_memory:853896
used_memory_human:833.88K
used_memory_rss:19177472
used_memory_rss_human:18.29M
used_memory_peak:853896
used_memory_peak_human:833.88K
used_memory_peak_perc:100.12%
used_memory_overhead:840694
used_memory_startup:791000
used_memory_dataset:13202
used_memory_dataset_perc:20.99%
allocator_allocated:1477016
allocator_active:1806336
allocator_resident:10813440
total_system_memory:3811012608
total_system_memory_human:3.55G
used_memory_lua:37888
used_memory_lua_human:37.00K
used_memory_scripts:0
used_memory_scripts_human:0B
number_of_cached_scripts:0
maxmemory:0
maxmemory_human:0B
maxmemory_policy:noeviction
allocator_frag_ratio:1.22
allocator_frag_bytes:329320
allocator_rss_ratio:5.99
allocator_rss_bytes:9007104
rss_overhead_ratio:1.77
rss_overhead_bytes:8364032
mem_fragmentation_ratio:23.62
mem_fragmentation_bytes:18365576
mem_not_counted_for_evict:0
mem_replication_backlog:0
mem_clients_slaves:0
mem_clients_normal:49694
mem_aof_buffer:0
mem_allocator:jemalloc-5.1.0
active_defrag_running:0
lazyfree_pending_objects:0

# Persistence
loading:0
rdb_changes_since_last_save:0
rdb_bgsave_in_progress:0
rdb_last_save_time:1693286533
rdb_last_bgsave_status:ok
rdb_last_bgsave_time_sec:-1
rdb_current_bgsave_time_sec:-1
rdb_last_cow_size:0
aof_enabled:0
aof_rewrite_in_progress:0
aof_rewrite_scheduled:0
aof_last_rewrite_time_sec:-1
aof_current_rewrite_time_sec:-1
aof_last_bgrewrite_status:ok
aof_last_write_status:ok
aof_last_cow_size:0

# Stats
total_connections_received:1
total_commands_processed:1
instantaneous_ops_per_sec:0
total_net_input_bytes:31
total_net_output_bytes:11468
instantaneous_input_kbps:0.00
instantaneous_output_kbps:0.00
rejected_connections:0
sync_full:0
sync_partial_ok:0
sync_partial_err:0
expired_keys:0
expired_stale_perc:0.00
expired_time_cap_reached_count:0
evicted_keys:0
keyspace_hits:0
keyspace_misses:0
pubsub_channels:0
pubsub_patterns:0
latest_fork_usec:0
migrate_cached_sockets:0
slave_expires_tracked_keys:0
active_defrag_hits:0
active_defrag_misses:0
active_defrag_key_hits:0
active_defrag_key_misses:0

# Replication
role:master
connected_slaves:0
master_replid:7ded7e1eb5893c377c3fd4f45059abe0bec6445b
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

# CPU
used_cpu_sys:0.122103
used_cpu_user:0.040787
used_cpu_sys_children:0.000000
used_cpu_user_children:0.000000

# Cluster
cluster_enabled:0

# Keyspace
127.0.0.1:6379> quit

[root@Rocky8 ~]# pstree -p

           ├─redis-server(31455)─┬─{redis-server}(31456)
           │                     ├─{redis-server}(31457)
           │                     └─{redis-server}(31458)
#健康度检查，支持非交互
[root@Rocky8 ~]# redis-cli 
127.0.0.1:6379> ping
PONG   /PONG为正常
```

### 1.2.3、编译安装Redis

```
#安装相关依赖
[root@Rocky8 ~/redis-6.2.13]# yum -y install gcc make jemalloc-devel

#如果支持systemd需要安装下面包
[root@centos8~]#yum -y install gcc jemalloc-devel systemd-devel
[root@ubuntu2004 ~]#apt -y install make gcc libjemalloc-dev libsystemd-dev

#下载编译redis包
[root@Rocky8 ~]# wget https://download.redis.io/releases/redis-6.2.13.tar.gz
[root@Rocky8 ~]# tar xf redis-6.2.13.tar.gz 
[root@Rocky8 ~]# ls
anaconda-ks.cfg  redis-6.2.13  redis-6.2.13.tar.gz
[root@Rocky8 ~]# cd redis-6.2.13/
[root@Rocky8 ~/redis-6.2.13]# ls
00-RELEASENOTES  CONDUCT       COPYING  INSTALL   MANIFESTO  redis.conf  runtest-cluster    runtest-sentinel  src    TLS.md
BUGS             CONTRIBUTING  deps     Makefile  README.md  runtest     runtest-moduleapi  sentinel.conf     tests  utils
[root@Rocky8 ~/redis-6.2.13]# make -j 2 USE_SYSTEMD=yes PREFIX=/apps/redis install

[root@Rocky8 ~/redis-6.2.13]# echo 'PATH=/apps/redis/bin:$PATH' > /etc/profile.d/redis.sh

#前台执行
[root@Rocky8 /]# redis-server 
38014:C 29 Aug 2023 13:58:50.726 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
38014:C 29 Aug 2023 13:58:50.726 # Redis version=6.2.13, bits=64, commit=00000000, modified=0, pid=38014, just started
38014:C 29 Aug 2023 13:58:50.726 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
38014:M 29 Aug 2023 13:58:50.726 * Increased maximum number of open files to 10032 (it was originally set to 1024).
38014:M 29 Aug 2023 13:58:50.726 * monotonic clock: POSIX clock_gettime
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 6.2.13 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                  
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 38014
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           https://redis.io       
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

38014:M 29 Aug 2023 13:58:50.726 # Server initialized
38014:M 29 Aug 2023 13:58:50.726 # WARNING Memory overcommit must be enabled! Without it, a background save or replication may fail under low memory condition. Being disabled, it can can also cause failures without low memory condition, see https://github.com/jemalloc/jemalloc/issues/1328. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
38014:M 29 Aug 2023 13:58:50.727 * Ready to accept connections


[root@Rocky8 ~]# mkdir /apps/redis/{etc,data,log,run}
[root@Rocky8 ~]# cd redis-6.2.13/
[root@Rocky8 ~/redis-6.2.13]# cp redis.conf /apps/redis/etc/
#创建redis账号
[root@Rocky8 ~]#useradd -r -s /sbin/nologin redis
[root@Rocky8 ~]# chown redis.redis /apps/redis/ -R
[root@Rocky8 ~]# ls /apps/redis/ -l
total 0
drwxr-xr-x 2 redis redis 134 Aug 29 13:53 bin
drwxr-xr-x 2 redis redis   6 Aug 29 21:35 data
drwxr-xr-x 2 redis redis   6 Aug 29 21:35 etc
drwxr-xr-x 2 redis redis   6 Aug 29 21:35 log
drwxr-xr-x 2 redis redis   6 Aug 29 21:35 run

/*查看redis.conf文件：grep -Ev "^#|^$" etc/redis.conf 
*/

#创建相关配置文件目录
[root@Rocky8 ~]# vim /etc/sysctl.conf
vm.overcommit_memory=1   
[root@Rocky8 ~]# sysctl -p
vm.overcommit_memory = 1
#此项默认为128，redis官方建议511
[root@Rocky8 /apps/redis]# cat /proc/sys/net/core/somaxconn 
2048

#编写redis.service文件，实现后台执行
[root@Rocky8 /apps/redis]# vim /lib/systemd/system/redis.service

[Unit]
Description=Redis persistent key-value database
After=network.target

[Service]
ExecStart=/apps/redis/bin/redis-server /apps/redis/etc/redis.conf --supervised systemd
ExecStop=/bin/kill -s QUIT $MAINPID
Type=notify #如果支持systemd可以启用此行
User=redis
Group=redis
RuntimeDirectory=redis
RuntimeDirectoryMode=0755
LimitNOFILE=1000000   #指定此值才支持更大的maxclients值

[Install]
WantedBy=multi-user.target
#启动redis
[root@Rocky8 /apps/redis]# systemctl daemon-reload
[root@Rocky8 /apps/redis]# systemctl enable --now redis.service
Created symlink /etc/systemd/system/multi-user.target.wants/redis.service → /usr/lib/systemd/system/redis.service.
[root@Rocky8 /apps/redis]# systemctl status redis.service 
```

### 1.2.4、以编译安装为例实现 redis 多实例

```
#生成的文件列表

[root@Rocky8 ~]# ls /apps/redis/ -l
total 4
drwxr-xr-x 2 redis redis 134 Aug 29 13:53 bin
drwxr-xr-x 2 redis redis   6 Aug 29 21:35 data
-rw-r--r-- 1 root  root   93 Aug 29 21:48 dump.rdb
drwxr-xr-x 2 redis redis  72 Aug 29 22:46 etc
drwxr-xr-x 2 redis redis   6 Aug 29 21:35 log
drwxr-xr-x 2 redis redis   6 Aug 29 21:35 run

[root@Rocky8 /apps/redis]# cat /apps/redis/etc/redis6379.conf |grep -Ev '^#|^$'
bind 0.0.0.0 -::1
protected-mode yes
port 6379
tcp-backlog 511
timeout 0
tcp-keepalive 300
daemonize no
pidfile /var/run/redis_6379.pid
loglevel notice
logfile ""
databases 16
always-show-logo no
set-proc-title yes
proc-title-template "{title} {listen-addr} {server-mode}"
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
rdb-del-sync-files no
dir ./
replica-serve-stale-data yes
replica-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-diskless-load disabled
repl-disable-tcp-nodelay no
replica-priority 100
acllog-max-len 128
lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
replica-lazy-flush no
lazyfree-lazy-user-del no
lazyfree-lazy-user-flush no
oom-score-adj no
oom-score-adj-values 0 200 800
disable-thp yes
appendonly no
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
aof-use-rdb-preamble yes
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
stream-node-max-bytes 4096
stream-node-max-entries 100
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit replica 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
dynamic-hz yes
aof-rewrite-incremental-fsync yes
rdb-save-incremental-fsync yes
jemalloc-bg-thread yes

[root@Rocky8 ~]# sed 's/6379/6380/' /apps/redis/etc/redis6379.conf > /apps/redis/etc/redis6380.conf
[root@Rocky8 ~]# sed 's/6379/6381/' /apps/redis/etc/redis6379.conf > /apps/redis/etc/redis6381.conf

#生成service文件
[root@Rocky8 ~]# cat /lib/systemd/system/redis6379.service 
[Unit]
Description=Redis persistent key-value database
After=network.target

[Service]
ExecStart=/apps/redis/bin/redis-server /apps/redis/etc/redis6379.conf --supervised systemd
ExecStop=/bin/kill -s QUIT $MAINPID
Type=notify
User=redis
Group=redis
RuntimeDirectory=redis
RuntimeDirectoryMode=0755
LimitNOFILE=1000000

[Install]
WantedBy=multi-user.target

[root@Rocky8 ~]# sed 's/6379/6380/' /lib/systemd/system/redis6379.service >  /lib/systemd/system/redis6380.service

[root@Rocky8 ~]# sed 's/6379/6381/' /lib/systemd/system/redis6379.service >  /lib/systemd/system/redis6381.service

[root@Rocky8 ~]# systemctl daemon-reload
[root@Rocky8 ~]# systemctl enable --now redis6379 redis6380 redis6381

[root@Rocky8 ~]# ss -ntl
State               Recv-Q              Send-Q                           Local Address:Port                            Peer Address:Port              Process              
LISTEN              0                   511                                    0.0.0.0:6379                                 0.0.0.0:*                                      
LISTEN              0                   511                                    0.0.0.0:6380                                 0.0.0.0:*                                      
LISTEN              0                   511                                    0.0.0.0:6381                                 0.0.0.0:*                                      
LISTEN              0                   128                                    0.0.0.0:111                                  0.0.0.0:*                                      
LISTEN              0                   128                                    0.0.0.0:22                                   0.0.0.0:*                                      
LISTEN              0                   511                                      [::1]:6379                                    [::]:*                                      
LISTEN              0                   511                                      [::1]:6380                                    [::]:*                                      
LISTEN              0                   511                                      [::1]:6381                                    [::]:*                                      
LISTEN              0                   128                                       [::]:111                                     [::]:*                                      
LISTEN              0                   128                                       [::]:22                                      [::]:*            
```

### 1.2.5、**Redis** **相关工具和客户端连接**

#### **1.2.5.1** **安装的相关程序介绍**

```
[root@Rocky8 ~]# ll /apps/redis/bin/
total 24920
-rwxr-xr-x 1 redis redis  6552808 Aug 29 13:53 redis-benchmark    #性能测试程序
lrwxrwxrwx 1 redis redis       12 Aug 29 13:53 redis-check-aof -> redis-server   #AOF文件检查程序
lrwxrwxrwx 1 redis redis       12 Aug 29 13:53 redis-check-rdb -> redis-server   #RDB文件检查程序
-rwxr-xr-x 1 redis redis  6769224 Aug 29 13:53 redis-cli          #客户端程序
lrwxrwxrwx 1 redis redis       12 Aug 29 13:53 redis-sentinel -> redis-server    #哨兵程
序，软连接到服务器端主程序
-rwxr-xr-x 1 redis redis 12190896 Aug 29 13:53 redis-server       #服务端主程序

```

**1.2.5.2** **客户端程序** **redis-cli**

```
#默认为本机无密码连接
redis-cli
#远程客户端连接,注意:Redis没有用户的概念
redis-cli -h <Redis服务器IP> -p <PORT> -a <PASSWORD>

#实例：
[root@Rocky8 ~]# redis-cli -p 6379 ping
PONG
[root@Rocky8 ~]# redis-cli -p 6379 info
```

**1.2.5.3** **python访问程序连接Redis**

```
Redis 支持多种开发语言访问
```

##### ①python访问

```
#下载python环境
[root@Rocky8 ~]# yum -y install python3
[root@Rocky8 ~]# rpm -qf `which pip3`
[root@Rocky8 ~]# pip3 install redis

#创建脚本批量写入数据
[root@Rocky8 ~]# cat redis_test.py 
#!/usr/bin/python3 
import redis
pool = redis.ConnectionPool(host="127.0.0.1",port=6379,password="",decode_responses=True)
r = redis.Redis(connection_pool=pool)
for i in range(100000):
    r.set("k%d" % i,"v%d" % i)
    data=r.get("k%d" % i)
    print(data)
[root@Rocky8 ~]# chmod +x redis_test.py ^C
[root@Rocky8 ~]# ./redis_test.py 


[root@centos8 ~]#redis-cli 
127.0.0.1:6379> get k10
"v10"
127.0.0.1:6379> GET class
"66"
```

##### ②修改最大内存数

```
[root@Rocky8 ~]# redis-cli 
127.0.0.1:6379> CONFIG GET maxmemory

1) "maxmemory"
2) "0"
   127.0.0.1:6379> CONFIG SET maxmemory 1905506304
   OK
   127.0.0.1:6379> CONFIG GET maxmemory
3) "maxmemory"
4) "1905506304"
```

## 1.3、**慢查询**

```
#查看关于慢查询的日志slowlog
[root@Rocky8 ~]# grep slowlog /apps/redis/etc/redis6379.conf 
slowlog-log-slower-than 10000
slowlog-max-len 128
127.0.0.1:6379> SLOWLOG len
(integer) 0
127.0.0.1:6379> CONFIG set slowlog-log-slower-than 1
OK
127.0.0.1:6379> CONFIG GET slowlog-log-slower-than
1) "slowlog-log-slower-than"
2) "1"
127.0.0.1:6379> SLOWLOG len
(integer) 2
127.0.0.1:6379> slowlog get
1) 1) (integer) 2
   2) (integer) 1693329358
   3) (integer) 2
   4) 1) "SLOWLOG"
      2) "len"
   5) "127.0.0.1:54910"
   6) ""
2) 1) (integer) 1
   2) (integer) 1693329355
   3) (integer) 10
   4) 1) "CONFIG"
      2) "GET"
      3) "slowlog-log-slower-than"
   5) "127.0.0.1:54910"
   6) ""
3) 1) (integer) 0
   2) (integer) 1693329351
   3) (integer) 6
   4) 1) "CONFIG"
      2) "set"
      3) "slowlog-log-slower-than"
      4) "1"
   5) "127.0.0.1:54910"
   6) ""

#清空慢查询日志
127.0.0.1:6379> slowlog reset


#永久保存slowlog配置
[root@Rocky8 ~]# vim /apps/redis/etc/redis6379.conf 

slowlog-log-slower-than 1000  

#日志相关介绍
[root@centos8 ~]#vim /etc/redis.conf
slowlog-log-slower-than 1    #指定超过1us即为慢的指令，默认值为10000us
slowlog-max-len 1024         #指定只保存最近的1024条慢记录，默认值为128
127.0.0.1:6379> SLOWLOG LEN  #查看慢日志的记录条数
(integer) 14
127.0.0.1:6379> SLOWLOG GET [n] #查看慢日志的n条记录
1) 1) (integer) 14
2) (integer) 1544690617
3) (integer) 4                #第3)行表示每条指令的执行时长
4) 1) "slowlog"
127.0.0.1:6379> SLOWLOG GET 3
1) 1) (integer) 7
   2) (integer) 1602901545
   3) (integer) 26
   4) 1) "SLOWLOG"
      2) "get"
   5) "127.0.0.1:38258"
   6) ""
2) 1) (integer) 6
   2) (integer) 1602901540
   3) (integer) 22
   4) 1) "SLOWLOG"
      2) "get"
      3) "2"
   5) "127.0.0.1:38258"
   6) ""
3) 1) (integer) 5
   2) (integer) 1602901497
   3) (integer) 22
   4) 1) "SLOWLOG"
      2) "GET"
   5) "127.0.0.1:38258"
   6) ""
127.0.0.1:6379> SLOWLOG RESET #清空慢日志
OK
```

## 1.4、Redis持久化

```
Redis支持两种数据持久化保存方法
RDB:Redis DataBase
AOF:AppendOnlyFile

1、实现 RDB 方法
save: 同步,不推荐使用，使用主进程完成快照，因此会阻赛其它命令执行
bgsave: 异步后台执行,不影响其它命令的执行，会开启独立的子进程，因此不会阻赛其它命令执行
配置文件实现自动保存: 在配置文件中制定规则,自动执行bgsave
2、RDB 模式的优缺点
RDB 模式优点：
RDB快照只保存某个时间点的数据，恢复的时候直接加载到内存即可，不用做其他处理，这种文件
适合用于做灾备处理.可以通过自定义时间点执行redis指令bgsave或者save保存快照，实现多个版
本的备份
比如: 可以在最近的24小时内，每小时备份一次RDB文件，并且在每个月的每一天，也备份一个
RDB文件。这样的话，即使遇上问题，也可以随时将数据集还原到指定的不同的版本。
RDB在大数据集时恢复的速度比AOF方式要快c

RDB 模式缺点
不能实时保存数据，可能会丢失自上一次执行RDB备份到当前的内存数据
如果需要尽量避免在服务器故障时丢失数据，那么RDB并不适合。虽然Redis允许设置不同的保存
点（save point）来控制保存RDB文件的频率，但是，因为RDB文件需要保存整个数据集的状态，
所以它可能并不是一个非常快速的操作。因此一般会超过5分钟以上才保存一次RDB文件。在这种
情况下，一旦发生故障停机，就可能会丢失较长时间的数据。
在数据集比较庞大时，fork()子进程可能会非常耗时，造成服务器在一定时间内停止处理客户端请
求,如果数据集非常巨大，并且CPU时间非常紧张的话，那么这种停止时间甚至可能会长达整整一秒
或更久。另外子进程完成生成RDB文件的时间也会花更长时间.
```

### 1.4.1、RDB（bgsave）

```
#查看当前配置
[root@Rocky8 ~]# grep ^dir /apps/redis/etc/redis6379.conf 
dir /apps/redis/data
[root@Rocky8 ~]# grep ^dbfilename /apps/redis/etc/redis6379.conf 
dbfilename dump6379.rdb

#开启并修改RDB保存路径
[root@Rocky8 ~]# vim /apps/redis/etc/redis6379.conf 
 433 dbfilename dump6379.rdb  
 457 dir /apps/redis/data


#  “&”后台执行
[root@Rocky8 ~]# redis-cli save&  pstree -p |grep redis ;ll /apps/redis/data/    
[1] 41362
           |-redis-server(40936)-+-{redis-server}(40944)
           |                     |-{redis-server}(40945)
           |                     |-{redis-server}(40946)
           |                     `-{redis-server}(40947)
           |-redis-server(40949)-+-{redis-server}(40955)
           |                     |-{redis-server}(40956)
           |                     |-{redis-server}(40957)
           |                     `-{redis-server}(40958)
           |-redis-server(41347)-+-{redis-server}(41348)
           |                     |-{redis-server}(41349)
           |                     |-{redis-server}(41350)
           |                     `-{redis-server}(41351)
           |-sshd(814)-+-sshd(39244)---sshd(39246)---bash(39247)---redis-cli(41353)
           |                                                     `-redis-cli(41362)
total 185608
-rw-r--r-- 1 root  root  189855682 Aug 30 01:38 dump6379.rdb
-rw-r--r-- 1 root  root         93 Aug 29 23:16 dump6380.rdb
-rw-r--r-- 1 root  root         93 Aug 29 23:16 dump6381.rdb
-rw-r--r-- 1 redis redis     77824 Aug 30 01:45 temp-41347.rdb


# bgsave默认后台执行
[root@Rocky8 ~]# redis-cli bgsave ;pstree -p |grep redis ; ll /apps/redis/data/
Background saving started
           |-redis-server(40936)-+-{redis-server}(40944)
           |                     |-{redis-server}(40945)
           |                     |-{redis-server}(40946)
           |                     `-{redis-server}(40947)
           |-redis-server(40949)-+-{redis-server}(40955)
           |                     |-{redis-server}(40956)
           |                     |-{redis-server}(40957)
           |                     `-{redis-server}(40958)
           |-redis-server(41347)-+-redis-server(41383)
           |                     |-{redis-server}(41348)
           |                     |-{redis-server}(41349)
           |                     |-{redis-server}(41350)
           |                     `-{redis-server}(41351)
           |-sshd(814)-+-sshd(39244)---sshd(39246)---bash(39247)---redis-cli(41353)
total 185608
-rw-r--r-- 1 redis redis 189855683 Aug 30 01:46 dump6379.rdb
-rw-r--r-- 1 root  root         93 Aug 29 23:16 dump6380.rdb
-rw-r--r-- 1 root  root         93 Aug 29 23:16 dump6381.rdb
-rw-r--r-- 1 redis redis    163840 Aug 30 01:52 temp-41383.rdb

```

#### 1.4.1.2、数据备份恢复

```
#创建备份脚本
[root@Rocky8 ~]# vim redis_backup_rdb.sh 
[root@Rocky8 ~]# cat redis_backup_rdb.sh 
#!/bin/bash

BACKUP=/backup/redis-rdb
DIR=/apps/redis/data/
FILE=dump6379.rdb
PASS=123456

color () {
    RES_COL=60
    MOVE_TO_COL="echo -en \\033[${RES_COL}G"
    SETCOLOR_SUCCESS="echo -en \\033[1;32m"
    SETCOLOR_FAILURE="echo -en \\033[1;31m"
    SETCOLOR_WARNING="echo -en \\033[1;33m"
    SETCOLOR_NORMAL="echo -en \E[0m"
    echo -n "$1" && $MOVE_TO_COL
    echo -n "["
    if [ $2 = "success" -o $2 = "0" ] ;then
        ${SETCOLOR_SUCCESS}
        echo -n $"  OK  "    
    elif [ $2 = "failure" -o $2 = "1"  ] ;then 
        ${SETCOLOR_FAILURE}
        echo -n $"FAILED"
    else
        ${SETCOLOR_WARNING}
        echo -n $"WARNING"
    fi
    ${SETCOLOR_NORMAL}
    echo -n "]"
    echo 
}

redis-cli -h 127.0.0.1 -a $PASS --no-auth-warning  bgsave 
result=`redis-cli -a $PASS --no-auth-warning info Persistence |grep rdb_bgsave_in_progress| sed -rn 's/.*:([0-9]+).*/\1/p'`
until  [ $result -eq 0 ] ;do
    sleep 1
    result=`redis-cli -a $PASS --no-auth-warning info Persistence |grep rdb_bgsave_in_progress| sed -rn 's/.*:([0-9]+).*/\1/p'`
done
DATE=`date +%F_%H-%M-%S`

[ -e $BACKUP ] || { mkdir -p $BACKUP ; chown -R redis.redis $BACKUP; }
cp $DIR/$FILE $BACKUP/dump_6379-${DATE}.rdb

color "Backup redis RDB" 0

[root@Rocky8 ~]# chmod +x redis_backup_rdb.sh 


#插入数据
[root@Rocky8 ~]# redis-cli 
127.0.0.1:6379> CONFIG SET requirepass 123456
OK
127.0.0.1:6379> set car baoma
OK
127.0.0.1:6379> set niu 'laoniu'
OK
127.0.0.1:6379> dbsize
(integer) 10100003
127.0.0.1:6379> 


#执行备份
[root@Rocky8 ~]# bash redis_backup_rdb.sh
[root@Rocky8 ~]# ls /backup/redis-rdb/ -l
total 370816
-rw-r--r-- 1 root root 189855683 Aug 30 02:13 dump_6379-2023-08-30_02-13-33.rdb
-rw-r--r-- 1 root root 189855706 Aug 30 02:15 dump_6379-2023-08-30_02-15-13.rdb

#模拟故障
[root@Rocky8 ~]# systemctl stop redis6379.service
[root@Rocky8 ~]# rm -rf /apps/redis/data/dump6379.rdb

#恢复数据
[root@Rocky8 ~]# cp /backup/redis-rdb/dump_6379-2023-08-30_02-15-13.rdb /apps/redis/data/dump6379.rdb
[root@Rocky8 ~]# systemctl start redis6379.service

#验证数据
[root@Rocky8 ~]# redis-cli 
127.0.0.1:6379> dbsize
(integer) 10100003
127.0.0.1:6379> GET niu
"laoniu"	
127.0.0.1:6379> GET car
"baoma"
127.0.0.1:6379> quit

```

#### 1.4.1.3、RDB相关配置

```
[root@Rocky8 ~]# cat /apps/redis/etc/redis6379.conf |grep save
# save <seconds> <changes>
# Redis will save the DB if both the given number of seconds and the given
# save ""
# Unless specified otherwise, by default Redis will save the DB:
# save 3600 1
# save 300 100
# save 60 10000   ##测试60s内修改10000个key,验证是否生成RDB文件


#开启自动保存
[root@Rocky8 ~]# vim /apps/redis/etc/redis6379.conf 
save 10 1
#插入数据
[root@Rocky8 ~]# redis-cli set boot oction
OK

#查看备份文件生成temp-42414.rdb备份
[root@Rocky8 ~]# ls /apps/redis/data/
dump6379.rdb  dump6380.rdb  dump6381.rdb
[root@Rocky8 ~]# ls /apps/redis/data/
dump6379.rdb  dump6380.rdb  dump6381.rdb  temp-42414.rdb

#关闭自动备份
[root@Rocky8 ~]# vim /apps/redis/etc/redis6379.conf 
save ""   //开启此项
#save 10 1   //注释此行
```

### 1.4.2、AOF（完全数据+增量变化）

**AOF原理**

```
AOF 即 AppendOnlyFile，AOF 和 RDB 都采有COW机制，AOF可以指定不同的保存策略,默认为每秒钟
执行一次 fsync,按照操作的顺序地将变更命令追加至指定的AOF日志文件尾部
在第一次启用AOF功能时，会做一次完全备份，后续将执行增量性备份，相当于完全数据备份+增量变化
如果同时启用RDB和AOF,进行恢复时,默认AOF文件优先级高于RDB文件,即会使用AOF文件进行恢复
在第一次开启AOF功能时,会自动备份所有数据到AOF文件中,后续只会记录数据的更新指令
注意: AOF 模式默认是关闭的,第一次开启AOF后,并重启服务生效后,会因为AOF的优先级高于RDB,而
AOF默认没有数据文件存在,从而导致所有数据丢失
```



#### 1.4.2.1、正确开启AOF

```
[root@Rocky8 ~]# redis-cli dbsize
(integer) 10100004

#错误开启，造成数据丢失
[root@Rocky8 ~]# vim /apps/redis/etc/redis6379.conf

1256 appendonly yes                                                                                                                                                        
1260 appendfilename "appendonly.aof"

[root@Rocky8 ~]# systemctl restart redis6379.service 
[root@Rocky8 ~]# ls /apps/redis/data/ -l  
total 185416
-rw-r--r-- 1 redis redis         0 Aug 30 03:21 appendonly.aof   #生成AOF文件
-rw-r--r-- 1 redis redis 189855719 Aug 30 03:11 dump6379.rdb
-rw-r--r-- 1 root  root         93 Aug 29 23:16 dump6380.rdb
-rw-r--r-- 1 root  root         93 Aug 29 23:16 dump6381.rdb

[root@Rocky8 ~]# redis-cli dbsize   空文件
(integer) 0


#正确开启，命令开启，然后修改配置文件
[root@Rocky8 ~]# redis-cli config set appendonly yes
OK
[root@Rocky8 ~]# ll /apps/redis/data/
total 250888
-rw-r--r-- 1 redis redis         0 Aug 30 03:28 appendonly.aof
-rw-r--r-- 1 redis redis 189855719 Aug 30 03:11 dump6379.rdb
-rw-r--r-- 1 root  root         93 Aug 29 23:16 dump6380.rdb
-rw-r--r-- 1 root  root         93 Aug 29 23:16 dump6381.rdb
-rw-r--r-- 1 redis redis  63897600 Aug 30 03:28 temp-rewriteaof-42696.aof  #生成临时aof配置文件

[root@Rocky8 ~]# pstree -p |grep redis
           |-redis-server(40936)-+-{redis-server}(40944)
           |                     |-{redis-server}(40945)
           |                     |-{redis-server}(40946)
           |                     `-{redis-server}(40947)
           |-redis-server(40949)-+-{redis-server}(40955)
           |                     |-{redis-server}(40956)
           |                     |-{redis-server}(40957)
           |                     `-{redis-server}(40958)
           |-redis-server(42680)-+-{redis-server}(42688)
           |                     |-{redis-server}(42689)
           |                     |-{redis-server}(42690)
           |                     |-{redis-server}(42691)
           |                     `-{redis-server}(42698)
[root@Rocky8 ~]# ll /apps/redis/data/
total 370824
-rw-r--r-- 1 redis redis 189855719 Aug 30 03:28 appendonly.aof
-rw-r--r-- 1 redis redis 189855719 Aug 30 03:11 dump6379.rdb
-rw-r--r-- 1 root  root         93 Aug 29 23:16 dump6380.rdb
-rw-r--r-- 1 root  root         93 Aug 29 23:16 dump6381.rdb

#然后重启服务
[root@Rocky8 ~]# vim /apps/redis/etc/redis6379.conf

appendonly yes     #修改此项                                                                                                                                                   
appendfilename "appendonly.aof"

[root@Rocky8 ~]# systemctl restart redis6379.servic
```

#### 1.4.2.2、误删除恢复

```
#若误删除则需修改appendonly.aof文件中相关行并重启服务即可
[root@Rocky8 ~]# cat /apps/redis/data/appendonly.aof |tail -n10
name
$9
shipanpan
*3
$3
set
$5
core
$9
100
[root@Rocky8 ~]# systemctl restart redis6379.service
[root@Rocky8 ~]# redis-cli get core
"100"
```

#### 1.4.2.3、相关配置

```
appendonly no #是否开启AOF日志记录，默认redis使用的是rdb方式持久化，这种方式在许多应用中已经
足够用了，但是redis如果中途宕机，会导致可能有几分钟的数据丢失(取决于dump数据的间隔时间)，根据
save来策略进行持久化，Append Only File是另一种持久化方式，可以提供更好的持久化特性，Redis会
把每次写入的数据在接收后都写入 appendonly.aof 文件，每次启动时Redis都会先把这个文件的数据读入
内存里，先忽略RDB文件。默认不启用此功能

appendfilename "appendonly.aof" #文本文件AOF的文件名，存放在dir指令指定的目录中

appendfsync everysec #aof持久化策略的配置
#no表示由操作系统保证数据同步到磁盘,Linux的默认fsync策略是30秒，最多会丢失30s的数据
#always表示每次写入都执行fsync，以保证数据同步到磁盘,安全性高,性能较差
#everysec表示每秒执行一次fsync，可能会导致丢失这1s数据,此为默认值,也生产建议值
dir /path
#rewrite相关
no-appendfsync-on-rewrite yes
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
```

```
#动态修改配置自动生成appendonly.aof文件
[root@Rocky8 ~]# redis-cli 
127.0.0.1:6379> CONFIG SET appendonly yes
```

#### 1.4.2.4、**AOF rewrite** **重写**

```
将一些重复的,可以合并的,过期的数据重新写入一个新的AOF文件,从而节约AOF备份占用的硬盘空间,也
能加速恢复过程
可以手动执行bgrewriteaof 触发AOF,第一次开启AOF功能,或定义自动rewrite 策略
```

**AOF rewrite** **过程**



 **手动执行AOF重写BGREWRITEAOF 命令**

```
#清理数据
[root@Rocky8 ~]# redis-cli 
127.0.0.1:6379> FLUSHALL
OK
(5.86s)
127.0.0.1:6379> dbsize
(integer) 0
127.0.0.1:6379> quit
[root@Rocky8 ~]# ll /apps/redis/data/ -h
total 182M
-rw-r--r-- 1 redis redis 182M Aug 30 03:48 appendonly.aof  #清理数据后文件不会小
-rw-r--r-- 1 redis redis   93 Aug 30 04:21 dump6379.rdb
-rw-r--r-- 1 root  root    93 Aug 29 23:16 dump6380.rdb
-rw-r--r-- 1 root  root    93 Aug 29 23:16 dump6381.rdb

#手动 bgrewriteaof
[root@Rocky8 ~]# redis-cli bgrewriteaof ; pstree -p |grep redis ;ll /apps/redis/data/
Background append only file rewriting started
           |-redis-server(40936)-+-{redis-server}(40944)
           |                     |-{redis-server}(40945)
           |                     |-{redis-server}(40946)
           |                     `-{redis-server}(40947)
           |-redis-server(40949)-+-{redis-server}(40955)
           |                     |-{redis-server}(40956)
           |                     |-{redis-server}(40957)
           |                     `-{redis-server}(40958)
           |-redis-server(42923)-+-redis-server(42988)
           |                     |-{redis-server}(42924)
           |                     |-{redis-server}(42925)
           |                     |-{redis-server}(42926)
           |                     `-{redis-server}(42927)
total 185424
-rw-r--r-- 1 redis redis 189855817 Aug 30 03:48 appendonly.aof
-rw-r--r-- 1 redis redis        93 Aug 30 04:21 dump6379.rdb
-rw-r--r-- 1 root  root         93 Aug 29 23:16 dump6380.rdb
-rw-r--r-- 1 root  root         93 Aug 29 23:16 dump6381.rdb
-rw-r--r-- 1 redis redis        93 Aug 30 04:25 temp-rewriteaof-42988.aof

[root@Rocky8 ~]# ll /apps/redis/data/
total 16
-rw-r--r-- 1 redis redis 93 Aug 30 04:25 appendonly.aof  #观察文件已被清理
-rw-r--r-- 1 redis redis 93 Aug 30 04:21 dump6379.rdb
-rw-r--r-- 1 root  root  93 Aug 29 23:16 dump6380.rdb
-rw-r--r-- 1 root  root  93 Aug 29 23:16 dump6381.rdb

```

**AOF重写相关配置**

```
#同时在执行bgrewriteaof操作和主进程写aof文件的操作，两者都会操作磁盘，而bgrewriteaof往往会涉及大量磁盘操作，这样就会造成主进程在写aof文件的时候出现阻塞的情形,以下参数实现控制no-appendfsync-on-rewrite no #在aof rewrite期间,是否对aof新记录的append暂缓使用文件同步策略,主要考虑磁盘IO开支和请求阻塞时间。
#默认为no,表示"不暂缓",新的aof记录仍然会被立即同步到磁盘，是最安全的方式，不会丢失数据，但是要忍受阻塞的问题
#为yes,相当于将appendfsync设置为no，这说明并没有执行磁盘操作，只是写入了缓冲区，因此这样并不会造成阻塞（因为没有竞争磁盘），但是如果这个时候redis挂掉，就会丢失数据。丢失多少数据呢？Linux的默认fsync策略是30秒，最多会丢失30s的数据,但由于yes性能较好而且会避免出现阻塞因此比较推荐
#rewrite 即对aof文件进行整理,将空闲空间回收,从而可以减少恢复数据时间auto-aof-rewrite-percentage 100 #当Aof log增长超过指定百分比例时，重写AOF文件，设置为0表示不自动重写Aof日志，重写是为了使aof体积保持最小，但是还可以确保保存最完整的数据

auto-aof-rewrite-min-size 64mb #触发aof rewrite的最小文件大小

aof-load-truncated yes #是否加载由于某些原因导致的末尾异常的AOF文件(主进程被kill/断电等)，建议yes
```

#### 1.4.2.5、**AOF** **模式优缺点**

**①AOF** **模式优点**

```
数据安全性相对较高，根据所使用的fsync策略(fsync是同步内存中redis所有已经修改的文件到存
储设备)，默认是appendfsync everysec，即每秒执行一次 fsync,在这种配置下，Redis 仍然可以保
持良好的性能，并且就算发生故障停机，也最多只会丢失一秒钟的数据( fsync会在后台线程执行，
所以主线程可以继续努力地处理命令请求)
由于该机制对日志文件的写入操作采用的是append模式，因此在写入过程中不需要seek, 即使出现
宕机现象，也不会破坏日志文件中已经存在的内容。然而如果本次操作只是写入了一半数据就出现
了系统崩溃问题，不用担心，在Redis下一次启动之前，可以通过 redis-check-aof 工具来解决数据
一致性的问题
Redis可以在 AOF文件体积变得过大时，自动地在后台对AOF进行重写,重写后的新AOF文件包含了
恢复当前数据集所需的最小命令集合。整个重写操作是绝对安全的，因为Redis在创建新 AOF文件
的过程中，append模式不断的将修改数据追加到现有的 AOF文件里面，即使重写过程中发生停
机，现有的 AOF文件也不会丢失。而一旦新AOF文件创建完毕，Redis就会从旧AOF文件切换到新
AOF文件，并开始对新AOF文件进行追加操作。
AOF包含一个格式清晰、易于理解的日志文件用于记录所有的修改操作。事实上，也可以通过该文
件完成数据的重建
```

**② AOF 模式缺点**

```
即使有些操作是重复的也会全部记录，AOF 的文件大小一般要大于 RDB 格式的文件
AOF 在恢复大数据集时的速度比 RDB 的恢复速度要慢
如果 fsync 策略是appendfsync no, AOF保存到磁盘的速度甚至会可能会慢于RDB
bug 出现的可能性更多
```

### 1.4.3、**RDB**和AOF **的选择**

```
如果主要充当缓存功能,或者可以承受较长时间,比如数分钟数据的丢失, 通常生产环境一般只需启用RDB

即可,此也是默认值

如果一点数据都不能丢失,可以选择同时开启RDB和AOF

一般不建议只开启AOF
```

# 2、**Redis** **常用命令**

## 2.1、**INFO**

显示当前节点redis运行状态信息

```
[root@Rocky8 ~]# redis-cli 
127.0.0.1:6379[1]> info
# Server
redis_version:6.2.13
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:6a4a30c076e6cee6
redis_mode:standalone
os:Linux 4.18.0-477.10.1.el8_8.x86_64 x86_64
arch_bits:64
monotonic_clock:POSIX clock_gettime
multiplexing_api:epoll
atomicvar_api:c11-builtin
gcc_version:8.5.0
process_id:43311
process_supervised:systemd
run_id:bcc39dea395b4cc8707198ad1fb64d62bf3978e0
tcp_port:6379
server_time_usec:1693344648708184
uptime_in_seconds:149
uptime_in_days:0
hz:10
configured_hz:10
lru_clock:15623048
executable:/apps/redis/bin/redis-server
config_file:/apps/redis/etc/redis6379.conf
io_threads_active:0

# Clients
connected_clients:1
cluster_connections:0
maxclients:10000
client_recent_max_input_buffer:16
client_recent_max_output_buffer:0
blocked_clients:0
tracking_clients:0
clients_in_timeout_table:0

# Memory
used_memory:861477728
used_memory_human:821.57M
used_memory_rss:890589184
used_memory_rss_human:849.33M
used_memory_peak:861535688
used_memory_peak_human:821.62M
used_memory_peak_perc:99.99%
used_memory_overhead:539050392
used_memory_startup:811912
used_memory_dataset:322427336
used_memory_dataset_perc:37.46%
allocator_allocated:861527376
allocator_active:861786112
allocator_resident:883892224
total_system_memory:3811012608
total_system_memory_human:3.55G
used_memory_lua:30720
used_memory_lua_human:30.00K
used_memory_scripts:0
used_memory_scripts_human:0B
number_of_cached_scripts:0
maxmemory:0
maxmemory_human:0B
maxmemory_policy:noeviction
allocator_frag_ratio:1.00
allocator_frag_bytes:258736
allocator_rss_ratio:1.03
allocator_rss_bytes:22106112
rss_overhead_ratio:1.01
rss_overhead_bytes:6696960
mem_fragmentation_ratio:1.03
mem_fragmentation_bytes:29154224
mem_not_counted_for_evict:0
mem_replication_backlog:0
mem_clients_slaves:0
mem_clients_normal:20496
mem_aof_buffer:0
mem_allocator:jemalloc-5.1.0
active_defrag_running:0
lazyfree_pending_objects:0
lazyfreed_objects:0

# Persistence
loading:0
current_cow_size:0
current_cow_size_age:0
current_fork_perc:0.00
current_save_keys_processed:0
current_save_keys_total:0
rdb_changes_since_last_save:0
rdb_bgsave_in_progress:0
rdb_last_save_time:1693344499
rdb_last_bgsave_status:ok
rdb_last_bgsave_time_sec:-1
rdb_current_bgsave_time_sec:-1
rdb_last_cow_size:0
aof_enabled:0
aof_rewrite_in_progress:0
aof_rewrite_scheduled:0
aof_last_rewrite_time_sec:-1
aof_current_rewrite_time_sec:-1
aof_last_bgrewrite_status:ok
aof_last_write_status:ok
aof_last_cow_size:0
module_fork_in_progress:0
module_fork_last_cow_size:0

# Stats
total_connections_received:1
total_commands_processed:3
instantaneous_ops_per_sec:0
total_net_input_bytes:103
total_net_output_bytes:20360
instantaneous_input_kbps:0.00
instantaneous_output_kbps:0.00
rejected_connections:0
sync_full:0
sync_partial_ok:0
sync_partial_err:0
expired_keys:0
expired_stale_perc:0.00
expired_time_cap_reached_count:0
expire_cycle_cpu_milliseconds:2
evicted_keys:0
keyspace_hits:0
keyspace_misses:0
pubsub_channels:0
pubsub_patterns:0
latest_fork_usec:0
total_forks:0
migrate_cached_sockets:0
slave_expires_tracked_keys:0
active_defrag_hits:0
active_defrag_misses:0
active_defrag_key_hits:0
active_defrag_key_misses:0
tracking_total_keys:0
tracking_total_items:0
tracking_total_prefixes:0
unexpected_error_replies:0
total_error_replies:2
dump_payload_sanitizations:0
total_reads_processed:6
total_writes_processed:5
io_threaded_reads_processed:0
io_threaded_writes_processed:0

# Replication
role:master
connected_slaves:0
master_failover_state:no-failover
master_replid:abd1c65d4601b2d9b1b416126b240063dcf4a542
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

# CPU
used_cpu_sys:0.553983
used_cpu_user:5.039465
used_cpu_sys_children:0.000000
used_cpu_user_children:0.000000
used_cpu_sys_main_thread:0.552826
used_cpu_user_main_thread:5.038687

# Modules

# Errorstats
errorstat_ERR:count=2

# Cluster
cluster_enabled:0

# Keyspace
db0:keys=10100002,expires=0,avg_ttl=0
db1:keys=2,expires=0,avg_ttl=0

#显示特定部分
127.0.0.1:6379[1]> info server
# Server
redis_version:6.2.13
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:6a4a30c076e6cee6
redis_mode:standalone
os:Linux 4.18.0-477.10.1.el8_8.x86_64 x86_64
arch_bits:64
monotonic_clock:POSIX clock_gettime
multiplexing_api:epoll
atomicvar_api:c11-builtin
gcc_version:8.5.0
process_id:43311
process_supervised:systemd
run_id:bcc39dea395b4cc8707198ad1fb64d62bf3978e0
tcp_port:6379
server_time_usec:1693344695682110
uptime_in_seconds:196
uptime_in_days:0
hz:10
configured_hz:10
lru_clock:15623095
executable:/apps/redis/bin/redis-server
config_file:/apps/redis/etc/redis6379.conf
io_threads_active:0
127.0.0.1:6379[1]> info cluster
# Cluster
cluster_enabled:0
```

## 2.2、**SELECT**

切换数据库，相当于在MySQL的 USE DBNAME 指令

```
[root@Rocky8 ~]# redis-cli 
127.0.0.1:6379[1]> SELECT 1
OK
127.0.0.1:6379[1]> dbsize
(integer) 2
127.0.0.1:6379[1]> SELECT 0
OK
127.0.0.1:6379> dbsize
(integer) 10100002

```

## 2.3、**KEYS**

查看当前库下的所有key，此命令慎用！

```
127.0.0.1:6379[15]> SELECT 0
OK
127.0.0.1:6379> KEYS *
1) "9527"
2) "9526"
3) "course"
4) "list1"
127.0.0.1:6379> SELECT 1
OK
127.0.0.1:6379[1]> KEYS *
(empty list or set)
127.0.0.1:6379[1]> 
redis>MSET one 1 two 2 three 3 four 4  # 一次设置 4 个 key
OK
redis> KEYS *o*
1) "four"
2) "two"
3) "one"
redis> KEYS t??
1) "two"
redis> KEYS t[w]*
1) "two"
redis> KEYS *  # 匹配数据库内所有 key
1) "four"
2) "three"
3) "two"
4) "one"
```

### 2.4、**BGSAVE**

手动在后台执行RDB持久化操作

```
#交互式执行
127.0.0.1:6379[1]> BGSAVE
Background saving started
#非交互式执行
[root@centos8 ~]#ll /var/lib/redis/
total 4
-rw-r--r-- 1 redis redis 326 Feb 18 22:45 dump.rdb


[root@Rocky8 ~]# redis-cli -h 127.0.0.1 -a '123456' BGSAVE
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
Background saving started
[root@Rocky8 ~]# ll /apps/redis/data/
total 185420
-rw-r--r-- 1 redis redis        93 Aug 30 04:25 appendonly.aof
-rw-r--r-- 1 redis redis 189855719 Aug 30 05:37 dump6379.rdb
-rw-r--r-- 1 root  root         93 Aug 29 23:16 dump6380.rdb
-rw-r--r-- 1 root  root         93 Aug 29 23:16 dump6381.rdb

```

## 2.4、**DBSIZE**

返回当前库下的所有key 数量

```
[root@Rocky8 ~]# redis-cli 
127.0.0.1:6379[1]> SELECT 1
OK
127.0.0.1:6379[1]> dbsize
(integer) 2
127.0.0.1:6379[1]> SELECT 0
OK
127.0.0.1:6379> dbsize
(integer) 10100002
```

## 2.5、**FLUSHDB**

强制清空当前库中的所有key，此命令慎用！

```
127.0.0.1:6379[1]> SELECT 0
OK
127.0.0.1:6379> DBSIZE
(integer) 4
127.0.0.1:6379> FLUSHDB
OK
127.0.0.1:6379> DBSIZE
(integer) 0
127.0.0.1:6379>
```

## 2.6、**FLUSHALL**

强制清空当前Redis服务器所有数据库中的所有key，即删除所有数据，此命令慎用！

```
127.0.0.1:6379> FLUSHALL
OK
#生产建议修改配置使用rename-command禁用此命令
vim /etc/redis.conf
rename-command FLUSHALL ""
#rename-command 可能会在后续版本淘汰
```

## 2.7、**SHUTDOWN**

```
可用版本： >= 1.0.0
时间复杂度： O(N)，其中 N 为关机时需要保存的数据库键数量。
SHUTDOWN 命令执行以下操作：
关闭Redis服务,停止所有客户端连接
如果有至少一个保存点在等待，执行 SAVE 命令
如果 AOF 选项被打开，更新 AOF 文件
关闭 redis 服务器(server)
如果持久化被打开的话， SHUTDOWN 命令会保证服务器正常关闭而不丢失任何数据。
另一方面，假如只是单纯地执行 SAVE 命令，然后再执行 QUIT 命令，则没有这一保证 —— 因为在执行
SAVE 之后、执行 QUIT 之前的这段时间中间，其他客户端可能正在和服务器进行通讯，这时如果执行 QUIT 
就会造成数据丢失
```

# 3、**Redis** **数据类型**

## 3.1**字符串** **string**

```
字符串是一种最基本的Redis值类型。Redis字符串是二进制安全的，这意味着一个Redis字符串能包含任
意类型的数据，例如： 一张JPEG格式的图片或者一个序列化的Ruby对象。一个字符串类型的值最多能
存储512M字节的内容。Redis 中所有 key 都是字符串类型的。
```

### **3.1.1** **创建一个**key

**set 指令可以创建一个key 并赋值, 使用格式**

```
SET key value [EX seconds] [PX milliseconds] [NX|XX]
时间复杂度： O(1)
将字符串值 value 关联到 key 。
如果 key 已经持有其他值， SET 就覆写旧值， 无视类型。
当 SET 命令对一个带有生存时间（TTL）的键进行设置之后， 该键原有的 TTL 将被清除。
从 Redis 2.6.12 版本开始， SET 命令的行为可以通过一系列参数来修改：
EX seconds ： 将键的过期时间设置为 seconds 秒。 执行 SET key value EX seconds 的效果等
同于执行 SETEX key seconds value 。
PX milliseconds ： 将键的过期时间设置为 milliseconds 毫秒。 执行 SET key value PX 
milliseconds 的效果等同于执行 PSETEX key milliseconds value 。
NX ： 只在键不存在时， 才对键进行设置操作。 执行 SET key value NX 的效果等同于执行 SETNX 
key value 。
XX ： 只在键已经存在时， 才对键进行设置操作。


#不论key是否存在.都设置
127.0.0.1:6379> set key1 value1
OK
127.0.0.1:6379> get key1
"value1"
127.0.0.1:6379> TYPE key1  #判断类型
string
127.0.0.1:6379> SET title ceo ex 3 #设置自动过期时间3s
OK
127.0.0.1:6379> set NAME wang
OK
127.0.0.1:6379> get NAME
"wang"
#Key大小写敏感
127.0.0.1:6379> get name
(nil)
127.0.0.1:6379> set name mage
OK
127.0.0.1:6379> get name
"mage"
127.0.0.1:6379> get NAME
"wang"
#key不存在,才设置,相当于add 
127.0.0.1:6379> get title
"ceo"
127.0.0.1:6379> setnx title coo  #set key value nx
(integer) 0
127.0.0.1:6379> get title
"ceo"
#key存在,才设置,相当于update
127.0.0.1:6379> get title
"ceo"
127.0.0.1:6379> set title coo xx
OK
127.0.0.1:6379> get title
"coo"
127.0.0.1:6379> get age
(nil)
127.0.0.1:6379> set age 20 xx
(nil)
127.0.0.1:6379> get age
(nil)


[root@Rocky8 ~]# redis-cli 
127.0.0.1:6379> set hongbao 100 ex 10
OK
127.0.0.1:6379> get hongbao
"100"
127.0.0.1:6379> get hongbao
"100"
127.0.0.1:6379> get hongbao
(nil)

```

### 3.1.2、查看一个key值

```
127.0.0.1:6379> get key1
"value1"
#get可能查看一个key的值
127.0.0.1:6379> get name age
(error) ERR wrong number of arguments for 'get' command
```

### 3.1.3、 **删除**key

```
127.0.0.1:6379> DEL key1
(integer) 1
127.0.0.1:6379> DEL key1 key2
(integer) 2
```

### 3.1.4、 **批量设置多个**key

```
127.0.0.1:6379> MSET key1 value1 key2 value2 
Ok
```

### **3.1.5** **批量获取多个key**

```
127.0.0.1:6379> MGET key1 key2
1) "value1"
2) "value2"
127.0.0.1:6379> KEYS n*
1) "n1"
2) "name"
127.0.0.1:6379> KEYS *
1) "k2"
2) "k1"
3) "key1"
4) "key2"
5) "n1"
6) "name"
7) "k3"
8) "title"
```

### 3.1.6、**追加**key的数据

```
127.0.0.1:6379> APPEND key1 " append new value"
(integer) 12              #添加数据后,key1总共9个字节
127.0.0.1:6379> get key1
"value1 append new value"
```

### 3.1.7、**设置新值并返回旧值**

```
127.0.0.1:6379> set name wang
OK
#set key newvalue并返回旧的value
127.0.0.1:6379> getset name magedu
"wang"
127.0.0.1:6379> get name
"magedu"
```

### **3.1.8** **返回字符串** **key** **对应值的字节数**、

```
127.0.0.1:6379> SET name wang
OK
127.0.0.1:6379> STRLEN name
(integer) 4
127.0.0.1:6379> APPEND name " xiaochun"
(integer) 13
127.0.0.1:6379> GET name
"wang xiaochun"
127.0.0.1:6379> STRLEN name  #返回字节数
(integer) 13
127.0.0.1:6379> set name 马哥教育
OK
127.0.0.1:6379> get name
"\xe9\xa9\xac\xe5\x93\xa5\xe6\x95\x99\xe8\x82\xb2"
127.0.0.1:6379> strlen name
(integer) 12
127.0.0.1:6379>
```

### 3.1.9、**判断** **key** **是否存在**

```
127.0.0.1:6379> SET name wang ex 10
OK
127.0.0.1:6379> set age 20
OK
127.0.0.1:6379> EXISTS NAME #key的大小写敏感
(integer) 0
127.0.0.1:6379> EXISTS name age #返回值为1,表示存在2个key,0表示不存在
(integer) 2
127.0.0.1:6379> EXISTS name  #过几秒再看
(integer) 0
```

### 3**.1.10** **获取** **key** **的过期时长**

```
ttl key #查看key的剩余生存时间,如果key过期后,会自动删除
-1 #返回值表示永不过期，默认创建的key是永不过期，重新对key赋值，也会从有剩余生命周期变成永不过
期
-2 #返回值表示没有此key
num #key的剩余有效期
127.0.0.1:6379> TTL key1
(integer) -1
127.0.0.1:6379> SET name wang EX 100
OK
127.0.0.1:6379> TTL name
(integer) 96
127.0.0.1:6379> TTL name
(integer) 93
127.0.0.1:6379> SET name mage #重新设置，默认永不过期
OK
127.0.0.1:6379> TTL name
(integer) -1
127.0.0.1:6379> SET name wang EX 200
OK
127.0.0.1:6379> TTL name
(integer) 198
127.0.0.1:6379> GET name
"wang"
```

### 3.1.11 **重置**key的过期时长

```
127.0.0.1:6379> TTL name
(integer) 148
127.0.0.1:6379> EXPIRE name 1000
(integer) 1
127.0.0.1:6379> TTL name
(integer) 999
127.0.0.1:6379>
```

### **3.1.12** **取消**key的期限

即永不过期

```
127.0.0.1:6379> TTL name
(integer) 999
127.0.0.1:6379> PERSIST name
(integer) 1
127.0.0.1:6379> TTL name
(integer) -1
```



### **3.1.13** **数字递增**

利用INCR命令簇（INCR, DECR, INCRBY,DECRBY)来把字符串当作原子计数器使用。

```
127.0.0.1:6379> set num 10 #设置初始值
OK
127.0.0.1:6379> INCR num
(integer) 11
127.0.0.1:6379> get num
"11
```

### 3**.1.14** **数字递减**

```
127.0.0.1:6379> set num 10
OK
127.0.0.1:6379> DECR num
(integer) 9
127.0.0.1:6379> get num
"9
```

### 3**.1.15** **数字增加**

将key对应的数字加decrement(可以是负数)。如果key不存在，操作之前，key就会被置为0。如果key的

value类型错误或者是个不能表示成数字的字符串，就返回错误。这个操作最多支持64位有符号的正型

数字。

```
redis> SET mykey 10
OK
redis> INCRBY mykey 5
(integer) 15
127.0.0.1:6379> get mykey
"15"
127.0.0.1:6379> INCRBY mykey -10
(integer) 5
127.0.0.1:6379> get mykey
"5"
127.0.0.1:6379> INCRBY nokey  5
(integer) 5
127.0.0.1:6379> get nokey
"5"
```

### 3.1.16 数字减少

decrby 可以减小数值(也可以增加)

```
127.0.0.1:6379> SET mykey 10
OK
127.0.0.1:6379> DECRBY mykey 8
(integer) 2
127.0.0.1:6379> get mykey
"2"
127.0.0.1:6379> DECRBY mykey -20
(integer) 22
127.0.0.1:6379> get mykey
"22"
127.0.0.1:6379> DECRBY nokey 3
(integer) -3
127.0.0.1:6379> get nokey
"-3"
```

## 3.2、列表list

```
1、Redis列表就是简单的字符串数组，按照插入顺序排序. 支持双向读写,可以添加一个元素到列表的头部（左边）或者尾部（右边），一个列表最多可以包含2^32-1=4294967295个元素，每个列表元素有下标来标识,下标 0 表示列表的第一个元素，以 1 表示列表的第二个元素，以此类推。 也可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，元素值可以重复，常用于存入日志等场景，此数据类型比较常用

2、列表特点
	有序
	可重复
	左右都可以操作
```

### 3.2.1、创建列表和数据

LPUSH和RPUSH都可以插入列表

```
LPUSH key value [value …]
时间复杂度： O(1)
将一个或多个值 value 插入到列表 key 的表头
如果有多个 value 值，那么各个 value 值按从左到右的顺序依次插入到表头： 比如说，对空列表
mylist 执行命令 LPUSH mylist a b c ，列表的值将是 c b a ，这等同于原子性地执行 LPUSH 
mylist a 、 LPUSH mylist b 和 LPUSH mylist c 三个命令。
如果 key 不存在，一个空列表会被创建并执行 LPUSH 操作。
当 key 存在但不是列表类型时，返回一个错误。
RPUSH key value [value …]

时间复杂度： O(1)
将一个或多个值 value 插入到列表 key 的表尾(最右边)。
如果有多个 value 值，那么各个 value 值按从左到右的顺序依次插入到表尾：比如对一个空列表 mylist 执行 RPUSH mylist a b c ，得出的结果列表为 a b c ，等同于执行命令 RPUSH mylist a 、RPUSH mylist b 、 RPUSH mylist c 。
如果 key 不存在，一个空列表会被创建并执行 RPUSH 操作。
当 key 存在但不是列表类型时，返回一个错误。
```

**范例：**

```
#从左边添加数据，已添加的需向右移
127.0.0.1:6379> LPUSH name mage wang zhang  #根据顺序逐个写入name，最后的zhang会在列表
的最左侧。
(integer) 3
127.0.0.1:6379> TYPE name
list
#从右边添加数据
127.0.0.1:6379> RPUSH course linux python go
(integer) 3
127.0.0.1:6379> type course
list

实例：
[root@Rocky8 ~]# redis-cli 
127.0.0.1:6379> lpush list1 c
(integer) 1
127.0.0.1:6379> lpush list1 b
(integer) 2
127.0.0.1:6379> lpush list1 a
(integer) 3
127.0.0.1:6379> lpush list1 3 2 1
(integer) 6
127.0.0.1:6379> type list
none
127.0.0.1:6379> type list1
list
127.0.0.1:6379> llen list1
(integer) 6
127.0.0.1:6379> LINDEX list1 0
"1"
127.0.0.1:6379> LINDEX list1 1
"2"
127.0.0.1:6379> LINDEX list1 2
"3"
127.0.0.1:6379> LINDEX list1 3
"a"
127.0.0.1:6379> LINDEX list1 4
"b"
127.0.0.1:6379> LINDEX list1 5
"c"
127.0.0.1:6379> LINDEX list1 6
(nil)

```

### 3.2.2、**列表追加新数据**

```
127.0.0.1:6379> LPUSH list1 tom
(integer) 7
#从右边添加数据，已添加的向左移
127.0.0.1:6379> rpush list1 jack
(integer) 8

```

### 3.2.3、**获取列表长度**(元素个数)

```
127.0.0.1:6379> llen list1
(integer) 8

```

### 3.2.4、**获取列表指定位置元素数据**

```
[root@Rocky8 ~]# redis-cli 
127.0.0.1:6379> lpush list1 c
(integer) 1
127.0.0.1:6379> lpush list1 b
(integer) 2
127.0.0.1:6379> lpush list1 a
(integer) 3
127.0.0.1:6379> lpush list1 3 2 1
(integer) 6
127.0.0.1:6379> type list
none
127.0.0.1:6379> type list1
list
127.0.0.1:6379> llen list1
(integer) 6
127.0.0.1:6379> LINDEX list1 0
"1"
127.0.0.1:6379> LINDEX list1 1
"2"
127.0.0.1:6379> LINDEX list1 2
"3"
127.0.0.1:6379> LINDEX list1 3
"a"
127.0.0.1:6379> LINDEX list1 4
"b"
127.0.0.1:6379> LINDEX list1 5
"c"
127.0.0.1:6379> LINDEX list1 6
(nil)
```

**指定位置**

```
127.0.0.1:6379> rpush list2 zhang wang li zhao
(integer) 4
127.0.0.1:6379> LRANGE list2 1 2
1) "wang"
2) "li"
127.0.0.1:6379> LRANGE list2 2 2
1) "li"
127.0.0.1:6379> LRANGE list2 0 -1
1) "zhang"
2) "wang"
3) "li"
4) "zhao"

```

### 3.2.5、**修改列表指定索引值**

```
127.0.0.1:6379> rpush listkey a b c d e f g h i j k l m n
(integer) 14
127.0.0.1:6379> LLEN listkey
(integer) 14
127.0.0.1:6379> LRANGE listkey 0 -1
 1) "a"
 2) "b"
 3) "c"
 4) "d"
 5) "e"
 6) "f"
 7) "g"
 8) "h"
 9) "i"
10) "j"
11) "k"
12) "l"
13) "m"
14) "n"
127.0.0.1:6379> LSET listkey 2 jave
OK
127.0.0.1:6379> LRANGE listkey 0 -1
 1) "a"
 2) "b"
 3) "jave"
 4) "d"
 5) "e"
 6) "f"
 7) "g"
 8) "h"
 9) "i"
10) "j"
11) "k"
12) "l"
13) "m"
14) "n"
```

### 3.2.6、**删除列表数据**

```
#从左边弹出
127.0.0.1:6379> LPOP listkey
"a"

#从右边弹出
127.0.0.1:6379> rPOP listkey
"n"
#一次性弹出两个值
127.0.0.1:6379> rPOP listkey 2
1) "m"
2) "l"
127.0.0.1:6379> LRANGE listkey 0 -1
 1) "b"
 2) "jave"
 3) "d"
 4) "e"
 5) "f"
 6) "g"
 7) "h"
 8) "i"
 9) "j"
10) "k"

```

## 3.3、集合set

### 3.3.1、创建集合

```
127.0.0.1:6379> SADD myfriend ming hong qiang cai
(integer) 4
127.0.0.1:6379> SADD yourfriend hou zhu ren sha
(integer) 4
127.0.0.1:6379> type myfriend
set
127.0.0.1:6379> type yourfriend
set

```

### 3.3.2、**集合中追加数据**

```
#追加时，只能追加不存在的数据，不能追加已经存在的数值
127.0.0.1:6379> SADD set1 v2 v3 v4
(integer) 3
127.0.0.1:6379> SADD set1 v2 #已存在的value,无法再次添加
(integer) 0
127.0.0.1:6379> TYPE set1
set
127.0.0.1:6379> TYPE set2
set
```

### 3.3.3、 **获取集合的所有数据**

```
127.0.0.1:6379> SMEMBERS myfriend
1) "qiang"
2) "cai"
3) "hong"
4) "ming"
127.0.0.1:6379> SMEMBERS yourfriend
1) "zhu"
2) "ren"
3) "sha"
4) "hou"
5) "hong"

```

### **3.3.4** **删除集合中的元素**

```
127.0.0.1:6379> SREM yourfriend zhu
(integer) 1
127.0.0.1:6379> SMEMBERS yourfriend
1) "ren"
2) "sha"
3) "hou"
4) "hong"

```

### **3.3.5** **取集合的交集**

```
交集：同时属于集合A且属于集合B的元素
可以实现共同的朋友
127.0.0.1:6379> SINTER myfriend yourfriend
(empty array)
127.0.0.1:6379> SADD yourfriend hong
(integer) 1
127.0.0.1:6379> SINTER myfriend yourfriend
1) "hong"

```

### **3.3.6** **取集合的并集**

并集：属于集合A或者属于集合B的元素

```
127.0.0.1:6379> SUNION myfriend yourfriend
1) "zhu"
2) "ming"
3) "ren"
4) "qiang"
5) "sha"
6) "cai"
7) "hong"
8) "hou"
```

### **3.3.7** **取集合的差集**

差集：属于集合A但不属于集合B的元素

可以实现我的朋友的朋友

```
#属于集合myfriend但是不属于集合yourfriend的元素
127.0.0.1:6379> SMEMBERS myfriend
1) "qiang"
2) "cai"
3) "hong"
4) "ming"
127.0.0.1:6379> SMEMBERS yourfriend
1) "ren"
2) "sha"
3) "hou"
4) "hong"

127.0.0.1:6379> sdiff myfriend yourfriend
1) "qiang"
2) "cai"
3) "ming"
```

## **3.4、有序集合** **sorted set**

Redis有序集合和Redis集合类似，是不包含相同字符串的合集。它们的差别是，每个有序集合的成员都

关联着一个双精度浮点型的评分，这个评分用于把有序集合中的成员按最低分到最高分排序。有序集合

的成员不能重复,但评分可以重复,一个有序集合中最多的成员数为 2^32 - 1=4294967295个，经常用于

排行榜的场景

```
#有序集合特点：
	有序
	无重复元素
	每个元素是由score和value组成
	score 可以重复
	value 不可以重复
```

### 3.4.1、创建有序集合

```
127.0.0.1:6379> ZADD course 90 linux 99 go 60 python 50 cloud
(integer) 4
127.0.0.1:6379> ZRANGE course 0 -1
1) "cloud"
2) "python"
3) "linux"
4) "go"
127.0.0.1:6379> ZREVRANGE course 0 -1
1) "go"
2) "linux"
3) "python"
4) "cloud"
127.0.0.1:6379> ZRANGE course 0 -1 withscores
1) "cloud"
2) "50"
3) "python"
4) "60"
5) "linux"
6) "90"
7) "go"
8) "99"
127.0.0.1:6379> ZREVRANGE course 0 -1 withscores
1) "go"
2) "99"
3) "linux"
4) "90"
5) "python"
6) "60"
7) "cloud"
8) "50"

```

### 3.4.2、查看集合的成员个数

```
127.0.0.1:6379> ZADD zset1 60 ming 70 ning 65 hong 75 qiang 99 hang
(integer) 5
127.0.0.1:6379> ZCARD course
(integer) 4

127.0.0.1:6379> ZCARD zset1
(integer) 5

```

### 3.4.3、实现排名

```
127.0.0.1:6379> ZADD course 90 linux 99 go 60 python 50 cloud
(integer) 4
127.0.0.1:6379> ZRANGE course 0 -1  #正序排序后显示集合内所有的key,按score从小到大显示
1) "cloud"
2) "python"
3) "linux"
4) "go"
127.0.0.1:6379> ZREVRANGE course 0 -1 #倒序排序后显示集合内所有的key,score从大到小显示
1) "go"
2) "linux"
3) "python"
4) "cloud"
127.0.0.1:6379> ZRANGE course 0 -1 WITHSCORES  #正序显示指定集合内所有key和得分情况
1) "cloud"
2) "50"
3) "python"
4) "60"
5) "linux"
6) "90"
7) "go"
8) "99"
127.0.0.1:6379> ZREVRANGE course 0 -1 WITHSCORES  #倒序显示指定集合内所有key和得分情况
1) "go"
2) "99"
3) "linux"
4) "90"
5) "python"
6) "60"
7) "cloud"
8) "50"
127.0.0.1:6379>
```

### 3.4.4、 **基于索引查找数据**

```
127.0.0.1:6379> ZRANGE course 0 2
1) "cloud"
2) "python"
3) "linux"
127.0.0.1:6379> ZRANGE course 0 10
1) "cloud"
2) "python"
3) "linux"
4) "go"

```

### 3.4.5、**查询指定数据的排名**

```
127.0.0.1:6379> ZADD course 90 linux 99 go 60 python 50 cloud
(integer) 4

127.0.0.1:6379> ZRANGE course 0 -1 withscores
1) "cloud"
2) "50"
3) "python"
4) "60"
5) "linux"
6) "90"
7) "go"
8) "99"


127.0.0.1:6379> ZRANK course go
(integer) 3   #第4个
127.0.0.1:6379> ZRANK course python
(integer) 1   #第2个
```

### **3.4.6** **获取分数**

```
127.0.0.1:6379> zscore course go
"99"
```

### **3.4.7** **删除元素**

```
127.0.0.1:6379> ZRANGE course 0 -1
1) "cloud"
2) "python"
3) "linux"
4) "go"
127.0.0.1:6379> ZREM course python linux
(integer) 2
127.0.0.1:6379> ZRANGE course 0 -1
1) "cloud"
2) "go"
```

## 3.5、哈希hash

```
hash 即字典, 用于保存字符串字段field和字符串值value之间的映射，即key/value做为数据部分,hash特
别适合用于存储对象场景.
一个hash最多可以包含2^32-1 个key/value键值对
#哈希特点：
	无序
	k/v 对
	适用于存放相关的数据
```

### 3.5.1、创建hash

```
#格式
HSET hash field value
时间复杂度： O(1)
将哈希表 hash 中域 field 的值设置为 value 。
如果给定的哈希表并不存在， 那么一个新的哈希表将被创建并执行 HSET 操作。
如果域 field 已经存在于哈希表中， 那么它的旧值将被新值 value 覆盖。

实例：
127.0.0.1:6379> HSET 1999 name dingbaohang age 24
(integer) 2
127.0.0.1:6379> type 1999
hash
127.0.0.1:6379> HGET 1999
(error) ERR wrong number of arguments for 'hget' command
127.0.0.1:6379> HGETALL 1999
1) "name"
2) "dingbaohang"
3) "age"
4) "24"

#增加字段
127.0.0.1:6379> HSET 1999 gender male
(integer) 1
127.0.0.1:6379> HGETALL 1999
1) "name"
2) "dingbaohang"
3) "age"
4) "24"
5) "gender"
6) "male"

```

### 3.5.2、**查看**hash的指定field的value

```
127.0.0.1:6379> HGET 1999 name
"dingbaohang"
127.0.0.1:6379> HGET 1999 age
"24"
127.0.0.1:6379> HGET 1999 gender
"male"

```

### 3.5.3、**删除**hash的指定的**field/value**

```
127.0.0.1:6379> HDEL 1999 gender
(integer) 1
127.0.0.1:6379> HGETALL 1999
1) "name"
2) "dingbaohang"
3) "age"
4) "24"

```

### 3.5.4、 **批量设置**hash key的多个field和value

```
127.0.0.1:6379> HMSET 1999 gender male city shanghai address zhuanqiao
OK
127.0.0.1:6379> HGETALL 1999
 1) "name"
 2) "dingbaohang"
 3) "age"
 4) "24"
 5) "gender"
 6) "male"
 7) "city"
 8) "shanghai"
 9) "address"
10) "zhuanqiao"	
```

### 3.5.5、查看hash指定field的value

```
127.0.0.1:6379> HGETALL 1999
 1) "name"
 2) "dingbaohang"
 3) "age"
 4) "24"
 5) "gender"
 6) "male"
 7) "city"
 8) "shanghai"
 9) "address"
10) "zhuanqiao"
127.0.0.1:6379> HMGET 1999 name age
1) "dingbaohang"
2) "24"
127.0.0.1:6379> HMGET 1999 name city
1) "dingbaohang"
2) "shanghai"

```

### 3.5.6、**查看**hash的所有field

```
127.0.0.1:6379> HMSET 1999 name shipanpan age 26 city shandong gender male
OK
127.0.0.1:6379> HGETALL 1999
 1) "name"
 2) "shipanpan"
 3) "age"
 4) "26"
 5) "gender"
 6) "male"
 7) "city"
 8) "shandong"
 9) "address"
10) "zhuanqiao"
127.0.0.1:6379> HKEYS 1999
1) "name"
2) "age"
3) "gender"
4) "city"
5) "address"
```

### 3.5.7、**查看**hash **所有**value

```
127.0.0.1:6379> HMSET 1999 name shipanpan age 26 city shandong gender male
OK
127.0.0.1:6379> HGETALL 1999
 1) "name"
 2) "shipanpan"
 3) "age"
 4) "26"
 5) "gender"
 6) "male"
 7) "city"
 8) "shandong"
 9) "address"
10) "zhuanqiao"

127.0.0.1:6379> HVALS 1999
1) "shipanpan"
2) "26"
3) "male"
4) "shandong"
5) "zhuanqiao"

```

### 3.5.8、**删除** **hash**

```
127.0.0.1:6379> DEL 1999
(integer) 1
127.0.0.1:6379> HMGET 1999 name city
1) (nil)
2) (nil)
```

## 3.6、消息队列

```
消息队列: 把要传输的数据放在队列中,从而实现应用之间的数据交换
常用功能: 可以实现多个应用系统之间的解耦,异步,削峰/限流等
常用的消息队列应用: Kafka,RabbitMQ,Redis

消息队列分为两种
生产者/消费者模式: Producer/Consumer
发布者/订阅者模式: Publisher/Subscriber
```

### 3.6.1、**生产者消费者模式**

```
模式说明
生产者消费者模式下，多个消费者同时监听一个频道(redis用队列实现)，但是生产者产生的一个消息只
能被最先抢到消息的一个消费者消费一次,队列中的消息由可以多个生产者写入，也可以有不同的消费者
取出进行消费处理.此模式应用广泛
```

```
1、生产者生成消息
127.0.0.1:6379> LPUSH channel1 message1
(integer) 1
127.0.0.1:6379> LPUSH channel1 message2
(integer) 2
127.0.0.1:6379> LPUSH channel1 message3
(integer) 3
127.0.0.1:6379> LPUSH channel1 message4
(integer) 4
127.0.0.1:6379> LPUSH channel1 message5
(integer) 5

2、获取所有消息
127.0.0.1:6379> LRANGE channel1 0 -1
1) "message5"
2) "message4"
3) "message3"
4) "message2"
5) "message1"

3、消费者消费消息
127.0.0.1:6379> LPOP channel1 
"message5"
127.0.0.1:6379> RPOP channel1 
"message1"
127.0.0.1:6379> LRANGE channel1 0 -1
1) "message4"
2) "message3"
3) "message2"
127.0.0.1:6379> RPOP channel1
"message2"
127.0.0.1:6379> RPOP channel1
"message3"
127.0.0.1:6379> RPOP channel1
"message4"

4、验证队列消息消费完成、
127.0.0.1:6379> LRANGE channel1 0 -1   #验证队列中的消息全部消费完成
(empty array)
```

### 3.6.2、**发布者订阅模式**

```
 模式说明
在发布者订阅者Publisher/Subscriber模式下，发布者Publisher将消息发布到指定的频道channel，事
先监听此channel的一个或多个订阅者Subscriber都会收到相同的消息。即一个消息可以由多个订阅者
获取到. 对于社交应用中的群聊、群发、群公告等场景适用于此模式
```

```\
1、订阅者订阅频道
[root@Rocky8 ~]# redis-cli 
127.0.0.1:6379> SUBSCRIBE message1
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "message1"
3) (integer) 1


[root@Ubuntu1804 ~]# redis-cli -h 192.168.188.88
192.168.188.88:6379> SUBSCRIBE message1
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "message1"
3) (integer) 1


127.0.0.1:6379> SUBSCRIBE message1
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "message1"
3) (integer) 1

2、发布者发布消息
[root@Rocky8 ~]# redis-cli 
127.0.0.1:6379> PUBLISH message1 duchele!
(integer) 3

3、各个订阅者都能收到消息
127.0.0.1:6379> SUBSCRIBE message1
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "message1"
3) (integer) 1
1) "message"
2) "message1"
3) "duchele!"

[root@Rocky8 ~]# redis-cli 
127.0.0.1:6379> SUBSCRIBE message1
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "message1"
3) (integer) 1
1) "message"
2) "message1"
3) "duchele!"

[root@Ubuntu1804 ~]# redis-cli -h 192.168.188.88
192.168.188.88:6379> SUBSCRIBE message1
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "message1"
3) (integer) 1
1) "message"
2) "message1"
3) "duchele!"

4、订阅多个频道
#订阅指定的多个频道
[root@Rocky8 ~]# redis-cli 
127.0.0.1:6379> SUBSCRIBE message1 message2 message3
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "message1"
3) (integer) 1
1) "subscribe"
2) "message2"
3) (integer) 2
1) "subscribe"
2) "message3"
3) (integer) 3

#发布多个消息
[root@Rocky8 ~]# redis-cli 
127.0.0.1:6379> PUBLISH message1 kaishile!
(integer) 2
127.0.0.1:6379> PUBLISH message2 qianggouzhong!
(integer) 1
127.0.0.1:6379> PUBLISH message3 jieshule!
(integer) 1

#接受消息
[root@Rocky8 ~]# redis-cli 
127.0.0.1:6379> SUBSCRIBE message1 message2 message3
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "message1"
3) (integer) 1
1) "subscribe"
2) "message2"
3) (integer) 2
1) "subscribe"
2) "message3"
3) (integer) 3
1) "message"
2) "message1"
3) "kaishile!"
1) "message"
2) "message2"
3) "qianggouzhong!"
1) "message"
2) "message3"
3) "jieshule!"

5、 订阅所有频道
127.0.0.1:6379> PSUBSCRIBE *  #支持通配符*

6、 订阅匹配的频道
127.0.0.1:6379> PSUBSCRIBE chann* #匹配订阅多个频道

7、取消订阅频道
[root@Rocky8 ~]# redis-cli 
127.0.0.1:6379> UNSUBSCRIBE message1
1) "unsubscribe"
2) "message1"
3) (integer) 0
```

# 4、 **Redis** **主从复制**

## **4.1.1 Redis** **主从复制架构**

```
主从模式（master/slave），和MySQL的主从模式类似,可以实现Redis数据的跨主机的远程备份。
常见客户端连接主从的架构:
程序APP先连接到高可用性 LB 集群提供的虚拟IP，再由LB调度将用户的请求至后端Redis 服务器来真正提供服务


主从复制特点：
一个master可以有多个slave
一个slave只能有一个master
数据流向是从master到slave单向的
master 可读可写
slave 只读
```



## **4.1.2** **主从复制实现**

```
当master出现故障后,可以会提升一个slave节点变成新的Mster,因此Redis Slave 需要设置和master相同的连接密码，此外当一个Slave提升为新的master 通过持久化实现数据的恢复


当配置Redis复制功能时，强烈建议打开主服务器的持久化功能。否则的话，由于延迟等问题，部署的主节点Redis服务应该要避免自动启动
```

参考案例: 导致主从服务器数据全部丢失

```
1.假设节点A为主服务器，并且关闭了持久化。并且节点B和节点C从节点A复制数据
2.节点A崩溃，然后由自动拉起服务重启了节点A.由于节点A的持久化被关闭了，所以重启之后没有任何数据
3.节点B和节点C将从节点A复制数据，但是A的数据是空的，于是就把自身保存的数据副本删除。
```

```
在关闭主服务器上的持久化，并同时开启自动拉起进程的情况下，即便使用Sentinel来实现Redis的高可
用性，也是非常危险的。因为主服务器可能拉起得非常快，以至于Sentinel在配置的心跳时间间隔内没
有检测到主服务器已被重启，然后还是会执行上面的数据丢失的流程。无论何时，数据安全都是极其重
要的，所以应该禁止主服务器关闭持久化的同时自动启动。
```

### **4.1.2.1** **主从命令配置**

#### **4.1.2.1.1** **启用主从同步**

**主节点master配置**

```
Redis Server 默认为 master节点，如果要配置为从节点,需要指定master服务器的IP,端口及连接密码在从节点执行 REPLICAOF MASTER_IP PORT 指令可以启用主从同步复制功能,早期版本使用 SLAVEOF 指令

#主节点master配置
[root@Rocky8 ~]#redis-cli -a 123456
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
127.0.0.1:6379> CONFIG set requirepass 123456
OK
127.0.0.1:6379> AUTH 123456
OK
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.188.81,port=6379,state=online,offset=42,lag=1
master_replid:5937ff079b9d1a82c5e15033c6c0617400435b95
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:42
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:42
127.0.0.1:6379> set class n75
OK
127.0.0.1:6379> dbsize
(integer) 1
127.0.0.1:6379> GET class
"n75"
127.0.0.1:6379> set car bwm
OK
127.0.0.1:6379> set people zhang
OK
127.0.0.1:6379> info replication   #查看实现一主二从
# Replication
role:master
connected_slaves:2
slave0:ip=192.168.188.81,port=6379,state=online,offset=1770,lag=0
slave1:ip=192.168.188.82,port=6379,state=online,offset=1770,lag=0
master_replid:5937ff079b9d1a82c5e15033c6c0617400435b95
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:1770
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:1770
127.0.0.1:6379> 


#查看master日志文件
[root@Rocky8 ~]#tail -f /var/log/redis/redis.log 
2248:M 31 Aug 2023 22:37:40.757 * Full resync requested by replica 192.168.188.81:6379
2248:M 31 Aug 2023 22:37:40.757 * Starting BGSAVE for SYNC with target: disk
2248:M 31 Aug 2023 22:37:40.758 * Background saving started by pid 2254
2254:C 31 Aug 2023 22:37:40.765 * DB saved on disk
2254:C 31 Aug 2023 22:37:40.766 * RDB: 0 MB of memory used by copy-on-write
2248:M 31 Aug 2023 22:37:40.829 * Background saving terminated with success
2248:M 31 Aug 2023 22:37:40.829 * Synchronization with replica 192.168.188.81:6379 succeeded
2248:M 31 Aug 2023 22:42:48.672 # Connection with replica client id #38 lost.
2248:M 31 Aug 2023 22:43:41.979 * Replica 192.168.188.81:6379 asks for synchronization
2248:M 31 Aug 2023 22:43:41.979 * Partial resynchronization request from 192.168.188.81:6379 accepted. Sending 0 bytes of backlog starting from offset 522.
2248:M 31 Aug 2023 22:51:59.191 * Replica 192.168.188.82:6379 asks for synchronization
2248:M 31 Aug 2023 22:51:59.191 * Full resync requested by replica 192.168.188.82:6379
2248:M 31 Aug 2023 22:51:59.191 * Starting BGSAVE for SYNC with target: disk
2248:M 31 Aug 2023 22:51:59.191 * Background saving started by pid 2312
2312:C 31 Aug 2023 22:51:59.193 * DB saved on disk
2312:C 31 Aug 2023 22:51:59.193 * RDB: 0 MB of memory used by copy-on-write
2248:M 31 Aug 2023 22:51:59.258 * Background saving terminated with success
2248:M 31 Aug 2023 22:51:59.259 * Synchronization with replica 192.168.188.82:6379 succeeded
```

**slave节点配置**

```
#从slave1、slave2配置
[root@Rocky8 ~]#vim /etc/redis.conf     #修改配置文件replicaof及masterauth两项

# replicaof <masterip> <masterport>
replicaof 192.168.188.8 6379 

# If the master is password protected (using the "requirepass" configuration
# directive below) it is possible to tell the replica to authenticate before
# starting the replication synchronization process, otherwise the master will
# refuse the replica request.
#
masterauth 123456

[root@Rocky8 ~]#systemctl enable --now redis
[root@Rocky8 ~]#redis-cli -a 123456
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
127.0.0.1:6379> dbsize
(integer) 2
127.0.0.1:6379> dbsize
(integer) 3
127.0.0.1:6379> 
```

#### 4.1.2.1.2、查看数据库状态

```
requirepass 123456
127.0.0.1:6379> info replication
# Replication
role:slave
master_host:192.168.188.81
master_port:6379
master_link_status:up     #slave状态值
master_last_io_seconds_ago:3
master_sync_in_progress:0
slave_repl_offset:11822
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:a356c851711d418e725a32e9098cd45d31a5d2f5
master_replid2:5937ff079b9d1a82c5e15033c6c0617400435b95
master_repl_offset:11822
second_repl_offset:11316
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2569
repl_backlog_histlen:9254

#模拟中断主从通信
127.0.0.1:6379> info replication
# Replication
role:slave
master_host:192.168.188.8
master_port:6379
master_link_status:down    #中断slave状态值
```

#### 4.1.2.1.3、模拟master节点故障

```
#master
[root@Rocky8 ~]#systemctl stop redis.service 
[root@Rocky8 ~]#tail -f /var/log/redis/redis.log   #查看log日志
2248:M 31 Aug 2023 23:28:34.645 * 1 changes in 900 seconds. Saving...
2248:M 31 Aug 2023 23:28:34.645 * Background saving started by pid 2475
2475:C 31 Aug 2023 23:28:34.647 * DB saved on disk
2475:C 31 Aug 2023 23:28:34.647 * RDB: 0 MB of memory used by copy-on-write
2248:M 31 Aug 2023 23:28:34.745 * Background saving terminated with success
2248:M 31 Aug 2023 23:33:35.081 * 10 changes in 300 seconds. Saving...
2248:M 31 Aug 2023 23:33:35.082 * Background saving started by pid 2576
2576:C 31 Aug 2023 23:33:35.194 * DB saved on disk
2576:C 31 Aug 2023 23:33:35.195 * RDB: 0 MB of memory used by copy-on-write
2248:M 31 Aug 2023 23:33:35.284 * Background saving terminated with success
2248:signal-handler (1693498101) Received SIGTERM scheduling shutdown...
2248:M 01 Sep 2023 00:08:21.892 # User requested shutdown...
2248:M 01 Sep 2023 00:08:21.892 * Calling fsync() on the AOF file.
2248:M 01 Sep 2023 00:08:21.893 * Saving the final RDB snapshot before exiting.
2248:M 01 Sep 2023 00:08:21.894 * DB saved on disk
2248:M 01 Sep 2023 00:08:21.894 * Removing the pid file.
2248:M 01 Sep 2023 00:08:21.894 # Redis is now ready to exit, bye bye...


#slave1、2状态
24641:S 01 Sep 2023 00:10:20.474 * Connecting to MASTER 192.168.188.8:6379
24641:S 01 Sep 2023 00:10:20.474 * MASTER <-> REPLICA sync started
24641:S 01 Sep 2023 00:10:20.475 # Error condition on socket for SYNC: Connection refused
24641:S 01 Sep 2023 00:10:21.481 * Connecting to MASTER 192.168.188.8:6379
24641:S 01 Sep 2023 00:10:21.481 * MASTER <-> REPLICA sync started
24641:S 01 Sep 2023 00:10:21.481 # Error condition on socket for SYNC: Connection refused
24641:S 01 Sep 2023 00:10:22.488 * Connecting to MASTER 192.168.188.8:6379
24641:S 01 Sep 2023 00:10:22.489 * MASTER <-> REPLICA sync started
24641:S 01 Sep 2023 00:10:22.489 # Error condition on socket for SYNC: Connection refused
24641:S 01 Sep 2023 00:10:23.495 * Connecting to MASTER 192.168.188.8:6379
24641:S 01 Sep 2023 00:10:23.496 * MASTER <-> REPLICA sync started
24641:S 01 Sep 2023 00:10:23.496 # Error condition on socket for SYNC: Connection refused
24641:S 01 Sep 2023 00:10:24.503 * Connecting to MASTER 192.168.188.8:6379
24641:S 01 Sep 2023 00:10:24.503 * MASTER <-> REPLICA sync started
24641:S 01 Sep 2023 00:10:24.504 # Error condition on socket for SYNC: Connection refused
24641:S 01 Sep 2023 00:10:25.510 * Connecting to MASTER 192.168.188.8:6379
24641:S 01 Sep 2023 00:10:25.510 * MASTER <-> REPLICA sync started
24641:S 01 Sep 2023 00:10:25.510 # Error condition on socket for SYNC: Connection refused
24641:S 01 Sep 2023 00:10:26.517 * Connecting to MASTER 192.168.188.8:6379
24641:S 01 Sep 2023 00:10:26.517 * MASTER <-> REPLICA sync started
24641:S 01 Sep 2023 00:10:26.518 # Error condition on socket for SYNC: Connection refused
24641:S 01 Sep 2023 00:10:27.523 * Connecting to MASTER 192.168.188.8:6379
24641:S 01 Sep 2023 00:10:27.523 * MASTER <-> REPLICA sync started
24641:S 01 Sep 2023 00:10:27.523 # Error condition on socket for SYNC: Connection refused

#将slave1生成新master
127.0.0.1:6379> REPLICAOF no one    #输入命令，升为新主
OK
127.0.0.1:6379> info replication
# Replication
role:master   #角色变为master
connected_slaves:0
master_replid:a356c851711d418e725a32e9098cd45d31a5d2f5
master_replid2:5937ff079b9d1a82c5e15033c6c0617400435b95
master_repl_offset:11315
second_repl_offset:11316
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:11315
127.0.0.1:6379> set book xiyouji
OK

#slave2重新建立与新主 的通信
[root@Rocky8 ~]#redis-cli -a 123456
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
127.0.0.1:6379> REPLICAOF 192.168.188.81 6379
OK

127.0.0.1:6379> get book
"xiyouji"
127.0.0.1:6379> info replication
# Replication
role:slave
master_host:192.168.188.81
master_port:6379
master_link_status:up
master_last_io_seconds_ago:3
master_sync_in_progress:0
slave_repl_offset:11822
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:a356c851711d418e725a32e9098cd45d31a5d2f5
master_replid2:5937ff079b9d1a82c5e15033c6c0617400435b95
master_repl_offset:11822
second_repl_offset:11316
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2569
repl_backlog_histlen:9254



#查看日志
[root@Rocky8 ~]#tail -f /var/log/redis/redis.log 

24641:S 01 Sep 2023 00:11:45.958 * REPLICAOF 192.168.188.81:6379 enabled (user request from 'id=8 addr=127.0.0.1:58772 fd=7 name= age=38 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=50 qbuf-free=32718 obl=0 oll=0 omem=0 events=r cmd=replicaof')
24641:S 01 Sep 2023 00:11:46.041 * Connecting to MASTER 192.168.188.81:6379
24641:S 01 Sep 2023 00:11:46.041 * MASTER <-> REPLICA sync started
24641:S 01 Sep 2023 00:11:46.043 * Non blocking connect for SYNC fired the event.
24641:S 01 Sep 2023 00:11:46.043 * Master replied to PING, replication can continue...
24641:S 01 Sep 2023 00:11:46.046 * Trying a partial resynchronization (request 5937ff079b9d1a82c5e15033c6c0617400435b95:11316).
24641:S 01 Sep 2023 00:11:46.047 * Successful partial resynchronization with master.
24641:S 01 Sep 2023 00:11:46.047 # Master replication ID changed to a356c851711d418e725a32e9098cd45d31a5d2f5
24641:S 01 Sep 2023 00:11:46.047 * MASTER <-> REPLICA sync: Master accepted a Partial Resynchronization.
24641:S 01 Sep 2023 00:13:24.949 * 1 changes in 900 seconds. Saving...
24641:S 01 Sep 2023 00:13:24.950 * Background saving started by pid 24944
24944:C 01 Sep 2023 00:13:24.952 * DB saved on disk
24944:C 01 Sep 2023 00:13:24.953 * RDB: 0 MB of memory used by copy-on-write
24641:S 01 Sep 2023 00:13:25.051 * Background saving terminated with success

```

#### 4.1.2.1.4、实现主从复制

##### ①创建master节点（命令实现）

```
[root@Rocky8 ~]#redis-cli 
127.0.0.1:6379> dbsize
(error) NOAUTH Authentication required.
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> INFO replication
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.188.81,port=6379,state=online,offset=1096,lag=1
master_replid:ddc360c7df985ae8c7cc4c1265c71004585dc556
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:1096
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:1096


注：127.0.0.1:6379> CONFIG SET requirepass 123456 //创建数据库登录认证密码
```

##### ②创建master2，slave1节点

```
[root@Rocky8 ~]#redis-cli 
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> REPLICAOF 192.168.188.8 6379   #设置master节点ip，永久保存需修改配置文件
OK 
127.0.0.1:6379> CONFIG SET masterauth 123456   #创建复制认证密码
OK
127.0.0.1:6379> info replication
# Replication
role:slave
master_host:192.168.188.8
master_port:6379
master_link_status:up
master_last_io_seconds_ago:10
master_sync_in_progress:0
slave_repl_offset:1362
slave_priority:100
slave_read_only:1
connected_slaves:1
slave0:ip=192.168.188.82,port=6379,state=online,offset=1362,lag=0
master_replid:ddc360c7df985ae8c7cc4c1265c71004585dc556
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:1362
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:1362

```

##### ③创建slave2节点

```
[root@Rocky8 ~]#redis-cli 
127.0.0.1:6379> AUTH 123456
OK
127.0.0.1:6379> REPLICAOF 192.168.188.81 6379
OK 
127.0.0.1:6379> CONFIG SET masterauth 123456 
OK
127.0.0.1:6379> INFO replication
# Replication
role:slave
master_host:192.168.188.81
master_port:6379
master_link_status:up
master_last_io_seconds_ago:8
master_sync_in_progress:0
slave_repl_offset:1614
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:ddc360c7df985ae8c7cc4c1265c71004585dc556
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:1614
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:211
repl_backlog_histlen:1404

```

# 5、哨兵模式

## 5.1、介绍

```
主从架构和MySQL的主从复制一样,无法实现master和slave角色的自动切换，即当master出现故障时,
不能实现自动的将一个slave 节点提升为新的master节点,即主从复制无法实现自动的故障转移功能,如果
想实现转移,则需要手动修改配置,才能将 slave 服务器提升新的master节点.此外只有一个主节点支持写
操作,所以业务量很大时会导致Redis服务性能达到瓶颈
```

**需要解决的主从复制的存在以下弊端：**

```
1、master和slave角色的自动切换，且不能影响业务
2、提升Redis服务整体性能，支持更高并发访问
```

## 5.2、哨兵Sentinel工作原理

```
哨兵Sentinel从Redis2.6版本开始引用，Redis 2.8版本之后稳定可用。生产环境如果要使用此功能建议
使用Redis的2.8版本以上版本
```

## 5.3、哨兵模式实现

```
首先实现主从复制：
master：192.168.188.8

slave1：192.168.188.81

slave2：192.168.188.82
```

sentinel配置：

```
#master、slave1、slave2的sentinel配置文件
[root@Rocky8 ~]#vim /etc/redis-sentinel.conf

bind 0.0.0.0
logfile "/var/log/redis/sentinel.log"
sentinel monitor mymaster 192.168.188.8 6379 2
sentinel auth-pass mymaster 123456
sentinel down-after-milliseconds mymaster 3000

[root@Rocky8 ~]#systemctl enable --now redis-sentinel.service 

#验证
[root@Rocky8 ~]#ss -ntl
State               Recv-Q              Send-Q                           Local Address:Port                            Peer Address:Port              Process              
LISTEN              0                   128                                    0.0.0.0:26379                                0.0.0.0:*                                      
LISTEN              0                   128                                  127.0.0.1:6379                                 0.0.0.0:*                                      
LISTEN              0                   128                                    0.0.0.0:22                                   0.0.0.0:*                                      
LISTEN              0                   128                                       [::]:22                                      [::]:*                                 [root@Rocky8 ~]#redis-cli -p 26379
127.0.0.1:26379> info sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=192.168.188.8:6379,slaves=2,sentinels=3  #主要配置，#两个
slave,三个sentinel服务器,如果sentinels值不符合,检查myid可能冲突量
     
#模拟故障,停止Master 实现故障转移
[root@redis-master ~]#killall redis-server
[root@Rocky8 ~]#ss -ntl
State               Recv-Q              Send-Q                           Local Address:Port                            Peer Address:Port              Process              
LISTEN              0                   128                                    0.0.0.0:26379                                0.0.0.0:*                                      
LISTEN              0                   128                                    0.0.0.0:22                                   0.0.0.0:*                                      
LISTEN              0                   128                                       [::]:22                                      [::]:*                   

#查看各节点哨兵状态及日志
[root@Rocky8 ~]#tail -f /var/log/redis/sentinel.log   #主节点日志，观察主节点转移至81主机
25407:X 12 Sep 2023 15:04:19.469 # Configuration loaded
25407:X 12 Sep 2023 15:04:19.469 * supervised by systemd, will signal readiness
25407:X 12 Sep 2023 15:04:19.470 * Running mode=sentinel, port=26379.
25407:X 12 Sep 2023 15:04:19.470 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
25407:X 12 Sep 2023 15:04:19.471 # Sentinel ID is e14ebe6a10bd2f458dff58b0306f6cb6e01d1259
25407:X 12 Sep 2023 15:04:19.471 # +monitor master mymaster 192.168.188.8 6379 quorum 2
25407:X 12 Sep 2023 15:04:19.472 * +slave slave 192.168.188.81:6379 192.168.188.81 6379 @ mymaster 192.168.188.8 6379
25407:X 12 Sep 2023 15:04:19.473 * +slave slave 192.168.188.82:6379 192.168.188.82 6379 @ mymaster 192.168.188.8 6379
25407:X 12 Sep 2023 15:04:20.708 * +sentinel sentinel 2f94db21c4c5afc83b777081eb5d215bad6ebcf1 192.168.188.82 26379 @ mymaster 192.168.188.8 6379
25407:X 12 Sep 2023 15:04:21.476 * +sentinel sentinel 9e355c1f92c9f4a43c8b3e647a9026ba7a076a14 192.168.188.81 26379 @ mymaster 192.168.188.8 6379
25407:X 12 Sep 2023 15:17:29.700 # +sdown master mymaster 192.168.188.8 6379
25407:X 12 Sep 2023 15:17:29.763 # +odown master mymaster 192.168.188.8 6379 #quorum 2/2
25407:X 12 Sep 2023 15:17:29.763 # +new-epoch 1
25407:X 12 Sep 2023 15:17:29.763 # +try-failover master mymaster 192.168.188.8 6379
25407:X 12 Sep 2023 15:17:29.765 # +vote-for-leader e14ebe6a10bd2f458dff58b0306f6cb6e01d1259 1
25407:X 12 Sep 2023 15:17:29.768 # 2f94db21c4c5afc83b777081eb5d215bad6ebcf1 voted for e14ebe6a10bd2f458dff58b0306f6cb6e01d1259 1
25407:X 12 Sep 2023 15:17:29.768 # 9e355c1f92c9f4a43c8b3e647a9026ba7a076a14 voted for e14ebe6a10bd2f458dff58b0306f6cb6e01d1259 1
25407:X 12 Sep 2023 15:17:29.837 # +elected-leader master mymaster 192.168.188.8 6379
25407:X 12 Sep 2023 15:17:29.837 # +failover-state-select-slave master mymaster 192.168.188.8 6379
25407:X 12 Sep 2023 15:17:29.895 # +selected-slave slave 192.168.188.81:6379 192.168.188.81 6379 @ mymaster 192.168.188.8 6379
25407:X 12 Sep 2023 15:17:29.896 * +failover-state-send-slaveof-noone slave 192.168.188.81:6379 192.168.188.81 6379 @ mymaster 192.168.188.8 6379
25407:X 12 Sep 2023 15:17:29.958 * +failover-state-wait-promotion slave 192.168.188.81:6379 192.168.188.81 6379 @ mymaster 192.168.188.8 6379
25407:X 12 Sep 2023 15:17:30.150 # +promoted-slave slave 192.168.188.81:6379 192.168.188.81 6379 @ mymaster 192.168.188.8 6379
25407:X 12 Sep 2023 15:17:30.150 # +failover-state-reconf-slaves master mymaster 192.168.188.8 6379
25407:X 12 Sep 2023 15:17:30.226 * +slave-reconf-sent slave 192.168.188.82:6379 192.168.188.82 6379 @ mymaster 192.168.188.8 6379
25407:X 12 Sep 2023 15:17:30.921 # -odown master mymaster 192.168.188.8 6379
25407:X 12 Sep 2023 15:17:31.242 * +slave-reconf-inprog slave 192.168.188.82:6379 192.168.188.82 6379 @ mymaster 192.168.188.8 6379
25407:X 12 Sep 2023 15:17:31.242 * +slave-reconf-done slave 192.168.188.82:6379 192.168.188.82 6379 @ mymaster 192.168.188.8 6379
25407:X 12 Sep 2023 15:17:31.331 # +failover-end master mymaster 192.168.188.8 6379
25407:X 12 Sep 2023 15:17:31.332 # +switch-master mymaster 192.168.188.8 6379 192.168.188.81 6379
25407:X 12 Sep 2023 15:17:31.332 * +slave slave 192.168.188.82:6379 192.168.188.82 6379 @ mymaster 192.168.188.81 6379
25407:X 12 Sep 2023 15:17:31.332 * +slave slave 192.168.188.8:6379 192.168.188.8 6379 @ mymaster 192.168.188.81 6379
25407:X 12 Sep 2023 15:18:01.382 # +sdown slave 192.168.188.8:6379 192.168.188.8 6379 @ mymaster 192.168.188.81 6379
25407:X 12 Sep 2023 15:23:43.445 # -sdown slave 192.168.188.8:6379 192.168.188.8 6379 @ mymaster 192.168.188.81 6379
25407:X 12 Sep 2023 15:23:53.404 * +convert-to-slave slave 192.168.188.8:6379 192.168.188.8 6379 @ mymaster 192.168.188.81 6379


#验证节点复制情况
[root@Rocky8 ~]#redis-cli -a 123456  #.81
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
127.0.0.1:6379> dbsize
(integer) 100005

[root@Rocky8 ~]#redis-cli -a 123456   #.82
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
127.0.0.1:6379> dbsize
(integer) 100005

注：#配置文件
#[root@Rocky8 ~]# vim /etc/redis.conf
replicaof 192.168.188.8 6379 
requirepass 123456
masterauth 123456

#[root@Rocky8 ~]# vim /etc/redis-sentinel.conf
logfile "/var/log/redis/sentinel.log"
sentinel monitor mymaster 192.168.188.8 6379 2
sentinel auth-pass mymaster 123456
```

## 5.4、**python** **连接** **Sentinel** **哨兵**

```
[root@Rocky8 ~]#vim sentinel_test.py 

#!/usr/bin/python3
import redis
from redis.sentinel import Sentinel

#连接哨兵服务器(主机名也可以用域名)
sentinel = Sentinel([('192.168.188.8', 26379),
                     ('192.168.188.81', 26379),
                     ('192.168.188.82', 26379)
             ],  
                    socket_timeout=0.5)

redis_auth_pass='123456'

#mymaster 是运维人员配置哨兵模式的数据库名称，实际名称按照个人部署案例来填写
#获取主服务器地址
master = sentinel.discover_master('mymaster')
print(master)


#获取从服务器地址
slave = sentinel.discover_slaves('mymaster')
print(slave)



#获取主服务器进行写入
master = sentinel.master_for('mymaster', socket_timeout=0.5, password=redis_auth_pass, db=0)
w_ret = master.set('name', 'successly')
#输出：True


#获取从服务器进行读取（默认是round-roubin）
slave = sentinel.slave_for('mymaster', socket_timeout=0.5, password=redis_auth_pass, db=0)
r_ret = slave.get('name')
print(r_ret)
#输出:successly



#加执行权限，运行脚本
[root@Rocky8 ~]#./sentinel_test.py 
('192.168.188.8', 6379)   #主节点
[('192.168.188.82', 6379), ('192.168.188.81', 6379)]   #slave节点
b'successly'
```

# 6、Redis Cluster

## 6.1、安装启动redis cluster集群

### 6.1.1、准备6台主机：

```
三主三从：
192.168.188.8
192.168.188.81
192.168.188.82
192.168.188.83
192.168.188.84
192.168.188.85
```

### 6.1.2、安装部署cluster集群

```
#yum安装并修改配置
[root@Rocky8 ~]#sed -i.bak -e 's/bind 127.0.0.1/bind 0.0.0.0/' -e '/masterauth/a masterauth 123456' -e '/# requirepass/a requirepass 123456' -e '/# cluster-enabled yes/a cluster-enabled yes' -e '/# cluster-config-file nodes-6379.conf/a cluster-config-file nodes-6379.conf' -e '/cluster-requirefull-coverage yes/c cluster-require-full-coverage no' /etc/redis.conf
[root@Rocky8 ~]#systemctl enable --now redis.service
#启动cluster主从关系
[root@Rocky8 ~]#redis-cli -a 123456 --cluster create 192.168.188.8:6379 192.168.188.81:6379 192.168.188.82:6379 192.168.188.83:6379 192.168.188.84:6379 192.168.188.85:6379 --cluster-replicas 1
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 192.168.188.83:6379 to 192.168.188.8:6379
Adding replica 192.168.188.84:6379 to 192.168.188.81:6379
Adding replica 192.168.188.85:6379 to 192.168.188.82:6379
M: 86792cc551f17549ca90ddc6b2c8f1006fd4c974 192.168.188.8:6379
   slots:[0-5460] (5461 slots) master
M: 5c425a605cb60e78111fff001bfa4fc0d831e07a 192.168.188.81:6379
   slots:[5461-10922] (5462 slots) master
M: ae4755464a1791e18f38b1a59af2ce3135f39fa1 192.168.188.82:6379
   slots:[10923-16383] (5461 slots) master
S: 2e4b7972d1b0ab02b6e816ef522dd71290055ce9 192.168.188.83:6379
   replicates 86792cc551f17549ca90ddc6b2c8f1006fd4c974
S: e4162d73be9dbdd4c086149766a3a532bf5aca01 192.168.188.84:6379
   replicates 5c425a605cb60e78111fff001bfa4fc0d831e07a
S: 4d1b44cb817de8f88c767e0c32b279e2afbabba2 192.168.188.85:6379
   replicates ae4755464a1791e18f38b1a59af2ce3135f39fa1
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
..
>>> Performing Cluster Check (using node 192.168.188.8:6379)
M: 86792cc551f17549ca90ddc6b2c8f1006fd4c974 192.168.188.8:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: 5c425a605cb60e78111fff001bfa4fc0d831e07a 192.168.188.81:6379
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 4d1b44cb817de8f88c767e0c32b279e2afbabba2 192.168.188.85:6379
   slots: (0 slots) slave
   replicates ae4755464a1791e18f38b1a59af2ce3135f39fa1
S: e4162d73be9dbdd4c086149766a3a532bf5aca01 192.168.188.84:6379
   slots: (0 slots) slave
   replicates 5c425a605cb60e78111fff001bfa4fc0d831e07a
M: ae4755464a1791e18f38b1a59af2ce3135f39fa1 192.168.188.82:6379
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: 2e4b7972d1b0ab02b6e816ef522dd71290055ce9 192.168.188.83:6379
   slots: (0 slots) slave
   replicates 86792cc551f17549ca90ddc6b2c8f1006fd4c974
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
[root@Rocky8 ~]#redis-cli -a 123456 cluster info
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:3
cluster_stats_messages_ping_sent:386
cluster_stats_messages_pong_sent:361
cluster_stats_messages_meet_sent:3
cluster_stats_messages_sent:750
cluster_stats_messages_ping_received:357
cluster_stats_messages_pong_received:389
cluster_stats_messages_meet_received:4
cluster_stats_messages_received:750
[root@Rocky8 ~]#redis-cli -a 123456
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
127.0.0.1:6379> set car abc
(error) MOVED 9461 192.168.188.81:6379
127.0.0.1:6379> get car
(error) MOVED 9461 192.168.188.81:6379
127.0.0.1:6379> quit
[root@Rocky8 ~]#redis-cli -a 123456 -c
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
127.0.0.1:6379> set car abc
-> Redirected to slot [9461] located at 192.168.188.81:6379
OK
192.168.188.81:6379> get car
"abc"
```

### 6.1.3、确认集群各节点状态

```
#主从信息，节点状态
[root@Rocky8 ~]#cat /var/lib/redis/nodes-6379.conf 
5c425a605cb60e78111fff001bfa4fc0d831e07a 192.168.188.81:6379@16379 master - 0 1683817474035 8 connected 5461-10922
4d1b44cb817de8f88c767e0c32b279e2afbabba2 192.168.188.85:6379@16379 slave ae4755464a1791e18f38b1a59af2ce3135f39fa1 0 1683817476049 6 connected
e4162d73be9dbdd4c086149766a3a532bf5aca01 192.168.188.84:6379@16379 slave 5c425a605cb60e78111fff001bfa4fc0d831e07a 0 1683817478474 8 connected
ae4755464a1791e18f38b1a59af2ce3135f39fa1 192.168.188.82:6379@16379 master - 0 1683817478070 3 connected 10923-16383
86792cc551f17549ca90ddc6b2c8f1006fd4c974 192.168.188.8:6379@16379 myself,master - 0 0 1 connected 0-5460
2e4b7972d1b0ab02b6e816ef522dd71290055ce9 192.168.188.83:6379@16379 slave 86792cc551f17549ca90ddc6b2c8f1006fd4c974 0 1683817477060 4 connected
vars currentEpoch 8 lastVoteEpoch 8

#三主三从，确认主从对应关系
[root@Rocky8 ~]#redis-cli -a 123456
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
127.0.0.1:6379> role
1) "master"
2) (integer) 172704
3) 1) 1) "192.168.188.83"
      2) "6379"
      3) "172704"


[root@Rocky8 ~]#redis-cli -a 123456 role
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
1) "master"
2) (integer) 172993
3) 1) 1) "192.168.188.84"
      2) "6379"
      3) "172993"

[root@Rocky8 ~]#redis-cli -a 123456 role
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
1) "master"
2) (integer) 172848
3) 1) 1) "192.168.188.85"
      2) "6379"
      3) "172848"

[root@Rocky8 ~]#redis-cli -a 123456 role
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
1) "slave"
2) "192.168.188.8"
3) (integer) 6379
4) "connected"
5) (integer) 172928

[root@Rocky8 ~]#redis-cli -a 123456 role
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
1) "slave"
2) "192.168.188.81"
3) (integer) 6379
4) "connected"
5) (integer) 172993

[root@Rocky8 ~]#redis-cli -a 123456 role
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
1) "slave"
2) "192.168.188.82"
3) (integer) 6379
4) "connected"
5) (integer) 172848
```

### 6.1.4、python程序实现redis cluster访问

```
[root@Rocky8 ~]#yum -y install python3
[root@Rocky8 ~]#pip3 install redis-py-cluster
[root@Rocky8 ~]#vim redis_cluster_test.py 

#!/usr/bin/env python3
from rediscluster  import RedisCluster

if __name__ == '__main__':

    startup_nodes = [ 
        {"host":"192.168.188.8", "port":6379},
        {"host":"192.168.188.81", "port":6379},
        {"host":"192.168.188.82", "port":6379},
        {"host":"192.168.188.83", "port":6379},
        {"host":"192.168.188.84", "port":6379},
        {"host":"192.168.188.85", "port":6379}]
    try:
        redis_conn= RedisCluster(startup_nodes=startup_nodes,password='123456', decode_responses=True)
    except Exception as e:
        print(e)

    for i in range(0, 10000):
        redis_conn.set('key'+str(i),'value'+str(i))
        print('key'+str(i)+':',redis_conn.get('key'+str(i)))

[root@Rocky8 ~]#chmod +x redis_cluster_test.py
[root@Rocky8 ~]#./redis_cluster_test.py

#验证数据插入10000条记录
可以看出三台主节点各有数据3000多条
[root@Rocky8 ~]#redis-cli -a 123456 dbsize
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
(integer) 3331

[root@Rocky8 ~]#redis-cli -a 123456 dbsize
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
(integer) 3341

[root@Rocky8 ~]#redis-cli -a 123456 dbsize
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
(integer) 3329

```

## 6.2、实现缩扩容

### 6.2.1、添加新节点到集群

```
扩容适用场景：
当前客户量激增，现有的Redis cluster架构已经无法满足越来越高的并发访问请求，为解决此问题,新购置两台服务器，要求将其动态添加到现有集群，但不能影响业务的正常访问。
注意: 生产环境一般建议master节点为奇数个,比如:3,5,7,以防止脑裂现象
```

#### 6.2.1.1、**添加节点准备**

```
增加Redis 新节点，需要与之前的Redis node版本和配置一致，然后分别再启动两台Redis node，应为
一主一从。

#
#yum安装并修改redis配置文件
[root@Rocky8 ~]#sed -i.bak -e 's/bind 127.0.0.1/bind 0.0.0.0/' -e '/masterauth/a masterauth 123456' -e '/# requirepass/a requirepass 123456' -e '/# cluster-enabled yes/a cluster-enabled yes' -e '/# cluster-config-file nodes-6379.conf/a cluster-config-file nodes-6379.conf' -e '/cluster-requirefull-coverage yes/c cluster-require-full-coverage no' /etc/redis.conf
[root@Rocky8 ~]#systemctl enable --now redis.service
```

#### 6.2.1.2、**添加新的**master节点到集群

```
使用以下命令添加新节点，要添加的新redis节点IP和端口添加到的已有的集群中任意节点的IP:端口

add-node new_host:new_port existing_host:existing_port [--slave --master-id
<arg>]
#说明：
new_host:new_port #指定新添加的主机的IP和端口
existing_host:existing_port #指定已有的集群中任意节点的IP和端口


#使用redis cluser命令添加新的master节点
[root@Rocky8 ~]#redis-cli -a 123456 --cluster add-node 192.168.188.86:6379  192.168.188.8:6379  #任意node
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Adding node 192.168.188.86:6379 to cluster 192.168.188.8:6379
>>> Performing Cluster Check (using node 192.168.188.8:6379)
M: 86792cc551f17549ca90ddc6b2c8f1006fd4c974 192.168.188.8:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: 5c425a605cb60e78111fff001bfa4fc0d831e07a 192.168.188.81:6379
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 4d1b44cb817de8f88c767e0c32b279e2afbabba2 192.168.188.85:6379
   slots: (0 slots) slave
   replicates ae4755464a1791e18f38b1a59af2ce3135f39fa1
S: e4162d73be9dbdd4c086149766a3a532bf5aca01 192.168.188.84:6379
   slots: (0 slots) slave
   replicates 5c425a605cb60e78111fff001bfa4fc0d831e07a
M: ae4755464a1791e18f38b1a59af2ce3135f39fa1 192.168.188.82:6379
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: 2e4b7972d1b0ab02b6e816ef522dd71290055ce9 192.168.188.83:6379
   slots: (0 slots) slave
   replicates 86792cc551f17549ca90ddc6b2c8f1006fd4c974
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
>>> Send CLUSTER MEET to node 192.168.188.86:6379 to make it join the cluster.
[OK] New node added correctly.

[root@Rocky8 ~]#redis-cli -a 123456 cluster nodes  #查看新增节点状态
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
e4162d73be9dbdd4c086149766a3a532bf5aca01 192.168.188.84:6379@16379 slave 5c425a605cb60e78111fff001bfa4fc0d831e07a 0 1694757956857 8 connected
ae4755464a1791e18f38b1a59af2ce3135f39fa1 192.168.188.82:6379@16379 master - 0 1694757954842 3 connected 10923-16383
5201b067425cb7e6f837a33b55ec630272d293c1 192.168.188.86:6379@16379 myself,master - 0 1694703174000 0 connected
4d1b44cb817de8f88c767e0c32b279e2afbabba2 192.168.188.85:6379@16379 slave ae4755464a1791e18f38b1a59af2ce3135f39fa1 0 1694757959875 3 connected
2e4b7972d1b0ab02b6e816ef522dd71290055ce9 192.168.188.83:6379@16379 slave 86792cc551f17549ca90ddc6b2c8f1006fd4c974 0 1694757957863 1 connected
5c425a605cb60e78111fff001bfa4fc0d831e07a 192.168.188.81:6379@16379 master - 0 1694757955851 8 connected 5461-10922
86792cc551f17549ca90ddc6b2c8f1006fd4c974 192.168.188.8:6379@16379 master - 0 1694757958870 1 connected 0-5460

[root@Rocky8 ~]#redis-cli -a 123456 cluster info  #查看集群状态
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:7
cluster_size:3
cluster_current_epoch:8
cluster_my_epoch:0
cluster_stats_messages_ping_sent:248
cluster_stats_messages_pong_sent:259
cluster_stats_messages_meet_sent:6
cluster_stats_messages_sent:513
cluster_stats_messages_ping_received:259
cluster_stats_messages_pong_received:254
cluster_stats_messages_received:513

#重新分配槽位
[root@Rocky8 ~]#redis-cli -a 123456 --cluster reshard 192.168.188.8:6379
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Performing Cluster Check (using node 192.168.188.8:6379)
M: 86792cc551f17549ca90ddc6b2c8f1006fd4c974 192.168.188.8:6379
   slots:[0-1363],[1365-5460] (5460 slots) master
   1 additional replica(s)
M: 5c425a605cb60e78111fff001bfa4fc0d831e07a 192.168.188.81:6379
   slots:[1364],[5461-6824],[6827-10922] (5461 slots) master
   1 additional replica(s)
S: 4d1b44cb817de8f88c767e0c32b279e2afbabba2 192.168.188.85:6379
   slots: (0 slots) slave
   replicates ae4755464a1791e18f38b1a59af2ce3135f39fa1
S: e4162d73be9dbdd4c086149766a3a532bf5aca01 192.168.188.84:6379
   slots: (0 slots) slave
   replicates 5c425a605cb60e78111fff001bfa4fc0d831e07a
M: ae4755464a1791e18f38b1a59af2ce3135f39fa1 192.168.188.82:6379
   slots:[6825-6826],[10923-16383] (5463 slots) master
   1 additional replica(s)
M: ceb4d5c6e92a5f38f9cb5e82ec7135274380944c 192.168.188.86:6379
   slots: (0 slots) master
S: 2e4b7972d1b0ab02b6e816ef522dd71290055ce9 192.168.188.83:6379
   slots: (0 slots) slave
   replicates 86792cc551f17549ca90ddc6b2c8f1006fd4c974
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
How many slots do you want to move (from 1 to 16384)? 1363
What is the receiving node ID? 86792cc551f17549ca90ddc6b2c8f1006fd4c974
Please enter all the source node IDs.
  Type 'all' to use all the nodes as source nodes for the hash slots.
  Type 'done' once you entered all the source nodes IDs.
Source node #1: ceb4d5c6e92a5f38f9cb5e82ec7135274380944c
Source node #2: done

Ready to move 1363 slots.
  Source nodes:
    M: ceb4d5c6e92a5f38f9cb5e82ec7135274380944c 192.168.188.86:6379
       slots: (0 slots) master
  Destination node:
    M: 86792cc551f17549ca90ddc6b2c8f1006fd4c974 192.168.188.8:6379
       slots:[0-1363],[1365-5460] (5460 slots) master
       1 additional replica(s)
  Resharding plan:
Do you want to proceed with the proposed reshard plan (yes/no)? ye

```

#### 6.2.1.3、为新的master指定新的slave节点

```
#任意节点执行
[root@Rocky8 ~]#redis-cli -a 123456 --cluster add-node 192.168.188.87:6379 192.168.188.8:6379 --cluster-slave --cluster-master-id 5201b067425cb7e6f837a33b55ec630272d293c1
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Adding node 192.168.188.87:6379 to cluster 192.168.188.8:6379
>>> Performing Cluster Check (using node 192.168.188.8:6379)
M: 86792cc551f17549ca90ddc6b2c8f1006fd4c974 192.168.188.8:6379
   slots:[1365-5460] (4096 slots) master
   1 additional replica(s)
M: 5c425a605cb60e78111fff001bfa4fc0d831e07a 192.168.188.81:6379
   slots:[6827-10922] (4096 slots) master
   1 additional replica(s)
S: 4d1b44cb817de8f88c767e0c32b279e2afbabba2 192.168.188.85:6379
   slots: (0 slots) slave
   replicates ae4755464a1791e18f38b1a59af2ce3135f39fa1
S: e4162d73be9dbdd4c086149766a3a532bf5aca01 192.168.188.84:6379
   slots: (0 slots) slave
   replicates 5c425a605cb60e78111fff001bfa4fc0d831e07a
M: ae4755464a1791e18f38b1a59af2ce3135f39fa1 192.168.188.82:6379
   slots:[12288-16383] (4096 slots) master
   1 additional replica(s)
M: 5201b067425cb7e6f837a33b55ec630272d293c1 192.168.188.86:6379
   slots:[0-1364],[5461-6826],[10923-12287] (4096 slots) master
S: 2e4b7972d1b0ab02b6e816ef522dd71290055ce9 192.168.188.83:6379
   slots: (0 slots) slave
   replicates 86792cc551f17549ca90ddc6b2c8f1006fd4c974
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
>>> Send CLUSTER MEET to node 192.168.188.87:6379 to make it join the cluster.
Waiting for the cluster to join

>>> Configure node as replica of 192.168.188.86:6379.
[OK] New node added correctly.

#查看集群状态
[root@Rocky8 ~]#redis-cli -a 123456 cluster nodes
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
e4162d73be9dbdd4c086149766a3a532bf5aca01 192.168.188.84:6379@16379 slave 5c425a605cb60e78111fff001bfa4fc0d831e07a 0 1694758792000 8 connected
ae4755464a1791e18f38b1a59af2ce3135f39fa1 192.168.188.82:6379@16379 master - 0 1694758793737 3 connected 12288-16383
5201b067425cb7e6f837a33b55ec630272d293c1 192.168.188.86:6379@16379 myself,master - 0 1694758793000 9 connected 0-1364 5461-6826 10923-12287
4d1b44cb817de8f88c767e0c32b279e2afbabba2 192.168.188.85:6379@16379 slave ae4755464a1791e18f38b1a59af2ce3135f39fa1 0 1694758788700 3 connected
2e4b7972d1b0ab02b6e816ef522dd71290055ce9 192.168.188.83:6379@16379 slave 86792cc551f17549ca90ddc6b2c8f1006fd4c974 0 1694758792730 1 connected
5c425a605cb60e78111fff001bfa4fc0d831e07a 192.168.188.81:6379@16379 master - 0 1694758790715 8 connected 6827-10922
86792cc551f17549ca90ddc6b2c8f1006fd4c974 192.168.188.8:6379@16379 master - 0 1694758789706 1 connected 1365-5460
0f2d5e5b805b6e38d975594def99148249fd25eb 192.168.188.87:6379@16379 slave 5201b067425cb7e6f837a33b55ec630272d293c1 0 1694758794743 9 connected
```

### 6.2.2、缩减集群中节点

```
#查看槽位分配情况
[root@Rocky8 ~]#cat /var/lib/redis/nodes-6379.conf 
5c425a605cb60e78111fff001bfa4fc0d831e07a 192.168.188.81:6379@16379 master - 0 1683823411636 8 connected 6827-10922
4d1b44cb817de8f88c767e0c32b279e2afbabba2 192.168.188.85:6379@16379 slave ae4755464a1791e18f38b1a59af2ce3135f39fa1 0 1683823411132 6 connected
e4162d73be9dbdd4c086149766a3a532bf5aca01 192.168.188.84:6379@16379 slave 5c425a605cb60e78111fff001bfa4fc0d831e07a 0 1683823407099 8 connected
ae4755464a1791e18f38b1a59af2ce3135f39fa1 192.168.188.82:6379@16379 master - 0 1683823409114 3 connected 12288-16383
86792cc551f17549ca90ddc6b2c8f1006fd4c974 192.168.188.8:6379@16379 myself,master - 0 0 1 connected 1365-5460
0f2d5e5b805b6e38d975594def99148249fd25eb 192.168.188.87:6379@16379 slave 5201b067425cb7e6f837a33b55ec630272d293c1 0 1683823410022 9 connected
5201b067425cb7e6f837a33b55ec630272d293c1 192.168.188.86:6379@16379 master - 0 1683823408108 9 connected 0-1364 5461-6826 10923-12287
2e4b7972d1b0ab02b6e816ef522dd71290055ce9 192.168.188.83:6379@16379 slave 86792cc551f17549ca90ddc6b2c8f1006fd4c974 0 1683823410123 4 connected
vars currentEpoch 9 lastVoteEpoch 8

#释放0-1364槽位
[root@Rocky8 ~]#redis-cli -a 123456 --cluster reshard 192.168.188.8:6379
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Performing Cluster Check (using node 192.168.188.8:6379)
M: 86792cc551f17549ca90ddc6b2c8f1006fd4c974 192.168.188.8:6379
   slots:[1365-5460] (4096 slots) master
   1 additional replica(s)
M: 5c425a605cb60e78111fff001bfa4fc0d831e07a 192.168.188.81:6379
   slots:[6827-10922] (4096 slots) master
   1 additional replica(s)
S: 4d1b44cb817de8f88c767e0c32b279e2afbabba2 192.168.188.85:6379
   slots: (0 slots) slave
   replicates ae4755464a1791e18f38b1a59af2ce3135f39fa1
S: e4162d73be9dbdd4c086149766a3a532bf5aca01 192.168.188.84:6379
   slots: (0 slots) slave
   replicates 5c425a605cb60e78111fff001bfa4fc0d831e07a
M: ae4755464a1791e18f38b1a59af2ce3135f39fa1 192.168.188.82:6379
   slots:[12288-16383] (4096 slots) master
   1 additional replica(s)
S: 0f2d5e5b805b6e38d975594def99148249fd25eb 192.168.188.87:6379
   slots: (0 slots) slave
   replicates 5201b067425cb7e6f837a33b55ec630272d293c1
M: 5201b067425cb7e6f837a33b55ec630272d293c1 192.168.188.86:6379
   slots:[0-1364],[5461-6826],[10923-12287] (4096 slots) master
   1 additional replica(s)
S: 2e4b7972d1b0ab02b6e816ef522dd71290055ce9 192.168.188.83:6379
   slots: (0 slots) slave
   replicates 86792cc551f17549ca90ddc6b2c8f1006fd4c974
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
How many slots do you want to move (from 1 to 16384)? 1364
What is the receiving node ID? 86792cc551f17549ca90ddc6b2c8f1006fd4c974
Please enter all the source node IDs.
  Type 'all' to use all the nodes as source nodes for the hash slots.
  Type 'done' once you entered all the source nodes IDs.
Source node #1: 5201b067425cb7e6f837a33b55ec630272d293c1
Source node #2: done


.。。。。。。。。
    Moving slot 1357 from 5201b067425cb7e6f837a33b55ec630272d293c1
    Moving slot 1358 from 5201b067425cb7e6f837a33b55ec630272d293c1
    Moving slot 1359 from 5201b067425cb7e6f837a33b55ec630272d293c1
    Moving slot 1360 from 5201b067425cb7e6f837a33b55ec630272d293c1
    Moving slot 1361 from 5201b067425cb7e6f837a33b55ec630272d293c1
    Moving slot 1362 from 5201b067425cb7e6f837a33b55ec630272d293c1
    Moving slot 1363 from 5201b067425cb7e6f837a33b55ec630272d293c1
Do you want to proceed with the proposed reshard plan (yes/no)? yes



#释放5461-6826槽位
[root@Rocky8 ~]#redis-cli -a 123456 --cluster reshard 192.168.188.8:6379
#释放10923-12287槽位
[root@Rocky8 ~]#redis-cli -a 123456 --cluster reshard 192.168.188.8:6379


#移除集群节点
[root@Rocky8 ~]#redis-cli -a 123456 --cluster del-node 192.168.188.8:6379 1b067425cb7e6f837a33b55ec630272d293c1  #删除后从节点会找新主
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Removing node 5201b067425cb7e6f837a33b55ec630272d293c1 from cluster 192.168.188.8:6379
>>> Sending CLUSTER FORGET messages to the cluster...
>>> SHUTDOWN the node.

#删除从节点
[root@Rocky8 ~]#redis-cli -a 123456 --cluster del-node 192.168.188.8:6379 0f2d5e5b805b6e38d975594def99148249fd25eb
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Removing node 0f2d5e5b805b6e38d975594def99148249fd25eb from cluster 192.168.188.8:6379
>>> Sending CLUSTER FORGET messages to the cluster...
>>> SHUTDOWN the node.

```



# 常见面试题

```
Redis 做什么的,即在哪些场景下使用
如果监控 Redis 是否出现故障
Redis客户端timeout报错突然增加，排查思路是怎样的？
请简单描述pipeline功能，为什么pipeline功能会提升redis性能?
本地redis-client访问远程Redis服务出错，说出几种常见的错误?
key-value的大小超大或单key的qps超高，会对Redis本身造成什么样的影响、会对访问Redis的其
他客户端造成什么样的影响？
Zabbix 监控 Redis 哪些监控项
RDB和AOF持久化区别
docker拉取一个Redis如何实现数据持久化保存
Redis 支持哪些数据类型
Redis 如何实现消息队列
描述下常见的redis集群架构有哪些，他们之间的优缺点对比
主从复制工作原理
Redis 如何实现高可用
哨兵工作原理
Redis 集群的工作原理
Redis 集群如果避免脑裂
Redis 集群最少几个节点为什么?
Redis的集群槽位多少个
Redis集群中某个节点缺少一个槽位是否能使用
Redis数据写入的时候是怎么在各个节点槽位分配数据的
Redis的数据存储是以什么样的方式存储
Redis集群的各槽位和总槽位之间什么关系
```

**删除主从同步**

```
#保留数据，但是会删除slave角色
127.0.0.1:6379> REPLICAOF no one 
OK
127.0.0.1:6379> dbsize
(integer) 3
127.0.0.1:6379> keys
(error) ERR wrong number of arguments for 'keys' command
127.0.0.1:6379> keys *
1) "car"
2) "people"
3) "class"
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:0
master_replid:6bd3853984d90962278c50964d08f2eae19635bf
master_replid2:5937ff079b9d1a82c5e15033c6c0617400435b95
master_repl_offset:2022
second_repl_offset:2023
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1208
repl_backlog_histlen:815
```

```

logfile "/var/log/redis/sentinel.log"
sentinel monitor mymaster 192.168.188.8 6379 2
sentinel auth-pass mymaster 123456
sentinel down-after-milliseconds mymaster 30

```





```
# CentOS-Base.repo
[base]
name=CentOS-8.5.2111 - Base - mirrors.cloud.aliyuncs.com
baseurl=http://mirrors.cloud.aliyuncs.com/centos-vault/8.5.2111/BaseOS/$basearch/os/
        http://mirrors.aliyuncs.com/centos-vault/8.5.2111/BaseOS/$basearch/os/
        http://mirrors.aliyun.com/centos-vault/8.5.2111/BaseOS/$basearch/os/
gpgcheck=0
gpgkey=http://mirrors.cloud.aliyuncs.com/centos/RPM-GPG-KEY-CentOS-Official

#additional packages that may be useful
[extras]
name=CentOS-8.5.2111 - Extras - mirrors.cloud.aliyuncs.com
baseurl=http://mirrors.cloud.aliyuncs.com/centos-vault/8.5.2111/extras/$basearch/os/
        http://mirrors.aliyuncs.com/centos-vault/8.5.2111/extras/$basearch/os/
        http://mirrors.aliyun.com/centos-vault/8.5.2111/extras/$basearch/os/
gpgcheck=0
gpgkey=http://mirrors.cloud.aliyuncs.com/centos/RPM-GPG-KEY-CentOS-Official

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-8.5.2111 - Plus - mirrors.cloud.aliyuncs.com
baseurl=http://mirrors.cloud.aliyuncs.com/centos-vault/8.5.2111/centosplus/$basearch/os/
        http://mirrors.aliyuncs.com/centos-vault/8.5.2111/centosplus/$basearch/os/
        http://mirrors.aliyun.com/centos-vault/8.5.2111/centosplus/$basearch/os/
gpgcheck=0
enabled=0
gpgkey=http://mirrors.cloud.aliyuncs.com/centos/RPM-GPG-KEY-CentOS-Official

[PowerTools]
name=CentOS-8.5.2111 - PowerTools - mirrors.cloud.aliyuncs.com
baseurl=http://mirrors.cloud.aliyuncs.com/centos-vault/8.5.2111/PowerTools/$basearch/os/
        http://mirrors.aliyuncs.com/centos-vault/8.5.2111/PowerTools/$basearch/os/
        http://mirrors.aliyun.com/centos-vault/8.5.2111/PowerTools/$basearch/os/
gpgcheck=0
enabled=0
gpgkey=http://mirrors.cloud.aliyuncs.com/centos/RPM-GPG-KEY-CentOS-Official

                                                                                   [AppStream]
name=CentOS-8.5.2111 - AppStream - mirrors.cloud.aliyuncs.com
baseurl=http://mirrors.cloud.aliyuncs.com/centos-vault/8.5.2111/AppStream/$basearch/os/
        http://mirrors.aliyuncs.com/centos-vault/8.5.2111/AppStream/$basearch/os/
        http://mirrors.aliyun.com/centos-vault/8.5.2111/AppStream/$basearch/os/
gpgcheck=0
gpgkey=http://mirrors.cloud.aliyuncs.com/centos/RPM-GPG-KEY-CentOS-Official
                                                                                                                                       
```

