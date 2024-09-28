# ZABBIX可视化监控

# 1、zabbix介绍

## 1.1、**Zabbix** **功能**

```
https://www.zabbix.com/documentation/5.0/zh/manual/introduction/features
```

![image-20231129124120640](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231129124120640.png)

![image-20231129124133843](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231129124133843.png)

```
1、数据采集
	历史数据, 即记录每个监控项采集到的每个监控值
	趋势数据, 主要保留某个监控项一个小时内历史数据的最大值、最小值和平均值以及该监控项一个小时内所采集到的数据个数可用性和性能检查；
	支持 SNMP（包括主动轮询和被动捕获）、IPMI、JMX、VMware 监控；
	自定义检查；
	按照自定义的时间间隔采集需要的数据；
	通过 Server/Proxy 和 Agents 来执行数据采集。
2、灵活的阈值定义
	您可以参考后端数据库定义非常灵活的告警阈值，即触发器
3、高度可配置化的告警
	可以根据递增计划、接收者、媒介类型自定义发送告警通知；
	使用宏变量可以使告警通知变得更加高效有用；
	自动操作包含远程执行命令。
4、实时图形
	使用内置图形功能可以将监控项实时绘制成图形。
5、Web 监控功能
	https://www.zabbix.com/documentation/5.0/zh/manual/introduction/features；Zabbix可以追踪模拟鼠标在 Web 网站上的点击操作，来检查 Web 网站的功能和响应时间。
6、丰富的可视化选项
	可以组合多个监控项到单个视图中，创建自定义图表；
	网络拓扑图；
	以仪表盘样式展示自定义聚合图形和幻灯片演示；
	报表；
	监控资源的更高层次展示视图（业务视图）。
7、历史数据存储
	存储在数据库中的数据；
	历史配置；
	内置数据管理机制
8、配置简单
	将被监控设备添加为主机；
	主机一旦添加到数据库中，就会采集数据用于监控；
	将模板用于监控设备。
9、使用模板
	模板中分组检查；
	模板可以关联模板，继承已关联模板的属性。
10、网络发现
	自动发现网络设备；
	Zabbix Agent 发现设备后自动注册；
	自动发现文件系统、网络接口和 SNMP OIDs 值。
11、快捷的 Web 界面
	基于 PHP 的 Web 前端；
	可以从任何地方访问；
	您可以定制自己的操作方式；
	您可以通过审计日志来查看你的操作。
12、Zabbix API
	Zabbix API 为 Zabbix 提供可编程接口，用于批量操作、第三方软件集成和其他用途。
13、权限管理系统
	安全的用户身份验证；
	指定的用户只能查看指定的权限范围内的视图。
14、功能强大且易于扩展的 Zabbix Agent
	部署于被监控对象上；
	支持 Linux 和 Windows ；
15、二进制守护进程
	为了更好的性能和更少的内存占用，采用 C 语言编写；
	便于移植。
16、适应更复杂的环境
	使用 Zabbix Proxy 代理，可以轻松实现分布式远程监控
```

## 1.2、**Zabbix** **架构**

![image-20231129124608333](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231129124608333.png)

![image-20231129124622152](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231129124622152.png)

### 1.2.1、SERVER

```
Zabbix server 是 Zabbix 软件的核心组件
Zabbix Agent 向其报告可用性、系统完整性信息和统计信息。
Zabbix server也是存储所有配置信息、统计信息和操作信息的核心存储库。
Zabbix server也是Zabbix监控系统的告警中心。在监控的系统中出现任何异常，将发出通知给管理员。
基本的 Zabbix Server 的功能分解成为三个不同的组件。他们是：Zabbix server、Web前端和数据库。


Zabbix 的所有配置信息都存储在 Server和Web前端进行交互的数据库中。例如，当你通过Web前端（或者API）新增一个监控项时，它会被添加到数据库的监控项表里。然后，Zabbix server 以每分钟一次的频率查询监控项表中的有效项，接着将它存储在 Zabbix server 中的缓存里。这就是为什么 Zabbix前端所做的任何更改需要花费两分钟左右才能显示在最新的数据段的原因。
```

### 1.2.2、数据库

```
所有配置信息以及 Zabbix 采集到的数据都被持久存储在数据库中
可以支持MySQL,PostgreSQL,Oracle 等多种数据库
```

### 1.2.3、WEB 界面

```
WEB 界面是 Zabbix server 的一部分，用于实现展示和配置的界面通常（但不一定）和 Zabbix server 运行在同一台物理机器上基于 Apache(Nginx)+PHP 实现,早期只支持LAMP架构,从Zabbix5.0开始支持LNMP
```

### 1.2.4、AGENT

```
Zabbix agents 部署在被监控目标上，用于主动监控本地资源和应用程序，并将收集的数据发送给Zabbix server。从Zabbix5.0开始支技Zabbix Agent2
```

### 1.2.5、PROXY

```
Zabbix Proxy 可以代替 Zabbix Server 采集性能和可用性数据
Zabbix Proxy 在 Zabbix 的部署是可选部分
Zabbix Proxy 的部署可以很好的分担单个Zabbix server的负载
```

### 1.2.6、Java 网关

```
Zabbix 要监控 tomcat 服务器和其它JAVA程序，需要使用 Java gateway 做为代理,才能从JAVA程序中获取数据
```

### 1.2.7、内部配置的数据流程

```
Zabbix 内部的数据流对Zabbix的使用也很重要。首先，为了创建一个采集数据的监控项，就必须先创建主机。其次，在任务的另外一端，必须要有监控项才能创建触发器（trigger），必须要有触发器来创建动作（action）。因此，如果您想要收到类似“X个server上CPU负载过高”这样的告警，您必须首先为 Server X 创建一个主机条目，其次创建一个用于监控其 CPU的监控项，最后创建一个触发器，用来触发 CPU负载过高这个动作，并将其发送到您的邮箱里。虽然这些步骤看起来很繁琐，但是使用模板的话，实际操作非常简单。也正是由于这种设计，使得 Zabbix 的配置变得更加灵活易用。
```

## 1.3、**Zabbix Server** **启动进程**

![image-20231129125123354](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231129125123354.png)

![image-20231129125130187](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231129125130187.png)

```
zabbix server 进程说明
Zabbix服务器端进程有多达二十多种
当在操作系统下用ps aux命令来查看时，往往看到很多zabbix系统进程，而这些系统进程在zabbix内部称为实例。这些实例各处负责不同的工作，就形成了不同种类型的进程)，各种类型进程的说明如下:
报警器(alerter)——该类型的进程是用来发送报警通知的
配置同步器(configuration syncer)——用于将配置文件中的配置信息同步到内存中缓存
数据发送器(data sender)——服务器代理节点用于发送数据的进程（服务器端没有这类进程）
数据库看门狗(db watchdog)——该进程用于监视zabbix系统的数据库状态，当数据库状态变为不可用时，发送警告信息（服务器代理端不支持这类型进程）。
自动发现器(discoverer)——用于自动发现设备的进程
步骤(escalator)——用于处理动作中的步骤的进程
心跳发送器(heartbeat sender)——服务器代理端用于发送心跳信息（服务器端没有这类型的进程）
历史数据同步器(history syncer)——用于写历史数据表
管家(housekeeper)——用于清理过期的历史数据的进程
HTTP 轮询器(http poller)——用于轮询web类的监控项目
Ping检查器(icmp pinger)——用于定期的进行ICMP PING检查
ipmi 轮询器（ipmi poller）——用于定期进行ipmi监控项目的检查
java 轮询器(java poller)——用于轮询java 监控项目
分布式节点看守器(node watcher)——用于在不同的分布式节点发送历史数据和配置信息更新的进程
轮询器(poller)——用于普通的被动监控项目的轮询
服务器代理轮询(proxy poller)——用于服务器代理的被动轮询
自我监控(self-monitoring)——用于收集Zabbix系统内部的监控信息
定时器(timer)——用于处理触发器中也时间相关的函数和维护模式的进程
陷入器(trapper)——用于处理主动采集、陷入以及分布式节点间或服务器代理的通信
不可到达轮询器(unreachable poller)——用于轮询不可到达到的设备
vmware 收集器(vmware collector)——负责从vmware服务进程中收集数据（服务器代理端不支持这种类型的进程）；
可用的mode参数包括：
avg——指定类型所有进程的平均值
count——返回创建的指定类型进程的数量
max——最大值
min——最小值
——进程号，含义参见“描述”中所述的；
可用的state参数包括:
繁忙(busy)——表示处于繁忙状态的进程；
空闲(idle)——表示处于空闲状态的进程；
```

# 2、Zabbix安装部署

ubuntu尝试修复所有存在的依赖关系问题

```
sudo apt --fix-broken install
```

rocky系统修复所有存在的依赖关系问题

```
yum --allowerasing 允许在升级或安装软件包时删除其依赖的其他软件包。这个选项有时在解决软件包依赖性问题时很有用，但它需要小心使用，因为它可能会删除一些你不希望删除的软件包。

sudo yum update --allowerasing
或
sudo yum install package_name --allowerasing
```

## 2.1、二进制安装部署

### 2.1.1、下载zabbix依赖包

Rocky

```
[root@zabbix_server ~]#rpm -Uvh https://repo.zabbix.com/zabbix/6.0/rhel/8/x86_64/zabbix-release-6.0-4.el8.noarch.rpm
[root@zabbix_server ~]#yum clean all
67 files removed
[root@zabbix_server ~]#yum install zabbix-server-mysql zabbix-web-mysql zabbix-nginx-conf zabbix-sql-scripts zabbix-selinux-policy zabbix-agent
```

ubuntu

```
[root@zabbix-server ~]# wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-4+ubuntu20.04_all.deb
[root@zabbix-server ~]# dpkg -i zabbix-release_6.0-4+ubuntu20.04_all.deb
[root@zabbix-server ~]# apt update
[root@zabbix-server ~]# apt install zabbix-server-mysql zabbix-frontend-php zabbix-nginx-conf zabbix-sql-scripts zabbix-agent
```

### 2.1.2、数据库及nginx安装部署

```
[root@zabbix_server ~]#mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.26 Source distribution

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> create database zabbix character set utf8mb4 collate utf8mb4_bin;
Query OK, 1 row affected (0.05 sec)

mysql> create user zabbix@'192.168.188.%' identified by '123456';
Query OK, 0 rows affected (0.01 sec)

mysql> grant all on zabbix.* to zabbix@'192.168.188.%';
Query OK, 0 rows affected (0.00 sec)

mysql> set global log_bin_trust_function_creators = 1;
Query OK, 0 rows affected (0.00 sec)

mysql> quit

[root@zabbix_server ~]#ALTER USER zabbix@'192.168.188.%' IDENTIFIED WITH mysql_native_password BY '123456';
或
[root@zabbix_server ~]#echo 'default-authentication-plugin=mysql_native_password' >> /etc/my.cnf.d/mysql-server.cnf


#设置数据库密码
[root@zabbix-server ~]#vim /etc/zabbix/zabbix_server.conf 

		.......
		
DBPassword=123456

		.......

#修改nginx配置
[root@zabbix-server ~]#vim /etc/zabbix/nginx.conf 

server {
        listen          8080;   #取消注释
        server_name     zabbix.dingbh.top;    #设置域名

        root    /usr/share/zabbix;

        index   index.php;

......

#重启服务并设置开机自启动
[root@zabbix_server ~]# systemctl restart zabbix-server zabbix-agent nginx php7.4-fpm
[root@zabbix_server ~]# systemctl enable zabbix-server zabbix-agent nginx php7.4-fpm
```

### 2.1.3、访问测试

### LNMP架构访问路径：http://192.168.188.100:8080或通过域名访问zabbix.dingbh.top

![image-20231130142628789](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231130142628789.png)

### 2.1.4下载配置文件并上传至目录/var/www/html/zabbix/conf/

```
[root@zabbix-server ~]#cat /var/www/html/zabbix/conf/zabbix.conf.php
<?php
// Zabbix GUI configuration file.
global $DB;
$DB['TYPE']     = 'MYSQL';
$DB['SERVER']   = '10.0.0.101';
$DB['PORT']     = '0';
$DB['DATABASE'] = 'zabbix';
$DB['USER']     = 'zabbix';
$DB['PASSWORD'] = '123456';
// Schema name. Used for IBM DB2 and PostgreSQL.
$DB['SCHEMA'] = '';
$ZBX_SERVER      = '10.0.0.100';
$ZBX_SERVER_PORT = '10051';
$ZBX_SERVER_NAME = 'Zabbix Server';
$IMAGE_FORMAT_DEFAULT = IMAGE_FORMAT_PNG;
```

### 2.1.5、LAMP和LNMP安装区别

```
lamp需要带zabbix子目录

ngin使用域名访问
```

## 2.2、添加agent

监控其他主机windos、linux主机、网络设备

### 2.2.1、选择agent版本

官网选择合适的版本https://www.zabbix.com/cn/download

![image-20231201150414692](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231201150414692.png)

按照官方文档下载相应的agent包

```
#此处以Rocky8系统为例

[root@Agent ~]# rpm -Uvh https://repo.zabbix.com/zabbix/6.4/rhel/8/x86_64/zabbix-release-6.4-1.el8.noarch.rpm
[root@Agent ~]# dnf clean all
[root@Agent ~]# dnf install zabbix-agent
[root@Agent ~]# systemctl restart zabbix-agent
[root@Agent ~]# systemctl enable zabbix-agent
```

### 2.2.2、修改agent配置文件

#### 监控linux主机

```
[root@Agent ~]#vim /etc/zabbix/zabbix_agentd.conf 
......

Server=192.168.188.100   #修改为zabbix—server服务端IP或域名

......

#重启服务生效
[root@Agent ~]# systemctl restart zabbix-agent
```

#### 监控windos主机

```
官网下载安装包：https://cdn.zabbix.com/zabbix/binaries/stable/6.0/6.0.24/zabbix_agent-6.0.24-windows-amd64-openssl.msi
```

#### 监控nginx服务

使用模板Template App Nginx by HTTP，监控nginx的状态页面status

```
#配置nginx服务生成状态页
[root@ROCKY ~]#vim /etc/nginx/nginx.conf
location /basic_status {       #注意zabbix的宏定义监控的是basic_status，因此搭建的状态页面名称也要保持一致，除非更改宏定义
	stub_status;
	allow 192.168.188.100;    #白名单
	deny all;
}
[root@ROCKY ~]#systemctl restart nginx
```

#### 监控php服务

使用模板Template App PHP-FPM by Zabbix agent

```
[root@Rocky ~]#vim /etc/php-fpm.d/www.conf 
#listen = /run/php-fpm/www.sock   #php-fpm默认无监听端口，通过sock文件启动
listen = 127.0.0.1:9000
pm.status_path = /status          #取消该行注释，注意页面名称和zabbix的宏定义相同
ping.path = /ping                 #取消注释开启ping功能
ping.response = pong pong pong    #可自定义

[root@Rocky ~]#mv /etc/nginx/conf.d/php-fpm.conf{,.bak}     #如不是编译安装的php，会自动添加这主要配置文件需提前备份
[root@Rocky ~]#mv /etc/nginx/default.d/php.conf{,.bak}
[root@Rocky ~]#systemctl restart php-fpm.service
[root@Rocky ~]#vim /etc/nginx/nginx.conf

location ~ ^/(.*\.php|ping|php_status)$ {
	root /data/php;
	#fastcgi_pass 127.0.0.1:9000;
	fastcgi_pass unix:/run/php-fpm/www.sock;
	fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
	include fastcgi_params;
}

[root@Rocky ~]#nginx -t
[root@Rocky ~]#nginx -s reload
```

#### 监控memcache服务

用模板memcached-zabbix agent 2模板

```
[root@Ubuntu1804 ~]# apt install -y memcached
[root@Ubuntu1804 ~]#vim /etc/zabbix/zabbix_agentd.conf 
......

Server=192.168.188.100   #修改为zabbix—server服务端IP或域名

......

#zabbix_server WEB端添加主机并关联模板
```

### 2.2.3、使用root账号运行zabbix_agent

```
#需要修改以下两个配置文件
[root@mysql ~]#vim /etc/zabbix/zabbix_agentd.conf 
### Option: AllowRoot
#   Allow the agent to run as 'root'. If disabled and the agent is started by 'root', the agent
#   will try to switch to the user specified by the User configuration option instead.
#   Has no effect if started under a regular user.
#   0 - do not allow
#   1 - allow
#
# Mandatory: no
# Default:
# AllowRoot=0
AllowRoot=1



[root@mysql ~]#vim /lib/systemd/system/zabbix-agent.service 

[Unit]
Description=Zabbix Agent
After=syslog.target
After=network.target

[Service]
Environment="CONFFILE=/etc/zabbix/zabbix_agentd.conf"
EnvironmentFile=-/etc/sysconfig/zabbix-agent
Type=forking
Restart=on-failure
PIDFile=/run/zabbix/zabbix_agentd.pid
KillMode=control-group
ExecStart=/usr/sbin/zabbix_agentd -c $CONFFILE
ExecStop=/bin/kill -SIGTERM $MAINPID
RestartSec=10s
#User=zabbix
#Group=zabbix

[Install]
WantedBy=multi-user.target
```



# 3、 **自定义模板** **Templates** **和监控项** **Items**

上面对Linux系统的监控是使用系统内置模板的监控项，虽然 Zabbix 内置的模板及其包括的监控项已经很丰富但是内置的监控项可能不一定适合当前环境,或者有些不需要的监控项,或者需要的监控项并没有提供因此实际生产中仍可能需要通过自定义监控项来满足当前环境的监控需求

**自定义模板和监控项实现过程**

```
在Zabbix 被监控主机上编写自定义监控项的取值的脚本,并加执行权限
在Zabbix 被监控主机上的配置文件中添加自定义监控项,指定 key 和 对 key 赋值的脚本及参数
在Zabbix Server 上使用 zabbix_get 工具测试是否能取到自定义监控项信息
在Zabbix Web 创建模板,在模板中添加自定义监控项
在Zabbix Web 将模板关联至被监控的主机
导出自定义模板可以在其它 Zabbix Server 复用
```

除了自定义模板,也可以从zabbix网站查找适合自已的模板:https://share.zabbix.com/

## 3.1、**自定义监控项**

### 3.1.1、自定义监控项介绍

系统内置的监控项如下

```
https://www.zabbix.com/documentation/6.0/zh/manual/config/items/itemtypes/zabbix_agent
https://www.zabbix.com/documentation/5.0/zh/manual/config/items/itemtypes/zabbix_agent
```

内置的监控项无法满足要求,可以自定义监控项

```
1、自定义监控项配置
监控项键值的格式
https://www.zabbix.com/documentation/6.0/zh/manual/config/items/item/key
https://www.zabbix.com/documentation/5.0/zh/manual/config/items/item/key

#客户端可以自定义监控项，在Zabbix Agent 配置文件添加内容,格式如下
#cat /etc/zabbix/zabbix_agentd.conf
#cat /etc/zabbix/zabbix_agent2.conf
UserParameter=<key>,<shell command>
Include=/etc/zabbix/zabbix_agentd.d/*.conf

#或者创建独立的自定义文件
#cat /etc/zabbix/zabbix_agentd.d/*.conf
#cat /etc/zabbix/zabbix_agent2.d/*.conf
UserParameter=<key>,<shell command>

说明：
	key 必须整个系统唯一。注意大小写是敏感的, Key名允许的字符如下
	
	 0-9a-zA-Z_-.
	 
	key 使用 [*] 用于定义该key接受括号内的参数。参数需在配置监控项时给出；参数禁止使用下列字符：
	
	\ ’ ” ` * ? [ ] { } ~ $ ! & ; ( ) <>
	
	Command：命令用于生成key对应的值。
	
	可以在命令中使用位置引用$1 … $9来引用监控项Key中的相应参数。Zabbix解析监控项Key的[]中
	包含的参数，并相应地替换$1，…，$9。$0会替换为完整的原始命令（在对$0，…，$9执行替换之
	前的命令）运行。不管位置参数（$0,…,$9)是用双引号( “ )还是单引号( ’ )括起来，都会解析位置引用

测试监控项
#不需要重启服务：
zabbix_agentd -t "在客户端定义的key名[arg1,arg2,...]"
zabbix_agent2 -t "在客户端定义的key名[arg1,arg2,...]"
在Zabbix Server上可以使用zabbix_get工具获取自定义监控项
#需要重启服务：systemctl restart zabbix-agent2.service
zabbix_get -s 客户端IP -p 10050 -k "在客户端定义的key名[arg1,arg2,...]"

注意: 
如果用脚本实现首行必须加shebang机制
如果编译安装的程序,需要写程序的完整路径

2、宏Macros
Zabbix 支持用户自定义宏,即支持变量定义，自定义宏格式为: {$macrosz_name}
https://www.zabbix.com/documentation/5.0/zh/manual/appendix/macros
```

### 3.1.2、范例：取根文件系统的空间利用率

```
[root@Rocky8 ~]#vim /etc/zabbix/zabbix_agentd.d/test.conf 

UserParameter=use_df,df /boot|awk -F' +|%' 'NR!=1{print $5}'

#客户端测试
[root@Rocky8 ~]#zabbix_agentd -t use_df
use_df 

#重启服务后服务器端才能获取数据
[root@Rocky8 ~]#systemctl restart zabbix-agent.service 
#服务器端测试

[root@zabbix-server ~]#yum -y install zabbix-get
或
[root@zabbix-server ~]#apt -y install zabbix-get

[root@zabbix-server ~]# zabbix_get -s 192.168.188.8 -p 10050 -k 'use_df'
17
```

### 3.1.3、范例：自定义监控项实现连接数

```
1、取值脚本
[root@Rocky8 ~]#cat /etc/zabbix/zabbix_agentd.d/tcp_state.sh 

#! /bin/bash
netstat -nat|awk -v STATE=$1 '$NF ~ STATE{state[$NF]++}END{for(i in state){print state[i]}}'

2、agnet本机测试
[root@Rocky8 ~]#bash /etc/zabbix/zabbix_agentd.d/tcp_state.sh LISTEN
8

3、zabbix_agent文件配置
[root@Rocky8 ~]#cat /etc/zabbix/zabbix_agentd.d/test.conf 
#UserParameter=tcp_state_ESTABLISHED,netstat -nat|grep ESTABLISHED |wc -l
#UserParameter=tcp_state_TIME_WAIT,netstat -nat|grep TIME_WAIT |wc -l
#UserParameter=tcp_state_listen,netstat -nat|grep LISTEN |wc -l
UserParameter=tcp_state[*],/etc/zabbix/zabbix_agentd.d/tcp_state.sh $1  #添加相应键值及关联脚本
UserParameter=use_df,df /boot|awk -F' +|%' 'NR!=1{print $5}'

4、agent测试
[root@Rocky8 ~]#zabbix_agentd -t tcp_state[LISTEN]
tcp_state[LISTEN]                             [t|8]

5、服务器端测试
[root@zabbix-server ~]# zabbix_get -s 192.168.188.8 -p 10050 -k 'tcp_state[LISTEN]'
8
```

### 3.1.4、范例：实现自定义监控项的参数

```
#修改agent文件配置
[root@Rocky8 ~]#vim /etc/zabbix/zabbix_agentd.d/test.conf 

UserParameter=test[*],echo $1

#agent端测试
[root@Rocky8 ~]#zabbix_agentd -t test["This is test"]
test[This is test]                            [t|This is test]

#zabbix-server服务端测试
[root@zabbix-server ~]# zabbix_get -s 192.168.188.8 -p 10050 -k 'test["this is test"]'
this is test
```

### 3.1.5、范例：利用自定义监控项的参数功能监控MySQL的存活状态

```
[root@Rocky8 ~]#yum -y install mysql
[root@Rocky8 ~]#mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 8.0.26 Source distribution

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> create user zabbixtest@'192.168.188.%' identified by '123456';  #创建zabbix测试账号
Query OK, 0 rows affected (0.02 sec)

mysql> quit


#mysql取消登录warning信息
[root@Rocky8 ~]#vim /etc/my.cnf.d/client.cnf 

[client]
user=zabbixtest
password=123456

#修改zabbix_agent配置，--defaults-file=/etc/my.cnf.d/.client.cnf，不仅可以避免警告，还可以增强MySQL凭据的安全性
[root@Rocky8 my.cnf.d]#mysqladmin  --defaults-file=/etc/my.cnf.d/.client.cnf  -h192.168.188.8 ping 2>/dev/null|grep -c "mysqld is alive"  #将错误信息丢进回收站
1


#agent端测试
[root@Rocky8 ~]#zabbix_agentd -t mysql_ping
1


#服务端测试
[root@zabbix-server ~]# zabbix_get -s 192.168.188.8 -p 10050 -k 'mysql_ping'
1
或者使用自带模板，检测MySQL健康度
[root@zabbix-server ~]# zabbix_get -s 192.168.188.8 -p 10050 -k 'net.tcp.listen[3306]'
1
```

## 3.2、自定义模板

所谓模板,就是一些相关的监控项的集合,还可以在模板中包括图形,触发器等

### 3.2.1、为什么需要模板?

```
为每个主机单独添加相同的一组监控项比较麻烦，例如:100台主机增加一个tcp80的监控项
修改监控项比较麻烦，例如:100台服务器的监控项将80改成8080
删除每个主机的同一组监控项，例如:100台服务器的监控项删除
删除主机会连同主机的监控项一起被删除；不能复用监控项
所以可以使用模板的方式，来解决上述的一些问题
```

### 3.2.2、**自定义模板使用流程**

```
创建模板，模板必须属于某个主机组(一般属于主机组Templates)
在模板中创建监控项、图形、触发器
创建需要监控的主机，然后关联对应的模板
更改模板的监控项目，所以使用模板的都会自动更改
导出模板，后期可以至其他系统继续使用
```

## 3.3、**监控项的值映射** **Value Mapping**

```
https://www.zabbix.com/documentation/6.0/zh/manual/config/items/mapping?hl=%E5%80%BC%E6%98%A0%E5%B0%84
https://www.zabbix.com/documentation/5.0/zh/manual/config/items/mapping?hl=%E5%80%BC%E6%98%A0%E5%B0%84
```

**什么是值映射**？

```
为了接收到的值能更“人性化”的显示，可以通过值映射方式，将数值与字符串之间进行关系绑定
示例: 
http 响应码
'200' → 'OK'
'403' → 'Forbidden'
'404' → 'Not Found'
```

**创建值映射**

Zabbix 6.0 以上版本配置

要定义一个值映射:

```
1、打开主机或者模板配置表单
2、前往 值映射 标签
3、点击 增加 来增加一个新映射
4、点击一个已存在的值映射名字来进行编辑
```

## **3.4** **触发器** **Triggers**

### 3.4.1、磁盘已满告警

问题表达式

```
last(/N75/use_df)>80
```

恢复表达式

```
last(/N75/use_df)<=70
```

告警通知



# 4、图形

## 4.2、**Zabbix** **利用** **Grafana** **进行图形展示**

```
虽然 Zabbix 自身带有图形功能，但是并不美观，而利用 Grafana 可以实现相当精美的 Web 图形显示Grafana 是一款采用 go 语言编写的开源应用，主要用于大规模指标数据的可视化展现，是网络架构和应用分析中最流行的时序数据展示工具，目前已经支持绝大部分常用的时序数据库
Grafana支持许多不同的数据源。每个数据源都有一个特定的查询编辑器,该编辑器定制的特性和功能是公开的特定数据来源。 官方支持以下数据源:Zabbix，Graphite，Elasticsearch，InfluxDB，Prometheus，Cloudwatch，MySQL和OpenTSDB等。
官方站点: https://grafana.com/

使用 Grafana 显示Zabbix的监控数据过程
	安装 grafana
	安装 zabbix 插件
	创建 zabbix 数据源
	导入 dashboard 模板
	
```

**注意**:如果浏览器无法正常显示grafana的图形，可能是浏览器版本问题，建议更换浏览器再尝试

### 4.2.1、下载grafana包

```
[root@zabbix-server ~]# wget https://mirrors.aliyun.com/grafana/apt/pool/main/g/grafana-enterprise/grafana-enterprise_10.2.2_amd64.deb
[root@zabbix-server ~]# dpkg -i grafana-enterprise_10.2.2_amd64.deb 

[root@zabbix-server ~]# dpkg -i grafana-enterprise_9.5.2_amd64.deb 
Selecting previously unselected package grafana-enterprise.
(Reading database ... 113349 files and directories currently installed.)
Preparing to unpack grafana-enterprise_9.5.2_amd64.deb ...
Unpacking grafana-enterprise (9.5.2) ...
Setting up grafana-enterprise (9.5.2) ...
Adding system user `grafana' (UID 117) ...
Adding new user `grafana' (UID 117) with group `grafana' ...
Not creating home directory `/usr/share/grafana'.
### NOT starting on installation, please execute the following statements to configure grafana to start automatically using systemd
 sudo /bin/systemctl daemon-reload
 sudo /bin/systemctl enable grafana-server
### You can start grafana-server by executing
 sudo /bin/systemctl start grafana-server
Processing triggers for systemd (245.4-4ubuntu3.20) ...

#安装之后需进行指定开机重启
[root@zabbix-server ~]# systemctl daemon-reload
[root@zabbix-server ~]# systemctl enable --now grafana-server.service 

#安装zabbix插件
[root@zabbix-server ~]# grafana-cli plugins list-remote|grep zabbix
id: alexanderzobnin-zabbix-app version: 4.4.4

#筛选网络zabbix插件资源及版本
[root@zabbix-server ~]# grafana-cli plugins list-remote|grep zabbix
id: alexanderzobnin-zabbix-app version: 4.4.4
[root@zabbix-server ~]# grafana-cli plugins list-versions alexanderzobnin-zabbix-app
4.4.4
4.4.3
4.4.2
4.4.1
4.4.0
4.3.1
4.3.0
4.2.10
4.2.9
4.2.8
4.2.7
4.2.6
4.2.5
4.2.4
4.2.3
4.2.2
4.2.1
4.2.0
4.1.5
4.1.4
4.1.3
4.1.2
4.1.1
4.1.0
4.0.2
4.0.1
4.0.0
3.12.4
3.12.3
3.12.2
3.12.1
3.12.0
3.11.0
3.10.5
3.10.4
3.10.3
3.10.2
3.10.1
3.10.0
3.9.1
3.9.0
3.8.1
3.8.0
3.7.0
3.6.1
3.6.0
3.5.1
3.5.0
3.4.0
3.3.0
3.2.1
3.2.0
3.1.2
3.1.1
3.1.0
3.0.0
3.0.0-beta8
3.0.0-beta7
3.0.0-beta6
3.0.0-beta5
3.0.0-beta4
3.0.0-beta3

#安装zabbix插件
[root@zabbix-server ~]# grafana-cli plugins install alexanderzobnin-zabbix-app

✔ Downloaded and extracted alexanderzobnin-zabbix-app v4.4.4 zip successfully to /var/lib/grafana/plugins/alexanderzobnin-zabbix-app

Please restart Grafana after installing or removing plugins. Refer to Grafana documentation for instructions if necessary.

#重启服务
[root@zabbix-server ~]# systemctl restart grafana-server.service 

#查看插件路径
[root@zabbix-server ~]# ls -l /var/lib/grafana/
total 1132
drwxr-x--- 3 grafana grafana    4096 Dec 11 14:56 alerting
drwx------ 2 grafana grafana    4096 Dec 11 14:56 csv
-rw-r----- 1 grafana grafana 1138688 Dec 11 15:29 grafana.db   #
drwxr-xr-x 3 root    root       4096 Dec 11 15:28 plugins       适合离线安装
drwx------ 2 grafana grafana    4096 Dec 11 14:56 png

[root@zabbix-server ~]# ls -l /var/lib/grafana/plugins/alexanderzobnin-zabbix-app
total 21432
-rw-r--r-- 1 root root    38514 Dec 11 15:28 CHANGELOG.md
drwxr-xr-x 4 root root     4096 Dec 11 15:28 datasource
-rwxr-xr-x 1 root root     3177 Dec 11 15:28 go_plugin_build_manifest
-rwxr-xr-x 1 root root 21852160 Dec 11 15:28 gpx_zabbix-plugin_linux_amd64
drwxr-xr-x 2 root root     4096 Dec 11 15:28 img
-rw-r--r-- 1 root root    11379 Dec 11 15:28 LICENSE
-rw-r--r-- 1 root root     3602 Dec 11 15:28 MANIFEST.txt
-rw-r--r-- 1 root root      921 Dec 11 15:28 module.js
-rw-r--r-- 1 root root     3807 Dec 11 15:28 module.js.map
drwxr-xr-x 3 root root     4096 Dec 11 15:28 panel-triggers
-rw-r--r-- 1 root root     1760 Dec 11 15:28 plugin.json
-rw-r--r-- 1 root root     3035 Dec 11 15:28 README.md
drwxr-xr-x 2 root root     4096 Dec 11 15:28 styles

#查看已安装插件
[root@zabbix-server ~]# grafana-cli plugins ls
installed plugins:
alexanderzobnin-zabbix-app @ 4.4.4

```

### 4.2.2、添加插件

![image-20231211154054552](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231211154054552.png)



![image-20231211154113830](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231211154113830.png)



![image-20231211154131231](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231211154131231.png)

默认不启用；点击Enable插件

![image-20231211154159360](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231211154159360.png)

### 4.2.3、添加数据源

![image-20231211154405114](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231211154405114.png)



![image-20231211154422389](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231211154422389.png)

添加访问url

```
添加访问域名：
#nginx
http://zabbix.dingbh.top:8080/api_jsonrpc.php

#apache
http://zabbix.dingbh.top:8080/zabbix/api_jsonrpc.php
```

![image-20231211154742140](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231211154742140.png)



![image-20231211155030190](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231211155030190.png)

保存配置

![image-20231211155104313](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231211155104313.png)





# 5、**用户** **Users** **和组** **Groups** **管理**

```
用户组：指定可访问资源的范围，比如哪些主机可以被访问到
角色： 加入角色可以控制用户是否有管理能力，还有更多的细小权限分派，比如：具体某个菜单项的访问
```

**用户管理**

```
系统默认有Admin和guest两个用户，其中guest，密码为空，且默认被禁用

	创建新用户

	管理 --- 用户

	创建新用户,指定群组
```

# 6、告警

```
生产环境中，通常当达到一定的指标,比如硬盘使用率为80%左右时，会触发触发器的条件,而触发器可以再触发动作,实现事件通知告警或执行指定命令等.要实现事件通知告警，就需要一个中间介质来发送告警消息给运维

Zabbix 有如下常见报警介质:
	E-mail	
	微信
	短信
	
```

## 6.1、邮件告警

### 6.1.1、ubuntu实现邮件告警脚本

```
#查看放松邮件的相关配置

[root@zabbix-server ~]# grep alert /etc/zabbix/zabbix_server.conf 
#	Number of pre-forked instances of alerters.
#	Full path to location of custom alert scripts.
# AlertScriptsPath=/usr/lib/zabbix/alertscripts

#告警发送脚本路径
[root@zabbix-server ~]# ls /usr/lib/zabbix/alertscripts
send_email.sh

```



```
ubuntu邮件发送脚本
[root@zabbix-server ~]#vim /usr/lib/zabbix/alertscripts/send_email.sh 

#!/bin/bash
# 
#********************************************************************
#Author:            dingbaohang
#QQ:                904748581
#Date:              2023-11-31
#FileName:          send_email.sh
#URL:               http://www.wangxiaochun.com
#Description:       The test script
#Copyright (C):     2023 All rights reserved
#********************************************************************

email_send='904748581@qq.com'
email_passwd='kqchnzcqsvyubfac'
email_smtp_server='smtp.qq.com'

. /etc/os-release

msg_error() {
  echo -e "\033[1;31m$1\033[0m"
}

msg_info() {
  echo -e "\033[1;32m$1\033[0m"
}

msg_warn() {
  echo -e "\033[1;33m$1\033[0m"
}


color () {
    RES_COL=60
    MOVE_TO_COL="echo -en \\033[${RES_COL}G"
    SETCOLOR_SUCCESS="echo -en \\033[1;32m"
    SETCOLOR_FAILURE="echo -en \\033[1;31m"
    echo -n "$1" && $MOVE_TO_COL
    echo -n "["
    if [ $2 = "success" -o $2 = "0" ] ;then
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


install_sendemail () {
    if [[ $ID =~ rhel|centos|rocky ]];then
        rpm -q sendemail &> /dev/null ||  yum install -y sendemail
    elif [ $ID = 'ubuntu' ];then
        dpkg -l |grep -q sendemail  || { apt update; apt install -y libio-socket-ssl-perl libnet-ssleay-perl sendemail ; }
    else
        color "不支持此操作系统，退出!" 1
        exit
    fi
}

send_email () {
    local email_receive="$1"
    local email_subject="$2"
    local email_message="$3"
    sendemail -f $email_send -t $email_receive -u $email_subject -m $email_message -s $email_smtp_server -o message-charset=utf-8 -o tls=yes -xu $email_send -xp $email_passwd
    [ $? -eq 0 ] && color "邮件发送成功!" 0 || color "邮件发送失败!" 1
}

if [ $# -ne 3 ];then
    color "脚本参数不正确!" 1
    msg_info "Usage: `basename $0` <mail_address> <subject> <message>"
    exit 1
fi

install_sendemail

send_email "$1" "$2" "$3"
```

![路径](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231204181845137.png)

![image-20231205182336827](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231205182336827.png)

### 6.1.2、邮件本机测试

```
[root@zabbix-server ~]#./send_email.sh m15269032515@163.com 紧急报警 "this is test"
Dec 01 15:24:47 zabbix-server sendemail[108475]: Email was sent successfully!
邮件发送成功!                                              [  OK  ]
[root@zabbix-server ~]#mv send_email.sh /usr/lib/zabbix/alertscripts/
```

### 6.1.3、邮件模板

告警

```
主题: 告警: {EVENT.NAME}
告警主机：{HOST.NAME1}
告警服务: {ITEM.NAME1}
告警Key1: {ITEM.KEY1}：{ITEM.VALUE1}
告警Key2: {ITEM.KEY2}：{ITEM.VALUE2}
严重级别: {TRIGGER.SEVERITY}
```

恢复

```
主题: 恢复: {EVENT.DURATION}: {EVENT.NAME}
恢复主机：{HOST.NAME1}
恢复服务： {ITEM.NAME1}
恢复Key1：{ITEM.KEY1}：{ITEM.VALUE1}
恢复Key2: {ITEM.KEY2}：{ITEM.VALUE2}
```

# 7、实现故障自愈

## 7.1、zabbix Agent开启远程命令功能

```
配置远程命令的操作类似于发送消息，区别在于一个执行命令，一个发送消息
远程命令可以直接在ZabbixServer, ZabbixProxy和ZabbixAgent上执行。
但在Zabbix agent和Zabbix proxy上，远程命令默认是不开启的，它们可以通过以下方式启用：
在agent配置中添加
	AllowKey=system.run[*]
	UnsafeUserParameters=1
在proxy配置中，将enableremotecommands参数设置为1
```

范例: 在故障自愈的客户端主机开启 Zabbix Agent 远程命令功能

### 7.1.1、客户端配置

```
[root@Rocky8 ~]#vim /etc/zabbix/zabbix_agentd.conf 

AllowKey=system.run[*]    #开启执行功能，此项适合zabbix5.0版本以上,agent2默认没有配置,需要手工配置，用于代替早期版本的中EnableRemoteCommands
#EnableRemoteCommands=1   #开启远程执行命令,此指令在zabbix5.0版本以上淘汰


UnsafeUserParameters=1     #允许远程执行命令的时候使用不安全的参数(特殊字符串,如: \ ' * ? [] {} ~ $ ! & ; ( ) < > 等 )
                           #1表示允许，默认为0不允许



#使用sudo提高zabbix账户权限，sudo授权
[root@Rocky8 ~]#vim /etc/sudoers

zabbix  ALL=(ALL) NOPASSWD: ALL


#因为默认zabbix账户是nologin模式，需增加shebang机制提高权限
[root@Rocky8 ~]#su -s /bin/bash zabbix -c "sudo systemctl restart nginx"
[root@Rocky8 ~]#systemctl status nginx.service 
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/nginx.service.d
           └─php-fpm.conf
   Active: active (running) since Sat 2023-12-02 16:33:31 CST; 5s ago
  Process: 207917 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 207915 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 207913 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 207918 (nginx)
    Tasks: 5 (limit: 11176)
   Memory: 7.9M
   CGroup: /system.slice/nginx.service
           ├─207918 nginx: master process /usr/sbin/nginx
           ├─207919 nginx: worker process
           ├─207920 nginx: worker process
           ├─207921 nginx: worker process
           └─207922 nginx: worker process

Dec 02 16:33:31 Rocky8 systemd[1]: nginx.service: Succeeded.
Dec 02 16:33:31 Rocky8 systemd[1]: Stopped The nginx HTTP and reverse proxy server.
Dec 02 16:33:31 Rocky8 systemd[1]: Starting The nginx HTTP and reverse proxy server...
Dec 02 16:33:31 Rocky8 nginx[207915]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Dec 02 16:33:31 Rocky8 nginx[207915]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Dec 02 16:33:31 Rocky8 systemd[1]: Started The nginx HTTP and reverse proxy server.

```



### 7.1.2、添加远程执行脚本

![image-20231205101800108](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231205101800108.png)

### 7.1.3、填写scope、类型、命令信息

![image-20231205101851689](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231205101851689.png)

### 7.1.4、查看脚本添加完成

![image-20231205101915887](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231205101915887.png)



### 7.1.5、添加故障自愈动作

![image-20231205101009776](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231205101009776.png)

### 7.1.6、填写名称、计算方式（或 满足其一就触发）、条件信息

![image-20231205101338303](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231205101338303.png)

### 7.1.6、动作→操作

![image-20231205102604522](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231205102604522.png)

### 7.6.7、填写操作细节（目标列表：当前主机√）

![image-20231205102627965](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231205102627965.png)

### 7.1.8、查看动作添加完成

![image-20231205102832452](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231205102832452.png)

### 7.1.9、客户端测试、关闭nginx服务

```
[root@Rocky8 ~]#systemctl stop nginx.service 
[root@Rocky8 ~]#systemctl status nginx.service 
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/nginx.service.d
           └─php-fpm.conf
   Active: inactive (dead) since Sat 2023-12-02 17:48:02 CST; 3s ago
  Process: 214542 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 214539 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 214536 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 214543 (code=exited, status=0/SUCCESS)

Dec 02 17:18:23 Rocky8 systemd[1]: Starting The nginx HTTP and reverse proxy server...
Dec 02 17:18:23 Rocky8 nginx[214539]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Dec 02 17:18:23 Rocky8 nginx[214539]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Dec 02 17:18:23 Rocky8 systemd[1]: Started The nginx HTTP and reverse proxy server.
Dec 02 17:48:02 Rocky8 systemd[1]: Stopping The nginx HTTP and reverse proxy server...
Dec 02 17:48:02 Rocky8 systemd[1]: nginx.service: Succeeded.
Dec 02 17:48:02 Rocky8 systemd[1]: Stopped The nginx HTTP and reverse proxy server.
[root@Rocky8 ~]#ss -ntl
State               Recv-Q              Send-Q                           Local Address:Port                            Peer Address:Port             Process              
LISTEN              0                   128                                    0.0.0.0:10050                                0.0.0.0:*                                     
LISTEN              0                   128                                    0.0.0.0:22                                   0.0.0.0:*                                     
LISTEN              0                   128                                       [::]:10050                                   [::]:*                                     
LISTEN              0                   70                                           *:33060                                      *:*                                     
LISTEN              0                   128                                          *:3306                                       *:*                                     
LISTEN              0                   128                                       [::]:22                                      [::]:*    
```

### 7.1.10、服务端查看仪表盘告警

![image-20231205174920862](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231205174920862.png)

```
[root@Rocky8 zabbix_agentd.d]#systemctl status nginx.service 
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/nginx.service.d
           └─php-fpm.conf
   Active: active (running) since Sat 2023-12-02 20:22:48 CST; 43s ago
  Process: 241186 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 241184 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 241182 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 241187 (nginx)
    Tasks: 5 (limit: 11176)
   Memory: 7.9M
   CGroup: /system.slice/nginx.service
           ├─241187 nginx: master process /usr/sbin/nginx
           ├─241188 nginx: worker process
           ├─241189 nginx: worker process
           ├─241190 nginx: worker process
           └─241191 nginx: worker process

Dec 02 20:22:48 Rocky8 systemd[1]: Starting The nginx HTTP and reverse proxy server...
Dec 02 20:22:48 Rocky8 nginx[241184]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Dec 02 20:22:48 Rocky8 nginx[241184]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Dec 02 20:22:48 Rocky8 systemd[1]: Started The nginx HTTP and reverse proxy server.

[root@Rocky8 zabbix_agentd.d]#ss -ntl
State               Recv-Q              Send-Q                           Local Address:Port                            Peer Address:Port             Process              
LISTEN              0                   128                                    0.0.0.0:10050                                0.0.0.0:*                                     
LISTEN              0                   128                                    0.0.0.0:80                                   0.0.0.0:*                                     
LISTEN              0                   128                                    0.0.0.0:22                                   0.0.0.0:*                                     
LISTEN              0                   128                                       [::]:10050                                   [::]:*                                     
LISTEN              0                   70                                           *:33060                                      *:*                                     
LISTEN              0                   128                                          *:3306                                       *:*                                     
LISTEN              0                   128                                       [::]:80                                      [::]:*                                     
LISTEN              0                   128                                       [::]:22                                      [::]:*   
```

# 8、zabbix主动模式与被动模式

```
Zabbix 有两种工作模式: 主动和被动模式

无论是模式还是被动模式，都是站在zabbix agent 角度来说的工作模式
```

## 8.1、**Zabbix** **的主动和被动模式工作原理**

![image-20231206113959890](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231206113959890.png)

```
Zabbix 监控流程中 Agent 收集数据分为主动和被动两种模式
主动模式和被动模式都是相对 zabbix agent 而言的
Zabbix默认是被动模式，如果有100个监控项，被动模式需要 Zabbix Server找 Zabbix Agent要100次
主动模式是Zabbix Server给 Zabbix Agent发送一个包括100个监控项的任务清单，Zabbix Agent根据任
务清单，采集好100个监控项的值，主动汇报给 Zabbix Server这100个监控项，Zabbix Agent主动模式
只需要发送一次数据，大大提高了传输效率。
	被动模式: Server 向 Agent 请求获取监控项的数据，Agent返回数据。此为默认模式,如果有100个监控项,则需要100次交互
	主动模式: Agent 请求 Server 获取主动的监控项列表，并主动将监控项内需要检测的数据提交给server/proxy,如有100个监控项,只需要1次交互即可
	
注意: 两种模式可以在Zabbix Server上同时存在,可以将一部分监控项设为主动,其它设为被动模式
```

**被动监测通信过程如下**:

```
Server打开一个TCP连接
Server发送请求agent.ping
Agent接收到请求并且响应<HEADER><DATALEN>
Server处理接收到的数据
关闭TCP连接
被动模式每获取一个监控项都需要打开一个tcp连接，这样当监控项越来越多时，Zabbix Server会打开很多端口,就会出现server端性能瓶颈问题。
```

**主动模式监测通信过程如下:**

```
#获取ACTIVE ITEMS列表
	Agent主动打开一个TCP连接（主动检测变成Agent打开）
	Agent请求items检测列表
	Server返回items列表
	Agent 处理响应
	关闭TCP连接
	Agent开始收集数据
```

```
#主动检测提交数据过程
	Agent建立TCP连接
	Agent批量提交items列表收集的所有监控项数据
	Server处理数据，并返回响应状态
	关闭TCP连接
```

## 8.2、zabbix被动模式

### 8.2.1、zabbix被动模式介绍

```
被动模式是指 zabbix agent 被动的接受zabbix server(或者Zabbix Proxy)周期性发送过来的数据收集指令，此为默认的工作方式。

在被动模式之下，zabbix server会根据主机关联的模板中的监控项和数据采集间隔时间，周期性的打开随机端口并向zabbix agent服务器的10050端口发起tcp连接，然后发送获取监控项数据的指令，即zabbix server发送什么指令那么zabbix agent就收集什么数据，zabbix server什么时候发送指令，zabbix agent就什么时候采集，zabbix server不发送指令，zabbix agent就一直不响应，所以zabbix agent也不用关心其监控项和数据采集周期间隔时间。

被动模式的优点就是配置简单，安装后即可使用，因此也成为zabbix的默认工作模式，但是被动模式的最大问题就是会加大zabbix server的 负载，在数百甚至数千台服务器的环境下会导致zabbix server需要轮训向每个zabbix agent发送数据采集指令，如果zabbix server负载过高还会导致不能及时获取到最新数据，被动模式因为性能的原因,一台 Zabbix Server 一般只能监控500台以下的主机,更多的主机建议使用主动模式
```

**Zabbix Server 通过Poller进程实现被动模式**

```
[root@zabbix-server ~]# grep StartPollers /etc/zabbix/zabbix_server.conf  
### Option: StartPollers
# StartPollers=5					#此数量可以适当进行调整
### Option: StartPollersUnreachable
# StartPollersUnreachable=1

[root@zabbix-server ~]# ps ax|grep poller
     84 ?        I<     0:00 [edac-poller]
   9460 ?        S      0:00 /usr/sbin/zabbix_server: http poller #1 [got 0 values in 0.000776 sec, idle 5 sec]
   9470 ?        S      0:00 /usr/sbin/zabbix_server: proxy poller #1 [exchanged data with 0 proxies in 0.000027 sec, idle 5 sec]
   9473 ?        S      0:02 /usr/sbin/zabbix_server: poller #1 [got 0 values in 0.000043 sec, idle 1 sec]
   9476 ?        S      0:03 /usr/sbin/zabbix_server: poller #2 [got 9 values in 0.012213 sec, idle 1 sec]
   9477 ?        S      0:02 /usr/sbin/zabbix_server: poller #3 [got 0 values in 0.000038 sec, idle 1 sec]
   9478 ?        S      0:02 /usr/sbin/zabbix_server: poller #4 [got 0 values in 0.000026 sec, idle 1 sec]
   9479 ?        S      0:02 /usr/sbin/zabbix_server: poller #5 [got 0 values in 0.000015 sec, idle 1 sec]
   9481 ?        S      0:00 /usr/sbin/zabbix_server: unreachable poller #1 [got 0 values in 0.000035 sec, idle 5 sec]
   9490 ?        S      0:00 /usr/sbin/zabbix_server: history poller #1 [got 0 values in 0.000043 sec, idle 1 sec]
   9491 ?        S      0:00 /usr/sbin/zabbix_server: history poller #2 [got 2 values in 0.000094 sec, idle 1 sec]
   9495 ?        S      0:00 /usr/sbin/zabbix_server: history poller #3 [got 0 values in 0.000121 sec, idle 1 sec]
   9497 ?        S      0:00 /usr/sbin/zabbix_server: history poller #4 [got 0 values in 0.000027 sec, idle 1 sec]
   9502 ?        S      0:00 /usr/sbin/zabbix_server: history poller #5 [got 0 values in 0.000042 sec, idle 1 sec]
   9507 ?        S      0:00 /usr/sbin/zabbix_server: odbc poller #1 [got 0 values in 0.000046 sec, idle 5 sec]
  14514 pts/8    S+     0:00 grep --color=auto poller

```

**查看被动模式下 Zabbix agent 进程和端口打开情况**

```
[root@Rocky8 ~]# grep StartAgents /etc/zabbix/zabbix_agentd.conf
### Option: StartAgents
# StartAgents=3

[root@Rocky8 ~]# ps ax|grep zabbix
  4639 pts/0    S+     0:00 grep --color=auto zabbix
 81117 ?        Ss     0:00 /usr/sbin/zabbix_agentd --foreground
 81126 ?        S      0:06 /usr/sbin/zabbix_agentd: collector [idle 1 sec]
 81127 ?        S      0:00 /usr/sbin/zabbix_agentd: listener #1 [waiting for connection]
 81128 ?        S      0:00 /usr/sbin/zabbix_agentd: listener #2 [waiting for connection]
 81129 ?        S      0:00 /usr/sbin/zabbix_agentd: listener #3 [waiting for connection]
 81130 ?        S      0:06 /usr/sbin/zabbix_agentd: active checks #1 [idle 1 sec]
[root@Rocky8 ~]#ss -ant|grep 10050
LISTEN    0      128          0.0.0.0:10050         0.0.0.0:*           
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:51698       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:51584       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:44574       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:56038       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:52630       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:56004       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:44680       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:57564       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:51624       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:52442       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:56032       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:51634       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:44616       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:56184       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:51740       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:53760       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:44660       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:57478       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:44600       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:53806       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:51596       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:51654       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:52526       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:33862       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:57372       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:33822       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:56154       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:57360       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:52594       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:53816       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:52518       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:52452       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:33774       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:56146       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:57470       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:56100       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:44548       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:44580       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:53744       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:51688       
TIME-WAIT 0      0      192.168.188.8:10050 192.168.188.100:44558       

#被动模式下agent主机的日志中会提示以下错误
#被动模式下agent主机的日志中会提示以下错误
[root@Rocky8 ~]#tail /var/log/zabbix/zabbix_agentd.log 
  1414:20200830:180435.723 TLS support:           YES
  1414:20200830:180435.723 **************************
  1414:20200830:180435.723 using configuration file: 
/etc/zabbix/zabbix_agentd.conf
  1414:20200830:180435.724 agent #0 started [main process]
  1415:20200830:180435.724 agent #1 started [collector]
  1416:20200830:180435.725 agent #2 started [listener #1]
  1419:20200830:180435.725 agent #4 started [listener #3]
    1420:20200830:180435.726 agent #5 started [active checks #1]
  1418:20200830:180435.726 agent #3 started [listener #2]
  1420:20200830:180435.727 active check configuration update from 
[127.0.0.1:10051] started to fail (cannot connect to [[127.0.0.1]:10051]: [111] 
Connection refused)
```

## 8.3、zabbix主动模式

**主动模式介绍**

```
主动模式是由每个zabbix agent 打开本机的随机端口, 主动向 Zabbix Server(或者Zabbix Proxy)的10051端口发起tcp连接请求
主动模式下必须在zabbix agent配置文件中指定zabbix server的IP或者主机名(必须可以被解析为IP地址)，在连接到zabbix server之前zabbix agent是不知道自己要采集那些数据以及间隔多久采集一次数据的，然后在连接到zabbix server以后获取到自己的监控项和数据采集间隔周期时间，之后再根据监控项采集数据并返回给zabbix server，在主动模式下不再需要zabbix serve向zabbix agent发起连接请求，因此主动模式在一定程度上可减轻zabbix server打开的本地随机端口和进程数，在一定程度就减轻了zabbix server的压力。经过优化后可以轻松监控2000台，甚至5000以上的主机
```

**以下场景下生产推荐使用主动模式**

```
Zabbix Server 出现性能瓶颈
监控队列出现大量延迟的监控项,告警不及时
监控主机超过 500 台以上
```

**注意：主动模式模板Linux by Zabbix agent active的agent.ping监控项ZBX标记不会变绿，需要修改为被动模式ZBX标记才能被绿**

### 8.3.1、模板全克隆

![image-20231206080126931](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231206080126931.png)



### 8.3.2、重命名N75模板

![image-20231206080229800](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231206080229800.png)



### 8.3.3、查看新克隆模板N75-active

![image-20231206080354761](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231206080354761.png)



### 8.3.4、全选监控项，点击批量更新

![image-20231206080456637](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231206080456637.png)



### 8.3.5、类型选择zabbix客户端（主动式）

![image-20231206080543930](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231206080543930.png)



# 9、监控 JAVA 程序

## **9.1、Zabbix Java Gateway** **介绍**

```
Zabbix 不支持直接监控JAVA应用
如果要监控JAVA程序比如Tomcat等，需要使用 Java gateway 做为代理,才能从JAVA应用中获取数据,Zabbix Java Gateway 通过监听10052/tcp 端口实现监控JAVA应用

注意: JAVA应用要求开启JMX功能才被被监控
```

![image-20231206113423658](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231206113423658.png)

```
Zabbix 监控JVM流程
	zabbix-server 通知 zabbix-Java-Gateway需要获取监控主机的哪些监控项
	Zabbix-Java-Gateway 通过 JMX 协议请求采集 Java进程数据
	Java程序通过 JMX 协议返回数据给 zabbix-Java-Gateway 
	zabbix-Java-Gateway 最终返回数据给 zabbix-server
	zabbix-server 将采集的 JAVA 数据存储至数据库，然后进行 Web 展示
```

**Zabbix Java Gateway** **架构**

![image-20231206113509926](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231206113509926.png)

## 9.2、**实战案例:** **监控** **Java** **应用** **Tomcat** **服务**

```
CATALINA_OPTS="$CATALINA__OPTS -Dcom.sun.management.jmxremote-Dcom.sun.management.jmxremote -Djava.rmi.server.hostname=192.168.188.200 -Dcom.sun.management.jmxremote.port=12345 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false
```

### 9.2.1、**部署** **tomcat** **服务并开启** **JMX** **功能**

#### 9.2.1.1、安装java环境

```
1、安装java

[root@Rocky8 ~]#ll
total 151344
-rw-------. 1 root root      1464 Mar 29  2023 anaconda-ks.cfg
-rw-r--r--  1 root root  11645293 Dec  3 06:16 apache-tomcat-9.0.75.tar.gz
-rwxr-xr-x  1 root root      3608 Dec  3 06:16 install_tomcat.sh
-rw-r--r--  1 root root 139219380 Dec  3 06:16 jdk-8u371-linux-x64.tar.gz

2、解压到/usr/local/目录下

[root@Rocky8 ~]#tar xf jdk-8u371-linux-x64.tar.gz -C /usr/local/
[root@Rocky8 ~]#cd /usr/local/

[root@Rocky8 local]#ln -s jdk1.8.0_371/ jdk

[root@Rocky8 local]#ll
total 8
drwxr-xr-x  9 tomcat tomcat 4096 Dec  3 06:41 apache-tomcat-9.0.75
drwxr-xr-x. 2 root   root      6 Oct 11  2021 bin
drwxr-xr-x. 2 root   root      6 Oct 11  2021 etc
drwxr-xr-x. 2 root   root      6 Oct 11  2021 games
drwxr-xr-x. 2 root   root      6 Oct 11  2021 include
lrwxrwxrwx  1 root   root     13 Dec  3 06:19 jdk -> jdk1.8.0_371/
drwxr-xr-x  8 root   root   4096 Dec  3 06:18 jdk1.8.0_371
drwxr-xr-x. 2 root   root      6 Oct 11  2021 lib
drwxr-xr-x. 3 root   root     17 Mar 29  2023 lib64
drwxr-xr-x. 2 root   root      6 Oct 11  2021 libexec
drwxr-xr-x. 2 root   root      6 Oct 11  2021 sbin
drwxr-xr-x. 5 root   root     49 Mar 29  2023 share
drwxr-xr-x. 2 root   root      6 Oct 11  2021 src
lrwxrwxrwx  1 root   root     21 Dec  3 06:20 tomcat -> apache-tomcat-9.0.75/

#创建环境变量
[root@Rocky8 local]#vim /etc/profile.d/jdk.sh 

   export JAVA_HOME=/usr/local/jdk
   export PATH=$PATH:$JAVA_HOME/bin

#测试
[root@Rocky8 ~]#java -version
java version "1.8.0_371"
Java(TM) SE Runtime Environment (build 1.8.0_371-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.371-b11, mixed mode)
```

#### 9.2.1.2、安装tomcat

```
[root@Rocky8 ~]#ll
total 151344
-rw-------. 1 root root      1464 Mar 29  2023 anaconda-ks.cfg
-rw-r--r--  1 root root  11645293 Dec  3 06:16 apache-tomcat-9.0.75.tar.gz
-rwxr-xr-x  1 root root      3608 Dec  3 06:16 install_tomcat.sh
-rw-r--r--  1 root root 139219380 Dec  3 06:16 jdk-8u371-linux-x64.tar.gz
[root@Rocky8 ~]#tar xf apache-tomcat-9.0.75.tar.gz -C /usr/local/ 
#创建软链接
[root@Rocky8 local]#ln -s apache-tomcat-9.0.75/ tomcat
#创建账户
[root@Rocky8 local]#useradd -r -s /sbin/nologin tomcat
#创建启动配置文件
[root@Rocky8 local]#touch tomcat/conf/tomcat.conf
[root@Rocky8 local]#vim tomcat/conf/tomcat.conf 

   JAVA_HOME=/usr/local/jdk     #java环境路径
#设置账户权限
[root@Rocky8 local]#chown -R tomcat.tomcat /usr/local/tomcat/

#设置环境变量
[root@Rocky8 ~]#vim /etc/profile.d/tomcat.sh 

   PATH=/usr/local/tomcat/bin:$PATH

#查看启动脚本
[root@Rocky8 ~]#ls /usr/local/tomcat/bin/ -l
total 900
-rw-r----- 1 tomcat tomcat  35213 May  4  2023 bootstrap.jar
-rw-r----- 1 tomcat tomcat  16852 May  4  2023 catalina.bat
-rwxr-x--- 1 tomcat tomcat  25593 Dec  6 17:51 catalina.sh
-rw-r----- 1 tomcat tomcat   1664 May  4  2023 catalina-tasks.xml
-rw-r----- 1 tomcat tomcat   2123 May  4  2023 ciphers.bat
-rwxr-x--- 1 tomcat tomcat   1997 May  4  2023 ciphers.sh
-rw-r----- 1 tomcat tomcat  25765 May  4  2023 commons-daemon.jar
-rw-r----- 1 tomcat tomcat 214019 May  4  2023 commons-daemon-native.tar.gz
-rw-r----- 1 tomcat tomcat   2040 May  4  2023 configtest.bat
-rwxr-x--- 1 tomcat tomcat   1922 May  4  2023 configtest.sh
-rwxr-x--- 1 tomcat tomcat   9100 May  4  2023 daemon.sh
-rw-r----- 1 tomcat tomcat   2091 May  4  2023 digest.bat
-rwxr-x--- 1 tomcat tomcat   1965 May  4  2023 digest.sh
-rw-r----- 1 tomcat tomcat   3606 May  4  2023 makebase.bat
-rwxr-x--- 1 tomcat tomcat   3382 May  4  2023 makebase.sh
-rw-r----- 1 tomcat tomcat   3814 May  4  2023 setclasspath.bat
-rwxr-x--- 1 tomcat tomcat   4317 May  4  2023 setclasspath.sh
-rw-r----- 1 tomcat tomcat   2020 May  4  2023 shutdown.bat
-rwxr-x--- 1 tomcat tomcat   1902 May  4  2023 shutdown.sh   #关闭tomcat
-rw-r----- 1 tomcat tomcat   2022 May  4  2023 startup.bat
-rwxr-x--- 1 tomcat tomcat   1904 May  4  2023 startup.sh   #启动tomcat
-rw-r----- 1 tomcat tomcat  48970 May  4  2023 tomcat-juli.jar
-rw-r----- 1 tomcat tomcat 437622 May  4  2023 tomcat-native.tar.gz
-rw-r----- 1 tomcat tomcat   4574 May  4  2023 tool-wrapper.bat
-rwxr-x--- 1 tomcat tomcat   5540 May  4  2023 tool-wrapper.sh
-rw-r----- 1 tomcat tomcat   2026 May  4  2023 version.bat
-rwxr-x--- 1 tomcat tomcat   1908 May  4  2023 version.sh


#创建tomcat.service服务启动脚本
[root@Rocky8 ~]#cat /lib/systemd/system/tomcat.service 
[Unit]
Description=Tomcat
#After=syslog.target network.target remote-fs.target nss-lookup.target
After=syslog.target network.target

[Service]
Type=forking
EnvironmentFile=/usr/local/tomcat/conf/tomcat.conf
ExecStart=/usr/local/tomcat/bin/startup.sh
ExecStop=/usr/local/tomcat/bin/shutdown.sh
RestartSec=3
PrivateTmp=true
User=tomcat
Group=tomcat

[Install]
WantedBy=multi-user.target


#加载启动脚本并查看状态
[root@Rocky8 ~]#systemctl daemon-reload
[root@Rocky8 ~]#systemctl enable --now tomcat.service
[root@Rocky8 ~]#systemctl status tomcat.service 
● tomcat.service - Tomcat
   Loaded: loaded (/usr/lib/systemd/system/tomcat.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2023-12-06 17:51:10 CST; 19min ago
 Main PID: 325075 (java)
    Tasks: 62 (limit: 11176)
   Memory: 290.2M
   CGroup: /system.slice/tomcat.service
           └─325075 /usr/local/jdk/bin/java -Djava.util.logging.config.file=/usr/local/tomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoa>

Dec 06 17:51:10 Rocky8 systemd[1]: Starting Tomcat...
Dec 06 17:51:10 Rocky8 systemd[1]: Started Tomcat.

```

#### 9.2.1.3、tomcat开启**JMX** **功能**

JMX（Java Management Extensions，即Java管理扩展）是一个为应用程序、设备、系统等植入管理功能的框架。JMX可以跨越一系列异构操作系统平台、系统体系结构和网络传输协议，灵活的开发无缝集成的系统、网络和服务管理应用。

JMX在Java编程语言中定义了应用程序以及网络管理和监控的体系结构、设计模式、应用程序接口以及服务。通常使用JMX来监控系统的运行状态或管理系统的某些方面，比如清空缓存、重新加载配置文件等

**说明：**

```
#二进制安装tomcat
vim /usr/local/tomcat/bin/catalina.sh
CATALINA_OPTS="$CATALINA__OPTS
-Dcom.sun.management.jmxremote                   #启用远程监控JMX 
-Djava.rmi.server.hostname=<JAVA主机IP>             #是tomcat主机的IP地址,不是zabbix
服务器地址
-Dcom.sun.management.jmxremote.port=XXXXX          #默认启动的JMX端口号
-Dcom.sun.management.jmxremote.authenticate=false  #不使用用户名密码
-Dcom.sun.management.jmxremote.ssl=false"         #不使用ssl认证
```

**tomcat 开启远程 JMX 环境配置**

```
1、范例：Ubuntu22.04 包安装tomcat开启JMX功能
[root@ubuntu2204 ~]#apt install tomcat9 -y
[root@ubuntu2204 ~]#dpkg -L tomcat9-common |grep catalina.sh
/usr/share/tomcat9/bin/catalina.sh
#开启JMX功能
[root@ubuntu2204 ~]#vim /usr/share/tomcat9/bin/catalina.sh
......
CATALINA_OPTS="$CATALINA__OPTS -Dcom.sun.management.jmxremote -
Dcom.sun.management.jmxremote.port=12345 -
Dcom.sun.management.jmxremote.authenticate=false -
Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=10.0.0.201"
# OS specific support. $var _must_ be set to either true or false.
[root@ubuntu2204 ~]#systemctl restart tomcat9
[root@ubuntu2204 ~]#ss -ntl|grep 12345
LISTEN 0      50                 *:12345           *:*



2、范例: 二进制安装Tomcat 开启 JMX 功能
[root@Rocky8 ~]#vim /usr/local/tomcat/bin/catalina.sh

		......

CATALINA_OPTS="$CATALINA__OPTS -Dcom.sun.management.jmxremote-Dcom.sun.management.jmxremote -Djava.rmi.server.hostname=192.168.188.200 -Dcom.sun.management.jmxremote.port=12345 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false"
		
		......

#查看端口12345是否打开
[root@Rocky8 ~]#systemctl restart tomcat.service 
[root@Rocky8 ~]#ss -ntl |grep 12345
LISTEN 0      50                      *:12345            *:*  
```

#### 9.2.1.4、**测试通过java工具 **jconsole **进行** **JMX** **监控**

![image-20231206223736299](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231206223736299.png)

**填写IP信息**

![image-20231206223835005](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231206223835005.png)



![image-20231206223917536](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231206223917536.png)





![image-20231206223936177](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231206223936177.png)

#### **9.2.1.5、使用** **java** **客户端获取** **JMX** **信息** *(可选*)

cmdline-jmxclient-0.10.3.jar 是一个开源jar包，在zabbix 中常用于测试的一个工具，可以用来测试 jmx 是否配置正确。在监控方面后还可以对Tomcat各种属性进行监控。下面使用java客户端cmdline-jmxclient进行测试

下载链接

```
https://github.com/qiueer/zabbix/blob/master/Tomcat/cmdline-jmxclient-0.10.3.jar
```

**范例: 测试能否获取到java当前线程数和最大线程数**

```
#下载cmdline-jmxclient-0.10.3.jar文件
[root@Rocky8 ~]#ll
total 151364
-rw-------. 1 root root      1464 Mar 29  2023 anaconda-ks.cfg
-rw-r--r--  1 root root  11645293 Dec  3 06:16 apache-tomcat-9.0.75.tar.gz
-rw-r--r--  1 root root     20124 Dec  6 22:46 cmdline-jmxclient-0.10.3.jar
-rwxr-xr-x  1 root root      3608 Dec  3 06:16 install_tomcat.sh
-rw-r--r--  1 root root 139219380 Dec  3 06:16 jdk-8u371-linux-x64.tar.gz
-rw-r--r--  1 root root      1452 May 11  2023 reset2.sh
-rw-r--r--  1 root root      4213 May 11  2023 reset.sh
-rw-r--r--  1 root root   4080320 Nov 30 18:03 zabbix_all.sql.gz

#测试能否获取到java当前线程数和最大线程数:
[root@zabbix-server ~]#yum -y install java
#Ubuntu安装下面包
[root@zabbix-server ~]#apt install openjdk-8-jdk -y


[root@Rocky8 ~]#java -jar cmdline-jmxclient-0.10.3.jar - 192.168.188.8:12345 'Catalina:name="http-nio-8080",type=ThreadPool' currentThreadCount
12/06/2023 22:50:33 +0800 org.archive.jmx.Client currentThreadCount: 10   #当前线程数10

#测试能否获取到java最大线程数:
[root@Rocky8 ~]#java -jar cmdline-jmxclient-0.10.3.jar - 192.168.188.8:12345 'Catalina:name="http-nio-8080",type=ThreadPool' maxThreads

12/06/2023 22:49:07 +0800 org.archive.jmx.Client maxThreads: 200   #最大线程数200

```

### 9.3.1、**部署** **Java Gateway** **服务器**

```
Java gateway 是一个独立于zabbix server和 zabbix agent的组件，默认使用端口10052/tcp, 所以java gateway可以是单独的一台服务器，但是也可以和zabbix server或者zabbix agent 共用一台服务器

Java gatway 可以用两种方法安装
	包安装
	编译安装
```

**注意：**

​	**Zabbix Server** **无需安装** **JDK**

​	**但 Java gateway 本质就是一个JAVA程序，所以需要安装JDK**

#### 9.3.1.1、**包安装** **Java gateway**

范例：Ubuntu20.04 安装zabbix6 java gateway

```
[root@zabbix-server ~]#wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbixrelease/zabbixrelease_6.0-4+ubuntu22.04_all.deb
[root@zabbix-server ~]# dpkj -i zabbix-release_6.0-4+ubuntu20.04_all.deb
[root@zabbix-server ~]# apt update
[root@zabbix-server ~]# cat /etc/apt/sources.list.d/zabbix.list|grep -v '^#' 

deb https://repo.zabbix.com/zabbix/6.0/ubuntu focal main
deb-src https://repo.zabbix.com/zabbix/6.0/ubuntu focal main

#安装JDK(可选)
[root@zabbix-server ~]# apt -y install openjdk-11-jdk

#安装zabbix-java-gateway会自动安装JDK，但默认首次无法启动成功，提示无法找到JAVA，需要再次手动重新启动服务才能找到Java
[root@zabbix-server ~]# apt -y install zabbix-java-gateway

..........

Created symlink /etc/systemd/system/multi-user.target.wants/zabbix-java-gateway.service → /lib/systemd/system/zabbix-java-gateway.service.
Job for zabbix-java-gateway.service failed because the control process exited with error code.
See "systemctl status zabbix-java-gateway.service" and "journalctl -xe" for details.
invoke-rc.d: initscript zabbix-java-gateway, action "start" failed.
● zabbix-java-gateway.service - Zabbix Java Gateway
     Loaded: loaded (/lib/systemd/system/zabbix-java-gateway.service; enabled; vendor preset: enabled)
     Active: failed (Result: exit-code) since Thu 2023-12-07 02:30:47 UTC; 6ms ago
    Process: 47468 ExecStart=/usr/sbin/zabbix_java_gateway_startup (code=exited, status=1/FAILURE)

Dec 07 02:30:46 zabbix-server.dingbh.top systemd[1]: Starting Zabbix Java Gateway...
Dec 07 02:30:46 zabbix-server.dingbh.top zabbix_java_gateway_startup[47485]: /usr/sbin/zabbix_java_gateway_startup: 1: java: not found
Dec 07 02:30:47 zabbix-server.dingbh.top zabbix_java_gateway_startup[47468]: Zabbix Java Gateway did not start
Dec 07 02:30:47 zabbix-server.dingbh.top systemd[1]: zabbix-java-gateway.service: Control process exited, code=exited, status=1/FAILURE
Dec 07 02:30:47 zabbix-server.dingbh.top systemd[1]: zabbix-java-gateway.service: Failed with result 'exit-code'.
Dec 07 02:30:47 zabbix-server.dingbh.top systemd[1]: Failed to start Zabbix Java Gateway.
dpkg: error processing package zabbix-java-gateway (--configure):
 installed zabbix-java-gateway package post-installation script subprocess returned error exit status 1
Processing triggers for libc-bin (2.31-0ubuntu9.7) ...
Processing triggers for systemd (245.4-4ubuntu3.20) ...
Processing triggers for man-db (2.9.1-1) ...
Processing triggers for ca-certificates (20230311ubuntu0.20.04.1) ...
Updating certificates in /etc/ssl/certs...
0 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...

........


[root@zabbix-server ~]# java -version
openjdk version "11.0.21" 2023-10-17
OpenJDK Runtime Environment (build 11.0.21+9-post-Ubuntu-0ubuntu120.04)
OpenJDK 64-Bit Server VM (build 11.0.21+9-post-Ubuntu-0ubuntu120.04, mixed mode, sharing)
#安装后启动失败，需要手动启动
[root@zabbix-server ~]# systemctl restart zabbix-java-gateway.service 

[root@zabbix-server ~]# ps aux|grep java
zabbix     49961  5.7  1.4 3561412 57488 ?       Sl   02:39   0:00 java -server -Dlogback.configurationFile=/etc/zabbix/zabbix_java_gateway_logback.xml -classpath lib:lib/android-json-4.3_r3.1.jar:lib/logback-classic-1.2.9.jar:lib/logback-core-1.2.9.jar:lib/slf4j-api-1.7.32.jar:bin/zabbix-java-gateway-6.0.24.jar -Dzabbix.pidFile=/var/run/zabbix/zabbix_java_gateway.pid -Dsun.rmi.transport.tcp.responseTimeout=3000 com.zabbix.gateway.JavaGateway
root       49987  0.0  0.0   9384   720 pts/0    R+   02:39   0:00 grep --color=auto java

[root@zabbix-server ~]# ss -ntlp|grep 10052
LISTEN    0         50                       *:10052                  *:*        users:(("java",pid=49961,fd=12))   

#修改zabbix-server.conf配置
[root@zabbix-server ~]#vim /apps/zabbix_server/etc/zabbix_server.conf

JavaGateway=192.168.188.100   #指向JAVA gateway主机

StartJavaPollers=1			  #指定开启的进程数,默认为0，即不开启,所以必须修改此行

JavaGatewayPort=10052         #指定端口,此为默认值,可不修改

#重启zabbix_server服务
[root@zabbix-server ~]# systemctl restart zabbix-server.service 

#查看进程
[root@zabbix-server ~]# ps aux|grep java
zabbix     49961  0.1  1.4 3568608 57496 ?       Sl   02:39   0:01 java -server -Dlogback.configurationFile=/etc/zabbix/zabbix_java_gateway_logback.xml -classpath lib:lib/android-json-4.3_r3.1.jar:lib/logback-classic-1.2.9.jar:lib/logback-core-1.2.9.jar:lib/slf4j-api-1.7.32.jar:bin/zabbix-java-gateway-6.0.24.jar -Dzabbix.pidFile=/var/run/zabbix/zabbix_java_gateway.pid -Dsun.rmi.transport.tcp.responseTimeout=3000 com.zabbix.gateway.JavaGateway
zabbix     50745  0.0  0.1 140916  6904 ?        S    02:54   0:00 /usr/sbin/zabbix_server: java poller #1 [got 0 values in 0.000019 sec, idle 5 sec]     #StartJavaPollers=1	值为1，一个java poller进程
root       50797  0.0  0.0   9516   720 pts/0    S+   02:54   0:00 grep --color=auto java

[root@zabbix-server ~]# vim /etc/zabbix/zabbix_server.conf # 将StartJavaPollers的值修改为2 ，两个java poller进程
[root@zabbix-server ~]# ps aux|grep java
zabbix     49961  0.1  1.4 3568608 57496 ?       Sl   02:39   0:01 java -server -Dlogback.configurationFile=/etc/zabbix/zabbix_java_gateway_logback.xml -classpath lib:lib/android-json-4.3_r3.1.jar:lib/logback-classic-1.2.9.jar:lib/logback-core-1.2.9.jar:lib/slf4j-api-1.7.32.jar:bin/zabbix-java-gateway-6.0.24.jar -Dzabbix.pidFile=/var/run/zabbix/zabbix_java_gateway.pid -Dsun.rmi.transport.tcp.responseTimeout=3000 com.zabbix.gateway.JavaGateway
zabbix     50871  0.0  0.1 140916  6920 ?        S    02:54   0:00 /usr/sbin/zabbix_server: java poller #1 [got 0 values in 0.000036 sec, idle 5 sec]
zabbix     50872  0.0  0.1 140916  6920 ?        S    02:54   0:00 /usr/sbin/zabbix_server: java poller #2 [got 0 values in 0.000015 sec, idle 5 sec]   #StartJavaPollers=2	值为2，两个java poller进程
root       50918  0.0  0.0   9516   656 pts/0    S+   02:54   0:00 grep --color=auto java

```

#### 9.3.1.2、**二进制安装** **Java gateway**

编译安装使用的是zabbix agent，zabbix server 是同一个源码包,只需指定相关的编译选项即可

范例： 编译安装 zabbix java gateway 5.0

```
[root@centos8 ~]#tar xf zabbix-5.0.13.tar.gz
[root@centos8 ~]#wget https://cdn.zabbix.com/zabbix/sources/stable/5.0/zabbix-
5.0.13.tar.gz
[root@centos8 ~]#cd zabbix-5.0.13/
[root@centos8 zabbix-5.0.13]#./configure --prefix=/apps/zabbix_java_gateway --
enable-java --with-net-snmp --with-mysql -with-ssh2
[root@centos8 zabbix-5.0.13]#make && make install
[root@centos8 zabbix-
5.0.13]#/apps/zabbix_java_gateway/sbin/zabbix_java/startup.sh
[root@centos8 ~]#ss -ntl
State     Recv-Q   Send-Q Local Address:Port     Peer Address:Port            
LISTEN     0        100         127.0.0.1:25            0.0.0.0:*              

LISTEN     0        128           0.0.0.0:22            0.0.0.0:*               
LISTEN     0        100           [::1]:25             [::]:*                
LISTEN     0        50                 *:10052             *:*                
LISTEN     0        128             [::]:22               [::]:*               
[root@centos8 ~]#ps aux|grep java
root        9253  2.1  2.9 2570612 54684 pts/1   Sl   09:29   0:00 java -server
-classpath lib:lib/android-json-4.3_r3.1.jar:lib/logback-classic-
1.2.3.jar:lib/logback-core-1.2.3.jar:lib/slf4j-api-1.7.30.jar:bin/zabbix-javagateway-5.0.13.jar -Dzabbix.pidFile=/tmp/zabbix_java.pid -
Dsun.rmi.transport.tcp.responseTimeout=3000 com.zabbix.gateway.JavaGateway
root        9270  0.0  0.0  12112   988 pts/0   S+   09:29   0:00 grep --
color=auto java
范例： 编译安装 zabbix java gateway 4.0
[root@centos8 ~]#yum -y install gcc java-1.8.0-openjdk-devel
[root@centos8 ~]#wget https://cdn.zabbix.com/zabbix/sources/stable/4.0/zabbix-
4.0.24.tar.gz
[root@centos8 ~]#tar xf zabbix-4.0.24.tar.gz
[root@centos8 ~]#cd zabbix-4.0.24/
[root@centos8 zabbix-4.0.24]# ./configure --prefix=/usr/local/zabbix --enablejava --with-net-snmp --with-mysql -with-ssh2
[root@centos8 zabbix-4.0.24]#make && make install
[root@centos8 ~]#tree /usr/local/zabbix/
/usr/local/zabbix/
└── sbin
   └── zabbix_java
       ├── bin
       │   └── zabbix-java-gateway-4.0.24.jar
       ├── lib
       │   ├── android-json-4.3_r3.1.jar
       │   ├── logback-classic-0.9.27.jar
       │   ├── logback-console.xml
       │   ├── logback-core-0.9.27.jar
       │   ├── logback.xml
       │   └── slf4j-api-1.6.1.jar
       ├── settings.sh
       ├── shutdown.sh
       └── startup.sh
4 directories, 10 files
[root@centos8 ~]#/usr/local/zabbix/sbin/zabbix_java/startup.sh 
[root@centos8 ~]#ss -ntl
State   Recv-Q Send-Q     Local Address:Port     Peer Address:Port  
LISTEN   0      128              0.0.0.0:22             0.0.0.0:*               
  
LISTEN   0      100            127.0.0.1:25             0.0.0.0:*
LISTEN   0      128                 [::]:22               [::]:*               
  
LISTEN   0      100               [::1]:25               [::]:*               
  
LISTEN   0      50                     *:10052               *:*               
   
[root@centos8 ~]#ps -ef|grep java
root       15568       1  3 10:58 pts/0    00:00:00 java -server -classpath
lib:lib/android-json-4.3_r3.1.jar:lib/logback-classic-0.9.27.jar:lib/logbackcore-0.9.27.jar:lib/slf4j-api-1.6.1.jar:bin/zabbix-java-gateway-4.0.24.jar -
Dzabbix.pidFile=/tmp/zabbix_java.pid -
Dsun.rmi.transport.tcp.responseTimeout=3000 com.zabbix.gateway.JavaGateway
root       15587    1165  0 10:58 pts/0    00:00:00 grep --color=auto java
```



### 9.4.1、**配置** **Zabbix Server** **支持** **Java gateway**

#### 9.4.1.1、**Zabbix Server** **开启** **Java gateway** **功能**

默认有关java的监控项是不支持的,如下图

![不支持](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231207113300980.png)

配置下面后,上面相关监控项会支持

```
#包安装zabbix server
[root@zabbix-server ~]#vim /etc/zabbix/zabbix_server.conf 
#编译安装zabbix server
[root@zabbix-server ~]#vim /apps/zabbix_server/etc/zabbix_server.conf
JavaGateway=10.0.0.105   #指向JAVA gateway主机
StartJavaPollers=10 #指定开启的进程数,默认为0，即不开启,所以必须修改此行
JavaGatewayPort=10052    #指定端口,此为默认值,可不修改
[root@zabbix-server ~]#systemctl restart zabbix-server.service
#查看进程
[root@zabbix-server ~]# ps -ef|grep java
zabbix     50871   50839  0 02:54 ?        00:00:00 /usr/sbin/zabbix_server: java poller #1 [got 0 values in 0.000065 sec, idle 5 sec]
zabbix     50872   50839  0 02:54 ?        00:00:00 /usr/sbin/zabbix_server: java poller #2 [got 0 values in 0.000029 sec, idle 5 sec]
zabbix     52821       1  0 03:25 ?        00:00:03 java -server -Dlogback.configurationFile=/etc/zabbix/zabbix_java_gateway_logback.xml -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=9999 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false -classpath lib:lib/android-json-4.3_r3.1.jar:lib/logback-classic-1.2.9.jar:lib/logback-core-1.2.9.jar:lib/slf4j-api-1.7.32.jar:bin/zabbix-java-gateway-6.0.24.jar -Dzabbix.pidFile=/var/run/zabbix/zabbix_java_gateway.pid -Dsun.rmi.transport.tcp.responseTimeout=3000 com.zabbix.gateway.JavaGateway
root       53383   45293  0 03:34 pts/0    00:00:00 grep --color=auto java

```

####  9.4.1.2、**使用系统内置模板监控** **tomcat** **主机**（Java gateway 开启JMX功能）

**如果 Java gateway 开启JMX功能，也可以直接监控 java gateway 自身**

```
#zabbix-server开启JMX功能

[root@zabbix-server ~]# vim /lib/systemd/system/zabbix-java-gateway.service   #查看启动文件或脚本路径

[Unit]
Description=Zabbix Java Gateway
After=syslog.target
After=network.target

[Service]
Type=forking
KillMode=process
PIDFile=/run/zabbix/zabbix_java_gateway.pid
ExecStart=/usr/sbin/zabbix_java_gateway_startup
ExecStop=/usr/sbin/zabbix_java_gateway_shutdown
SuccessExitStatus=143
User=zabbix
Group=zabbix

[Install]
WantedBy=multi-user.target

#修改启动脚本开启JMX功能
[root@zabbix-server ~]# vim /usr/sbin/zabbix_java_gateway_startup 
.......

JAVA_OPTIONS="$JAVA_OPTIONS -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=12345 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false"

.......

**********************************************************************************************************
参数解释说明：
com.sun.management.jmxremote ：启用 JMX。
com.sun.management.jmxremote.port=12345 ：指定 JMX 连接的端口号（在此示例中是 12345）。
com.sun.management.jmxremote.ssl=false ：指定是否启用 SSL。
com.sun.management.jmxremote.authenticate=false ：指定是否启用身份验证。
**********************************************************************************************************

#使用cmdline-jmxclient-0.10.3.jar工具测试
[root@zabbix-server ~]# java -jar cmdline-jmxclient-0.10.3.jar - 192.168.188.100:12345
java.lang:name=Metaspace,type=MemoryPool
java.lang:name=CodeHeap 'profiled nmethods',type=MemoryPool
java.lang:type=Runtime
java.nio:name=mapped,type=BufferPool
java.lang:type=OperatingSystem
jdk.management.jfr:type=FlightRecorder
java.lang:name=Compressed Class Space,type=MemoryPool
java.lang:name=CodeHeap 'non-nmethods',type=MemoryPool
java.lang:name=G1 Eden Space,type=MemoryPool
java.util.logging:type=Logging
java.lang:type=Compilation
java.lang:name=CodeHeap 'non-profiled nmethods',type=MemoryPool
java.lang:name=Metaspace Manager,type=MemoryManager
JMImplementation:type=MBeanServerDelegate
java.lang:type=ClassLoading
java.lang:name=CodeCacheManager,type=MemoryManager
java.lang:type=Threading
java.nio:name=direct,type=BufferPool
java.lang:name=G1 Old Gen,type=MemoryPool
java.lang:name=G1 Survivor Space,type=MemoryPool
java.lang:name=G1 Young Generation,type=GarbageCollector
java.lang:type=Memory
com.sun.management:type=HotSpotDiagnostic
java.lang:name=G1 Old Generation,type=GarbageCollector
com.sun.management:type=DiagnosticCommand

```

#### 9.4.1.3、添加需要监控的java服务器，指定通过JMX接口实现

![image-20231207131803185](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231207131803185.png)

**选中系统内置的模板**:

**Template App Generic Java JMX**

**Template App Apache Tomcat JMX**

**注意: Template App Apache Tomcat 只支持老版本的tomcat,所以此处不选择它**

![image-20231207131608253](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231207131608253.png)

**注意: 主机的 JMX 标识正常情况下会变绿**

![image-20231207131932863](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231207131932863.png)

查看到 JMX 数据

![image-20231207132031544](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231207132031544.png)

# 10、**监控网络设备** SNMP

##  10.1、SNMP介绍和组成

### 10.1.1、SNMP 介绍

![image-20231207133403898](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231207133403898.png)

```
SNMP 即 Simple Network Management Protocol 简单网络管理协议，属于TCP/IP五层协议中的应用层协议，用于网络管理的协议，SNMP主要用于网络设备的管理。
SNMP的基本思想:为不同种类的设备、不同厂家生产的设备、不同型号的设备，定义为一个统一的接口和协议，使得管理员可以是使用统一的外观面对这些需要管理的网络设备进行管理。通过网络，管理员可以管理位于不同物理空间的设备，从而大大提高网络管理的效率，简化网络管理员的工作。
通常一线的网络硬件厂商的设备都支持SNMP协议,如:华为,华三,思科,锐捷等,另外Linux,Unix,Windows,MacOS等也都支持SNMP协议在1988年被制定，并被Internet体系结构委员会(IAB，Internet Architecture Board）采纳作为一个短期的网络管理解决方案，由于SNMP的简单性，在Internet时代得到了蓬勃的发展，1992年发布了SNMPv2版本，以增强SNMPv1的安全性和功能，SNMP的协议版本目前有SNMP v1、SNMPV 2c 和 SNMP v3三种版本，其具体差别如下:
SNMP v1采用团体名(Community Name)认证，团体名用来定义SNMP NMS和SNMP Agent的关系，如果SNMP报文携带的团体名没有得到设备的认可，该报文将被丢弃，团体名起到了类似于密码的作用，用来限制SNMP NMS对SNMP Agent的访问SNMP v2c也采用团体名认证，它在兼容SNMP v1的同时又扩充了SNMP v1的功能，它提供了更多的操作类型（批量获取GetBulk和通知请求InformRequest)、支持更多的数据类型(Counter64等)、提供了更丰富的错误代码且能够更细致地区分错误。此版本使用较多.
SNMP v3提供了基于用户的安全模型(USM,User-Based Security Model)的认证机制，用户可以设置认证和加密功能，认证用于验证报文发送方的合法性，避免非法用户的访问，加密则是对NMS和Agent之间的传输报文进行加密，以免被窃听。通过有无认证和有无加密等功能组合，可以为SNMP NMS和SNMP Agent之间的通信提供更高的安全性。此版本安全性高,但效率不高.
```

####  10.1.1.1、**SNMP** **工作原理**

```
http://www.oid-info.com/
```

```
SNMP网络设备分为NMS和Agent两种:
	NMS(Network Management Station，网络管理站)是SNMP网络的管理者,NMS是运行SNMP客户端管理程序的工作站，能够提供友好的人机	交互界面，方便网络管理员完成绝大多数的网络管理工作。
	
	Agent是SNMP网络的被管理者。Agent是驻留在设备上的一个进程，负责接收、处理来自NMS的请求报文。在一些紧急情况下，如接口状态发	生改变等，Agent也会主动通知NMS。
	
	NMS和Agent之间通过SNMP协议来交互管理信息。
```

SNMP管理进程与代理进程之前为了交互信息，定义了5种报文

```
get-request操作:这个操作是由管理进程发起,向代理进程处请求提取一个或多个参数值。
get-next-request操作:这个操作是由管理进程发起,向从代理进程处提取一个或多个参数的下一个参数值。
get-response操作:这个操作是由代理进程发出的,向管理进程返回的一个或多个参数值。
set-request操作:这个操作是由管理进程发起,对代理进程设置一个或多个参数值。
trap操作:这个操作是由代理进程主动发出的报文，通知管理进程有某些事情发生。
```

![image-20231208103130849](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231208103130849.png)

### 10.1.2、SNMP组成

基于TCP/IP的SNMP网络管理包含以下组成部分

```
管理信息库MIB（Management Information Base）: 管理信息库包含所有代理进程的所有可被查询和修改的参数。RFC 1213 [McCloghrie and Rose 1991]定义了第二版的MIB，叫做MIB-II。OID（object identifiers）: 是与对象相关联的用来无歧义地标识对象的全局唯一的值,一个OID是一个唯一的键值对，用于标识具体某一个设备的某个具体信息(对象标识)，如端口信息、设备名称等。即一个OID就是网络通信中对象的身份证。

管理信息结构SMI（Structure of Management Information）: 关于MIB的一套公用的结构和表示符号。这个在RFC 1155 [Rose and McCloghrie 1990]中定义。SMI 是一种语言，是为了确保网络管理数据的语法和语义明确和无二义性而定义的语言。它是定义被管理网络实体中特定数据的语言。它定义了数据类型、对象模型，以及写入和修改管理信息的规则。比如: 支持INTEGER,OCTERSTRING,DisplayString,IpAddress等数据类型

SNMP（Simple Network Management Protocol）: 管理进程和代理进程之间的通信协议，叫做简单网络管理协议。在RFC 1157 [Case et al.1990]中定义。SNMP包括数据报交换的格式等。尽管可以在运输层采用各种各样的协议，但是在SNMP中，用得最多的协议还是UDP。
```

上述三部分相互独立，每部分都定义了单独标准（RFC）。SNMP定义通信的方式和格式，但不指明具体设备上的具体数据，每种设备的数据细节在MIB中定义，这样做达到了“控制与数据相分离”的目的，能提供很好的兼容性和可扩展性。而SMI又为保持MIB的简单性和可扩展性提供了很好的支持。

#### 10.1.2.1、 **MIB** **和** **OID**

管理信息库MIB可以分为公有MIB和私有MIB两种

```
公有MIB：一般由RFC定义，主要用来对各种公有协议进行结构化设计和接口标准化处理。例如：OSPF-MIB（RFC1850）/BGP4-MIB（RFC1657）都是典型的公有MIB。大多数的设备制造商都需要按照RFC的定义来提供SNMP接口。

私有MIB：是公有MIB的必要补充，当公司自行开发私有协议或者特有功能时，可以利用私有MIB来完善SNMP接口的管理功能，同时对第三方网管软件管理存在私有协议或特有功能的设备提供支持。

MIB中存放了对每个对象都指定唯 一的对象标识OID, OID是一种数据类型，它指明一种“授权”命名的对象。“授权”的意思就是这些标识不是随便分配的，它是由一些权威机构进行管理和分配的。对象标识是一个整数序列，以点（“.”）分隔。这些整数构成一个树型结构，类似于DNS或Unix的文件系统。对象标识从树的顶部开始，顶部没有标识，以root表示

树上的每个结点同时还有一个文字名。例如标识 .1.3.6.1.2.1和iso.org.dod.internet.memt.mib对应。这主要是为了人们阅读方便。在实际应用中，也就是说在管理进程和代理进程进行数据报交互时，MIB变量名是以对象标识来标识的，都是以1.3.6.1.2.1开头的。

一般情况下（如交换机、防火墙、PC）使用的路径如图中所示，iso-->org-->dod-->internet，所以这些
设备信息的oid必然以1.3.6.1开头，之后mgmt为公有库，包含了一些设备共同具备的信息，例如1.3.6.1.2.1.1.5为主机名，而private为私有库，再根据不同的公司，MIB树继续扩展下去，最后到每一个具体值的叶子节点，例如1.3.6.1.4.1.2021.4.3.0为Linux操作系统的交换区大小
```

下图显示了在SNMP中用到的这种树型结构。所有的MIB变量都从 .1.3.6.1.2.1这个标识开始。

![image-20231208103733898](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231208103733898.png)

#### 10.1.2.2、 **常见** **OID**

oid-info是一个内容丰富的网站，包含了大量公有或私有的MIB信息，在知道部分OID的情况，可以方便的进行节点具体信息的查看。但是树太过庞大，仅仅从网站上查看效率十分低下，直接从设备商处获取mib库是最方便的做法。

参考链接:

```
http://www.oid-info.com/index.htm

#注意:查询时无需输入最前面的.
```

系统信息:

```
sysDescr .1.3.6.1.2.1.1.1
sysobjectID .1.3.6.1.2.1.1.2
sysUpTime .1.3.6.1.2.1.1.3
syscontact .1.3.6.1.2.1.1.4
sysName .1.3.6.1.2.1.1.5
```

CPU负载:

```
l minute Load: .1.3.6.1.4.1.2021.10.1.3.1
5 minute Load: .1.3.6.1.4.1.2021.10.1.3.2
15minute Load: .1.3.6.1.4.1.2021.10.1.3.3
```

CPU信息:

```
percentage of user Cpu time: .1.3.6.1.4.1.2021.11.9.0
raw user cpu time: .1.3.6.1.4.1.2021.11.50.0
percentages of system Cpu time: .1.3.6.1.4.1.2021.11.10.0
raw system cpu time: .1.3.6.1.4.1.2021.11.52.0
percentages of idle Cpu time:.1.3.6.1.4.1.2021.11.11.0
raw idle cpu time:.1.3.6.1.4.1.2021.11.53.0
raw nice cpu time: .1.3.6.1.4.1.2021.11.51.0
```

内存使用:

```
Total swap size:.1.3.6.1.4.1.2021.4.3.0
Available swap Space: .1.3.6.1.4.1.2021.4.4.0
Total RAM in machine: .1.3.6.1.4.1.2021.4.5.0
Total RAM used: .1.3.6.1.4.1.2021.4.6.0
Total RAMFree: .1.3.6.1.4.1.2021.4.11.0
Total RAM Shared: .1.3.6.1.4.1.2021.4.13.0
Tota7 RAM Buffered: .1.3.6.1.4.1.2021.4.14.0
Total cached Memory: .1.3.6.1.4.1.2021.4.15.0
```

硬盘使用：

```
Path where the disk is mounted: .1.3.6.1.4.1.2021.9.1.2.1
Path of the device for the partition: .1.3.6.1.4.1.2021.9.1.3.1
Total size of the disk/partion (kBytes): .1.3.6.1.4.1.2021.9.1.6.1
Available space on the disk: .1.3.6.1.4.1.2021.9.1.7.1
Used space on the disk: .1.3.6.1.4.1.2021.9.1.8.1
Percentage of space used on disk: .1.3.6.1.4.1.2021.9.1.9.1
Percentage of inodes used on disk: .1.3.6.1.4.1.2021.9.1.10.1
```

## 10.2、**使用** **SNMP**工具监控网络设备

### 10.2.1、开启设备的SNMP

登录网络设备，找到如下图中的设备管理--SNMP类似的设置界面，开启SNMP功能，然后设置团体名称即完成

![image-20231208104209016](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231208104209016.png)

设置团体名（验证功能，类似于密码）,默认可以使用 Public 

![image-20231208104224596](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231208104224596.png)

![image-20231208104230827](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231208104230827.png)

### 10.2.2、配置 Zabbix Web 监控

#### 10.2.2.1、配置-->主机-->添加主机-->选择类型为SNMP，输入路由器设备的IP地址

![image-20231208104624467](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231208104624467.png)

![image-20231208104542384](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231208104542384.png)

![image-20231208104632486](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231208104632486.png)

关联以下三个模板

```
Template Module EtherLike-MIB SNMP
Template Module Generic SNMP
Template Module Interfaces SNMP
```

#### 10.2.2.2、**查看网络设备的监测结果**

监测-->主机-->图形

![image-20231208104735144](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231208104735144.png)

![image-20231208104739408](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231208104739408.png)

## 10.3、**利用** **Zabbix** **监控** **Linux** 主机

### 10.3.1、 在 Zabbix Agent 安装 SNMP 协议并配置

#### 10.3.1.1、范例： Ubuntu 系统实现 SNMP

```
[root@Ubuntu1804 ~]# apt install snmpd		#安装下载snmp
[root@Rocky8 ~]#yum -y install net-snmp     #Rocky安装
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Suggested packages:
  snmptrapd
The following NEW packages will be installed:
  snmpd
0 upgraded, 1 newly installed, 0 to remove and 241 not upgraded.
Need to get 57.1 kB of archives.
After this operation, 144 kB of additional disk space will be used.
Get:1 https://mirrors.aliyun.com/ubuntu bionic-security/main amd64 snmpd amd64 5.7.3+dfsg-1.8ubuntu3.8 [57.1 kB]
Fetched 57.1 kB in 1s (63.6 kB/s)                      
Preconfiguring packages ...
Selecting previously unselected package snmpd.
(Reading database ... 68466 files and directories currently installed.)
Preparing to unpack .../snmpd_5.7.3+dfsg-1.8ubuntu3.8_amd64.deb ...
Unpacking snmpd (5.7.3+dfsg-1.8ubuntu3.8) ...
Setting up snmpd (5.7.3+dfsg-1.8ubuntu3.8) ...
adduser: Warning: The home directory `/var/lib/snmp' does not belong to the user you are currently creating.
Created symlink /etc/systemd/system/multi-user.target.wants/snmpd.service → /lib/systemd/system/snmpd.service.
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
Processing triggers for ureadahead (0.100.0-21) ...
Processing triggers for systemd (237-3ubuntu10.42) ...
[root@Ubuntu1804 ~]# snmpd -v    #查看版本号

NET-SNMP version:  5.7.3
Web:               http://www.net-snmp.org/
Email:             net-snmp-coders@lists.sourceforge.net

[root@Ubuntu1804 ~]# vim /etc/snmp/snmpd.conf 


                                                 #  system + hrSystem groups only
view   systemonly  included   .1.3.6.1.2.1.1
view   systemonly  included   .1.3.6.1.2.1.1
view   systemonly  included   .1
#view   systemonly  included   .1.3.6.1.2.1.25.1

agentAddress  udp:127.0.0.1:161


[root@Ubuntu1804 ~]# systemctl restart snmpd.service 
[root@Ubuntu1804 ~]# systemctl status snmpd.service 
● snmpd.service - Simple Network Management Protocol (SNMP) Daemon.
   Loaded: loaded (/lib/systemd/system/snmpd.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2023-12-08 10:57:52 CST; 9s ago
  Process: 79134 ExecStartPre=/bin/mkdir -p /var/run/agentx (code=exited, status=0/SUCCESS)
 Main PID: 79141 (snmpd)
    Tasks: 1 (limit: 4631)
   CGroup: /system.slice/snmpd.service
           └─79141 /usr/sbin/snmpd -Lsd -Lf /dev/null -u Debian-snmp -g Debian-snmp -I -smux mteTrigger mteTriggerConf -f

Dec 08 10:57:52 Ubuntu1804 systemd[1]: Starting Simple Network Management Protocol (SNMP) Daemon....
Dec 08 10:57:52 Ubuntu1804 systemd[1]: Started Simple Network Management Protocol (SNMP) Daemon..
Dec 08 10:57:52 Ubuntu1804 snmpd[79141]: /etc/snmp/snmpd.conf: line 147: Warning: Unknown token: defaultMonitors.
Dec 08 10:57:52 Ubuntu1804 snmpd[79141]: /etc/snmp/snmpd.conf: line 149: Warning: Unknown token: linkUpDownNotifications.
Dec 08 10:57:52 Ubuntu1804 snmpd[79141]: Turning on AgentX master support.
Dec 08 10:57:52 Ubuntu1804 snmpd[79141]: NET-SNMP version 5.7.3

[root@Ubuntu1804 ~]# ss -ntlu
Netid            State              Recv-Q             Send-Q                                 Local Address:Port                            Peer Address:Port             
udp              UNCONN             0                  0                                      127.0.0.53%lo:53                                   0.0.0.0:*                
udp              UNCONN             0                  0                                            0.0.0.0:34930                                0.0.0.0:*                
udp              UNCONN             0                  0                                            0.0.0.0:161                                  0.0.0.0:*             #161端口已打开被监听  
tcp              LISTEN             0                  128                                        127.0.0.1:6010                                 0.0.0.0:*                
tcp              LISTEN             0                  128                                        127.0.0.1:6011                                 0.0.0.0:*                
tcp              LISTEN             0                  128                                          0.0.0.0:10050                                0.0.0.0:*                
tcp              LISTEN             0                  80                                           0.0.0.0:3306                                 0.0.0.0:*                
tcp              LISTEN             0                  128                                          0.0.0.0:11211                                0.0.0.0:*                
tcp              LISTEN             0                  128                                          0.0.0.0:80                                   0.0.0.0:*                
tcp              LISTEN             0                  128                                    127.0.0.53%lo:53                                   0.0.0.0:*                
tcp              LISTEN             0                  128                                          0.0.0.0:22                                   0.0.0.0:*                
tcp              LISTEN             0                  50                                                 *:12345                                      *:*                
tcp              LISTEN             0                  128                                            [::1]:6010                                    [::]:*                
tcp              LISTEN             0                  128                                            [::1]:6011                                    [::]:*                
tcp              LISTEN             0                  128                                             [::]:10050                                   [::]:*                
tcp              LISTEN             0                  1                                 [::ffff:127.0.0.1]:8005                                       *:*                
tcp              LISTEN             0                  50                                                 *:39019                                      *:*                
tcp              LISTEN             0                  100                                                *:8080                                       *:*                
tcp              LISTEN             0                  128                                             [::]:80                                      [::]:*                
tcp              LISTEN             0                  50                                                 *:43793                                      *:*                
tcp              LISTEN             0                  128                                             [::]:22                                      [::]:*   

```

####  10.3.1.2、**测试** **SNMP** **是否可以访问**

```
在 Zabbix Server (也可以在其它主机上)安装 SNMP工具 net-snmp-utils
此工具可以测试是否可以获取Zabbix Agent的SNMP数据

注意: 在实际生产环境中Zabbix Server 在进行 SNMP 监测时,无需在 Zabbix Server 安装此工具snmpwalk 命令格式


USAGE:snmpwalk[OPTIONS]AGENT[OID]
-h:显示帮助。
-v:指定snmp的版本，1或者2c或者3。
-c:指定连接设备SNMP密码。
—r:指定重次次数.默认为0次
-l:指定安全级别: noAuthNoPriv|authNoPriv|authPriv。
-a:验证协议:MD5|SHA。只有-l指定为authNoPriv或authPriv时才需要。
-A∶验证字符串。只有-1指定为authNoPriv或authPriv时才需要
-x:加密协议:DES。只有-l指定为authPriv时才需要
-x:加密字符串。只有-l指定为authPriv时才需要
```

范例: snmpwalk 获取 SNMP代理端的信息

```
[root@Rocky8 ~]#snmpwalk -v 2c -cpublic 127.0.0.1  .1.3.6.1.2.1.1.5.0
SNMPv2-MIB::sysName.0 = STRING: Rocky8
[root@Rocky8 ~]#snmpwalk -v 2c -cpublic 127.0.0.1   .1.3.6.1.4.1.2021.4.5.0
UCD-SNMP-MIB::memTotalReal.0 = INTEGER: 1828180 kB
```

#### 10.3.1.3、zabbix添加主机

![image-20231208160128009](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231208160128009.png)



![image-20231208160249883](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231208160249883.png)

#### 10.3.1.4、图像展示

![image-20231208165629650](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231208165629650.png)

![image-20231208165641335](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231208165641335.png)

# 11、Zabbix分布式实现Proxy

## 11.1、Zabbix Proxy 工作原理

### 11.1.1、Zabbix Proxy 概述

```
zabbix作为一个分布式监控系统(分布式监控解决方案)，支持通过代理(proxy)收集zabbix agent的监控数据,然后由zabbix proxy再把数据发送给zabbix server，也就是zabbix proxy 可以代替zabbix server收集监控数据，然后把数据汇报给zabbix server，所以zabbix proxy可以在一定程度上分担了zabbix server的数据收集压力，从而降低了数据的采集时间、也相应的增加了zabbix server的监控能力。

官方文档:
https://www.zabbix.com/documentation/6.0/zh/manual/distributed_monitoring
https://www.zabbix.com/documentation/5.0/zh/manual/distributed_monitoring
https://www.zabbix.com/documentation/4.0/zh/manual/distributed_monitoring
```

### 11.1.2、**Zabbix Proxy** **使用场景**

```
1、监控远程区域设备,尤其是多机房和多个云环境时
2、监控网络不稳定区域,避免跨网段监控的告警不及时
3、当需要监控设备众多时,使用它来减轻 Zabbix Server 的压力
4、简化分布式监控的维护,无需各位创建 Zabbix Server,统一管理策略
```

### 11.1.3、**Zabbix Proxy** **架构**

![image-20231208183051004](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231208183051004.png)

### 11.1.4、**Zabbix Proxy** **功能列表**

官方文档: 

```
https://www.zabbix.com/documentation/6.0/zh/manual/distributed_monitoring/proxies
https://www.zabbix.com/documentation/5.0/zh/manual/distributed_monitoring/proxies
https://www.zabbix.com/documentation/4.0/zh/manual/distributed_monitoring/proxies
```

zabbix proxy 是一个数据收集器,它不计算触发器、不处理事件、不发送报警。有关proxy功能的概述，

如下表:

![image-20231208183204273](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231208183204273.png)

### 11.1.5、Zabbix Server **和** **Zabbix Proxy** **区别**

Zabbix Proxy 只执行数据收集，不运行触发器，无处理事件，也不发送告警，无Web管理界面所以最终导致告警延时可能会比较长

![image-20231208183335736](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231208183335736.png)

### 11.1.6、**Zabbix Proxy** **的主动模式和被动模式**

```
1、Zabbix Proxy 也分主动模式和被动模式，通信方式与zabbix server主动模式和被动模式一样

2、Zabbix Proxy的模式 是从Zabbix Proxy 角度来说的

3、zabbix proxy在主动模式下要主动地向zabbix server周期性的申请获取zabbix agent的监控项信息,此模式可以大幅降低Zabbix Server的压力,生产推荐使用

4、Zabbix Proxy在被动模式下要被动地等待zabbix server的连接,并接受zabbix server发送的监控项指令，然后再由zabbix proxy向zabbix agent发起请求获取数据。

注意: Zabbix Proxy 的主动模式和被动模式和Agent的主动被动模式没有关系
```

## 11.2、Zabbix Proxy安装和配置

**注意: Zabbix Proxy的大版本必须要和zabbix server版本相同，否则很可能会导致出现 zabbix server与zabbix proxy不兼容问题**

**安装环境准备**：

**注意：所有主机的主机名要唯一，否则被动模式可能会影响数据采集**

![jj](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231208183605394.png)

```
主机准备：
192.168.188.100：zabbix-server
192.168.188.101：zabbix-proxy、zabbix-agent
192.168.188.102：zabbix-proxy(被动)、zabbix-agent
192.168.188.8：zabbix-agent1
192.168.188.81：zabbix-agent1（被动）
192.168.188.7：dns、zabbix-agent


官方安装文档
https://www.zabbix.com/documentation/5.0/zh/manual/installation/install_from_packages/debian_ubuntu
https://www.zabbix.com/documentation/5.0/zh/manual/installation/install_from_packages/rhel_centos
```

### 11.2.1、**安装** **Zabbix Proxy**

#### 11.2.1.1、二进制包安装Zabbix Proxy 6.0

```
https://www.zabbix.com/cn/download?zabbix=6.0&os_distribution=ubuntu&os_version=18.04&components=proxy&db=mysql&ws=
```

![image-20231208184034822](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231208184034822.png)

范例：Ubuntu 安装 Zabbix Proxy 6.0

注：提前安装数据库

```
[root@Ubuntu1804 ~]# wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-4+ubuntu18.04_all.deb
[root@Ubuntu1804 ~]# dpkg -i zabbix-release_6.0-4+ubuntu18.04_all.deb
[root@Ubuntu1804 ~]# apt update

[root@Ubuntu1804 ~]# apt install zabbix-proxy-mysql zabbix-sql-scripts


mysql> create database zabbix_proxy character set utf8mb4 collate utf8mb4_bin;

mysql> create user zabbix@"192.168.188.%" identified by '123456';

mysql> grant all on zabbix_proxy.* to zabbix@"192.168.188.%";

mysql> set global log_bin_trust_function_creators = 1;

mysql> quit

[root@Ubuntu1804 ~]# cat /usr/share/zabbix-sql-scripts/mysql/proxy.sql | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix_proxy -h192.168.188.200

[root@Ubuntu1804 ~]# mysql -uroot -p

mysql> set global log_bin_trust_function_creators = 0;
mysql> quit;


[root@zabbix-proxy ~]# vim /etc/zabbix/zabbix_proxy.conf

DBPassword=123456

[root@zabbix-proxy ~]# systemctl enable --now zabbix-proxy.service 

[root@zabbix-proxy ~]# systemctl restart zabbix-proxy.service 
```



遇到的问题：E: Unable to locate package zabbix-sql-scripts

```
[root@zabbix_proxy2 apt]# apt install zabbix-proxy-mysql zabbix-sql-scripts
Reading package lists... Done
Building dependency tree       
Reading state information... Done
E: Unable to locate package zabbix-sql-scripts
```

解决办法：下载缺失包

```
[root@zabbix_proxy1 ~]# wget https://mirrors.aliyun.com/zabbix/zabbix/6.0/ubuntu/pool/main/z/zabbix/zabbix-sql-scripts_6.0.9-1+ubuntu18.04_all.deb
[root@zabbix_proxy1 ~]# dpkg -i zabbix-sql-scripts_6.0.9-1+ubuntu18.04_all.deb
[root@zabbix_proxy1 ~]# apt update
[root@zabbix_proxy1 ~]# apt --fix-broken install
```

```
mysql>ALTER USER zabbix@localhost IDENTIFIED WITH mysql_native_password BY '123456'; #修改密码加密策略
mysql>FLUSH PRIVILEGES;
```



#### 11.2.1.2、修改zabbix_proxy配置文件

配置文件说明

```
[root@zabbix-proxy-active ~]#grep '^[^#]' /etc/zabbix/zabbix_proxy.conf 
ProxyMode=0                         #主动模式为0，被动模式为1,默认为0即主动模式
Server=10.0.0.100                   #指向Zabbix Server
Hostname=zabbix-proxy-active        #此名称必须和后面Web管理页的agent代理程序名称相同
DBHost=localhost                    #MySQL服务器地址
DBName=zabbix_proxy_active          #MySQL数据库名
DBUser=proxy                        #连接MySQL的用户
DBPassword=123456                   #连接MySQL的用户密码
LogFile=/var/log/zabbix/zabbix_proxy.log
LogFileSize=0
DebugLevel=4                        #指定日志级别，默认为3，值越大日志越详细
EnableRemoteCommands=1              #开启远程命令，允许server到proxy上执行命令，在故障自愈时使用
PidFile=/var/run/zabbix/zabbix_proxy.pid
SocketDir=/var/run/zabbix
ProxyLocalBuffer=360             #当proxy将数据发送给server后将数据仍在本地保存多少时间,默认不保留
ProxyOfflineBuffer=720           #当proxy和server无法连接时将数据在本地保存多长时间,默认1小时
HeartbeatFrequency=60            #server端用来检测proxy可用性的心跳信息的时间间隔,被动模式无效
ConfigFrequency=60               #每间隔多少时间到server获取监控项，在agent更新端监控项，只在主动模式有效,默认3600s
DataSenderFrequency=60           #数据发送的间隔时间，只在主动模式有效,默认1s,建议加长
JavaGateway=10.0.0.101           #指向JAVA gateway主机
StartJavaPollers=10              #指定开启的进程数,默认为0，即不开启
SNMPTrapperFile=/var/log/snmptrap/snmptrap.log
CacheSize=8M                      #当主机数量很多时，会将获取的监控项存放在缓存中，生产中设置2G
StartDBSyncers=4                  #启动多少个线程和数据库连接
HistoryCacheSize=16M              #保存agent发送过来的监控数据的内存空间大小，生产中设置2G 
HistoryIndexCacheSize=4M          #历史数据的索引
Timeout=30                        #获取数据的最长等待时间
ExternalScripts=/usr/lib/zabbix/externalscripts
LogSlowQueries=3000               #慢查询时长

#默认zabbix-proxy.service不自动启动，设置服务开机自启
[root@zabbix-proxy-active ~]#systemctl enable --now zabbix-proxy.service
```

**修改zabbix_proxy配置文件及实现proxy主机监控**

```
#zabbix_proxy被动模式
[root@zabbix-proxy1 ~]#grep  -Ev '^(#|$)' /etc/zabbix/zabbix_proxy.conf 
ProxyMode=1   	#被动模式
Server=192.168.188.100
Hostname=zabbix-passive
LogFile=/var/log/zabbix/zabbix_proxy.log
LogFileSize=0
PidFile=/run/zabbix/zabbix_proxy.pid
SocketDir=/run/zabbix
DBName=zabbix_proxy
DBUser=zabbix
DBPassword=123456
HeartbeatFrequency=60
SNMPTrapperFile=/var/log/snmptrap/snmptrap.log
Timeout=4
FpingLocation=/usr/bin/fping
Fping6Location=/usr/bin/fping6
LogSlowQueries=3000
StatsAllowedIP=127.0.0.1
[root@zabbix-proxy1 ~]#grep  -Ev '^(#|$)' /etc/zabbix/zabbix_agentd.conf 
PidFile=/run/zabbix/zabbix_agentd.pid
LogFile=/var/log/zabbix/zabbix_agentd.log
LogFileSize=0
Server=127.0.0.1
ServerActive=127.0.0.1
Hostname=Zabbix server
Include=/etc/zabbix/zabbix_agentd.d/*.conf
[root@zabbix-proxy1 ~]#systemctl restart zabbix-agent.service 
[root@zabbix-proxy1 ~]#systemctl restart zabbix-proxy.service 

#zabbix_proxy主动模式
[root@zabbix-proxy2 ~]#grep  -Ev '^(#|$)' /etc/zabbix/zabbix_proxy.conf 
Server=192.168.188.100
Hostname=192.168.188.102
LogFile=/var/log/zabbix/zabbix_proxy.log
LogFileSize=0
PidFile=/run/zabbix/zabbix_proxy.pid
SocketDir=/run/zabbix
DBName=zabbix_proxy
DBUser=zabbix
DBPassword=password
SNMPTrapperFile=/var/log/snmptrap/snmptrap.log
Timeout=4
FpingLocation=/usr/bin/fping
Fping6Location=/usr/bin/fping6
LogSlowQueries=3000
StatsAllowedIP=127.0.0.1
[root@zabbix-proxy2 ~]#grep  -Ev '^(#|$)' /etc/zabbix/zabbix_agentd.conf 
PidFile=/run/zabbix/zabbix_agentd.pid
LogFile=/var/log/zabbix/zabbix_agentd.log
LogFileSize=0
Server=127.0.0.1
ServerActive=127.0.0.1
Hostname=192.168.188.102
Include=/etc/zabbix/zabbix_agentd.d/*.conf
[root@zabbix-proxy2 ~]#systemctl restart zabbix-agent.service 
[root@zabbix-proxy2 ~]#systemctl restart zabbix-proxy.service 
```

#### 11.2.1.3、**在** **Zabbix Server** **上添加代理** **Proxies**

##### 11.2.1.3.1、被动模式 添加代理proxies

​	![image-20231209230546505](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231209230546505.png)



注：**添加被动代理→接口填写zabbix-proxy主机**（agent代理程序名称需与主机配置文件Hostname一致）



![image-20231209230713489](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231209230713489.png)

**创建监控主机**



![image-20231209231002541](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231209231002541.png)



![image-20231209231058039](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231209231058039.png)

##### 11.2.1.3.2、主动模式 添加代理proxies

**注：代理名称和配置文件Hostname命名保持一致**

![image-20231209231133457](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231209231133457.png)

查看添加成功

![image-20231209231142464](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231209231142464.png)

##### **注：创建监控主机→主机名要与配置文件Hostname保持一致**

![image-20231209232945556](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231209232945556.png)

##### 11.2.1.3.3、查看数据

![image-20231209233158861](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231209233158861.png)

被监控主机及java应用展示

![image-20231209234346609](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231209234346609.png)

被监控主机展示

![image-20231209233225508](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231209233225508.png)

被动模式监控数据

![image-20231209233242683](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231209233242683.png)

主动模式监控数据

![image-20231209233259593](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231209233259593.png)

![image-20231209233324417](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231209233324417.png)

被动模式监控图形

![image-20231209233348777](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231209233348777.png)

JAVA进程监控

![image-20231209234406524](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231209234406524.png)

# 12、Zabbix实现自动化运维

## 12.1、 Zabbix网络发现

12.1.1、准备zabbix-agent



## 12.2、Zabbix自动注册



## 12.3、ZabbixAPI实现自动化运维
