# 负载均衡LVS(**Linux Virtual Server**)

# 1、**集群概念**

Cluster：集群,为解决某个特定问题将多台计算机组合起来形成的单个系统

```yaml
Cluster分为三种类型：
LB: Load Balancing，负载均衡，多个主机组成，每个主机只承担一部分访问请求
HA: High Availiablity，高可用，避免SPOF（single Point Of failure）
 MTBF: Mean Time Between Failure 平均无故障时间，正常时间
 MTTR: Mean Time To Restoration（ repair）平均恢复前时间，故障时间
 A = MTBF /（MTBF+MTTR） (0,1)：99%,99.5%,99.9%,99.99%,99.999%
```

**SLA**：服务等级协议（简称：SLA，全称：service level agreement）。是在一定开销下为保障服

务的性能和可用性，服务提供商与用户间定义的一种双方认可的协定。通常这个开销是驱动提供服

务质量的主要因素。在常规的领域中，总是设定所谓的三个9，四个9来进行表示，当没有达到这种

水平的时候，就会有一些列的惩罚措施，而运维，最主要的目标就是达成这种服务水平。

```
1年 = 365天 = 8760小时
90 = (1-90%)*365=36.5天
99 = 8760 * 1% = 87.6小时
99.9 = 8760 * 0.1% = 8760 * 0.001 = 8.76小时
99.99 = 8760 * 0.0001 = 0.876小时 = 0.876 * 60 = 52.6分钟
99.999 = 8760 * 0.00001 = 0.0876小时 = 0.0876 * 60 = 5.26分钟
99.9999= (1-99.9999%)*365*24*60*60=31秒
```

停机时间又分为两种，一种是计划内停机时间，一种是计划外停机时间，而运维则主要关注计划外

停机时间。

HPC：High-performance computing，高性能 www.top500.org



集群与分布式区别

```
集群：同一个业务系统，部署在多台服务器上。集群中，每一台服务器实现的功能没有差别，数据和代码都是一样的

分布式：一个业务被拆成多个子业务，或者本身就是不同的业务，部署在多台服务器上。分布式中，每一台服务器实现的功能是有差别的，数据和代码也是不一样的，分布式每台服务器功能加起来，才是完整的业务
```



# 2、**LVS** **模型**

## 2.1、LVS介绍

```
VS：Linux Virtual Server，负载调度器，内核集成，章文嵩（花名 正明）, 阿里的四层SLB(Server Load Balance)是基于LVS+keepalived实现
LVS 是全球最流行的四层负载均衡开源软件，由章文嵩博士（当前阿里云产品技术负责人）在1998年5月创立，可以实现LINUX平台下的负载均衡。
LVS 官网：http://www.linuxvirtualserver.org/
```

阿里SLB和LVS：

```
https://yq.aliyun.com/articles/1803
https://github.com/alibaba/LVS
```

## **2.2、LVS**工作原理

VS根据请求报文的目标IP和目标协议及端口将其调度转发至某RS，根据调度算法来挑选RS。LVS是内核级功能，工作在INPUT链的位置，将发往INPUT的流量进行“处理”

**范例：查看内核支持LVS**

```
[root@Rocky8 ~]#grep -i -C 10 ipvs /boot/config-4.18.0-425.3.1.el8.x86_64 
CONFIG_NETFILTER_XT_MATCH_DCCP=m
CONFIG_NETFILTER_XT_MATCH_DEVGROUP=m
CONFIG_NETFILTER_XT_MATCH_DSCP=m
CONFIG_NETFILTER_XT_MATCH_ECN=m
CONFIG_NETFILTER_XT_MATCH_ESP=m
CONFIG_NETFILTER_XT_MATCH_HASHLIMIT=m
CONFIG_NETFILTER_XT_MATCH_HELPER=m
CONFIG_NETFILTER_XT_MATCH_HL=m
# CONFIG_NETFILTER_XT_MATCH_IPCOMP is not set
CONFIG_NETFILTER_XT_MATCH_IPRANGE=m
CONFIG_NETFILTER_XT_MATCH_IPVS=m
# CONFIG_NETFILTER_XT_MATCH_L2TP is not set
CONFIG_NETFILTER_XT_MATCH_LENGTH=m
CONFIG_NETFILTER_XT_MATCH_LIMIT=m
CONFIG_NETFILTER_XT_MATCH_MAC=m
CONFIG_NETFILTER_XT_MATCH_MARK=m
CONFIG_NETFILTER_XT_MATCH_MULTIPORT=m
# CONFIG_NETFILTER_XT_MATCH_NFACCT is not set
CONFIG_NETFILTER_XT_MATCH_OSF=m
CONFIG_NETFILTER_XT_MATCH_OWNER=m
CONFIG_NETFILTER_XT_MATCH_POLICY=m
--
CONFIG_IP_SET_HASH_NETNET=m
CONFIG_IP_SET_HASH_NETPORT=m
CONFIG_IP_SET_HASH_NETIFACE=m
CONFIG_IP_SET_LIST_SET=m
CONFIG_IP_VS=m
CONFIG_IP_VS_IPV6=y
# CONFIG_IP_VS_DEBUG is not set
CONFIG_IP_VS_TAB_BITS=12

#
# IPVS transport protocol load balancing support
#
CONFIG_IP_VS_PROTO_TCP=y
CONFIG_IP_VS_PROTO_UDP=y
CONFIG_IP_VS_PROTO_AH_ESP=y
CONFIG_IP_VS_PROTO_ESP=y
CONFIG_IP_VS_PROTO_AH=y
CONFIG_IP_VS_PROTO_SCTP=y

#
# IPVS scheduler
#
CONFIG_IP_VS_RR=m
CONFIG_IP_VS_WRR=m
CONFIG_IP_VS_LC=m
CONFIG_IP_VS_WLC=m
CONFIG_IP_VS_FO=m
CONFIG_IP_VS_OVF=m
CONFIG_IP_VS_LBLC=m
CONFIG_IP_VS_LBLCR=m
CONFIG_IP_VS_DH=m
CONFIG_IP_VS_SH=m
CONFIG_IP_VS_MH=m
CONFIG_IP_VS_SED=m
CONFIG_IP_VS_NQ=m

#
# IPVS SH scheduler
#
CONFIG_IP_VS_SH_TAB_BITS=8

#
# IPVS MH scheduler
#
CONFIG_IP_VS_MH_TAB_INDEX=12

#
# IPVS application helper
#
CONFIG_IP_VS_FTP=m
CONFIG_IP_VS_NFCT=y
CONFIG_IP_VS_PE_SIP=m

#
# IP: Netfilter Configuration
#
CONFIG_NF_DEFRAG_IPV4=m
CONFIG_NF_SOCKET_IPV4=m

```

## **2.3、LVS**集群体系架构

![image-20230924224458773](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20230924224458773.png)

## **2.4、LVS**集群类型中的术语

```
VS：Virtual Server，Director Server(DS), Dispatcher(调度器)，Load Balancer
RS：Real Server(lvs), upstream server(nginx), backend server(haproxy)
CIP：Client IP
VIP：Virtual serve IP VS外网的IP
DIP：Director IP VS内网的IP
RIP：Real server IP 
访问流程：CIP <--> VIP == DIP <--> RIP
```

# 3、LVS 工作模式和相关命令

## 3.1、**3种LVS** 工作模式

```
lvs-nat：修改请求报文的目标IP,多目标IP的DNAT
lvs-dr：操纵封装新的MAC地址，常用
lvs-tun：在原请求IP报文之外新加一个IP首部
lvs-fullnat：修改请求报文的源和目标IP,默认内核不支持
```

### **3.1.1 LVS**的NAT模式

官方链接

```
http://www.linuxvirtualserver.org/VS-NAT.html
```

### **3.1.2、 LVS**的DR模式

### **3.1.3、 LVS**的TUN模式



## **3.2、LVS** **调度算法**

### **3.2.1**、**静态方法**

```
仅根据算法本身进行调度
1、RR：roundrobin，轮询,较常用，雨露均沾，大锅饭
2、WRR：Weighted RR，加权轮询,较常用
3、SH：Source Hashing，实现session sticky，源IP地址hash；将来自于同一个IP地址的请求始终发往第一次挑中的RS，从而实现会话绑定
4、DH：Destination Hashing；目标地址哈希，第一次轮询调度至RS，后续将发往同一个目标地址的请求始终转发至第一次挑中的RS，典型使用场景是正向代理缓存场景中的负载均衡,如: Web缓存
```

### **3.2.2、动态方法**

```
主要根据每RS当前的负载状态及调度算法进行调度Overhead=value 较小的RS将被调度
```

```
1、LC：least connections 适用于长连接应用
Overhead=activeconns*256+inactiveconns

2、WLC：Weighted LC（带权重的最小连接），默认调度方法,较常用（默认）  注：默认工作模式DR
Overhead=(activeconns*256+inactiveconns)/weight

[root@Rocky8 ~]#grep -i WLC /boot/config-4.18.0-425.3.1.el8.x86_64 
CONFIG_IP_VS_WLC=m

3、SED：Shortest Expection Delay，初始连接高权重优先,只检查活动连接,而不考虑非活动连接
Overhead=(activeconns+1)*256/weight

4、NQ：Never Queue，第一轮均匀分配，后续SED

5、LBLC：Locality-Based LC，动态的DH算法，使用场景：根据负载状态实现正向代理,实现Web Cache等

6、LBLCR：LBLC with Replication，带复制功能的LBLC，解决LBLC负载不均衡问题，从负载重的复制到负载轻的RS,,实现Web Cache等
```



## 3.3、安装ipvsadm管理工具

### **3.3.1** **程序包：ipvsadm**

```
Unit File: ipvsadm.service

主程序：/usr/sbin/ipvsadm

规则保存工具：/usr/sbin/ipvsadm-save

规则重载工具：/usr/sbin/ipvsadm-restore

配置文件：/etc/sysconfig/ipvsadm-config

ipvs调度规则文件：/etc/sysconfig/ipvsadm
```

### **3.3.2、ipvsadm** **命令**

```
ipvsadm核心功能：

集群服务管理：增、删、改

集群服务的RS管理：增、删、改

查看
```

```
[root@Rocky8 ~]#rpm -qi ipvsadm
Name        : ipvsadm
Version     : 1.31
Release     : 1.el8
Architecture: x86_64
Install Date: Fri 29 Sep 2023 12:31:35 PM CST
Group       : Unspecified
Size        : 87573
License     : GPLv2+
Signature   : RSA/SHA256, Sun 26 Apr 2020 10:10:12 AM CST, Key ID 05b555b38483c65d
Source RPM  : ipvsadm-1.31-1.el8.src.rpm
Build Date  : Fri 24 Apr 2020 09:50:03 AM CST
Build Host  : x86-01.mbox.centos.org
Relocations : (not relocatable)
Packager    : CentOS Buildsys <bugs@centos.org>
Vendor      : CentOS
URL         : https://kernel.org/pub/linux/utils/kernel/ipvsadm/
Summary     : Utility to administer the Linux Virtual Server
Description :
ipvsadm is used to setup, maintain, and inspect the virtual server
table in the Linux kernel. The Linux Virtual Server can be used to
build scalable network services based on a cluster of two or more
nodes. The active node of the cluster redirects service requests to a
collection of server hosts that will actually perform the
services. Supported Features include:
  - two transport layer (layer-4) protocols (TCP and UDP)
  - three packet-forwarding methods (NAT, tunneling, and direct routing)
  - eight load balancing algorithms (round robin, weighted round robin,
    least-connection, weighted least-connection, locality-based
    least-connection, locality-based least-connection with
    replication, destination-hashing, and source-hashing)

[root@Rocky8 ~]#rpm -ql ipvsadm
/etc/sysconfig/ipvsadm-config
/usr/lib/.build-id
/usr/lib/.build-id/e9
/usr/lib/.build-id/e9/2e0bac9713532c12e56e8290b695eafc6ffb50
/usr/lib/systemd/system/ipvsadm.service
/usr/sbin/ipvsadm
/usr/sbin/ipvsadm-restore
/usr/sbin/ipvsadm-save
/usr/share/doc/ipvsadm
/usr/share/doc/ipvsadm/MAINTAINERS
/usr/share/doc/ipvsadm/README
/usr/share/man/man8/ipvsadm-restore.8.gz
/usr/share/man/man8/ipvsadm-save.8.gz
/usr/share/man/man8/ipvsadm.8.gz

[root@Rocky8 ~]#cat /usr/lib/systemd/system/ipvsadm.service
[Unit]
Description=Initialise the Linux Virtual Server
After=syslog.target network.target

[Service]
Type=oneshot
ExecStart=/bin/bash -c "exec /sbin/ipvsadm-restore < /etc/sysconfig/ipvsadm"
ExecStop=/bin/bash -c "exec /sbin/ipvsadm-save -n > /etc/sysconfig/ipvsadm"
ExecStop=/sbin/ipvsadm -C
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

**ipvsadm 工具用法**

```
[root@Rocky8 ~]#ipvsadm --help
ipvsadm v1.31 2019/12/24 (compiled with popt and IPVS v1.2.1)
Usage:
  ipvsadm -A|E virtual-service [-s scheduler] [-p [timeout]] [-M netmask] [--pe persistence_engine] [-b sched-flags]   #管理集群服务
  ipvsadm -D virtual-service   #删除
  ipvsadm -C      #清空
  ipvsadm -R      #重载，相当于ipvsadm-restore
  ipvsadm -S [-n] #保存，相当于ipvsadm-save
 
 ipvsadm -a|e virtual-service -r server-address [options]  #管理集群中的RS	
  ipvsadm -d virtual-service -r server-address
  ipvsadm -L|l [virtual-service] [options]
  ipvsadm -Z [virtual-service]
  ipvsadm --set tcp tcpfin udp
  ipvsadm --start-daemon {master|backup} [daemon-options]
  ipvsadm --stop-daemon {master|backup}
  ipvsadm -h
```

### 实例：实现lvs集群的增、删、改、查

```
[root@Rocky8 ~]#ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
[root@Rocky8 ~]#ipvsadm -A -t 192.168.188.100:80
[root@Rocky8 ~]#ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.188.100:80 wlc
[root@Rocky8 ~]#ipvsadm -L
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.188.100:http wlc
[root@Rocky8 ~]#ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.188.100:80 wlc
[root@Rocky8 ~]#ipvsadm -a -t 192.168.188.100:80 -r 192.168.188.81:80
[root@Rocky8 ~]#ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.188.100:80 wlc
  -> 192.168.188.81:80            Route   1      0          0         
[root@Rocky8 ~]#ipvsadm -a -t 192.168.188.100:80 -r 192.168.188.82:80
[root@Rocky8 ~]#ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.188.100:80 wlc
  -> 192.168.188.81:80            Route   1      0          0         
  -> 192.168.188.82:80            Route   1      0          0         
[root@Rocky8 ~]#ipvsadm -A -t 192.168.188.200:80 -s wrr
[root@Rocky8 ~]#ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.188.100:80 wlc
  -> 192.168.188.81:80            Route   1      0          0         
  -> 192.168.188.82:80            Route   1      0          0         
TCP  192.168.188.200:80 wrr
[root@Rocky8 ~]#ipvsadm -a -t 192.168.188.200:80 -r 192.168.188.101:8080 -m -w 3
[root@Rocky8 ~]#ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.188.100:80 wlc
  -> 192.168.188.81:80            Route   1      0          0         
  -> 192.168.188.82:80            Route   1      0          0         
TCP  192.168.188.200:80 wrr
  -> 192.168.188.101:8080         Masq    3      0          0         
[root@Rocky8 ~]#ipvsadm -a -t 192.168.188.200:80 -r 192.168.188.102:6060 -m -w 6
[root@Rocky8 ~]#ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.188.100:80 wlc
  -> 192.168.188.81:80            Route   1      0          0         
  -> 192.168.188.82:80            Route   1      0          0         
TCP  192.168.188.200:80 wrr
  -> 192.168.188.101:8080         Masq    3      0          0         
  -> 192.168.188.102:6060         Masq    6      0          0         
[root@Rocky8 ~]#ipvsadm -d -t 192.168.188.200:80 -r 192.168.188.102:6060
[root@Rocky8 ~]#ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.188.100:80 wlc
  -> 192.168.188.81:80            Route   1      0          0         
  -> 192.168.188.82:80            Route   1      0          0         
TCP  192.168.188.200:80 wrr
  -> 192.168.188.101:8080         Masq    3      0          0         
[root@Rocky8 ~]#ipvsadm -D -t 192.168.188.200:80
[root@Rocky8 ~]#ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.188.100:80 wlc
  -> 192.168.188.81:80            Route   1      0          0         
  -> 192.168.188.82:80            Route   1      0          0         
[root@Rocky8 ~]#ipvsadm -C
[root@Rocky8 ~]#ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn


[root@Rocky8 ~]#ipvsadm -A -t 192.168.188.100:80
[root@Rocky8 ~]#ipvsadm -a -t 192.168.188.100:80 -r 192.168.188.81:80
[root@Rocky8 ~]#ipvsadm -a -t 192.168.188.100:80 -r 192.168.188.82:80
[root@Rocky8 ~]#ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.188.100:80 wlc
  -> 192.168.188.81:80            Route   1      0          0         
  -> 192.168.188.82:80            Route   1      0          0         
[root@Rocky8 ~]#cat /proc/net/ip_vs
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port Forward Weight ActiveConn InActConn
TCP  C0A8BC64:0050 wlc  
  -> C0A8BC52:0050      Route   1      0          0         
  -> C0A8BC51:0050      Route   1      0          0  
```

# 4、**LVS** **实战案例**

## **4.1 LVS-NAT**模式案例

### 4.1.1、服务器准备

```
一台internet主机：
192.168.10.200（仅主机）
一台lvs服务器：
eth0：192.168.188.81（NAT）
eth1：192.168.10.100（仅主机）
两台web服务器：
web1：192.168.88.82(NAT)
web2：192.168.88.83(NAT)
```

### 4.1.2、web服务器准备

```
[root@web1 ~]#route -n  #路由指向
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.188.81  0.0.0.0         UG    100    0        0 eth0
192.168.188.0   0.0.0.0         255.255.255.0   U     100    0        0 eth0

[root@web2 ~]#route -n  #路由指向
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.188.81  0.0.0.0         UG    100    0        0 eth0
192.168.188.0   0.0.0.0         255.255.255.0   U     100    0        0 eth0

[root@web1 ~]#yum -y install httpd mod_ssl redis  #安装httpd服务
[root@web1 ~]#systemctl enable --now redis
[root@web2 ~]#yum -y install httpd mod_ssl redis
[root@web2 ~]#systemctl enable --now redis

[root@web1 ~]#ss -ntl   #查看服务端口
State               Recv-Q              Send-Q                           Local Address:Port                            Peer Address:Port              Process              
LISTEN              0                   128                                    0.0.0.0:22                                   0.0.0.0:*                                      
LISTEN              0                   128                                  127.0.0.1:6379                                 0.0.0.0:*                                      
LISTEN              0                   128                                          *:80                                         *:*                                      
LISTEN              0                   128                                       [::]:22                                      [::]:*                                      
LISTEN              0                   128                                          *:443                                        *:*    
```

### 4.1.3、LVS服务器

```
1、检查网卡：
[root@lvs ~]#ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:3e:16:52 brd ff:ff:ff:ff:ff:ff
    altname enp3s0
    altname ens160
    inet 192.168.188.81/24 brd 192.168.188.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe3e:1652/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:3e:16:5c brd ff:ff:ff:ff:ff:ff
    altname enp19s0
    altname ens224
    inet 192.168.10.100/24 brd 192.168.10.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe3e:165c/64 scope link 
       valid_lft forever preferred_lft forever

2、安装ipsadm
[root@lvs ~]#yum -y install ipvsadm
[root@lvs ~]#ipvsadm -A -t 192.168.10.100:80 -s rr
[root@lvs ~]#ipvsadm -a -t 192.168.10.100 -r 192.168.188.82:80 -m
[root@lvs ~]#ipvsadm -a -t 192.168.10.100 -r 192.168.188.83:80 -m

[root@lvs ~]#ipvsadm -Ln   #查看ipvsadm规则
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.10.100:80 rr
  -> 192.168.188.82:80            Masq    1      0          0         
  -> 192.168.188.83:80            Masq    1      0          0     
[root@lvs ~]#ipvsadm -Ln --stats
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port               Conns   InPkts  OutPkts  InBytes OutBytes
  -> RemoteAddress:Port
TCP  192.168.10.100:80                  76      455      300    30150    35700
  -> 192.168.188.82:80                  38      228      152    15124    18088
  -> 192.168.188.83:80                  38      227      148    15026    17612
[root@lvs ~]#cat /proc/net/ip_vs
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port Forward Weight ActiveConn InActConn
TCP  C0A80A64:0050 rr  
  -> C0A8BC53:0050      Masq    1      0          25        
  -> C0A8BC52:0050      Masq    1      0          26  

3、开启ip_forward路由功能
[root@lvs ~]#vim /etc/sysctl.conf·

net.ipv4.ip_forward=1

[root@lvs ~]#sysctl -p  #生效加载规则

4、验证
[root@internet ~]#while :;do curl 192.168.10.100;sleep 0.5;done
web1
web2
web1
web2
web1
web2
web1
web2
web1
web2
web1

[root@lvs ~]#ipvsadm -Lnc   #监听状态
IPVS connection entries
pro expire state       source             virtual            destination
TCP 00:27  TIME_WAIT   192.168.10.200:47554 192.168.10.100:80  192.168.188.82:80
TCP 01:04  TIME_WAIT   192.168.10.200:48292 192.168.10.100:80  192.168.188.82:80
TCP 00:19  TIME_WAIT   192.168.10.200:47404 192.168.10.100:80  192.168.188.82:80
TCP 01:56  TIME_WAIT   192.168.10.200:54608 192.168.10.100:80  192.168.188.83:80
TCP 01:13  TIME_WAIT   192.168.10.200:52148 192.168.10.100:80  192.168.188.82:80
TCP 00:25  TIME_WAIT   192.168.10.200:47510 192.168.10.100:80  192.168.188.83:80
TCP 01:11  TIME_WAIT   192.168.10.200:52108 192.168.10.100:80  192.168.188.83:80
TCP 01:17  TIME_WAIT   192.168.10.200:52228 192.168.10.100:80  192.168.188.82:80
TCP 00:56  TIME_WAIT   192.168.10.200:35712 192.168.10.100:80  192.168.188.83:80



#保存规则
[root@lvs ~]#ipvsadm -Sn > /etc/sysconfig/ipvsadm
[root@lvs ~]#cat /etc/sysconfig/ipvsadm
-A -t 192.168.10.100:80 -s rr
-a -t 192.168.10.100:80 -r 192.168.188.82:80 -m -w 1
-a -t 192.168.10.100:80 -r 192.168.188.83:80 -m -w 1

[root@lvs ~]#systemctl enable --now ipvsadm.service

#reboot验证
[root@lvs ~]#reboot
[root@lvs ~]#ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.10.100:80 rr
  -> 192.168.188.82:80            Masq    1      0          0         
  -> 192.168.188.83:80            Masq    1      0          0         
```

### 4.1.4、internet主机（仅主机）

```
[root@internet ~]#cat /etc/sysconfig/network-scripts/ifcfg-eth0 
DEVICE=eth0
NAME=eth0
BOOTPROTO=static
IPADDR=192.168.10.200
PREFIX=24

[root@internet ~]#while :;do curl 192.168.10.100;sleep 0.5;done
web1
web2
web1
web2
web1
web2
web1
web2
web1
web2
web1
web2
web1
web2
web1
web2
```

## 4.2、实现redis连接

### 4.2.1、安装部署redis服务

```
#redis-server安装redis服务，修改bind ，ti
[root@web1 ~]#yum -y install redis

[root@web1 ~]#sed -i '/^bind/c bind 0.0.0.0'  /etc/redis.conf 
[root@web2 ~]#sed -i '/^bind/c bind 0.0.0.0'  /etc/redis.conf 

[root@web1 ~]#systemctl restart redis
[root@web1 ~]#systemctl enable --now redis
[root@web1 ~]#redis-cli 
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> set host 192.168.188.82
OK
127.0.0.1:6379> quit
```

### 4.2.2、添加LVS规则

```
#添加lvs规则
[root@lvs ~]#ipvsadm -A -t 192.168.10.100:6379
[root@lvs ~]#ipvsadm -a -t 192.168.10.100:6379 -r 192.168.188.82:6379 -m
[root@lvs ~]#ipvsadm -a -t 192.168.10.100:6379 -r 192.168.188.83:6379 -m
[root@lvs ~]#ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.10.100:80 rr
  -> 192.168.188.82:80            Masq    1      0          0         
  -> 192.168.188.83:80            Masq    1      0          0         
TCP  192.168.10.100:6379 wlc
  -> 192.168.188.82:6379          Masq    1      0          0         
  -> 192.168.188.83:6379          Masq    1      0          0   

#查看监听状态
[root@lvs ~]#ipvsadm -Lnc
IPVS connection entries
pro expire state       source             virtual            destination
TCP 01:21  TIME_WAIT   192.168.10.200:47084 192.168.10.100:6379 192.168.188.83:6379
TCP 01:11  TIME_WAIT   192.168.10.200:38416 192.168.10.100:6379 192.168.188.83:6379
TCP 14:56  ESTABLISHED 192.168.10.200:51712 192.168.10.100:6379 192.168.188.83:6379
TCP 01:02  TIME_WAIT   192.168.10.200:57322 192.168.10.100:6379 192.168.188.82:6379
```

### 4.2.3、验证连接

```
[root@web1 ~]#redis-cli  #后端redis添加数据
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> set host 192.168.188.82
OK
127.0.0.1:6379> quit

[root@web2 ~]#redis-cli 
127.0.0.1:6379> ping 
PONG
127.0.0.1:6379> set host 192.168.188.83
OK
127.0.0.1:6379> quit

[root@internet ~]#yum -y install redis  #客户端验证
[root@internet ~]#redis-cli -h 192.168.10.100
192.168.10.100:6379> get host
"192.168.188.83"
192.168.10.100:6379> quit
[root@internet ~]#redis-cli -h 192.168.10.100
192.168.10.100:6379> get host
"192.168.188.82"
192.168.10.100:6379> 
```

## 4.3、DR模型

### 4.3.1、准备工作

![DR模型](C:\Users\m1526\Desktop\DR模型.png)

```
环境：五台主机
一台：客户端 eth0:仅主机 192.168.10.6/24 GW:192.168.10.200
一台：ROUTER
eth0:  NAT  192.168.188.200/24
eth1:  仅主机 192.168.10.200/24
启用 IP_FORWARD
一台：LVS
eth0:NAT:DIP:192.168.188.8/24    GW:192.168.188.200
两台RS：
RS1：eth0:NAT:192.168.188.81/24   GW：192.168.188.200
RS2：eth0:NAT:192.168.188.82/24   GW：192.168.188.200
```

### 4.3.2、rs1、rs2服务器、router准备

```
1、安装并启动服务

[root@web1 ~]# yum -y install redis httpd mod_ssl
[root@web1 ~]# systemctl enable --now httpd redis
[root@web1 ~]#hostname > /var/www/html/index.html
[root@web2 ~]#hostname > /var/www/html/index.html

######################################################################################################
2、修改web1、web2网关
######################################################################################################
[root@web1 ~]#cat /etc/sysconfig/network-scripts/ifcfg-ens160 
BOOTPROTO=static
NAME=eth0
DEVICE=eth0
IPADDR=192.168.188.81
PREFIX=24
GATEWAY=192.168.188.200
DNS1=192.168.188.2
DNS2=223.6.6.6
ONBOOT=yes
[root@web2 ~]#cat /etc/sysconfig/network-scripts/ifcfg-eth0 
DEVICE=eth0
NAME=eth0
BOOTPROTO=static
IPADDR=192.168.188.82
PREFIX=24
GATEWAY=192.168.188.200
DNS1=8.8.8.8
DNS2=192.168.188.2
ONBOOT=yes

[root@web1 ~]#route -n  #查看路由
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.188.200 0.0.0.0         UG    100    0        0 eth0
192.168.188.0   0.0.0.0         255.255.255.0   U     100    0        0 eth0
[root@web2 ~]#route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.188.200 0.0.0.0         UG    100    0        0 eth0
192.168.188.0   0.0.0.0         255.255.255.0   U     100    0        0 eth0


######################################################################################################
3、后端RS1、2的IPVS配置
######################################################################################################
#RS1的IPVS配置、rs2同rs1
[root@web1 ~]#echo 1 >   /proc/sys/net/ipv4/conf/all/arp_ignore
[root@web1 ~]#echo 2 >   /proc/sys/net/ipv4/conf/all/arp_announce 
[root@web1 ~]#echo 1 >   /proc/sys/net/ipv4/conf/lo/arp_ignore
[root@web1 ~]#echo 2 >   /proc/sys/net/ipv4/conf/lo/arp_announce

#添加回环网卡信息两种方法
方法1：
[root@web1 ~]#ip a a 192.168.188.100/32 dev lo label lo:1
方法2：
[root@web1 ~]#ifconfig lo:1 192.168.188.100/32

[root@web1 ~]#ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.188.81  netmask 255.255.255.0  broadcast 192.168.188.255
        inet6 fe80::20c:29ff:fe3e:1652  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:3e:16:52  txqueuelen 1000  (Ethernet)
        RX packets 40795  bytes 36580322 (34.8 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 19145  bytes 1923795 (1.8 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 152  bytes 13532 (13.2 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 152  bytes 13532 (13.2 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo:1: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 192.168.188.100  netmask 255.255.255.255
        loop  txqueuelen 1000  (Local Loopback)

#RS2的IPVS配置
[root@rs2 ~]#echo 1 >   /proc/sys/net/ipv4/conf/all/arp_ignore
[root@rs2 ~]#echo 1 >   /proc/sys/net/ipv4/conf/lo/arp_ignore
[root@rs2 ~]#echo 2 >   /proc/sys/net/ipv4/conf/all/arp_announce 
[root@rs2 ~]#echo 2 >   /proc/sys/net/ipv4/conf/lo/arp_announce
[root@rs2 ~]#ifconfig lo:1 10.0.0.100/32
[root@rs2 ~]#ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group 
default qlen 1000
   link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
   inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
   inet 10.0.0.100/0 scope global lo:1
       valid_lft forever preferred_lft forever
   inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP 
group default qlen 1000
   link/ether 00:0c:29:94:1a:f6 brd ff:ff:ff:ff:ff:ff
   inet 10.0.0.17/24 brd 10.0.0.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
   inet6 fe80::20c:29ff:fe94:1af6/64 scope link 
       valid_lft forever preferred_lft forever

######################################################################################################
4、服务器模拟router
######################################################################################################
[root@router ~]#echo 'net.ipv4.ip_forward=1' >> /etc/sysctl.conf   #开启路由功能，
[root@router ~]#sysctl -p

[root@router ~]# cat /etc/netplan/01-netcfg.yaml
# This file describes the network interfaces available on your system
# For more information, see netplan(5).
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
      addresses: [192.168.188.200/24]
      gateway4: 192.168.188.2
      nameservers: 
          addresses: [8.8.8.8,192.168.188.2]
    eth1:
      addresses: [192.168.10.200/24]

[root@router ~]# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.188.200  netmask 255.255.255.0  broadcast 192.168.188.255
        inet6 fe80::20c:29ff:fe7c:8eeb  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:7c:8e:eb  txqueuelen 1000  (Ethernet)
        RX packets 9939  bytes 971089 (971.0 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 4827  bytes 473398 (473.3 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.10.200  netmask 255.255.255.0  broadcast 192.168.10.255
        inet6 fe80::20c:29ff:fe7c:8ef5  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:7c:8e:f5  txqueuelen 1000  (Ethernet)
        RX packets 7949  bytes 575758 (575.7 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 401  bytes 36723 (36.7 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 162  bytes 13652 (13.6 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 162  bytes 13652 (13.6 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0


######################################################################################################
5、internet主机环境
######################################################################################################
[root@internet ~]#hostname
internet
[root@internet ~]#hostname -I
192.168.10.6 
[root@internet ~]#route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref   Use Iface
192.168.10.0     0.0.0.0         255.255.255.0   U     1      0        0 eth0
0.0.0.0         192.168.10.200   0.0.0.0         UG    0      0        0 eth0
[root@intneter ~]# ping 192.168.188.8 -c1
PING 192.168.188.8 (192.168.188.8) 56(84) bytes of data.
64 bytes from 192.168.188.8: icmp_seq=1 ttl=63 time=1.11 ms

--- 192.168.188.8 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1.119/1.119/1.119/0.000 ms
[root@intneter ~]# ping 192.168.188.81 -c1
PING 192.168.188.81 (192.168.188.81) 56(84) bytes of data.
64 bytes from 192.168.188.81: icmp_seq=1 ttl=63 time=0.971 ms

--- 192.168.188.81 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.971/0.971/0.971/0.000 ms
[root@intneter ~]# ping 192.168.188.82 -c1
PING 192.168.188.82 (192.168.188.82) 56(84) bytes of data.
64 bytes from 192.168.188.82: icmp_seq=1 ttl=63 time=1.86 ms

--- 192.168.188.82 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1.863/1.863/1.863/0.000 ms
```

### 4.3.3、lvs服务器配置

```
#在LVS上添加VIP
[root@lvs ~]#ifconfig lo:1 10.0.0.100/32
[root@lvs ~]#ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet 192.168.188.100/32 scope global lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:fb:1c:80 brd ff:ff:ff:ff:ff:ff
    altname enp3s0
    altname ens160
    inet 192.168.188.8/24 brd 192.168.188.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fefb:1c80/64 scope link 
       valid_lft forever preferred_lft forever

#查看路由表
[root@lvs ~]#route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.188.200 0.0.0.0         UG    100    0        0 eth0
192.168.188.0   0.0.0.0         255.255.255.0   U     100    0        0 eth0


#添加ipvs规则
[root@lvs ~]#ipvsadm -A -t 192.168.188.100:80 -s rr
[root@lvs ~]#ipvsadm -a -t 192.168.188.100:80 -r 192.168.188.81:80 -g
[root@lvs ~]#ipvsadm -a -t 192.168.188.100:80 -r 192.168.188.82:80 -g
[root@lvs ~]#ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.188.100:80 rr
  -> 192.168.188.81:80            Route   1      0          0         
  -> 192.168.188.82:80            Route   1      0          0  
```

### **4.3.4** **测试访问**

```
[root@intneter ~]# while true;do curl 192.168.188.100 ;sleep 1;done
web2.dingbh.top
web1.dingbh.top
web2.dingbh.top
web1.dingbh.top
web2.dingbh.top
web1.dingbh.top
web2.dingbh.top
web1.dingbh.top
........


#实时查看日志
[root@web1 ~]#tail -f /var/log/httpd/access_log -n0
192.168.10.201 - - [12/Oct/2023:15:44:15 +0800] "GET / HTTP/1.1" 200 16 "-" "curl/7.58.0"
192.168.10.201 - - [12/Oct/2023:15:44:41 +0800] "GET / HTTP/1.1" 200 16 "-" "curl/7.58.0"
```

```
抓包分析：
1
src
7c:8e:eb   router  192.168.10.201：38982 

dest
fb：1c：80   lvs 192.168.188.100

2
src
fb:1c:80  lvs:mac 192.168.10.201：38982   

dest
3e:16:52 web1 192.168.188.100

3
src
3e:16:52  web1  192.168.188.100

dest
7c:8e:eb  router  192.168.10.201：38982 
```

### **4.3 LVS-DR**模式多网段案例

**lvs_dr_rs.sh**

```
[root@web1 ~]#vim lvs_dr_rs.sh 

#!/bin/bash
#Author:wangxiaochun
#Date:2017-08-13
vip=172.16.0.100
mask='255.255.255.255'
dev=lo:1

case $1 in
start)
    echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
    echo 1 > /proc/sys/net/ipv4/conf/lo/arp_ignore
    echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
    echo 2 > /proc/sys/net/ipv4/conf/lo/arp_announce
    ifconfig $dev $vip netmask $mask 
    echo "The RS Server is Ready!"
    ;;  
stop)
    ifconfig $dev down
    echo 0 > /proc/sys/net/ipv4/conf/all/arp_ignore
    echo 0 > /proc/sys/net/ipv4/conf/lo/arp_ignore
    echo 0 > /proc/sys/net/ipv4/conf/all/arp_announce
    echo 0 > /proc/sys/net/ipv4/conf/lo/arp_announce
    echo "The RS Server is Canceled!"
    ;;  
*) 
    echo "Usage: $(basename $0) start|stop"
    exit 1
    ;;  
esac
```

**lvs_dr_vs.sh**

```
[root@lvs ~]#vim lvs_dr_vs.sh 

#Date:2017-08-13
vip='172.16.0.100'
iface='lo:1'
mask='255.255.255.255'
port='80'
rs1='192.168.188.81'
rs2='192.168.188.82'
scheduler='wrr'
type='-g'
rpm -q ipvsadm &> /dev/null || yum -y install ipvsadm &> /dev/null

case $1 in
start)
    ifconfig $iface $vip netmask $mask #broadcast $vip up
    iptables -F
 
    ipvsadm -A -t ${vip}:${port} -s $scheduler
    ipvsadm -a -t ${vip}:${port} -r ${rs1} $type -w 1
    ipvsadm -a -t ${vip}:${port} -r ${rs2} $type -w 1
    echo "The VS Server is Ready!"
    ;;  
stop)
    ipvsadm -C
    ifconfig $iface down
    echo "The VS Server is Canceled!"
    ;;  
*)
    echo "Usage: $(basename $0) start|stop"
    exit 1
    ;;  
esac
```

# 5、**LVS** **高可用性**

keepalived+lvs实现?

9、12、15