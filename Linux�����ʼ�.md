```
Magedu20211206
```

域名：dingbh.top

## 条件判断

```
-eq 等于
-ne 不等于
-gt 大于
-ge 大于等于
-lt 小于
-le 小于等于
```

## 第一天

```
linux哲学思想：	
            1一切皆是文件
            2小型单一用途的程序
			3连接程序共同完成复杂功能
			4避免令人困惑的用户界面
			5配置数据存储在文件中

改变提示符颜色：nano /etc/profile     nano .bashrc

将交互的操作变成非交互（自动化执行），提高效率

Linux下的文件类型：

  - 普通文件
  d 目录文件directory
  l 符号链接文件link
  b 块设备block
  c 字符设备character
  p 管道文件pipe
  s 套接字文件socket

basename 只取文件名不要路径
dirname 只取路径，不要文件名

面试答题：
创建一个账户要默认创建文件：改变系统配置文件 /etc/skel/abc,会默认创建文件
```

## 用户默认属性存放路径：

```
[root@Rocky8 mysql]# cat /etc/default/useradd 

# useradd defaults file

GROUP=100
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/bash
SKEL=/etc/skel
CREATE_MAIL_SPOOL=yes
```

## umask实现方式（root默认022，值越大权限越小）

```
新建文件：666-umask ，若结果为奇数（存在执行权限）则加1，偶数不变
新建文件夹：777-umask

持久保存：全局设置/etc/bashrc
         局部设置：针对用户-/.bashrc
. .bashrc重新载入

一次性改变umask：（umask 777；touch a.txt）

特殊权限：suid 4（只对二进制可执行程序有效，设置在目录上无意义）

         sgid 2（作用于目录上，此目录中新建的文件的所有组将自动从目录继承）
         
         sticky t 1  粘滞位 只有目录的所有者可以删

chattr +/-i a.txt  禁止修改  lsattr a.txt
chattr +/-a a.txt  添加内容 lsattr a.txt
```

## vim

```
插入模式
命令 模式
扩展命令模式


vim工具
%全选 s查找替换 d删除  y复制
:%s/usr/USR/g 贪婪模式，默认只替换一行中的第一个，懒惰模式

gc替换前询问


文件格式 ：set ff=dos
         set ff=unix
```



```
[root@Rocky8 ~]# df|tail -n +2 |tr -s ' '|cut -d " " -f5//  -d *指定确认分隔符类型*
[root@Rocky8]~# df -h|tr -s " " %|cut -d% -f5|sort -nr |head -n3
//磁盘占用由大到小排列取前三


[root@Rocky8 ~]# grep -c processor /proc/cpuinfo//取cpu核数


比较两个文档，输出相同项
[root@Rocky8 ~]# cat f1.txt f2.txt |sort |uniq -d
[root@Rocky8 ~]# grep -vf f1.txt f2.txt

搜索相关项
[root@Rocky8 ~]# grep -R/r root /etc/*

安装完整依赖包：dnf group install "Development Tools"

取出IP地址，指定字段// [root@Rocky8 ~]# ifconfig | grep -o '[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}'|head -n1

[root@Rocky8 ~]# ifconfig | sed -rn '2s#^(.*inet )([0-9].+)( netmask.*)$#\2#p'


 索引库文件 // ls /var/lib/mlocate/mlocate.db  updatedb更新

[root@Rocky8 ~]# find /etc -path '/etc/asciidoc' -a -prune -o -name "*.conf"  //排除指定文件路径，指定目录显示


  875 echo user{1..10} | xargs -n1 useradd//批量创建用户名
  876  getent passwd
  877  echo user{1..10} | xargs -n1 userdel//批量删除用户名


```

```
下载bilibili视频
    885  yum install python3-pip -y
  886  pip3 install you-get
  889  seq 192 | xargs -i you-get -p3 	https://www.bilibili.com/video/BV1PZ4y1k7m1?p={}
```

**打包或解包**

```
tar c etc.tar /etc/   打包
-c打包-t预览-x解包
```



## 初始化系统

```
初始化系统c'm
1.关闭selinux：
 sed -i.bak '/SELINUX=enforcing/c SELINUX=disabled' /etc/selinux/config
2.关闭防火墙
systemctl disable --now firewalld
3.别名

修改主机名：centos 6 ：需修改文件/etc/sysconfig/network/，重启后生效
 		centos 7 ：hostnamectl set-hostname XXX ,bash生效

ubuntu开启远程登录功能
sudo -i
passwd root
vim /etc/ssh/sshd.config
systemctl restart sshd

4.网卡名称及IP
[root@Rocky8 ~]# sed -ri '/GRUB_CMDLINE_LINUX=/s#(.*)"$#\1 net.ifnames=0' /etc/default/grub

[root@Rocky8 ~]# grub2-mkconfig  -o /boot/grub2/grub.cfg ; reboot

centos 6 ：vim /etc/udev/rules.d/70-persistent-net.rules
重启网卡：service network restart
centos 7 ：
[root@Rocky7 ~]# sed -ri '/GRUB_CMDLINE_LINUX=/s#(.*)"$#\1 net.ifnames=0' /etc/default/grub
[root@Rocky7 ~]# grub2-mkconfig  -o /boot/grub2/grub.cfg ; reboot
[root@Rocky7 ~]# vim /etc/sysconfig/network-scripts/ifcfg-eth0
[root@Rocky7 ~]# systemctl restart network //重启网卡
centos 8 及rocky:
[root@Rocky8 ~]# sed -ri '/GRUB_CMDLINE_LINUX=/s#(.*)"$#\1 net.ifnames=0#' /etc/default/grub
[root@Rocky8 ~]# grub2-mkconfig  -o /boot/grub2/grub.cfg ; reboot
[root@Rocky8 ~]# vim /etc/sysconfig/network-scripts/ifcfg-eth0 
[root@Rocky8 ~]# nmcli connection reload   //重启网卡
[root@Rocky8 ~]# nmcli connection up eth0   //重启网卡
验证修改结果：ip add / ifconfig
验证网关：route -n / cat /etc/resolv.conf
```

### 修改PS1值

```
export PS1='\[\e[1;34m\][\u@\h \W]\$ \[\e[0m\]'
```

***********************
### 网卡别名（ubuntu）

```
 请配置一块网卡，IP地址是 10.0.0.110，子网掩码是24位，网关是 10.0.0.2，DNS分别是 10.0.0.2 和 114.114.114.114，请写出配置文件内容，另外，写一个脚本，配置成别名，该别名对 root用户永久生效，要求传入网卡设备名可以让修改后的内容重载生效。(8分）
答案写这里：

1、打开终端，使用以下命令编辑网络配置文件（以Ubuntu为例）
sudo nano /etc/netplan/99-netcfg.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
      addresses: [10.0.0.110/24]
      gateway4: 10.0.0.2
      nameservers:
        addresses: [10.0.0.2, 114.114.114.114]

2、应用配置更改，使用以下命令使配置生效：
[root@Ubuntu ~]#sudo netplan apply

3、配置成别名创建一个脚本文件setnetwork.sh
[root@Ubuntu ~]#vim setnetwork.sh 

#!/bin/bash
# 网卡设备名
DEVICE=$1

# 判断是否为root用户
if [ "$(id -u)" != "0" ]; then
    echo "This script must be run as root."
    exit 1
fi

# 编辑网络配置文件
Sudo vim /etc/netplan/eth0.yaml

# 应用配置更改
sudo netplan apply

4、对root用户永久生效，将脚本复制到 /usr/local/bin/ 目录下，并更改文件的权限为可执行：
[root@Ubuntu ~]# cp setnetwork.sh /usr/local/bin/
[root@Ubuntu ~]# chmod +x setnetwork.sh 

5、在 /root/.bashrc 文件中添加以下行以创建别名：
[root@Ubuntu ~]# vim/root/.bashrc
alias setnetwork='/usr/local/bin/set_network_alias.sh'
保存并关闭文件。

6、使用以下命令配置设备名为 eth0 的网卡：

[root@Ubuntu ~]#  setnetwork eth0

7、重启或者netplay apply使配置生效
```

路由：路由的选择
路由表构成
目标网络ID：需要到达的网络ID

iface接口： 如果要到达目标网络ID，需要从本机的哪个接口将数据包发出、

gateway网关：如果目标网络不直接相连，需数据包发送到下一个路由器邻近我的接口的IP，即网关，如果网络和本主机直接，则无需配置网关

Metric花费： 此值越小，优先级越高
**********************
### 5.配置仓库

系统源 epel源

```
[root@Rocky8 ~]# yum -y install epel-release
```



### 6.最小化安装系统后，建议安装常用软件

```
yum -y install autofs vim-enhanced tcpdump autofs chrony lrzsz tree telnet ftp lftp redhat-lsb-core bash-completion net-tools postfix wget bzip2 zip unzip xz lsof mlocate man-pages rsync
```

### **7、解决screen安装失败**

```
994  dnf -y install epel-release
995  dnf -y install screen
```



**发现及扫描新硬盘**

```
alias scandisk="echo '- - -' > /sys/class/scsi_host/host0/scan;echo '- - -' > /sys/class/scsi_host/host1/scan;echo '- - -' > /sys/class/scsi_host/host2/scan"
```



# 硬盘使用的三个步骤:

```
Centos7,8同步分区表命令
 partprobe

分区 fdisk（MBR)  gdisk（GPT)
创建文件系统  mkfs.ext4|xfs   uuid号：blkid
挂载 mount |umount

以下命令适用于ext4 ，（xfs文件系统只能扩容不能缩减xfs_growfs）
扩展逻辑卷： 1.创建物理卷：pvcreate /dev/sdb3 /dev/sdc1
		2.创建卷组：vgcreate -s 16M zabbixvg0 /dev/sdb3 /dev/sdc1
		3.创建逻辑卷：lvcreate -L 5G -n lv-zabbix zabbixvg0
		4.逻辑卷扩容：lvextend -l +100%free /dev/zabbixvg0/lv-zabbix //只扩展逻辑卷，文件系统的大小不会改变 + -r 可以同时执行第5步
		5.扩展文件系统：resize2fs /dev/zabbixvg0/lv-zabbix
		6.缩减逻辑卷：lvreduce -r -L 5G /dev/zabbixvg0/lv-zabbix
pvmove /dev/sdc 将sdc中的空间转移到同卷组的空间中去，安全移除硬盘

实现逻辑卷快照：
1.lvcreate -n lv-zabbix-snapshot -s -p r -L 100M /dev/zabbixvg0/lv-zabbix //创建逻辑卷快照
2.还原逻辑卷快照，必须事先取消挂载
umount /date/zabbix
lvconvert --merge /dev/zabbixvg0/lv-zabbix-snapshot
3.删除逻辑卷（与创建顺序相反）
	lvremove /dev/zabbixvg0/lv-zabbix
	vgremove  zabbixvg0
	pvremove /dev/sdb3 /dev/sdc1


禁用swap
修改 /etc/fstab文件注释掉swap一行
swapon -s 查看swap状态
swapoff /dev/dm-1 禁用swap



RAID技术（1磁盘使用率 2最少几块硬盘可以实现 3容错性防止几块硬盘损坏 4性能好坏）

raid 0 将文件分成若干chunk块，将chunk分别同时存入硬盘中，利用率100，一块硬盘就可以实现，无容错能力，

raid 1 将文件数据同时存入两块硬盘中，两块硬盘的数据一样，有容错能力，利用率50%，最少两块硬盘

raid 4 将文件分别存入两块硬盘，利用一种算法生成一个备份盘，利用率2/3，有容错能力，允许损坏1块硬盘，需3+块硬盘实现。

raid 5 将文件数据以及利用算法生成的备份同时存到硬盘组中，利用率2/3，有容错能力，允许损坏1块硬盘，需3+块硬盘实现。

raid 6


```

## 范例：利用fdisk分区工具分区并创建逻辑卷

```
题目：新增一块20G的硬盘，分三个区，大小分别是3GB，4GB，5GB，将这三个分区做成物理卷，然后创建卷组，在该卷组上创建一个6GB的逻辑卷，挂载至/log/ 目录下，再将该逻辑卷扩容至10GB，请写出实现步骤，文件系统不限？

1、将主机添加并连接一个20G的新硬盘
2、使用fdisk分区工具创建三个大小分别是3GB，4GB，5GB的分区，命令行输入fdisk /dev/nvme0n2，按下 n 创建新分区，然后选择适当的分区类型（p 为主分区，e 为扩展分区）。按照提示创建一个3GB的分区，重复此步骤创建4GB和5GB的分区。最后，按下 w 保存分区表并退出。 

3、将这三个分区创建为物理卷
[root@Rocky8 ~]#pvcreate /dev/nvme0n2p1  /dev/nvme0n2p2  /dev/nvme0n2p3
  Physical volume "/dev/nvme0n2p1" successfully created.
  Physical volume "/dev/nvme0n2p2" successfully created.
  Physical volume "/dev/nvme0n2p3" successfully created.

4、创建一个卷组（Volume Group），将物理卷添加进去。假设卷组的名称为mysqlvg0
[root@Rocky8 ~]#vgcreate mysqlvg0 /dev/nvme0n2p1  /dev/nvme0n2p2  /dev/nvme0n2p3
  Volume group "mysqlvg0" successfully created

5、再卷组上创建一个逻辑卷（Logical Volume）。假设逻辑卷的名称为 mysqlvg0-lv，大小为6GB：
[root@Rocky8 ~]#lvcreate -L 6G -n mysqlvg0-lv mysqlvg0
  Logical volume "mysqlvg0-lv" created.

6、格式化逻辑卷，可以选择 ext4、XFS 等文件系统格式，这里以 ext4 为例：
[root@Rocky8 ~]#mkfs.ext4 /dev/mysqlvg0/mysqlvg0-lv 
mke2fs 1.45.6 (20-Mar-2020)
Creating filesystem with 1572864 4k blocks and 393216 inodes
Filesystem UUID: c658b7a5-56cd-4b1d-8f67-a5248f5f012e
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 

7、创建一个挂载点 /log/
[root@Rocky8 ~]# mkdir /log/

8、将逻辑卷挂载到 /log 目录下：
[root@Rocky8 ~]#mount /dev/mysqlvg0/mysqlvg0-lv /log/

9、查看挂载情况
[root@Rocky8 ~]#df -hl
Filesystem                         Size  Used Avail Use% Mounted on
devtmpfs                           875M     0  875M   0% /dev
tmpfs                              893M     0  893M   0% /dev/shm
tmpfs                              893M   87M  806M  10% /run
tmpfs                              893M     0  893M   0% /sys/fs/cgroup
/dev/mapper/rl-root                100G   20G   81G  20% /
/dev/nvme0n1p1                     974M  220M  687M  25% /boot
/dev/mapper/rl-date                 50G  390M   50G   1% /date
tmpfs                              179M     0  179M   0% /run/user/0
/dev/mapper/mysqlvg0-mysqlvg0--lv  5.9G   24K  5.6G   1% /log

10、扩容逻辑卷至10GB。首先，扩展逻辑卷的大小：
 [root@Rocky8 ~]# lvextend -L 10G /dev/mysqlvg0/mysqlvg0-lv 
  Size of logical volume mysqlvg0/mysqlvg0-lv changed from 6.00 GiB (1536 extents) to 10.00 GiB (2560 extents).
  Logical volume mysqlvg0/mysqlvg0-lv successfully resized.

11、实现文件系统的扩展 针ext 4文件系统
[root@Rocky8 ~]#resize2fs /dev/mysqlvg0/mysqlvg0-lv 
resize2fs 1.45.6 (20-Mar-2020)
Filesystem at /dev/mysqlvg0/mysqlvg0-lv is mounted on /log; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 2
The filesystem on /dev/mysqlvg0/mysqlvg0-lv is now 2621440 (4k) blocks long.

12、查看逻辑卷挂载及大小
[root@Rocky8 ~]#df -h
Filesystem                         Size  Used Avail Use% Mounted on
devtmpfs                           875M     0  875M   0% /dev
tmpfs                              893M     0  893M   0% /dev/shm
tmpfs                              893M   87M  806M  10% /run
tmpfs                              893M     0  893M   0% /sys/fs/cgroup
/dev/mapper/rl-root                100G   20G   81G  20% /
/dev/nvme0n1p1                     974M  220M  687M  25% /boot
/dev/mapper/rl-date                 50G  390M   50G   1% /date
tmpfs                              179M     0  179M   0% /run/user/0
/dev/mapper/mysqlvg0-mysqlvg0--lv  9.8G   24K  9.3G   1% /log

[root@Rocky8 ~]#lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0                        11:0    1 11.3G  0 rom  
nvme0n1                   259:0    0  200G  0 disk 
├─nvme0n1p1               259:1    0    1G  0 part /boot
└─nvme0n1p2               259:2    0  199G  0 part 
  ├─rl-root               253:0    0  100G  0 lvm  /
  ├─rl-swap               253:1    0   49G  0 lvm  
  └─rl-date               253:2    0   50G  0 lvm  /date
nvme0n2                   259:3    0   20G  0 disk 
├─nvme0n2p1               259:10   0    3G  0 part 
│ └─mysqlvg0-mysqlvg0--lv 253:3    0   10G  0 lvm  /log
├─nvme0n2p2               259:11   0    4G  0 part 
│ └─mysqlvg0-mysqlvg0--lv 253:3    0   10G  0 lvm  /log
└─nvme0n2p3               259:12   0    5G  0 part 
  └─mysqlvg0-mysqlvg0--lv 253:3    0   10G  0 lvm  /log


```



## 位置变量：在bash shell中内置的变量, 在脚本代码中调用通过命令行传递给脚本的参数

```
$1, $2, ... 对应第1个、第2个等参数，shift [n]换位置
$0 命令本身,包括路径
$* 传递给脚本的所有参数，全部参数合为一个字符串
$@ 传递给脚本的所有参数，每个参数为独立字符串
$# 传递给脚本的参数的个数
注意：$@ $* 只在被双引号包起来的时候才会有差异

实例：
[root@Rocky8 ~]#vim arg.sh

  1 #!/bin/bash
  2 # **********************************************************
  3 # * Author        : DingBaoHang
  4 # * Email         : m15269032515@163.com
  5 # * Create time   : 2023-06-14 23:15
  6 # * Filename      : arg.sh
  7 # * Description   : 
  8 # **********************************************************
  9 echo "1st arg is $1"
 10 echo "2st arg is $2"
 11 echo "3st arg is $3"
 12 echo "10st arg is ${10}"
 13 echo "11st arg is ${11}"
 14 echo "The number of arg is $#"
 15 echo "All args are $*"
 16 echo "All args are $@"
 17 echo "The scriptname is `basename $0`"

[root@Rocky8 ~]#bash arg.sh {a..z}
1st arg is a
2st arg is b
3st arg is c
10st arg is j
11st arg is k
The number of arg is 26
All args are a b c d e f g h i j k l m n o p q r s t u v w x y z
All args are a b c d e f g h i j k l m n o p q r s t u v w x y z
The scriptname is arg.sh


#随机颜色
[root@Rocky8 ~]#echo -e "\033[1;$[RANDOM%7+31]m`uname -r`\033[0m"
4.18.0-425.19.2.el8_7.x86_64
[root@Rocky8 ~]#echo -e "\033[1;$[RANDOM%7+31]m`uname -r`\033[0m"
4.18.0-425.19.2.el8_7.x86_64
[root@Rocky8 ~]#echo -e "\033[1;$[RANDOM%7+31]m`uname -r`\033[0m"
4.18.0-425.19.2.el8_7.x86_64
[root@Rocky8 ~]#echo -e "\033[1;$[RANDOM%7+31]m`uname -r`\033[0m"
4.18.0-425.19.2.el8_7.x86_64
[root@Rocky8 ~]#echo -e "\033[1;$[RANDOM%7+31]m`uname -r`\033[0m"
4.18.0-425.19.2.el8_7.x86_64

```



## shell脚本面试：

```
面试题，计算1+2+3+...+100 的结果
[root@openvpn-server ~]#sum=0;for i in {1..100};do let sum+=i;done ;echo sum=$sum
sum=5050
[root@openvpn-server ~]#seq -s+ 100|bc5050
5050
[root@openvpn-server ~]#echo {1..100}|tr ' ' +|bc
5050
[root@openvpn-server ~]#seq 100|paste -sd +|bc
5050

#创建账号生成随机密码
for i in {1..10};do
   useradd user$i
    PASS=`cat /dev/urandom | tr -dc '[:alnum:]' |head -c12`
    echo $PASS | passwd --stdin user$i &> /dev/null 
    echo user$i:$PASS >> /data/user.log
    echo "user$i is created"
done

#生成随机密码：
cat /dev/urandom | tr -dc '[:alnum:]' |head -c6
```



## ***面试du 和 df 的区别***

```
df查看的文件系统的空间使用，包含源数据和数据本身，删除文件后，如果业务正在使用，不会立即释放空间。
du 查看的是文件数据本身空间的使用，不包含元数据，删除文件后立即释放空间。
```



# 软件管理：

```
rpm -ivh //显示安装进度
-q 查看安装的包名
-ql 详细显示安装的包的list
-qi 查看包的版本、包名、版本、安装时间、发行者、功能
-qf 反向查询文件来自哪个包
[root@Rocky8 ~]# rpm -qf /etc/passwd
setup-2.12.2-7.el8.noarch
-K 检查包
-e 卸载

yum常用命令

yum provides */route 知道你要安装的那个命令，然后去查找对应的软件仓库
yum [options] [command] [package …]
yum -y 自动回答YES
yum install 软件 安装软件
yum info 软件 查询软件安装信息
yum remove 卸载程序包
yum -y install --downloadonly --downloaddir=/data/httpd httpd 只下载相关依赖包不安装
yum clean [ packages | metadata | expire-cache | rpmdb | plugins | all ] 清除缓存
yum makecache 构建缓存
yum provides  文件名路径



创建私有仓库
dnf reposync --repoid=REPOID --download-metadata -p /var


判断网络的连接状态两种方法
mii-tool ens33
ethtool ens33


统计链接状态的次数
root@ubuntu2204:~# cut -d" " -f1 tcp.log |sed -n '2,$p' |sort |uniq -c |sort -nr 
     10 LISTEN
      3 ESTAB
```



# 为什么需要三次握手：保证数据有去有回



```
调整客户端的动态端口范围
root@ubuntu2204:~# cat /proc/sys/net/ipv4/ip_local_port_range 
32768	60999
root@ubuntu2204:~# echo 20000 62000 > /proc/sys/net/ipv4/ip_local_port_range
root@ubuntu2204:~# cat /proc/sys/net/ipv4/ip_local_port_range 
20000	62000


root@ubuntu2204:~# cat /proc/sys/net/ipv4/tcp_max_syn_backlog 
256
root@ubuntu2204:~# echo 1024 > /proc/sys/net/ipv4/tcp_max_syn_backlog
root@ubuntu2204:~# cat /proc/sys/net/ipv4/tcp_max_syn_backlog 
1024

ping 测：目标端口不可达，目标主机不可达
```

# #网卡绑定bonding设置#

```
[root@Rocky8 network-scripts]# cat ifcfg-bond0 
NAME=bond0
TYPE=bond
DEVICE=bond0
BOOTPROTO=none
IPADDR=192.168.188.70
PREFIX=24
BONDING_OPTS="mode=1 miimon=100 "
[root@Rocky8 network-scripts]# cat ifcfg-eth0
DEVICE=eth0
NAME=eth0
BOOTPROTO=none
MASTER=bond0
SLAVE=yes
ONBOOT=yes
[root@Rocky8 network-scripts]# cat ifcfg-eth1
DEVICE=eth1
NAME=eth1
BOOTPROTO=none
MASTER=bond0
SLAVE=yes
ONBOOT=yes
[root@Rocky8 network-scripts]# cat /proc/net/bonding/bond0 
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)

Bonding Mode: fault-tolerance (active-backup)
Primary Slave: None
Currently Active Slave: eth0
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0
Peer Notification Delay (ms): 0

Slave Interface: eth0
MII Status: up
Speed: 10000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 00:0c:29:d5:8e:21
Slave queue ID: 0

Slave Interface: eth1
MII Status: up
Speed: 10000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 00:0c:29:d5:8e:2b
Slave queue ID: 0
```



## ##找到未知进程的执行程序文件路径##

```
root@Ubuntu1804:~/script# ll /proc/`pidof ksmd`/exe
```

## w和uptime显示的内容

```
系统时间
系统启动时间
当前的登录用户数
系统平均负载（1、5、15分钟的平均负载，一般不会超过1，超过5建议警报）
```

```
清理缓存：
free -h
#撑满缓存
root@Ubuntu1804:~/script# dd if=/dev/zero of=/fi.img bs=1M count=1024 //1024M大小
释放缓存：
root@Ubuntu1804:~/script# echo 3 > /proc/sys/vm/drop_caches
```

## ******安装可视化cockpit工具******

```
root@Ubuntu1804:~ apt -y install cockpit
root@Ubuntu1804:~/script/test-script# systemctl enable --now cockpit.socket //开启服务
访问主机https：//192.168.188.210:9090
```


###centos 6
chkconfig vsftpd on //开机自动启动
service vsftpd start //立即启动cd

## ssh服务

```
ssh连接：忽略询问yes/no
方法1[root@centos7 ~]$ssh -o StrictHostKeyChecking=no 192.168.188.71
方法二[root@centos7 ~]$vim /etc/ssh/ssh_config //客户端改为 no
 35 #   StrictHostKeyChecking no


自动登录sshpass
  888  2023-05-24 06:08:41 sshpass -p 123456 ssh -o StrictHostKeyChecking=no 192.168.188.71 hostname -I
  889  2023-05-24 06:08:53 sshpass -p 123456 ssh -o StrictHostKeyChecking=no 192.168.188.72 hostname -I

修改ssh登录策略vim /etc/ssh/sshd_config 提高连接速度
 69 ChallengeResponseAuthentication no
 79 GSSAPIAuthentication no
116 UseDNS no
```



## pam包

```
###pam包模块的存放目录（调用文件的存放目录）
[root@centos pam.d]# ls /lib64/security/  
pam_access.so    pam_echo.so       pam_ftp.so       pam_localuser.so  pam_permit.so      pam_selinux_permit.so  pam_tally2.so     pam_unix_passwd.so
pam_cap.so       pam_env.so        pam_group.so     pam_loginuid.so   pam_postgresok.so  pam_selinux.so         pam_time.so       pam_unix_session.so
pam_chroot.so    pam_exec.so       pam_issue.so     pam_mail.so       pam_pwhistory.so   pam_sepermit.so        pam_timestamp.so  pam_unix.so
pam_console.so   pam_faildelay.so  pam_keyinit.so   pam_mkhomedir.so  pam_pwquality.so   pam_shells.so          pam_tty_audit.so  pam_userdb.so
pam_cracklib.so  pam_faillock.so   pam_lastlog.so   pam_motd.so       pam_rhosts.so      pam_stress.so          pam_umask.so      pam_warn.so
pam_debug.so     pam_filter        pam_limits.so    pam_namespace.so  pam_rootok.so      pam_succeed_if.so      pam_unix_acct.so  pam_wheel.so
pam_deny.so      pam_filter.so     pam_listfile.so  pam_nologin.so    pam_securetty.so   pam_systemd.so         pam_unix_auth.so  pam_xauth.so

####一些特殊模块的配置文件
[root@centos pam.d]# ls /etc/security/
access.conf  console.apps      console.perms    group.conf   limits.d        namespace.d     opasswd       pwquality.conf  time.conf
chroot.conf  console.handlers  console.perms.d  limits.conf  namespace.conf  namespace.init  pam_env.conf  sepermit.conf

####应用程序调用模块的配置文件
[root@centos pam.d]# ls /etc/pam.d/
chfn         crond                login   password-auth     postlogin     runuser         smartcard-auth-ac  sshd  sudo-i       system-auth-ac  vmtoolsd
chsh         fingerprint-auth     other   password-auth-ac  postlogin-ac  runuser-l       smtp               su    su-l         systemd-user
config-util  fingerprint-auth-ac  passwd  polkit-1          remote        smartcard-auth  smtp.postfix       sudo  system-auth  vlock


```

## 时钟同步工具chrony

```
服务端
[root@openvpn-server ~]# yum -y install chrony
[root@centos8 ~]# vim /etc/chrony.conf 
  3 #pool 2.rocky.pool.ntp.org iburst
  4 server ntp.aliyun.com iburst
  6 server time1.aliyun.com iburst
 24 # Allow NTP client access from local network.
 25 allow 192.168.0.0/16
 34 # Serve time even if not synchronized to a time source.
 35 #local stratum 10
 36 local stratum 10
[root@centos8 ~]# systemctl restart chronyd

客户端
[root@centos7 ~]#vim /etc/chrony.conf
  5 server 192.168.188.81 iburst
[root@centos7 ~]# systemctl restart chronyd
[root@centos7 ~]# chronyc sources -v
210 Number of sources = 0

  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current synced, '+' = combined , '-' = not combined,
| /   '?' = unreachable, 'x' = time may be in error, '~' = time too variable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||      Reachability register (octal) -.           |  xxxx = adjusted offset,
||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
||                                \     |          |  zzzz = estimated error.
||                                 |    |           \
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================

```

## 本地名称解析配置不同系统文件存放路径：

```
Linux：/etc/hosts
windows：C:\Windows\System32\drivers\etc\hosts
```

## DNS域名解析

范例： 正向解析

```sh
[root@rocky8 named]#vi /etc/named.conf
options {
    listen-on port 53 { localhost;};
    。。。。。
    allow-query     { any; };


[root@rocky8 named]#cat /etc/named.rfc1912.zones
zone "magedu.org" {
    type master;
    file "magedu.org.zone";
};

[root@rocky8 named]#cat magedu.org.zone
$TTL 1D
@           IN  SOA master.magedu.org. admin.magedu.org ( 20220117 3H 10M 1D 6H  )
                NS  master.magedu.org.
master.magedu.org.   A   10.0.0.8
www                  A   10.0.0.18
db                   A   10.0.0.200
node1                A   1.1.1.1

```

范例： 反向解析

```sh
[root@rocky8 named]#cat /etc/named.rfc1912.zones
zone "0.0.10.in-addr.arpa" {
    type master;
    file "10.0.0.zone";
};


[root@rocky8 named]#cat 10.0.0.zone
$TTL 1D
@           IN  SOA master admin.magedu.org ( 20220117 3H 10M 1D 6H  )
                NS  master
master          A   10.0.0.8
18              PTR www.magedu.org.
28              PTR db.magedu.org
```

### 面试题：

```
tcp53实现DNS服务器的主从同步     

udp53实现DNS解析查询
```

```
CDN原理？
```

## 防火墙iptables工具

```
iptables -A INPUT -s 192.168.188.7 -j REJECT //拒绝来自192.168.188.7的访问
iptables -I INPUT  -i lo -j ACCEPT //打开自身限制127.0.0.1
#iptables 隐式扩展
	-p指定特定的协议，无需再用-m指定扩展模块
tcp、ucp、time
```



```

##实现仅自身访问web页面##

[root@Rocky8 ~]#iptables -A INPUT -s 192.168.188.99 -j ACCEPT //允许本地主机访问
[root@Rocky8 ~]#iptables -A INPUT -i lo  -j ACCEPT  //允许本地端口经过
[root@Rocky8 ~]#iptables -I INPUT 3 -p tcp  --dport 80  -j ACCEPT //允许http本地80端口访问
[root@Rocky8 ~]#iptables -I INPUT 3 -s 192.168.188.0/24 -j REJECT //拒绝192.168.188.0/24网段通过
[root@Rocky8 ~]#iptables -nvL --line-nu
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination         
1      867 74408 ACCEPT     all  --  *      *       192.168.188.99       0.0.0.0/0           
2        0     0 ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0           
3        0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:80
4        1    76 REJECT     all  --  *      *       192.168.188.0/24     0.0.0.0/0            reject-with icmp-port-unreachable

主机访问web
#http://192.168.188.81/epel/

[root@Rocky8 ~]#iptables -E web WEB //创建WEB链
#multiport扩展
[root@Rocky8 ~]#iptables -A WEB -p tcp -m multiport --dports 80,443 -j ACCEPT //添加iptables规则内容
[root@Rocky8 ~]#iptables -nvL --line-nu
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination         
1     3722  298K ACCEPT     all  --  *      *       192.168.188.99       0.0.0.0/0           
2       30  3177 ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0           
3        0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            multiport dports 80,443
4       60  9432 REJECT     all  --  *      *       192.168.188.0/24     0.0.0.0/0            reject-with icmp-port-unreachable

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination         

Chain WEB (0 references) //WEB链添加成功
num   pkts bytes target     prot opt in     out     source               destination         
1        0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            multiport dports 80,443
[root@Rocky8 ~]#iptables -IINPUT 3 -j WEB //直接调用WEB链 
[root@Rocky8 ~]#iptables -nvL
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
 3895  311K ACCEPT     all  --  *      *       192.168.188.99       0.0.0.0/0           
   30  3177 ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0           
    4   686 WEB        all  --  *      *       0.0.0.0/0            0.0.0.0/0  //直接调用WEB链         
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0   
    

[root@Rocky8 ~]#iptables -F WEB //清除WEB链，事先清除在用规则 
#mac扩展
[root@Rocky8 ~]#iptables -IINPUT 4 -m mac --mac-source 00:0c:29:7e:1d:c9 -j ACCEPT //以mac方式添加规则

#STRING扩展
[root@centos7 ~]#iptables -A OUTPUT -m string --alog bm --from 62 --string "goole" -j REJECT //过滤有关goole的访问

#time扩展


#connlimit扩展
	--connlimit-upto N    //连接的数量小于等于N匹配
	--connlimit-above N   //连接的数量最大为N时匹配
[root@Rocky8 ~]#iptables -AINPUT -m connlimit --connlimit-above 10 -j  REJECT //连接的数量最大为10时匹配
	
	--limit-burst number  #前多少个包不限制
	--limit
```

### 面：调整连接追踪的最大容量（调大nf_conntrack_max值、降低nf_conntrack timeout的时间）

```
（1）调大nf_conntrack_max值
[root@Rocky8 ~]#vim /etc/sysctl.conf

 15 net.nf_conntrack_max= 1000000

（2）降低nf_conntrack timeout的时间
[root@Rocky8 ~]#vim /etc/sysctl.conf

 11 net.ipv4.ip_local_port_range=20000 60000
 
 13 net.ipv4.ip_forward=1 //开启路由模式
 
 15 net.nf_conntrack_max= 1000000
 16 net.netfilter.nf_conntrack_tcp_timeout_established = 120
 17 net.netfilter.nf_conntrack_tcp_timeout_time_wait = 120
 18 net.netfilter.nf_conntrack_tcp_timeout_close_wait = 60
 19 net.netfilter.nf_conntrack_tcp_timeout_fin_wait = 120
 
查看最大追踪条目
 [root@Rocky8 ~]#cat /proc/sys/net/netfilter/nf_conntrack_max 
65536
```

# 实现开机启动iptables规则（通用）

```
1、将rules保存至文件
[root@Rocky8 ~]#iptables-save > /date/iptables.rules
2、恢复iptables规则并查看
[root@Rocky8 ~]#iptables-restore < /date/iptables.rules ；iptables-save
# Generated by iptables-save v1.8.4 on Sat Jun  3 07:47:15 2023
*filter
:INPUT ACCEPT [17:1192]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [16:1852]
:WEB - [0:0]
-A INPUT -s 192.168.188.7/32 -j ACCEPT
-A INPUT -s 192.168.188.71/32 -m state --state NEW -j REJECT --reject-with icmp-port-unreachable
-A OUTPUT -s 192.168.188.7/32 -j ACCEPT
COMMIT
# Completed on Sat Jun  3 07:47:15 2023
# Generated by iptables-save v1.8.4 on Sat Jun  3 07:47:15 2023
*nat
:PREROUTING ACCEPT [3:252]
:INPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [1:84]
:OUTPUT ACCEPT [1:84]
COMMIT
# Completed on Sat Jun  3 07:47:15 2023
# Generated by iptables-save v1.8.4 on Sat Jun  3 07:47:15 2023
*mangle
:PREROUTING ACCEPT [21:1496]
:INPUT ACCEPT [21:1496]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [18:2228]
:POSTROUTING ACCEPT [18:2228]
COMMIT
# Completed on Sat Jun  3 07:47:15 2023

3、设置开启自启动
[root@Rocky8 ~]# vim /etc/rc.local
 10 # Please note that you must run 'chmod +x /etc/rc.d/rc.local' to ensure
 11 # that this script will be executed during boot.
 12 
 13 touch /var/lock/subsys/local
 14 iptables-restore < /date/iptables.rules
4、添加执行权限
[root@Rocky8 ~]# chmod +x /etc/rc.d/rc.local                                               
```

# 实现内外网的互通NAT

```
注意：开启ip_forward功能

SNAT?
DNAT?
格式：

REDIRECT?

```

# mysql关系数据库

```
第一范式：每一列必须是唯一的属性
第二范式：属性完全依赖于主键，比如小明的年龄,必须依赖于那个city的name（PK复合主键）
第三范式：非主键不依赖其他主键属性
```

```
修改mysql_PS1信息

[mysql]
prompt="(\\u@\\h) [\\d]>\\_"


创建及删除库
  790  2023-06-10 16:25:45 mysqladmin -uroot -pHang,,123 creat db2
  791  2023-06-10 16:26:01 mysqladmin -uroot -pHang,,123 drop db2


mysql故障自愈
[root@centos7 ~]# mysqladmin -uroot -pHang,,123 ping || systemctl start mysqld

配置文件/etc/my.cnf添加内容
[root@centos7 ~]# vim /etc/my.cnf

 34 [mysql]
 35 prompt="(\\u@\\h) [\\d]>\\_"
 36 user=root
 37 password=123456
 38 
 39 [mysqld]   //数据库维护，跳过网络连接
 40 skip-networking=true //改成0恢复port：3306链接
 
 
验证结果port：3306端口关闭
[root@centos7 ~]# ss -ntl
State       Recv-Q Send-Q        Local Address:Port                       Peer Address:Port              
LISTEN      0      128                       *:22                                    *:*                  
LISTEN      0      100               127.0.0.1:25                                    *:*                  
LISTEN      0      128                    [::]:22                                 [::]:*                  
LISTEN      0      100                   [::1]:25                                 [::]:*   4

修改配置文件允许所有主机访问port
[root@Ubuntu1804]:~$ vim /etc/mysql/mysql.conf.d/mysqld.cnf 
 27 [mysqld]
 43 #bind-address       = 127.0.0.1

#重启mysql服务
root@Ubuntu1804:~$ systemctl restart mysql.service 

#验证port信息
root@Ubuntu1804:~$ ss -ntl
LISTEN      0             80                                  *:3306                          *:*     
```

## mysql二进制安装

```
1、下载mysql程序包
https://dev.mysql.com/downloads/mysql/
2、解压包
[root@localhost ~]$tar xf mysql-8.0.33-linux-glibc2.28-x86_64.tar.gz -C /usr/local/    //默认安装路径，编译后的路径不允许修改
3、创建软链接
[root@localhost local]$ln -sv mysql-8.0.33-linux-glibc2.28-x86_64/ mysql
4、添加权限
[root@localhost local]$chown -R root.root /usr/local/mysql
5、创建客户端路径及配置文件
[root@localhost mysql]$vim /etc/my.cnf

[mysqld]
datadir=/data/mysql
socket=/data/mysql/mysql.sock
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
# Settings user and group are ignored when systemd is used.
# If you need to run mysqld under a different user or group,
# customize your systemd unit file for mysql according to the
# instructions in http://fedoraproject.org/wiki/Systemd

[mysqld_safe]
log-error=/data/mysql/mysql.log
pid-file=/var/run/mysql/mysql.pid

[client]
socket=/data/mysql/mysql.sock

6、更改环境变量
[root@centos7 ]# echo 'PATH=/usr/local/mysql/bin/:$PATH' > /etc/profile.d/mysql.sh
  . /etc/profile.d/mysql.sh   //生效
7、启动文件
cp support-files/mysql.server /etc/init.d/mysqld
chmod a+x /etc/init.d/mysqld
chkconfig --add mysqld
chkconfig mysqld  on
service mysqld start
8、初始化数据库，生成root密码
[root@centos7 lib64]# mysqld --initialize-insecure --user=mysql --datadir=/data/mysql

```

## mysql使用

```
1、创建数据库
mysql> create database zabbix;
Query OK, 1 row affected (0.00 sec)

2、查看数据库
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| zabbix             |
+--------------------+
5 rows in set (0.01 sec)

3、切换数据库
mysql> use zabbix;
Database changed

4、创建表
mysql> create table student (id tinyint unsigned primary key auto_increment, name char(4) not null  , gender char(1), age tinyint unsigned );
Query OK, 0 rows affected (0.02 sec)

5、查看表
mysql> show tables;
+------------------+
| Tables_in_zabbix |
+------------------+
| student          |
+------------------+
1 row in set (0.00 sec)

6、查看表结构
mysql> desc student;
+--------+------------------+------+-----+---------+----------------+
| Field  | Type             | Null | Key | Default | Extra          |
+--------+------------------+------+-----+---------+----------------+
| id     | tinyint unsigned | NO   | PRI | NULL    | auto_increment |
| name   | char(4)          | NO   |     | NULL    |                |
| gender | char(1)          | YES  |     | NULL    |                |
| age    | tinyint unsigned | YES  |     | NULL    |                |
+--------+------------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)

7、删除表
mysql> drop table teacher;
Query OK, 0 rows affected (0.01 sec)

8、将查询结果创建新的表
mysql> create table teacher select user,host from mysql.user;
Query OK, 4 rows affected (0.02 sec)
Records: 4  Duplicates: 0  Warnings: 0

mysql> desc teacher
    -> ;
+-------+-----------+------+-----+---------+-------+
| Field | Type      | Null | Key | Default | Extra |
+-------+-----------+------+-----+---------+-------+
| user  | char(32)  | NO   |     |         |       |
| host  | char(255) | NO   |     |         |       |
+-------+-----------+------+-----+---------+-------+
2 rows in set (0.00 sec)

9、查询表内容
mysql> select * from emp;
Empty set (0.00 sec)

10、like创建相同表结构的表
mysql> create table emp like student;
Query OK, 0 rows affected (0.01 sec)

mysql> show tables ;
+------------------+
| Tables_in_zabbix |
+------------------+
| emp              |
| student          |
| teacher          |
+------------------+
3 rows in set (0.00 sec)

mysql> desc emp;
+--------+------------------+------+-----+---------+----------------+
| Field  | Type             | Null | Key | Default | Extra          |
+--------+------------------+------+-----+---------+----------------+
| id     | tinyint unsigned | NO   | PRI | NULL    | auto_increment |
| name   | char(4)          | NO   |     | NULL    |                |
| gender | char(1)          | YES  |     | NULL    |                |
| age    | tinyint unsigned | YES  |     | NULL    |                |
+--------+------------------+------+-----+---------+----------------+
4 rows in set (0.01 sec)

11、查看所有表
mysql> SHOW TABLE STATUS\G ;
*************************** 1. row ***************************
           Name: emp
         Engine: InnoDB
        Version: 10
     Row_format: Dynamic
           Rows: 0
 Avg_row_length: 0
    Data_length: 16384
Max_data_length: 0
   Index_length: 0
      Data_free: 0
 Auto_increment: 1
    Create_time: 2023-06-15 08:52:56
    Update_time: NULL
     Check_time: NULL
      Collation: utf8mb4_0900_ai_ci
       Checksum: NULL
 Create_options: 
        Comment: 
*************************** 2. row ***************************
           Name: student
         Engine: InnoDB
        Version: 10
     Row_format: Dynamic
           Rows: 0
 Avg_row_length: 0
    Data_length: 16384
Max_data_length: 0
   Index_length: 0
      Data_free: 0
 Auto_increment: 1
    Create_time: 2023-06-15 08:36:26
    Update_time: NULL
     Check_time: NULL
      Collation: utf8mb4_0900_ai_ci
       Checksum: NULL
 Create_options: 
        Comment: 
*************************** 3. row ***************************
           Name: teacher
         Engine: InnoDB
        Version: 10
     Row_format: Dynamic
           Rows: 4
 Avg_row_length: 4096
    Data_length: 16384
Max_data_length: 0
   Index_length: 0
      Data_free: 0
 Auto_increment: NULL
    Create_time: 2023-06-15 08:49:35
    Update_time: 2023-06-15 08:49:35
     Check_time: NULL
      Collation: utf8mb4_0900_ai_ci
       Checksum: NULL
 Create_options: 
        Comment: 
3 rows in set (0.00 sec)

ERROR: 
No query specified

12、在表中插入内容

mysql> insert into student (id,name,gender,age) values(1,'ding','M',21);
Query OK, 1 row affected (0.00 sec)

mysql> select * from student;
+----+------+--------+------+
| id | name | gender | age  |
+----+------+--------+------+
|  1 | ding | M      |   21 |
+----+------+--------+------+
1 row in set (0.00 sec)

13、更新及更改表内容
mysql> updata student set age = 19 where id = 4 ;  //必须加where限制条件
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from student;

|  3 | HANG | M      |   19 |

14、删除表及内容
mysql> delete from student where age >=23 ;  //删除age大于23的内容
Query OK, 4 rows affected (0.00 sec)
mysql> select * from student;

*********缩减表的大小*****
mysql> OPTIMIZE TABLE student;
+----------------+----------+----------+-------------------------------------------------------------------+
| Table          | Op       | Msg_type | Msg_text                                                          |
+----------------+----------+----------+-------------------------------------------------------------------+
| zabbix.student | optimize | note     | Table does not support optimize, doing recreate + analyze instead |
| zabbix.student | optimize | status   | OK                                                                |
+----------------+----------+----------+-------------------------------------------------------------------+
2 rows in set (0.03 sec)

*******清除并缩减表的大小******
mysql> TRUNCATE TABLE teacher;
Query OK, 0 rows affected (0.01 sec)


15、统计表的数量
mysql> select count(*) from student;
+----------+
| count(*) |
+----------+
|        2 |
+----------+
1 row in set (0.00 sec)


```

### mysql单表查询

```
mysql> select * from students where name = 'ma chao';
+-------+---------+-----+--------+---------+-----------+
| StuID | Name    | Age | Gender | ClassID | TeacherID |
+-------+---------+-----+--------+---------+-----------+
|    23 | Ma Chao |  23 | M      |       4 |      NULL |
+-------+---------+-----+--------+---------+-----------+
1 row in set (0.00 sec)

mysql> select * from students where name like '%s%';
+-------+--------------+-----+--------+---------+-----------+
| StuID | Name         | Age | Gender | ClassID | TeacherID |
+-------+--------------+-----+--------+---------+-----------+
|     1 | Shi Zhongyu  |  22 | M      |       2 |         3 |
|     2 | Shi Potian   |  22 | M      |       1 |         7 |
|     6 | Shi Qing     |  46 | M      |       5 |      NULL |
|    10 | Yue Lingshan |  19 | F      |       3 |      NULL |
|    14 | Lu Wushuang  |  17 | F      |       3 |      NULL |
|    25 | Sun Dasheng  | 100 | M      |    NULL |      NULL |
+-------+--------------+-----+--------+---------+-----------+
6 rows in set (0.00 sec)

mysql> select stuid,name 姓名,gender 性别 from students;
+-------+---------------+--------+
| stuid | 姓名          | 性别   |
+-------+---------------+--------+
|     1 | Shi Zhongyu   | M      |
|     2 | Shi Potian    | M      |
|     3 | Xie Yanke     | M      |
|     4 | Ding Dian     | M      |
|     5 | Yu Yutong     | M      |
|     6 | Shi Qing      | M      |

mysql> select * from students where Age <=20  and gender='F';

mysql> select * from students where classid in (1,2,3);

mysql> select * from students where age limit 3,5;

mysql> select * from students where age between 20 and 30;

mysql> select * from students where name ='ma chao' and age=''or '1'='1';

#按照年龄排序order by 关键字
mysql> select * from students order by age;
+-------+---------------+-----+--------+---------+-----------+
| StuID | Name          | Age | Gender | ClassID | TeacherID |
+-------+---------------+-----+--------+---------+-----------+
|    14 | Lu Wushuang   |  17 | F      |       3 |      NULL |
|     8 | Lin Daiyu     |  17 | F      |       7 |      NULL |
|    19 | Xue Baochai   |  18 | F      |       6 |      NULL |
|    20 | Diao Chan     |  19 | F      |       7 |      NULL |
|    15 | Duan Yu       |  19 | M      |       4 |      NULL |
|    12 | Wen Qingqing  |  19 | F      |       1 |      NULL |
|     7 | Xi Ren        |  19 | F      |       3 |      NULL |
|    10 | Yue Lingshan  |  19 | F      |       3 |      NULL |
|    22 | Xiao Qiao     |  20 | F      |       1 |      NULL |
|     9 | Ren Yingying  |  20 | F      |       6 |      NULL |
|    16 | Xu Zhu        |  21 | M      |       1 |      NULL |
|     1 | Shi Zhongyu   |  22 | M      |       2 |         3 |
|     2 | Shi Potian    |  22 | M      |       1 |         7 |
|    21 | Huang Yueying |  22 | F      |       6 |      NULL |
|    11 | Yuan Chengzhi |  23 | M      |       6 |      NULL |
|    23 | Ma Chao       |  23 | M      |       4 |      NULL |
|    18 | Hua Rong      |  23 | M      |       7 |      NULL |
|    17 | Lin Chong     |  25 | M      |       4 |      NULL |
|     5 | Yu Yutong     |  26 | M      |       3 |         1 |
|    24 | Xu Xian       |  27 | M      |    NULL |      NULL |
|     4 | Ding Dian     |  32 | M      |       4 |         4 |
|    13 | Tian Boguang  |  33 | M      |       2 |      NULL |
|     6 | Shi Qing      |  46 | M      |       5 |      NULL |
|     3 | Xie Yanke     |  53 | M      |       2 |        16 |
|    25 | Sun Dasheng   | 100 | M      |    NULL |      NULL |
+-------+---------------+-----+--------+---------+-----------+
25 rows in set (0.00 sec)
**倒序排序
mysql> select * from students order by age desc;

```

### mysql多表查询

#### 1、子查询

```
mysql> update teachers set age = (select max(age) from students) where tid=2;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
mysql> select * from teachers;
+-----+---------------+-----+--------+
| TID | Name          | Age | Gender |
+-----+---------------+-----+--------+
|   1 | Song Jiang    |  45 | M      |
|   2 | Zhang Sanfeng | 100 | M      |
|   3 | Miejue Shitai |  77 | F      |
|   4 | Lin Chaoying  |  93 | F      |
+-----+---------------+-----+--------+
4 rows in set (0.00 sec)



```

#### 2、联合查询  union字段

```
mysql> select * from teachers;
+-----+---------------+-----+--------+
| TID | Name          | Age | Gender |
+-----+---------------+-----+--------+
|   1 | Song Jiang    |  45 | M      |
|   2 | Zhang Sanfeng | 100 | M      |
|   3 | Miejue Shitai |  77 | F      |
|   4 | Lin Chaoying  |  93 | F      |
+-----+---------------+-----+--------+
4 rows in set (0.00 sec)
mysql> select * from students;
+-------+---------------+-----+--------+---------+-----------+
| StuID | Name          | Age | Gender | ClassID | TeacherID |
+-------+---------------+-----+--------+---------+-----------+
|     1 | Shi Zhongyu   |  22 | M      |       2 |         3 |
|     2 | Shi Potian    |  22 | M      |       1 |         7 |
|     3 | Xie Yanke     |  53 | M      |       2 |        16 |
|     4 | Ding Dian     |  32 | M      |       4 |         4 |
|     5 | Yu Yutong     |  26 | M      |       3 |         1 |
|     6 | Shi Qing      |  46 | M      |       5 |      NULL |
|     7 | Xi Ren        |  19 | F      |       3 |      NULL |
|     8 | Lin Daiyu     |  17 | F      |       7 |      NULL |
|     9 | Ren Yingying  |  20 | F      |       6 |      NULL |
|    10 | Yue Lingshan  |  19 | F      |       3 |      NULL |
|    11 | Yuan Chengzhi |  23 | M      |       6 |      NULL |
|    12 | Wen Qingqing  |  19 | F      |       1 |      NULL |
|    13 | Tian Boguang  |  33 | M      |       2 |      NULL |
|    14 | Lu Wushuang   |  17 | F      |       3 |      NULL |
|    15 | Duan Yu       |  19 | M      |       4 |      NULL |
|    16 | Xu Zhu        |  21 | M      |       1 |      NULL |
|    17 | Lin Chong     |  25 | M      |       4 |      NULL |
|    18 | Hua Rong      |  23 | M      |       7 |      NULL |
|    19 | Xue Baochai   |  18 | F      |       6 |      NULL |
|    20 | Diao Chan     |  19 | F      |       7 |      NULL |
|    21 | Huang Yueying |  22 | F      |       6 |      NULL |
|    22 | Xiao Qiao     |  20 | F      |       1 |      NULL |
|    23 | Ma Chao       |  23 | M      |       4 |      NULL |
|    24 | Xu Xian       |  27 | M      |    NULL |      NULL |
|    25 | Sun Dasheng   | 100 | M      |    NULL |      NULL |
+-------+---------------+-----+--------+---------+-----------+
25 rows in set (0.00 sec)

mysql> select stuid id,name,age,gender from students union select * from teachers;
+----+---------------+-----+--------+
| id | name          | age | gender |
+----+---------------+-----+--------+
|  1 | Shi Zhongyu   |  22 | M      |
|  2 | Shi Potian    |  22 | M      |
|  3 | Xie Yanke     |  53 | M      |
|  4 | Ding Dian     |  32 | M      |
|  5 | Yu Yutong     |  26 | M      |
|  6 | Shi Qing      |  46 | M      |
|  7 | Xi Ren        |  19 | F      |
|  8 | Lin Daiyu     |  17 | F      |
|  9 | Ren Yingying  |  20 | F      |
| 10 | Yue Lingshan  |  19 | F      |
| 11 | Yuan Chengzhi |  23 | M      |
| 12 | Wen Qingqing  |  19 | F      |
| 13 | Tian Boguang  |  33 | M      |
| 14 | Lu Wushuang   |  17 | F      |
| 15 | Duan Yu       |  19 | M      |
| 16 | Xu Zhu        |  21 | M      |
| 17 | Lin Chong     |  25 | M      |
| 18 | Hua Rong      |  23 | M      |
| 19 | Xue Baochai   |  18 | F      |
| 20 | Diao Chan     |  19 | F      |
| 21 | Huang Yueying |  22 | F      |
| 22 | Xiao Qiao     |  20 | F      |
| 23 | Ma Chao       |  23 | M      |
| 24 | Xu Xian       |  27 | M      |
| 25 | Sun Dasheng   | 100 | M      |
|  1 | Song Jiang    |  45 | M      |
|  2 | Zhang Sanfeng | 100 | M      |
|  3 | Miejue Shitai |  77 | F      |
|  4 | Lin Chaoying  |  93 | F      |
+----+---------------+-----+--------+
29 rows in set (0.00 sec)
```

#### 3、交叉连接

```
mysql> select count(*) from students cross join teachers;
+----------+
| count(*) |
+----------+
|      100 |
+----------+
1 row in set (0.01 sec)
```

#### 4、内连接 ，重叠的项

```
mysql> select s.stuid,s.name,s.teacherid,t.tid,t.name from students s inner join teachers t on s.teacherid=t.tid;
+-------+-------------+-----------+-----+---------------+
| stuid | name        | teacherid | tid | name          |
+-------+-------------+-----------+-----+---------------+
|     1 | Shi Zhongyu |         3 |   3 | Miejue Shitai |
|     4 | Ding Dian   |         4 |   4 | Lin Chaoying  |
|     5 | Yu Yutong   |         1 |   1 | Song Jiang    |
+-------+-------------+-----------+-----+---------------+
3 rows in set (0.00 sec)
```

#### 5、左右链接

##### 左连接

```

mysql> select * from students s inner join teachers t on s.teacherid=t.tid;
+-------+-------------+-----+--------+---------+-----------+-----+---------------+-----+--------+
| StuID | Name        | Age | Gender | ClassID | TeacherID | TID | Name          | Age | Gender |
+-------+-------------+-----+--------+---------+-----------+-----+---------------+-----+--------+
|     1 | Shi Zhongyu |  22 | M      |       2 |         3 |   3 | Miejue Shitai |  77 | F      |
|     4 | Ding Dian   |  32 | M      |       4 |         4 |   4 | Lin Chaoying  |  93 | F      |
|     5 | Yu Yutong   |  26 | M      |       3 |         1 |   1 | Song Jiang    |  45 | M      |
+-------+-------------+-----+--------+---------+-----------+-----+---------------+-----+--------+
3 rows in set (0.00 sec)

mysql> select * from students s left outer teachers t on s.teacherid=t.tid;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'teachers t on s.teacherid=t.tid' at line 1
mysql> select * from students s left outer join teachers t on s.teacherid=t.tid;
+-------+---------------+-----+--------+---------+-----------+------+---------------+------+--------+
| StuID | Name          | Age | Gender | ClassID | TeacherID | TID  | Name          | Age  | Gender |
+-------+---------------+-----+--------+---------+-----------+------+---------------+------+--------+
|     1 | Shi Zhongyu   |  22 | M      |       2 |         3 |    3 | Miejue Shitai |   77 | F      |
|     2 | Shi Potian    |  22 | M      |       1 |         7 | NULL | NULL          | NULL | NULL   |
|     3 | Xie Yanke     |  53 | M      |       2 |        16 | NULL | NULL          | NULL | NULL   |
|     4 | Ding Dian     |  32 | M      |       4 |         4 |    4 | Lin Chaoying  |   93 | F      |
|     5 | Yu Yutong     |  26 | M      |       3 |         1 |    1 | Song Jiang    |   45 | M      |
|     6 | Shi Qing      |  46 | M      |       5 |      NULL | NULL | NULL          | NULL | NULL   |
|     7 | Xi Ren        |  19 | F      |       3 |      NULL | NULL | NULL          | NULL | NULL   |
|     8 | Lin Daiyu     |  17 | F      |       7 |      NULL | NULL | NULL          | NULL | NULL   |
|     9 | Ren Yingying  |  20 | F      |       6 |      NULL | NULL | NULL          | NULL | NULL   |
|    10 | Yue Lingshan  |  19 | F      |       3 |      NULL | NULL | NULL          | NULL | NULL   |
|    11 | Yuan Chengzhi |  23 | M      |       6 |      NULL | NULL | NULL          | NULL | NULL   |
|    12 | Wen Qingqing  |  19 | F      |       1 |      NULL | NULL | NULL          | NULL | NULL   |
|    13 | Tian Boguang  |  33 | M      |       2 |      NULL | NULL | NULL          | NULL | NULL   |
|    14 | Lu Wushuang   |  17 | F      |       3 |      NULL | NULL | NULL          | NULL | NULL   |
|    15 | Duan Yu       |  19 | M      |       4 |      NULL | NULL | NULL          | NULL | NULL   |
|    16 | Xu Zhu        |  21 | M      |       1 |      NULL | NULL | NULL          | NULL | NULL   |
|    17 | Lin Chong     |  25 | M      |       4 |      NULL | NULL | NULL          | NULL | NULL   |
|    18 | Hua Rong      |  23 | M      |       7 |      NULL | NULL | NULL          | NULL | NULL   |
|    19 | Xue Baochai   |  18 | F      |       6 |      NULL | NULL | NULL          | NULL | NULL   |
|    20 | Diao Chan     |  19 | F      |       7 |      NULL | NULL | NULL          | NULL | NULL   |
|    21 | Huang Yueying |  22 | F      |       6 |      NULL | NULL | NULL          | NULL | NULL   |
|    22 | Xiao Qiao     |  20 | F      |       1 |      NULL | NULL | NULL          | NULL | NULL   |
|    23 | Ma Chao       |  23 | M      |       4 |      NULL | NULL | NULL          | NULL | NULL   |
|    24 | Xu Xian       |  27 | M      |    NULL |      NULL | NULL | NULL          | NULL | NULL   |
|    25 | Sun Dasheng   | 100 | M      |    NULL |      NULL | NULL | NULL          | NULL | NULL   |
+-------+---------------+-----+--------+---------+-----------+------+---------------+------+--------+
25 rows in set (0.00 sec)

```

##### 右连接

```
mysql> select * from teachers;
+-----+---------------+-----+--------+
| TID | Name          | Age | Gender |
+-----+---------------+-----+--------+
|   1 | Song Jiang    |  45 | M      |
|   2 | Zhang Sanfeng | 100 | M      |
|   3 | Miejue Shitai |  77 | F      |
|   4 | Lin Chaoying  |  93 | F      |
+-----+---------------+-----+--------+
4 rows in set (0.00 sec)

mysql> select * from students s right outer join teachers t on s.teacherid=t.tid;
+-------+-------------+------+--------+---------+-----------+-----+---------------+-----+--------+
| StuID | Name        | Age  | Gender | ClassID | TeacherID | TID | Name          | Age | Gender |
+-------+-------------+------+--------+---------+-----------+-----+---------------+-----+--------+
|     5 | Yu Yutong   |   26 | M      |       3 |         1 |   1 | Song Jiang    |  45 | M      |
|  NULL | NULL        | NULL | NULL   |    NULL |      NULL |   2 | Zhang Sanfeng | 100 | M      |
|     1 | Shi Zhongyu |   22 | M      |       2 |         3 |   3 | Miejue Shitai |  77 | F      |
|     4 | Ding Dian   |   32 | M      |       4 |         4 |   4 | Lin Chaoying  |  93 | F      |
+-------+-------------+------+--------+---------+-----------+-----+---------------+-----+--------+
4 rows in set (0.00 sec)

```

#### 6、完全外连接

```
mysql> select * from students s right outer join teachers t on s.teacherid=t.tid
    -> union
    -> select * from students s left outer join teachers t on s.teacherid=t.tid;
+-------+---------------+------+--------+---------+-----------+------+---------------+------+--------+
| StuID | Name          | Age  | Gender | ClassID | TeacherID | TID  | Name          | Age  | Gender |
+-------+---------------+------+--------+---------+-----------+------+---------------+------+--------+
|     5 | Yu Yutong     |   26 | M      |       3 |         1 |    1 | Song Jiang    |   45 | M      |
|  NULL | NULL          | NULL | NULL   |    NULL |      NULL |    2 | Zhang Sanfeng |  100 | M      |
|     1 | Shi Zhongyu   |   22 | M      |       2 |         3 |    3 | Miejue Shitai |   77 | F      |
|     4 | Ding Dian     |   32 | M      |       4 |         4 |    4 | Lin Chaoying  |   93 | F      |
|     2 | Shi Potian    |   22 | M      |       1 |         7 | NULL | NULL          | NULL | NULL   |
|     3 | Xie Yanke     |   53 | M      |       2 |        16 | NULL | NULL          | NULL | NULL   |
|     6 | Shi Qing      |   46 | M      |       5 |      NULL | NULL | NULL          | NULL | NULL   |
|     7 | Xi Ren        |   19 | F      |       3 |      NULL | NULL | NULL          | NULL | NULL   |
|     8 | Lin Daiyu     |   17 | F      |       7 |      NULL | NULL | NULL          | NULL | NULL   |
|     9 | Ren Yingying  |   20 | F      |       6 |      NULL | NULL | NULL          | NULL | NULL   |
|    10 | Yue Lingshan  |   19 | F      |       3 |      NULL | NULL | NULL          | NULL | NULL   |
|    11 | Yuan Chengzhi |   23 | M      |       6 |      NULL | NULL | NULL          | NULL | NULL   |
|    12 | Wen Qingqing  |   19 | F      |       1 |      NULL | NULL | NULL          | NULL | NULL   |
|    13 | Tian Boguang  |   33 | M      |       2 |      NULL | NULL | NULL          | NULL | NULL   |
|    14 | Lu Wushuang   |   17 | F      |       3 |      NULL | NULL | NULL          | NULL | NULL   |
|    15 | Duan Yu       |   19 | M      |       4 |      NULL | NULL | NULL          | NULL | NULL   |
|    16 | Xu Zhu        |   21 | M      |       1 |      NULL | NULL | NULL          | NULL | NULL   |
|    17 | Lin Chong     |   25 | M      |       4 |      NULL | NULL | NULL          | NULL | NULL   |
|    18 | Hua Rong      |   23 | M      |       7 |      NULL | NULL | NULL          | NULL | NULL   |
|    19 | Xue Baochai   |   18 | F      |       6 |      NULL | NULL | NULL          | NULL | NULL   |
|    20 | Diao Chan     |   19 | F      |       7 |      NULL | NULL | NULL          | NULL | NULL   |
|    21 | Huang Yueying |   22 | F      |       6 |      NULL | NULL | NULL          | NULL | NULL   |
|    22 | Xiao Qiao     |   20 | F      |       1 |      NULL | NULL | NULL          | NULL | NULL   |
|    23 | Ma Chao       |   23 | M      |       4 |      NULL | NULL | NULL          | NULL | NULL   |
|    24 | Xu Xian       |   27 | M      |    NULL |      NULL | NULL | NULL          | NULL | NULL   |
|    25 | Sun Dasheng   |  100 | M      |    NULL |      NULL | NULL | NULL          | NULL | NULL   |
+-------+---------------+------+--------+---------+-----------+------+---------------+------+--------+
26 rows in set (0.00 sec)
```

#### 7、实例：三表查询

```
mysql> select * from students;
+-------+---------------+-----+--------+---------+-----------+
| StuID | Name          | Age | Gender | ClassID | TeacherID |
+-------+---------------+-----+--------+---------+-----------+
|     1 | Shi Zhongyu   |  22 | M      |       2 |         3 |
|     2 | Shi Potian    |  22 | M      |       1 |         7 |
|     3 | Xie Yanke     |  53 | M      |       2 |        16 |
|     4 | Ding Dian     |  32 | M      |       4 |         4 |
|     5 | Yu Yutong     |  26 | M      |       3 |         1 |
|     6 | Shi Qing      |  46 | M      |       5 |      NULL |
|     7 | Xi Ren        |  19 | F      |       3 |      NULL |
|     8 | Lin Daiyu     |  17 | F      |       7 |      NULL |
|     9 | Ren Yingying  |  20 | F      |       6 |      NULL |
|    10 | Yue Lingshan  |  19 | F      |       3 |      NULL |
|    11 | Yuan Chengzhi |  23 | M      |       6 |      NULL |
|    12 | Wen Qingqing  |  19 | F      |       1 |      NULL |
|    13 | Tian Boguang  |  33 | M      |       2 |      NULL |
|    14 | Lu Wushuang   |  17 | F      |       3 |      NULL |
|    15 | Duan Yu       |  19 | M      |       4 |      NULL |
|    16 | Xu Zhu        |  21 | M      |       1 |      NULL |
|    17 | Lin Chong     |  25 | M      |       4 |      NULL |
|    18 | Hua Rong      |  23 | M      |       7 |      NULL |
|    19 | Xue Baochai   |  18 | F      |       6 |      NULL |
|    20 | Diao Chan     |  19 | F      |       7 |      NULL |
|    21 | Huang Yueying |  22 | F      |       6 |      NULL |
|    22 | Xiao Qiao     |  20 | F      |       1 |      NULL |
|    23 | Ma Chao       |  23 | M      |       4 |      NULL |
|    24 | Xu Xian       |  27 | M      |    NULL |      NULL |
|    25 | Sun Dasheng   | 100 | M      |    NULL |      NULL |
+-------+---------------+-----+--------+---------+-----------+
25 rows in set (0.00 sec)


mysql> select * from courses;
+----------+----------------+
| CourseID | Course         |
+----------+----------------+
|        1 | Hamo Gong      |
|        2 | Kuihua Baodian |
|        3 | Jinshe Jianfa  |
|        4 | Taiji Quan     |
|        5 | Daiyu Zanghua  |
|        6 | Weituo Zhang   |
|        7 | Dagou Bangfa   |
+----------+----------------+
7 rows in set (0.00 sec)

mysql> select * from scores;
+----+-------+----------+-------+
| ID | StuID | CourseID | Score |
+----+-------+----------+-------+
|  1 |     1 |        2 |    77 |
|  2 |     1 |        6 |    93 |
|  3 |     2 |        2 |    47 |
|  4 |     2 |        5 |    97 |
|  5 |     3 |        2 |    88 |
|  6 |     3 |        6 |    75 |
|  7 |     4 |        5 |    71 |
|  8 |     4 |        2 |    89 |
|  9 |     5 |        1 |    39 |
| 10 |     5 |        7 |    63 |
| 11 |     6 |        1 |    96 |
| 12 |     7 |        1 |    86 |
| 13 |     7 |        7 |    83 |
| 14 |     8 |        4 |    57 |
| 15 |     8 |        3 |    93 |
+----+-------+----------+-------+
15 rows in set (0.00 sec)


***合并三表并查询name、score、course***
mysql> select ss.name,sc.Score,co.Course from students  ss inner join scores sc on ss.StuID=sc.StuID inner join courses co on sc.CourseID=co.CourseID;
+-------------+-------+----------------+
| name        | Score | Course         |
+-------------+-------+----------------+
| Shi Zhongyu |    77 | Kuihua Baodian |
| Shi Zhongyu |    93 | Weituo Zhang   |
| Shi Potian  |    47 | Kuihua Baodian |
| Shi Potian  |    97 | Daiyu Zanghua  |
| Xie Yanke   |    88 | Kuihua Baodian |
| Xie Yanke   |    75 | Weituo Zhang   |
| Ding Dian   |    71 | Daiyu Zanghua  |
| Ding Dian   |    89 | Kuihua Baodian |
| Yu Yutong   |    39 | Hamo Gong      |
| Yu Yutong   |    63 | Dagou Bangfa   |
| Shi Qing    |    96 | Hamo Gong      |
| Xi Ren      |    86 | Hamo Gong      |
| Xi Ren      |    83 | Dagou Bangfa   |
| Lin Daiyu   |    57 | Taiji Quan     |
| Lin Daiyu   |    93 | Jinshe Jianfa  |
+-------------+-------+----------------+
15 rows in set (0.00 sec)
```

## mysql破解密码

```
[root@Rocky8 ~]# vim /etc/my.cnf   //添加以下内容

1 [mysqld] 

10 skip-grant-tables //直接跳过安全检查
11 skip-networking  #MySQL8.0不需要

[root@Rocky8 ~]# systemctl restart mysqld.service  //重启服务
#方法1
mysql> update mysql.user set authentication_string='' where user='root' and
host='localhost'; 
#方法2
mysql> flush privileges;  //刷新权限

#再执行下面任意一个命令
mysql> alter user root@'localhost' identified by '123456';
mysql> set password for root@'localhost'='123456';

[root@centos8 ~]#vim /etc/my.cnf
[mysqld]
#skip-grant-tables
#skip-networking  #MySQL8.0不需要

[root@centos8 ~]#systemctl restart mysqld
[root@centos8 ~]#mysql -uroot -p123456l


修改密码策略等级
mysql> SHOW VARIABLES LIKE 'validate_password%';
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password.check_user_name    | ON     |
| validate_password.dictionary_file    |        |
| validate_password.length             | 8      |
| validate_password.mixed_case_count   | 1      |
| validate_password.number_count       | 1      |
| validate_password.policy             | MEDIUM |
| validate_password.special_char_count | 1      |
+--------------------------------------+--------+
7 rows in set (0.01 sec)

mysql> set global validate_password.policy=LOW;  //策略等级为LOW

Query OK, 0 rows affected (0.00 sec)
mysql> set global validate_password.length=6;    //密码长度为6
Query OK, 0 rows affected (0.00 sec)
```

## 搭建私人博客

```
[root@Rocky8 ~]#yum -y install php php-mysqlnd php-json //安装lamp环境
##下载workpress包
[root@Rocky8 ~]# wordpress-6.2.2-zh_CN.zip
[root@Rocky8 ~]#unzip wordpress-6.2.2-zh_CN.zip 
[root@Rocky8 ~]#cd wordpress/
[root@Rocky8 html]#mv /root/wordpress/* .
[root@Rocky8 html]#chown -R apache. /var/www/html/*

#设置dns
/etc/hosts
192.168.188.81 blog.dingbh.top

#创建数据库及服务账号
mysql> create database workpress;
Query OK, 1 row affected (0.00 sec)

mysql> show database;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'database' at line 1
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| workpress          |
+--------------------+
5 rows in set (0.01 sec)

mysql> create user workpress@'localhost' identified by '123456';
Query OK, 0 rows affected (0.00 sec)

mysql> grant all on workpress.* to workpress@'localhost';
Query OK, 0 rows affected (0.00 sec)

[root@Rocky8 html]#chown -R apache. /var/www/html/  //添加权限
```

## 创建表

```
mysql> create table host(id tinyint unsigned auto_increment primary key,hostname VARCHAR(256) , ip VARCHAR(256),admin VARCHAR(256),password VARCHAR(256),date timestamp DEFAULT CURRENT_TIMESTAMP NOT NULL,area  ENUM('华南','华北','华东') NOT NULL ,port int unsigned ,outNET VARCHAR(256) , inNET VARCHAR(256) );
Query OK, 0 rows affected (0.02 sec)

mysql> SHOW TABLES;
+---------------+
| Tables_in_db1 |
+---------------+
| host          |
+---------------+
1 row in set (0.00 sec)

mysql> SELECT * FROM HOST;
ERROR 1146 (42S02): Table 'db1.HOST' doesn't exist
mysql> use db1;
Database changed
mysql> select * from host;
Empty set (0.02 sec)

mysql> desc host;
+----------+----------------------------------+------+-----+-------------------+-------------------+
| Field    | Type                             | Null | Key | Default           | Extra             |
+----------+----------------------------------+------+-----+-------------------+-------------------+
| id       | tinyint unsigned                 | NO   | PRI | NULL              | auto_increment    |
| hostname | varchar(256)                     | YES  |     | NULL              |                   |
| ip       | varchar(256)                     | YES  |     | NULL              |                   |
| admin    | varchar(256)                     | YES  |     | NULL              |                   |
| password | varchar(256)                     | YES  |     | NULL              |                   |
| date     | timestamp                        | NO   |     | CURRENT_TIMESTAMP | DEFAULT_GENERATED |
| area     | enum('华南','华北','华东')       | NO   |     | NULL              |                   |
| port     | int unsigned                     | YES  |     | NULL              |                   |
| outNET   | varchar(256)                     | YES  |     | NULL              |                   |
| inNET    | varchar(256)                     | YES  |     | NULL              |                   |
+----------+----------------------------------+------+-----+-------------------+-------------------+
10 rows in set (0.01 sec)

mysql> select * from host;
Empty set (0.00 sec)

mysql> 
mysql> 
mysql> insert host(hostname,ip ,admin,password)values('shanghai','192.168.188.88','root','123456');
Query OK, 1 row affected (0.01 sec)

mysql> insert host(hostname,ip ,admin,password,port,outNET,inNET)values('shanghai','192.168.188.88','root','123456',3306,'192.168.188.2','192.168.188.254');
Query OK, 1 row affected (0.00 sec)

mysql> select * from host;
+----+----------+----------------+-------+----------+---------------------+--------+------+---------------+-----------------+
| id | hostname | ip             | admin | password | date                | area   | port | outNET        | inNET           |
+----+----------+----------------+-------+----------+---------------------+--------+------+---------------+-----------------+
|  1 | shanghai | 192.168.188.88 | root  | 123456   | 2023-07-08 19:46:00 | 华南   | NULL | NULL          | NULL            |
|  2 | shanghai | 192.168.188.88 | root  | 123456   | 2023-07-08 19:46:13 | 华南   | 3306 | 192.168.188.2 | 192.168.188.254 |
+----+----------+----------------+-------+----------+---------------------+--------+------+---------------+-----------------+
2 rows in set (0.00 sec)

```



# 面试：MyISAM和Innodb 区别

```go
MyISAM引擎主要特点如下：

1. 不支持事务，无法实现原子性、一致性等特性。
2. 不支持行级锁定，只有表级锁定，会导致并发性能较差。
3. 能够处理较大的数据表，具有高效的查询性能。
4. 不支持外键约束和崩溃恢复。

InnoDB引擎主要特点如下：

1. 支持事务，能够实现原子性、一致性、隔离性和持久性等特性。
2. 支持行级锁定，能够避免表级锁定带来的并发性能问题。
3. 对于大量读写操作的数据库，采用InnoDB更稳定。
4. 支持外键约束和崩溃恢复，能够保证数据的完整性和可靠性。
```

# 面试：ACID特性

```
A：atomicity 原子性；整个事务中的所有操作要么全部成功执行，要么全部失败后回滚
C：consistency一致性；数据库总是从一个一致性状态转换为另一个一致性状态,类似于能量守恒定
律(N50周启皓语录)
I：Isolation隔离性；一个事务所做出的操作在提交之前，是不能为其它事务所见；隔离有多种隔离
级别，实现并发
D：durability持久性；一旦事务提交，其所做的修改会永久保存于数据库中
```

## 事务的隔离级别：

```
READ UNCOMMITTED 
可读取到未提交数据，产生脏读

READ COMMITTED
可读取到提交数据，但未提交数据不可读，产生不可重复读，即可读取到多个提交数据，导致每次
读取数据不一致

REPEATABLE READ 
可重复读，多次读取数据都一致，产生幻读，即读取过程中，即使有其它提交的事务修改数据，仍
只能读取到未修改前的旧数据。此为MySQL默认设置

SERIALIZABLE 
可串行化，未提交的读事务阻塞修改事务（加读锁，但不阻塞读事务），或者未提交的修改事务阻
塞其它事务的读写（加写锁，其它事务的读，写都不可以执行）。会导致并发性能差
```



# 创建索引

```go
# 观察索引表关键字possible_keys | key

mysql> select * from students where name = "Xu Zhu";
+-------+--------+-----+--------+---------+-----------+
| StuID | Name   | Age | Gender | ClassID | TeacherID |
+-------+--------+-----+--------+---------+-----------+
|    16 | Xu Zhu |  21 | M      |       1 |      NULL |
+-------+--------+-----+--------+---------+-----------+
1 row in set (0.00 sec)

mysql> create index idx_name on students(name);
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

#创建索引
mysql> create index idx_name on students(name);
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

#查看index表，发现新索引 Key_name: idx_name
mysql> show indexes from students\G;
*************************** 1. row ***************************
        Table: students
   Non_unique: 0
     Key_name: PRIMARY
 Seq_in_index: 1
  Column_name: StuID
    Collation: A
  Cardinality: 25
     Sub_part: NULL
       Packed: NULL
         Null: 
   Index_type: BTREE
      Comment: 
Index_comment: 
      Visible: YES
   Expression: NULL
*************************** 2. row ***************************
        Table: students
   Non_unique: 1
     Key_name: idx_name
 Seq_in_index: 1
  Column_name: Name
    Collation: A
  Cardinality: 25
     Sub_part: NULL
       Packed: NULL
         Null: 
   Index_type: BTREE
      Comment: 
Index_comment: 
      Visible: YES
   Expression: NULL
2 rows in set (0.01 sec)

ERROR: 
No query specified

#验证，利用索引查询
mysql> explain select * from students where name = "Xu Zhu";
+----+-------------+----------+------------+------+---------------+----------+---------+-------+------+----------+-------+
| id | select_type | table    | partitions | type | possible_keys | key      | key_len | ref   | rows | filtered | Extra |
+----+-------------+----------+------------+------+---------------+----------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | students | NULL       | ref  | idx_name      | idx_name | 152     | const |    1 |   100.00 | NULL  |
+----+-------------+----------+------------+------+---------------+----------+---------+-------+------+----------+-------+
1 row in set, 1 warning (0.00 sec)
```

# 二进制备份（ROW）

```go
#开启行备份服务
02:22:14(root@localhost) [mysql]> show variables like 'binlog_format';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| binlog_format | ROW   |  // msyql 8.0默认为row
+---------------+-------+
1 row in set (0.00 sec)

    02:26:30(root@localhost) [mysql]> select @@log_bin;
+-----------+
| @@log_bin |
+-----------+
|         1 |
+-----------+
1 row in set (0.00 sec)


#修改my.cnf配置文件
  5 [mysqld]
  6 character-set-server=utf8mb4
  7 #transaction-isolation=READ-UNCOMMITTED
  8 #innodb_flush_log_at_trx_commit=0
  9 #innodb_log_file_size=500000000
 10 #innodb_log_files_in_group=2
 11 #innodb_log_group_home_dir=/data/tran_logs
 12 log_error=/var/log/mysql/mysqld.log
 13 general_log
 14 log_output=TABLE
 
 sql_log_bin=ON|OFF：#是否记录二进制日志，默认ON，支持动态修改，系统变量，而非服务器选项
 15 log_bin=/opt/binlog/msyql-bin  //指定二进制备份文件路径，上述两项都开启才可以



#创建binlog目录路径
[root@zabbix-server ~]# mkdir /data/binlog

#增加权限
[root@zabbix-server ~]# chown mysql . /data/binlog/


03:19:54(root@localhost) [hellodb]> update teachers set age=30 where tid=2;
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

03:20:31(root@localhost) [hellodb]> show binlog events in 'msyql-bin.000001';
+------------------+-----+----------------+-----------+-------------+--------------------------------------+
| Log_name         | Pos | Event_type     | Server_id | End_log_pos | Info                                 |
+------------------+-----+----------------+-----------+-------------+--------------------------------------+
| msyql-bin.000001 |   4 | Format_desc    |         1 |         126 | Server ver: 8.0.32, Binlog ver: 4    |
| msyql-bin.000001 | 126 | Previous_gtids |         1 |         157 |                                      |
| msyql-bin.000001 | 157 | Anonymous_Gtid |         1 |         236 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS' |
| msyql-bin.000001 | 236 | Query          |         1 |         323 | BEGIN                                |
| msyql-bin.000001 | 323 | Table_map      |         1 |         390 | table_id: 97 (hellodb.teachers)      |
| msyql-bin.000001 | 390 | Update_rows    |         1 |         466 | table_id: 97 flags: STMT_END_F       |
| msyql-bin.000001 | 466 | Xid            |         1 |         497 | COMMIT /* xid=22 */                  |
+------------------+-----+----------------+-----------+-------------+--------------------------------------+
7 rows in set (0.00 sec)

03:21:08(root@localhost) [hellodb]> show master logs;
+------------------+-----------+-----------+
| Log_name         | File_size | Encrypted |
+------------------+-----------+-----------+
| msyql-bin.000001 |       497 | No        |
+------------------+-----------+-----------+
1 row in set (0.01 sec)


#查看更改日志文件binlog  （命令：mysqlbinlog /opt/binlog/msyql-bin.000001 -v）
[root@Rocky8 binlog]#mysqlbinlog /opt/binlog/msyql-bin.000001 -v
# The proper term is pseudo_replica_mode, but we use this compatibility alias
# to make the statement usable on server versions 8.0.24 and older.
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4
#230701  3:12:45 server id 1  end_log_pos 126 CRC32 0x04a8a784 	Start: binlog v 4, server v 8.0.32 created 230701  3:12:45 at startup
# Warning: this binlog is either in use or was not closed properly.
ROLLBACK/*!*/;
BINLOG '
LSmfZA8BAAAAegAAAH4AAAABAAQAOC4wLjMyAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAtKZ9kEwANAAgAAAAABAAEAAAAYgAEGggAAAAICAgCAAAACgoKKioAEjQA
CigAAYSnqAQ=
'/*!*/;
# at 126
#230701  3:12:45 server id 1  end_log_pos 157 CRC32 0xff9d799e 	Previous-GTIDs
# [empty]
# at 157
#230701  3:20:31 server id 1  end_log_pos 236 CRC32 0x910730dc 	Anonymous_GTID	last_committed=0	sequence_number=1	rbr_only=yes	original_committed_timestamp=1688152831965916	immediate_commit_timestamp=1688152831965916	transaction_length=340
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
# original_commit_timestamp=1688152831965916 (2023-07-01 03:20:31.965916 CST)
# immediate_commit_timestamp=1688152831965916 (2023-07-01 03:20:31.965916 CST)
/*!80001 SET @@session.original_commit_timestamp=1688152831965916*//*!*/;
/*!80014 SET @@session.original_server_version=80032*//*!*/;
/*!80014 SET @@session.immediate_server_version=80032*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 236
#230701  3:20:31 server id 1  end_log_pos 323 CRC32 0xc0b3ac54 	Query	thread_id=8	exec_time=0	error_code=0
SET TIMESTAMP=1688152831/*!*/;
SET @@session.pseudo_thread_id=8/*!*/;
SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=0, @@session.unique_checks=1, @@session.autocommit=1/*!*/;
SET @@session.sql_mode=1168113696/*!*/;
SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
/*!\C utf8mb4 *//*!*/;
SET @@session.character_set_client=255,@@session.collation_connection=255,@@session.collation_server=255/*!*/;
SET @@session.lc_time_names=0/*!*/;
SET @@session.collation_database=DEFAULT/*!*/;
/*!80011 SET @@session.default_collation_for_utf8mb4=255*//*!*/;
BEGIN
/*!*/;
# at 323
#230701  3:20:31 server id 1  end_log_pos 390 CRC32 0x49fdb086 	Table_map: `hellodb`.`teachers` mapped to number 97
# has_generated_invisible_primary_key=0
# at 390
#230701  3:20:31 server id 1  end_log_pos 466 CRC32 0x96771c53 	Update_rows: table id 97 flags: STMT_END_F

BINLOG '
/yqfZBMBAAAAQwAAAIYBAAAAAGEAAAAAAAEAB2hlbGxvZGIACHRlYWNoZXJzAAQCDwH+BCwB9wEI
AQHAAgEhhrD9SQ==
/yqfZB8BAAAATAAAANIBAAAAAGEAAAAAAAEAAgAE//8AAgANAFpoYW5nIFNhbmZlbmcyAgACAA0A
WmhhbmcgU2FuZmVuZx4CUxx3lg==
'/*!*/;
### UPDATE `hellodb`.`teachers`
### WHERE
###   @1=2
###   @2='Zhang Sanfeng'
###   @3=50
###   @4=2
### SET
###   @1=2
###   @2='Zhang Sanfeng'
###   @3=30
###   @4=2
# at 466
#230701  3:20:31 server id 1  end_log_pos 497 CRC32 0xe02eb70a 	Xid = 22
COMMIT/*!*/;
# at 497
#230701  3:24:21 server id 1  end_log_pos 576 CRC32 0x3c411023 	Anonymous_GTID	last_committed=1	sequence_number=2	rbr_only=yes	original_committed_timestamp=1688153061422622	immediate_commit_timestamp=1688153061422622	transaction_length=340
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
# original_commit_timestamp=1688153061422622 (2023-07-01 03:24:21.422622 CST)
# immediate_commit_timestamp=1688153061422622 (2023-07-01 03:24:21.422622 CST)
/*!80001 SET @@session.original_commit_timestamp=1688153061422622*//*!*/;
/*!80014 SET @@session.original_server_version=80032*//*!*/;
/*!80014 SET @@session.immediate_server_version=80032*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 576
#230701  3:24:21 server id 1  end_log_pos 663 CRC32 0x824e1d83 	Query	thread_id=8	exec_time=0	error_code=0
SET TIMESTAMP=1688153061/*!*/;
BEGIN
/*!*/;
# at 663
#230701  3:24:21 server id 1  end_log_pos 730 CRC32 0xe617dc3e 	Table_map: `hellodb`.`teachers` mapped to number 97
# has_generated_invisible_primary_key=0
# at 730
#230701  3:24:21 server id 1  end_log_pos 806 CRC32 0xa3955b5f 	Update_rows: table id 97 flags: STMT_END_F

BINLOG '
5SufZBMBAAAAQwAAANoCAAAAAGEAAAAAAAEAB2hlbGxvZGIACHRlYWNoZXJzAAQCDwH+BCwB9wEI
AQHAAgEhPtwX5g==
5SufZB8BAAAATAAAACYDAAAAAGEAAAAAAAEAAgAE//8AAwANAE1pZWp1ZSBTaGl0YWkyAQADAA0A
TWllanVlIFNoaXRhaSEBX1uVow==
'/*!*/;
### UPDATE `hellodb`.`teachers`
### WHERE
###   @1=3
###   @2='Miejue Shitai'
###   @3=50
###   @4=1
### SET
###   @1=3
###   @2='Miejue Shitai'
###   @3=33
###   @4=1
# at 806
#230701  3:24:21 s		erver id 1  end_log_pos 837 CRC32 0x4027de89 	Xid = 26
COMMIT/*!*/;
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;

#按日志文件大小筛选（mysqlbinlog /opt/binlog/msyql-bin.000001 --start-position=322 --stop-position=806）
[root@Rocky8 binlog]#mysqlbinlog /opt/binlog/msyql-bin.000001 --start-position=322 --stop-position=806;
# The proper term is pseudo_replica_mode, but we use this compatibility alias
# to make the statement usable on server versions 8.0.24 and older.
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 157
#230701  3:12:45 server id 1  end_log_pos 126 CRC32 0x04a8a784 	Start: binlog v 4, server v 8.0.32 created 230701  3:12:45 at startup
# Warning: this binlog is either in use or was not closed properly.
ROLLBACK/*!*/;
BINLOG '
LSmfZA8BAAAAegAAAH4AAAABAAQAOC4wLjMyAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAtKZ9kEwANAAgAAAAABAAEAAAAYgAEGggAAAAICAgCAAAACgoKKioAEjQA
CigAAYSnqAQ=
'/*!*/;
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;


#导出二进制文件
[root@Rocky8 binlog]#mysqlbinlog /opt/binlog/msyql-bin.000001 --start-position=322 --stop-position=806 > binlog.sql
[root@Rocky8 binlog]#ll
total 12
-rw-r--r-- 1 root  root  892 Jul  1 03:40 binlog.sql
-rw-r----- 1 mysql mysql 837 Jul  1 03:24 msyql-bin.000001
-rw-r----- 1 mysql mysql  29 Jul  1 03:12 msyql-bin.index

#同步数据库文件
[root@Rocky8 binlog]#mysql -uroot -p < binlog.sql 
Enter password: 
```

## 同步binlog数据：

```
[root@Rocky8 binlog]#mysqlbinlog -R --host=192.168.188.10 --user=wang --password=123456 --raw --stop-never msyql-bin.000001;
```

# 备份数据库mysqldump

```
mysqldump 常见通用选项：
-A, --all-databases #备份所有数据库，含create database
-B, --databases db_name…  #指定备份的数据库，包括create database语句
-E, --events：#备份相关的所有event scheduler
-R, --routines：#备份所有存储过程和自定义函数
--triggers：#备份表相关触发器，默认启用,用--skip-triggers，不备份触发器
--default-character-set=utf8 #指定字符集
--master-data[=#]： #此选项须启用二进制日志
#1：所备份的数据之前加一条记录为CHANGE MASTER TO语句，非注释，不指定#，默认为1，适合于主从复
制多机使用

#2：记录为被注释的#CHANGE MASTER TO语句，适合于单机使用,适用于备份还原
#此选项会自动关闭--lock-tables功能，自动打开-x | --lock-all-tables功能（除非开启--
single-transaction）
--single-transaction
#此选项Innodb中推荐使用，不适用MyISAM，此选项会开始备份前，先执行START TRANSACTION指令开启
事务
#此选项通过在单个事务中转储所有表来创建一致的快照。 仅适用于存储在支持多版本控制的存储引擎中的表
（目前只有InnoDB可以）; 转储不保证与其他存储引擎保持一致。 在进行单事务转储时，要确保有效的转储
文件（正确的表内容和二进制日志位置），没有其他连接应该使用以下语句：ALTER TABLE，DROP TABLE，
RENAME TABLE，TRUNCATE TABLE,此选项和--lock-tables（此选项隐含提交挂起的事务）选项是相互
排斥,备份大型表时，建议将--single-transaction选项和--quick结合一起使用

-F, --flush-logs #备份前滚动日志，锁定表完成后，执行flush logs命令,生成新的二进制日志文件，
配合-A 或 -B 选项时，会导致刷新多次数据库。建议在同一时刻执行转储和日志刷新，可通过和--singletransaction或-x，--master-data 一起使用实现，此时只刷新一次二进制日志
--compact        #去掉注释，适合调试，节约备份占用的空间,生产不使用
-d, --no-data    #只备份表结构,不备份数据,即只备份create table 
-t, --no-create-info #只备份数据,不备份表结构,即不备份create table 
-n,--no-create-db #不备份create database，可被-A或-B覆盖
--flush-privileges #备份mysql或相关时需要使用
-f, --force       #忽略SQL错误，继续执行
--hex-blob        #使用十六进制符号转储二进制列，当有包括BINARY， VARBINARY，
BLOB，BIT的数据类型的列时使用，避免乱码
-q, --quick     #不缓存查询，直接输出，加快备份速度

1、mysqldump
#备份数据
[root@zabbix-server backup]# mysqldump -uroot hellodb > /data/backup/hellodb.sql
或
[root@zabbix-server backup]# mysqldump -B zabbix > /data/backup/zabbix.sql  //同--database，同时备份多个数据库，-A备份所有数据库
#还原
mysql> \. /root/hellodb.sql  //同source，source /root/hellodb.sql

```

**InnoDB****建议备份策略**

```
mysqldump -uroot -p -A -F -E -R --triggers --single-transaction --master-data=1
--flush-privileges --default-character-set=utf8 --hex-blob
>${BACKUP}/fullbak_${BACKUP_TIME}.sql
```

# mysql实现主从复制



```
主节点配置
#开启二进制日志
[root@master my.cnf.d]#vim /etc/my.cnf.d/mysql-server.cnf
[mysqld]
log_bin=/data/logbin/mysql-bin
server-id=8  #为当前节点设置一个全局惟一的ID号
log-basename=master  #可选项，设置datadir中日志名称，确保不依赖主机名

[root@master ~]#mkdir -pv /data/logbin/
[root@master ~]#chown -R mysql.mysql /data/logbin/
[root@master my.cnf.d]#systemctl status mysqld.service

注：server-id的取值范围
1 to 4294967295 (>= MariaDB 10.2.2)，默认值为1
0 to 4294967295 (<= MariaDB 10.2.1)，默认值为0，如果从节点为0，所有master都将拒绝此
slave的连接

#创建有复制权限的用户账号
mysql> create user repluser@'192.168.188.%' identified by '123456';
mysql> grant replication slave on *.* to repluser@'192.168.188.%';

[root@master ~]#mkdir /backup/
mysql> show processlist;
+----+-----------------+-----------+------+---------+------+------------------------+------------------+
| Id | User            | Host      | db   | Command | Time | State                  | Info             |
+----+-----------------+-----------+------+---------+------+------------------------+------------------+
|  5 | event_scheduler | localhost | NULL | Daemon  |  158 | Waiting on empty queue | NULL             |
|  8 | root            | localhost | NULL | Query   |    0 | init                   | show processlist |
+----+-----------------+-----------+------+---------+------+------------------------+------------------+
2 rows in set (0.00 sec)
mysql> show master logs;
+------------------+-----------+-----------+
| Log_name         | File_size | Encrypted |
+------------------+-----------+-----------+
| mysql-bin.000001 |       180 | No        |
| mysql-bin.000002 |       739 | No        |
| mysql-bin.000003 |       180 | No        |
| mysql-bin.000004 |       758 | No        |
+------------------+-----------+-----------+
4 rows in set (0.00 sec)



[root@master ~]# mysqldump -uroot -p  -A -F --single-transaction --source-data > /backup//full-`date +%F`.sql
[root@master ~]# scp /backup/full-2023-06-14.sql 192.168.188.88:/backup/


从节点配置

[root@slave1 ~]# vim /etc/my.cnf
[mysqld]
server_id=18 #为当前节点设置一个全局惟的ID号
log-bin
read_only=ON #设置数据库只读，针对supper user无效
relay_log=relay-log #relay log的文件路径，默认值hostname-relay-bin
relay_log_index=relay-log.index  #默认值hostname-relay-bin.index


#使用有复制权限的用户账号连接至主服务器，并启动复制线程
CHANGE MASTER TO MASTER_HOST='192.168.188.82', 
MASTER_USER='repluser', 
MASTER_PASSWORD='123456', 
MASTER_PORT='binlog.000002', 
MASTER_LOG_POS=691;

START SLAVE [IO_THREAD|SQL_THREAD];
SHOW SLAVE STATUS;

/*binlog.000002 |  31460881 | No*/
binlog.000009 |       197 

#导入完全备份
mysql> set sql_log_bin=0;
mysql> source /backup/full-2023-06-14.sql
mysql> set sql_log_bin=1;

#开启从节点功能
mysql> start slave;

mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for source to send event
                  Master_Host: 192.168.188.81
                  Master_User: repluser
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000004
          Read_Master_Log_Pos: 758
               Relay_Log_File: Rocky8-relay-bin.000003
                Relay_Log_Pos: 974
        Relay_Master_Log_File: mysql-bin.000004
#             Slave_IO_Running: Yes     //查看重要事项，缺一不可，缺一证明复制有问题 
#           Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 758
              Relay_Log_Space: 1354
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
#        Seconds_Behind_Master: 0      //复制延时，
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 8
                  Master_UUID: 283b5726-0a5b-11ee-a79b-000c29c01547
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Replica has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
       Master_public_key_path: 
        Get_master_public_key: 0
            Network_Namespace: 
1 row in set, 1 warning (0.01 sec)

#验证
主节点操作
mysql> delete from teachers where tid = 4 ;
Query OK, 1 row affected (0.01 sec)

mysql> select * from teachers;\
+-----+---------------+-----+--------+
| TID | Name          | Age | Gender |
+-----+---------------+-----+--------+
|   1 | Song Jiang    |  45 | M      |
|   2 | Zhang Sanfeng |  94 | M      |
|   3 | Miejue Shitai |  77 | F      |
|   5 | kaishi        |  30 | M      |
|   6 | shixian       |  40 | M      |
+-----+---------------+-----+--------+
5 rows in set (0.00 sec)

#从节点查看
(root@localhost) [hellodb]> select * from teachers;
+-----+---------------+-----+--------+
| TID | Name          | Age | Gender |
+-----+---------------+-----+--------+
|   1 | Song Jiang    |  45 | M      |
|   2 | Zhang Sanfeng |  94 | M      |
|   3 | Miejue Shitai |  77 | F      |
|   5 | kaishi        |  30 | M      |
|   6 | shixian       |  40 | M      |
+-----+---------------+-----+--------+
5 rows in set (0.00 sec)
```

主主复制

```
在主从基础之上
#88上产看
(root@localhost) [hellodb]> show master logs;
+---------------+-----------+-----------+
| Log_name      | File_size | Encrypted |
+---------------+-----------+-----------+
| binlog.000001 |       180 | No        |
| binlog.000002 |  31460881 | No        |
+---------------+-----------+-----------+
2 rows in set (0.00 sec)


#81上添加内容
mysql> CHANGE MASTER TO MASTER_HOST='192.168.188.88', 
    -> MASTER_USER='repluser', 
    -> MASTER_PASSWORD='123456', 
    -> MASTER_LOG_FILE='binlog.000002', 
    -> MASTER_LOG_POS=31460881;
Query OK, 0 rows affected, 8 warnings (0.02 sec)

mysql> start slave;
Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql> show slave status\G;
```

## GTID复制

```
主节点
[mysqld]
	log_bin
	server-id=8
	gtid_mode=ON
	enforce_gtid_consistency  
	
#创建有复制权限的用户账号
mysql> create user repluser@'192.168.188.%' identified by '123456';
mysql> grant replication slave on *.* to repluser@'192.168.188.%';

#主从事务保持一致
[root@master mysql]# mysqldump -uroot -p -A -F --single-transaction --source-data=1 --all-databases --triggers --routines --events  > /data/full-`date +%F`.sql
Enter password: 


从节点
[mysqld]
	log_bin
	server-id=18
	gtid_mode=ON
	enforce_gtid_consistency  

#还原主完全备份
mysql> source /data/full-2023-07-07.sql
Query OK, 0 rows affected (0.00 sec)
...


CHANGE MASTER TO 
MASTER_HOST='192.168.188.82', 
MASTER_USER='supluser', 
MASTER_PASSWORD='123456', 
MASTER_PORT=3306, 
MASTER_AUTO_POSITION=1;

新版：
CHANGE REPLICATION SOURCE TO
SOURCE_HOST='192.168.188.82',
SOURCE_USER='supluser',
SOURCE_PASSWORD='123456',
SOURCE_PORT=3306,
GET_SOURCE_PUBLIC_KEY=1,
SOURCE_AUTO_POSITION=1;

mysql> CHANGE REPLICATION SOURCE TO
    -> SOURCE_HOST='192.168.188.82',
    -> SOURCE_USER='supluser',
    -> SOURCE_PASSWORD='123456',
    -> SOURCE_PORT=3306,
    -> GET_SOURCE_PUBLIC_KEY=1,
    -> SOURCE_AUTO_POSITION=1;
Query OK, 0 rows affected, 2 warnings (0.01 sec)

mysql> show warnings;
+-------+------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Level | Code | Message                                                                                                                                                                                                                                                                              |
+-------+------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Note  | 1759 | Sending passwords in plain text without SSL/TLS is extremely insecure.                                                                                                                                                                                                               |
| Note  | 1760 | Storing MySQL user name or password information in the master info repository is not secure and is therefore not recommended. Please consider using the USER and PASSWORD connection options for START SLAVE; see the 'START SLAVE Syntax' in the MySQL Manual for more information. |
+-------+------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)

mysql> start replica;
Query OK, 0 rows affected (0.01 sec)

mysql> show replica status\G;
*************************** 1. row ***************************
             Replica_IO_State: Waiting for source to send event
                  Source_Host: 192.168.188.82
                  Source_User: supluser
                  Source_Port: 3306
                Connect_Retry: 60
              Source_Log_File: binlog.000004
          Read_Source_Log_Pos: 157
               Relay_Log_File: slave1-relay-bin.000002
                Relay_Log_Pos: 367
        Relay_Source_Log_File: binlog.000004
           Replica_IO_Running: Yes  //两项都是yes
          Replica_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Source_Log_Pos: 157
              Relay_Log_Space: 578
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Source_SSL_Allowed: No
           Source_SSL_CA_File: 
           Source_SSL_CA_Path: 
              Source_SSL_Cert: 
            Source_SSL_Cipher: 
               Source_SSL_Key: 
        Seconds_Behind_Source: 0    //复制延时时间
Source_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Source_Server_Id: 8
                  Source_UUID: 72fb0803-1cb7-11ee-b488-000c29c51569
             Source_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
    Replica_SQL_Running_State: Replica has read all relay log; waiting for more updates
           Source_Retry_Count: 86400
                  Source_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Source_SSL_Crl: 
           Source_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 939a2d30-0a79-11ee-8276-000c295a81b1:1-2,
e16d470a-1cb3-11ee-8d1e-000c29c51569:1-2
                Auto_Position: 1
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Source_TLS_Version: 
       Source_public_key_path: 
        Get_Source_public_key: 1
            Network_Namespace: 
1 row in set (0.00 sec)

ERROR: 
No query specified
```

## 面试：**造成主从不一致的原因**

```
1、主库binlog格式为Statement，同步到从库执行后可能造成主从不一致。

2、主库执行更改前有执行set sql_log_bin=0，会使主库不记录binlog，从库也无法变更这部分数据。

3、从节点未设置只读，误操作写入数据

4、主库或从库意外宕机，宕机可能会造成binlog或者relaylog文件出现损坏，导致主从不一致

5、主从实例版本不一致，特别是高版本是主，低版本为从的情况下，主数据库上面支持的功能，从数据库上面可能不支持该功能。

6、主从sql_mode不一致

7、MySQL自身bug导致
```

重点：

1、主从复制的工作原理？

2、造成主从不一致的原因？

解决主从server-uuid一致的问题

```
mysql> select uuid();  //生成uuid
+--------------------------------------+
| uuid()                               |
+--------------------------------------+
| 7b370c10-1cb4-11ee-b638-000c29c51569 |
+--------------------------------------+
1 row in set (0.00 sec)


[root@master ~]# vim /data/mysql/auto.cnf
  1 [auto]
  2 #server-uuid=939a2d30-0a79-11ee-8276-000c295a81b1
  3 server-uuid=e16d470a-1cb3-11ee-8d1e-000c29c51569  
```

## mycat实现

```
[root@mycat ~]# vim /etc/profile.d/mycat.sh 
#!/bin/bash
PATH=/apps/mycat/bin/:$PATH  //添加变量




[root@mycat ~]# mysql -uroot -p -h 192.168.188.82 -P3306
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 18
Server version: 8.0.28 MySQL Community Server - GPL

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| db1                |
| db2                |
| hellodb            |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
7 rows in set (0.00 sec)

```





```
CHANGE MASTER TO MASTER_HOST='192.168.188.82', 
MASTER_USER='repluser', 
MASTER_PASSWORD='123456', 
MASTER_LOG_FILE='binlog.000002', 
MASTER_LOG_POS=691;
```

## mha实现

### 1、安装管理端软件，管理端需安装管理端、node软件，主从主机只需安装node软件

```
#安装管理端软件，管理端需安装管理端、node软件，主从主机只需安装node软件
[root@centos7 ~]# yum -y install mha4mysql-manager-0.58-0.el7.centos.noarch.rpm  mha4mysql-node-0.58-0.el7.centos.noarch.rpm

#生成ssh-key
[root@centos7 ~]# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:9vPuWqvnBLZs9lsPIYGuzpGeBhC0j6k/epevW6PoXHk root@centos7
The key's randomart image is:
+---[RSA 2048]----+
|   ..            |
|    ..     .     |
|    ..    . .    |
|    .+   .   .   |
|    o.. S + . .  |
|   .  .o * o . . |
|  .   ooE B o o  |
|   ooooB.* *.o o |
|  .+=o+=* oBO.  .|
+----[SHA256]-----+

[root@centos7 ~]# ls .ssh
authorized_keys  id_rsa  id_rsa.pub  known_hosts

#将验证ssh-key拷贝到其他主机
[root@centos7 ~]# rsync -a .ssh 192.168.188.82:/root/
root@192.168.188.82's password: 
[root@centos7 ~]# rsync -a .ssh 192.168.188.83:/root/
root@192.168.188.83's password: 
[root@centos7 ~]# rsync -a .ssh 192.168.188.84:/root/
root@192.168.188.84's password: 
```

### 2、管理端建立配置文件

```
[root@mha-manager ~]#mkdir /etc/mastermha/
[root@mha-manager ~]#vim /etc/mastermha/app1.cnf


mysql> create user mhauser@'192.168.188.%' identified by '123456';
Query OK, 0 rows affected (0.00 sec)

mysql> grant all on *.* to mhauser@'192.168.188.%';
Query OK, 0 rows affected (0.01 sec)
 
[root@centos7 bin]# rz -E
rz waiting to receive.
[root@centos7 bin]# ls
master_ip_failover
[root@centos7 bin]# chmod +x master_ip_failover 
[root@centos7 bin]# vim master_ip_failover 


[root@master date]#ifconfig eth0:1 192.168.188.100/24
[root@master date]#ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:3c:92:06 brd ff:ff:ff:ff:ff:ff
    altname enp3s0
    altname ens160
    inet 192.168.188.82/24 brd 192.168.188.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
    inet 192.168.188.100/24 brd 192.168.188.255 scope global secondary eth0:1
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe3c:9206/64 scope link 
       valid_lft forever preferred_lft forever


```

##### 修改二进制日志位置不一致问题

```
主从主机mysqld配置文件内容
mkdir -pv /data/mysql/    #二进制文件路径

chown mysql.mysql /data/mysql/
cat >> /etc/my.cnf.d/mysql-server.cnf <<EOF
log-bin=/data/mysql/mysql-bin
skip_name_resolve=1
general_log
relay_log_purge=0
EOF
systemctl restart mysqld.service

CHANGE MASTER TO MASTER_HOST='192.168.188.83', 
MASTER_USER='repluser', 
MASTER_PASSWORD='123456', 
MASTER_LOG_FILE='mysql-bin.000004', 
MASTER_LOG_POS=1204874;
mysql> stop slave ;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql-bin.000005 |       156
mysql-bin.000004 |   1204874

mysql> reset slave all;
Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql> CHANGE MASTER TO MASTER_HOST='192.168.188.82', 
    -> MASTER_USER='repluser', 
    -> MASTER_PASSWORD='123456', 
    -> MASTER_LOG_FILE='mysql-bin.000001', 
    -> MASTER_LOG_POS=156;
Query OK, 0 rows affected, 8 warnings (0.01 sec)

mysql> start slave;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for source to send event
                  Master_Host: 192.168.188.82
                  Master_User: repluser
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 156
               Relay_Log_File: slave2-relay-bin.000002
                Relay_Log_Pos: 324
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 156
              Relay_Log_Space: 534
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 8
                  Master_UUID: 7f681a2e-1d4e-11ee-8bc2-000c293c9206
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Replica has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
       Master_public_key_path: 
        Get_master_public_key: 0
            Network_Namespace: 
1 row in set, 1 warning (0.00 sec)

ERROR: 
No query specified

```

```
#mha-manager主机状态

[root@mha-manager mastermha]# masterha_check_ssh --conf=/etc/mastermha/app1.cnf
Sun Jul  9 18:52:56 2023 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Sun Jul  9 18:52:56 2023 - [info] Reading application default configuration from /etc/mastermha/app1.cnf..
Sun Jul  9 18:52:56 2023 - [info] Reading server configuration from /etc/mastermha/app1.cnf..
Sun Jul  9 18:52:56 2023 - [info] Starting SSH connection tests..
Sun Jul  9 18:52:57 2023 - [debug] 
Sun Jul  9 18:52:56 2023 - [debug]  Connecting via SSH from root@192.168.188.84(192.168.188.84:22) to root@192.168.188.83(192.168.188.83:22)..
Sun Jul  9 18:52:56 2023 - [debug]   ok.
Sun Jul  9 18:52:56 2023 - [debug]  Connecting via SSH from root@192.168.188.84(192.168.188.84:22) to root@192.168.188.82(192.168.188.82:22)..
Sun Jul  9 18:52:57 2023 - [debug]   ok.
Sun Jul  9 18:52:57 2023 - [debug] 
Sun Jul  9 18:52:56 2023 - [debug]  Connecting via SSH from root@192.168.188.83(192.168.188.83:22) to root@192.168.188.84(192.168.188.84:22)..
Sun Jul  9 18:52:57 2023 - [debug]   ok.
Sun Jul  9 18:52:57 2023 - [debug]  Connecting via SSH from root@192.168.188.83(192.168.188.83:22) to root@192.168.188.82(192.168.188.82:22)..
Sun Jul  9 18:52:57 2023 - [debug]   ok.
Sun Jul  9 18:52:58 2023 - [debug] 
Sun Jul  9 18:52:57 2023 - [debug]  Connecting via SSH from root@192.168.188.82(192.168.188.82:22) to root@192.168.188.84(192.168.188.84:22)..
Sun Jul  9 18:52:57 2023 - [debug]   ok.
Sun Jul  9 18:52:57 2023 - [debug]  Connecting via SSH from root@192.168.188.82(192.168.188.82:22) to root@192.168.188.83(192.168.188.83:22)..
Sun Jul  9 18:52:58 2023 - [debug]   ok.
Sun Jul  9 18:52:58 2023 - [info] All SSH connection tests passed successfully.
[root@mha-manager mastermha]# masterha_check_repl --conf=/etc/mastermha/app1.cnf

[root@mha-manager app1]# masterha_check_status --conf=/etc/mastermha/app1.cnf
app1 (pid:7774) is running(0:PING_OK), master:192.168.188.82




[root@master data]#tail -f /var/lib/mysql/master.log 
2023-07-09T01:06:14.305520Z	   34 Query	SELECT 1 As Value   //探测活跃度
2023-07-09T01:06:15.306934Z	   34 Query	SELECT 1 As Value
2023-07-09T01:06:16.307417Z	   34 Query	SELECT 1 As Value
2023-07-09T01:06:17.308165Z	   34 Query	SELECT 1 As Value
```

模拟故障

```

[root@master ~]#systemctl stop mysqld.service
[root@master ~]#

[root@mha-manager mastermha]# masterha_manager --conf=/etc/mastermha/app1.cnf
Sun Jul  9 19:00:38 2023 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Sun Jul  9 19:00:38 2023 - [info] Reading application default configuration from /etc/mastermha/app1.cnf..
Sun Jul  9 19:00:38 2023 - [info] Reading server configuration from /etc/mastermha/app1.cnf..

  Creating /data/mastermha/app1 if not exists..    ok.
  Checking output directory is accessible or not..
   ok.
  Binlog found at /data/mysql/, up to mysql-bin.000008
Sun Jul  9 19:24:03 2023 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Sun Jul  9 19:24:03 2023 - [info] Reading application default configuration from /etc/mastermha/app1.cnf..
Sun Jul  9 19:24:03 2023 - [info] Reading server configuration from /etc/mastermha/app1.cnf..


----- Failover Report -----

app1: MySQL Master failover 192.168.188.82(192.168.188.82:3306) to 192.168.188.84(192.168.188.84:3306) succeeded

Master 192.168.188.82(192.168.188.82:3306) is down!

Check MHA Manager logs at mha-manager:/data/mastermha/app1/manager.log for details.

[root@mha-manager ~]# cat /data/mastermha/app1/manager.log 
Started automated(non-interactive) failover.
Invalidated master IP address on 192.168.188.82(192.168.188.82:3306)
The latest slave 192.168.188.84(192.168.188.84:3306) has all relay logs for recovery.
Selected 192.168.188.84(192.168.188.84:3306) as a new master.
192.168.188.84(192.168.188.84:3306): OK: Applying all logs succeeded.
192.168.188.84(192.168.188.84:3306): OK: Activated master IP address.
192.168.188.83(192.168.188.83:3306): This host has the latest relay log events.
Generating relay diff files from the latest slave succeeded.
192.168.188.83(192.168.188.83:3306): OK: Applying all logs succeeded. Slave started, replicating from 192.168.188.84(192.168.188.84:3306)
192.168.188.84(192.168.188.84:3306): Resetting slave info succeeded.
Master failover to 192.168.188.84(192.168.188.84:3306) completed successfully.

[root@mha-manager ~]# masterha_check_status --conf=/etc/mastermha/app1.cnf  
app1 is stopped(2:NOT_RUNNING).


#验证VIP漂移至新的Master上
[root@slave2 ~]#ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.188.84  netmask 255.255.255.0  broadcast 192.168.188.255
        inet6 fe80::20c:29ff:fe07:de70  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:07:de:70  txqueuelen 1000  (Ethernet)
        RX packets 26929  bytes 6399805 (6.1 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 14157  bytes 2201837 (2.0 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth0:1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.188.100  netmask 255.255.255.0  broadcast 192.168.188.255
        ether 00:0c:29:07:de:70  txqueuelen 1000  (Ethernet)

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 602  bytes 89696 (87.5 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 602  bytes 89696 (87.5 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0



#如果再次运行MHA,需要先删除下面文件
[root@mha-manager ~]# ls /data/mastermha/app1/app1.failover.complete -l
-rw-r--r-- 1 root root 0 Jul  9 19:24 /data/mastermha/app1/app1.failover.complete
[root@mha-manager ~]#rm -f /data/mastermha/app1/app1.failover.complete
```

## PXC实现

```
192.168.188.7
192.168.188.27
192.168.188.37
192.168.188.47

#配置yum软件源
cat > /etc/yum.repos.d/pxc.repo  <<EOF
[percona]
name=percona_repo
baseurl=https://mirrors.tuna.tsinghua.edu.cn/percona/release/\$releasever\/RPMS/\$basearch\/
enabled= 1
gpgcheck= 0
EOF


#在三个节点都安装好PXC 5.7 
[root@pxc1 ~]#yum install Percona-XtraDB-Cluster-57 -y
[root@pxc2 ~]#yum install Percona-XtraDB-Cluster-57 -y
[root@pxc3 ~]#yum install Percona-XtraDB-Cluster-57 -y

192.168.188.7,192.168.188.27,192.168.188.37


[root@pxc1 yum.repos.d]# vim /etc/percona-xtradb-cluster.conf.d/wsrep.cnf

  8 wsrep_cluster_address=gcomm://192.168.188.7,192.168.188.27,192.168.188.37
 27 wsrep_cluster_name=pxc-cluster-Ding
 25 wsrep_node_address=192.168.188.7
 30 wsrep_node_name=pxc-cluster-node-1
 39 wsrep_sst_auth="sstuser:s3cretPass"


[root@pxc1 ~]# systemctl start mysql@bootstrap.service  //仅适用于主机
[root@pxc1 ~]# ss -ntl
State       Recv-Q Send-Q                                       Local Address:Port                                                      Peer Address:Port              
LISTEN      0      128                                                      *:22                                                                   *:*                  
LISTEN      0      128                                                      *:4567                                                                 *:*                  
LISTEN      0      100                                              127.0.0.1:25                                                                   *:*                  
LISTEN      0      128                                                   [::]:22                                                                [::]:*                  
LISTEN      0      100                                                  [::1]:25                                                                [::]:*                  
LISTEN      0      80                                                    [::]:3306                                                              [::]:*                  
[root@pxc1 ~]# cat /var/log/mysqld.log |grep password
2023-07-10T05:35:32.764874Z 1 [Note] A temporary password is generated for root@localhost: ixg/xPFUI6L/


#修改root密码
mysql> alter user 'root'@'localhost' identified by '123456';
Query OK, 0 rows affected (0.01 sec)
#创建相关用户并授权
mysql> CREATE USER 'sstuser'@'localhost' IDENTIFIED BY 's3cretPass';
Query OK, 0 rows affected (0.00 sec)
mysql> GRANT RELOAD, LOCK TABLES, PROCESS, REPLICATION CLIENT ON *.* TO 
'sstuser'@'localhost';
Query OK, 0 rows affected (0.01 sec)
#查看相关变量
mysql> SHOW VARIABLES LIKE 'wsrep%'\G

#查看当前活动节点
mysql> SHOW STATUS LIKE 'wsrep_cluster_size'\G
*************************** 1. row ***************************
Variable_name: wsrep_cluster_size
        Value: 3
1 row in set (0.00 sec)


#加新节点
[root@pxc4 ~]# yum install Percona-XtraDB-Cluster-57 -y
[root@pxc4 ~]# vim /etc/percona-xtradb-cluster.conf.d/wsrep.cnf
[root@pxc4 ~]# systemctl start mysql

```

# openvpn实现



## 1、前期准备

```
1、OpenVPN服务器端
[root@openvpn-server ~]#yum -y install openvpn 
#证书管理工具
[root@openvpn-server ~]#yum -y install easy-rsa


#查看openvpn包中相关文件
[root@openvpn-server ~]#rpm -ql openvpn
/etc/openvpn
/etc/openvpn/client
/etc/openvpn/server
/run/openvpn-client
/run/openvpn-server
/usr/lib/.build-id
/usr/lib/.build-id/66
/usr/lib/.build-id/66/bd0dab2368dc0d844282225cb7f20f1db4bd9b
/usr/lib/.build-id/9e
/usr/lib/.build-id/9e/360159708bfe37bf6bbae0fa9facffbd2556dc
/usr/lib/.build-id/ca
/usr/lib/.build-id/ca/29127991f2fbcd366ca4d99df93d6d333eebcd
/usr/lib/systemd/system/openvpn-client@.service
/usr/lib/systemd/system/openvpn-server@.service
/usr/lib/tmpfiles.d/openvpn.conf
/usr/lib64/openvpn
/usr/lib64/openvpn/plugins
/usr/lib64/openvpn/plugins/openvpn-plugin-auth-pam.so
/usr/lib64/openvpn/plugins/openvpn-plugin-down-root.so
/usr/sbin/openvpn
/usr/share/doc/openvpn
/usr/share/doc/openvpn/AUTHORS
/usr/share/doc/openvpn/COPYING
/usr/share/doc/openvpn/COPYRIGHT.GPL
/usr/share/doc/openvpn/ChangeLog
/usr/share/doc/openvpn/Changes.rst
/usr/share/doc/openvpn/README
/usr/share/doc/openvpn/README.auth-pam
/usr/share/doc/openvpn/README.down-root
/usr/share/doc/openvpn/README.systemd
/usr/share/doc/openvpn/contrib
/usr/share/doc/openvpn/contrib/OCSP_check
/usr/share/doc/openvpn/contrib/OCSP_check/OCSP_check.sh
/usr/share/doc/openvpn/contrib/README
/usr/share/doc/openvpn/contrib/openvpn-fwmarkroute-1.00
/usr/share/doc/openvpn/contrib/openvpn-fwmarkroute-1.00/README
/usr/share/doc/openvpn/contrib/openvpn-fwmarkroute-1.00/fwmarkroute.down
/usr/share/doc/openvpn/contrib/openvpn-fwmarkroute-1.00/fwmarkroute.up
/usr/share/doc/openvpn/contrib/pull-resolv-conf
/usr/share/doc/openvpn/contrib/pull-resolv-conf/client.down
/usr/share/doc/openvpn/contrib/pull-resolv-conf/client.up
/usr/share/doc/openvpn/management-notes.txt
/usr/share/doc/openvpn/sample
/usr/share/doc/openvpn/sample/sample-config-files
/usr/share/doc/openvpn/sample/sample-config-files/README
/usr/share/doc/openvpn/sample/sample-config-files/client.conf
/usr/share/doc/openvpn/sample/sample-config-files/firewall.sh
/usr/share/doc/openvpn/sample/sample-config-files/home.up
/usr/share/doc/openvpn/sample/sample-config-files/loopback-client
/usr/share/doc/openvpn/sample/sample-config-files/loopback-server
/usr/share/doc/openvpn/sample/sample-config-files/office.up
/usr/share/doc/openvpn/sample/sample-config-files/openvpn-shutdown.sh
/usr/share/doc/openvpn/sample/sample-config-files/openvpn-startup.sh
/usr/share/doc/openvpn/sample/sample-config-files/roadwarrior-client.conf
/usr/share/doc/openvpn/sample/sample-config-files/roadwarrior-server.conf
/usr/share/doc/openvpn/sample/sample-config-files/server.conf
/usr/share/doc/openvpn/sample/sample-config-files/static-home.conf
/usr/share/doc/openvpn/sample/sample-config-files/static-office.conf
/usr/share/doc/openvpn/sample/sample-config-files/tls-home.conf
/usr/share/doc/openvpn/sample/sample-config-files/tls-office.conf
/usr/share/doc/openvpn/sample/sample-config-files/xinetd-client-config
/usr/share/doc/openvpn/sample/sample-config-files/xinetd-server-config
/usr/share/doc/openvpn/sample/sample-scripts
/usr/share/doc/openvpn/sample/sample-scripts/auth-pam.pl
/usr/share/doc/openvpn/sample/sample-scripts/bridge-start
/usr/share/doc/openvpn/sample/sample-scripts/bridge-stop
/usr/share/doc/openvpn/sample/sample-scripts/ucn.pl
/usr/share/doc/openvpn/sample/sample-scripts/verify-cn
/usr/share/doc/openvpn/sample/sample-windows
/usr/share/doc/openvpn/sample/sample-windows/sample.ovpn
/usr/share/man/man8/openvpn.8.gz
/var/lib/openvpn

查看easy-rsa包中相关文件
[root@openvpn-server 3]#rpm -ql easy-rsa
/usr/share/doc/easy-rsa
/usr/share/doc/easy-rsa/COPYING.md
/usr/share/doc/easy-rsa/ChangeLog
/usr/share/doc/easy-rsa/README.md
/usr/share/doc/easy-rsa/README.quickstart.md
/usr/share/doc/easy-rsa/vars.example
/usr/share/easy-rsa
/usr/share/easy-rsa/3
/usr/share/easy-rsa/3.0
/usr/share/easy-rsa/3.0.8
/usr/share/easy-rsa/3.0.8/easyrsa
/usr/share/easy-rsa/3.0.8/openssl-easyrsa.cnf
/usr/share/easy-rsa/3.0.8/x509-types
/usr/share/easy-rsa/3.0.8/x509-types/COMMON
/usr/share/easy-rsa/3.0.8/x509-types/ca
/usr/share/easy-rsa/3.0.8/x509-types/client
/usr/share/easy-rsa/3.0.8/x509-types/code-signing
/usr/share/easy-rsa/3.0.8/x509-types/email
/usr/share/easy-rsa/3.0.8/x509-types/kdc
/usr/share/easy-rsa/3.0.8/x509-types/server
/usr/share/easy-rsa/3.0.8/x509-types/serverClient
/usr/share/licenses/easy-rsa
/usr/share/licenses/easy-rsa/gpl-2.0.txt



set_var EASYRSA_CA_EXPIRE       36500

# In how many days should certificates expire?

set_var EASYRSA_CERT_EXPIRE     8250
```

## 2、准备相关配置文件

```
#生成服务器配置文件
[root@openvpn-server ~]#cp /usr/share/doc/openvpn/sample/sample-config-files/server.conf 
/etc/openvpn/
#准备证书签发相关文件
[root@openvpn-server ~]#cp -r /usr/share/easy-rsa/ /etc/openvpn/easy-rsa-server
#准备签发证书相关变量的配置文件
[root@openvpn-server ~]#cp /usr/share/doc/easy-rsa/vars.example /etc/openvpn/easy-rsaserver/3/vars
#建议修改给CA和OpenVPN服务器颁发的证书的有效期,可适当加长
[root@openvpn-server ~]#vim /etc/openvpn/easy-rsa-server/3/vars
#CA的证书有效期默为为10年,可以适当延长,比如:36500天
#set_var EASYRSA_CA_EXPIRE     3650
set_var EASYRSA_CA_EXPIRE      36500
#服务器证书默为为825天,可适当加长,比如:3650天
#set_var EASYRSA_CERT_EXPIRE   825 
#将上面行修改为下面
set_var EASYRSA_CERT_EXPIRE    3650
[root@openvpn-server ~]#tree /etc/openvpn/
/etc/openvpn/
├── client
├── easy-rsa-server
│   ├── 3 -> 3.0.7
│   ├── 3.0 -> 3.0.7
│   └── 3.0.7
│       ├── easyrsa
│       ├── openssl-easyrsa.cnf
│       ├── vars
│       └── x509-types
│           ├── ca
│           ├── client
│           ├── code-signing
│           ├── COMMON
│           ├── email
│           ├── kdc
│           ├── server
│           └── serverClient
├── server
└── server.conf
7 directories,
```

## 3、准备证书相关文件

### 3.1、初始化PKI和CA签发机构环境

#### **3.1.1** 脚本easyrsa帮助用法**

```
[root@openvpn-server ~]#cd /etc/openvpn/easy-rsa-server/3/
[root@openvpn-server 3]#pwd
/etc/openvpn/easy-rsa-server/3
[root@openvpn-server 3]#file ./easyrsa
./easyrsa: POSIX shell script, ASCII text executable

#easy-rsa用法帮助
[root@openvpn-server 3]#./easyrsa 

Note: using Easy-RSA configuration from: /etc/openvpn/easy-rsa-server/3.0.8/vars

Easy-RSA 3 usage and overview

USAGE: easyrsa [options] COMMAND [command-options]

A list of commands is shown below. To get detailed usage and help for a
command, run:
  ./easyrsa help COMMAND

For a listing of options that can be supplied before the command, use:
  ./easyrsa help options

Here is the list of commands available with a short syntax reminder. Use the
'help' command above to get full usage details.

  init-pki
  build-ca [ cmd-opts ]
  gen-dh
  gen-req <filename_base> [ cmd-opts ]
  sign-req <type> <filename_base>
  build-client-full <filename_base> [ cmd-opts ]
  build-server-full <filename_base> [ cmd-opts ]
  revoke <filename_base> [cmd-opts]
  renew <filename_base> [cmd-opts]
  build-serverClient-full <filename_base> [ cmd-opts ]
  gen-crl
  update-db
  show-req <filename_base> [ cmd-opts ]
  show-cert <filename_base> [ cmd-opts ]
  show-ca [ cmd-opts ]
  import-req <request_file_path> <short_basename>
  export-p7 <filename_base> [ cmd-opts ]
  export-p8 <filename_base> [ cmd-opts ]
  export-p12 <filename_base> [ cmd-opts ]
  set-rsa-pass <filename_base> [ cmd-opts ]
  set-ec-pass <filename_base> [ cmd-opts ]
  upgrade <type>

DIRECTORY STATUS (commands would take effect on these locations)
  EASYRSA: /etc/openvpn/easy-rsa-server/3.0.8
      PKI: /etc/openvpn/easy-rsa-server/3/pki

```

#### **3.1.2** **初始化****PKI****生成****PKI****相关目录和文件**

```
[root@openvpn-server ~]#cd /etc/openvpn/easy-rsa-server/3/
[root@openvpn-server 3]#pwd
/etc/openvpn/easy-rsa-server/3
[root@openvpn-server 3]#ls
easyrsa  openssl-easyrsa.cnf  vars  x509-types


#初始化数据,在当前目录下生成pki目录及相关文件
[root@openvpn-server 3]#./easyrsa init-pki

Note: using Easy-RSA configuration from: /etc/openvpn/easy-rsa-server/3.0.8/vars

init-pki complete; you may now create a CA or requests.
Your newly created PKI dir is: /etc/openvpn/easy-rsa-server/3/pki


[root@openvpn-server 3]#ls
easyrsa  openssl-easyrsa.cnf  pki  vars  x509-types
[root@openvpn-server 3]#tree
.
├── easyrsa
├── openssl-easyrsa.cnf
├── pki
│   ├── openssl-easyrsa.cnf
│   ├── private
│   ├── reqs
│   └── safessl-easyrsa.cnf
├── vars
└── x509-types
    ├── ca
    ├── client
    ├── code-signing
    ├── COMMON
    ├── email
    ├── kdc
    ├── server
    └── serverClient

4 directories, 13 files

```

### 3.2 创建CA机构

```
[root@openvpn-server ~]#cd /etc/openvpn/easy-rsa-server/3
[root@openvpn-server 3]#tree pki
pki
├── openssl-easyrsa.cnf
├── private
├── reqs
└── safessl-easyrsa.cnf
2 directories, 2 files

[root@openvpn-server 3]#./easyrsa build-ca nopass

Note: using Easy-RSA configuration from: /etc/openvpn/easy-rsa-server/3.0.8/vars
Using SSL: openssl OpenSSL 1.1.1k  FIPS 25 Mar 2021
Generating RSA private key, 2048 bit long modulus (2 primes)
.......................+++++
...................................................+++++
e is 65537 (0x010001)
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [Easy-RSA CA]: #接受默认值,直接回
车

CA creation complete and you may now import and sign cert requests.
Your new CA certificate file for publishing is at:
/etc/openvpn/easy-rsa-server/3/pki/ca.crt    #生成自签名的证书文件


[root@openvpn-server 3]#tree pki
pki
├── ca.crt         #生成自签名的证书文件
├── certs_by_serial
├── index.txt
├── index.txt.attr
├── issued
├── openssl-easyrsa.cnf
├── private
│   └── ca.key     #生成私钥文件
├── renewed
│   ├── certs_by_serial
│   ├── private_by_serial
│   └── reqs_by_serial
├── reqs
├── revoked
│   ├── certs_by_serial
│   ├── private_by_serial
│   └── reqs_by_serial
├── safessl-easyrsa.cnf
└── serial

#生成CA相关的文件
[root@openvpn-server 3]#cat pki/serial 
01
[root@openvpn-server 3]#ll pki/index.txt
-rw------- 1 root root 0 Aug  2 16:42 pki/index.txt
[root@openvpn-server 3]#cat pki/serial 
01
[root@openvpn-server 3]#ll pki/ca.crt pki/private/ca.key 
-rw------- 1 root root 1204 Aug  2 16:42 pki/ca.crt
-rw------- 1 root root 1675 Aug  2 16:42 pki/private/ca.key
#查看生成的自签名证书
[root@openvpn-server 3]#cat pki/ca.crt 
-----BEGIN CERTIFICATE-----
MIIDTTCCAjWgAwIBAgIUdNALrWtH8/psCo/dbEdn8f5ZnhQwDQYJKoZIhvcNAQEL
BQAwFjEUMBIGA1UEAwwLRWFzeS1SU0EgQ0EwIBcNMjMwNjE0MDYyNzEwWhgPMjEy
MzA1MjEwNjI3MTBaMBYxFDASBgNVBAMMC0Vhc3ktUlNBIENBMIIBIjANBgkqhkiG
9w0BAQEFAAOCAQ8AMIIBCgKCAQEA1iYicCSe5g1Fqmqzt1PKNdQuiAolc7U/oXky
jQRR1B1TT8mj7OMl61qYhDH0XBajjGHKvK/RFYRPHo22Q8LjIpozrOU62kqR1atV
Wknxa3WVmL2EsOlsxwO8AUnaxbhIG2CtRNpIE7/XBReK3q/IChwev+3WoYWz7Cpy
g7XBxlhzBiMCuTA05LWQUEfdb4EmxxKKM417rMvgeXrGxSB0341F4iTfconJ3YHs
S6sGETWggWRZDnr+ugynRDypw03CUtHCy8axjdZ/WfiIv8IabcyPIL89G9IsijwT
kzUJ7A19tvKrO2mTj1AS68Hqbx2DYIUtgGupr31ECqHFKALi5QIDAQABo4GQMIGN
MB0GA1UdDgQWBBRV//dVODODalBx2hgNL3St3hctIzBRBgNVHSMESjBIgBRV//dV
ODODalBx2hgNL3St3hctI6EapBgwFjEUMBIGA1UEAwwLRWFzeS1SU0EgQ0GCFHTQ
C61rR/P6bAqP3WxHZ/H+WZ4UMAwGA1UdEwQFMAMBAf8wCwYDVR0PBAQDAgEGMA0G
CSqGSIb3DQEBCwUAA4IBAQBqYrWHhd/bYhKneRwAF7Apw6W2b+MlkcnX8jXoe7Zw
MJf/gapZXZQBSEHa02OflOMD/49rk++4IsmoDikJOWNOCCETEIHd2PX9PzgtRJ85
FPsfK/vp8wlBSOSBmB2Jsi8UxOZt7Azj7TJKy4cZE5U09N8HFCd08+i2RT1mzamc
syTg0qz0oWT5FSbHO7FI7dYq7Yek4lbQ76VirunCfqyiN9Vx2zG50tDAdEe7WGVx
Bn59gTxAKr6oqQcKk5aAQ0SzZPExdEuUiG9fuPjS8fnuBO8sfO+dAWjdf5blB303
O3jlQLzg/Wk/CQ23IPl788/itQFzKugSVXxnar+j4eB5
-----END CERTIFICATE-----

```

### 3.3 **创建服务端证书申请**

```
[root@openvpn-server ~]#cd /etc/openvpn/easy-rsa-server/3
[root@openvpn-server 3]#pwd
/etc/openvpn/easy-rsa-server/3

#创建服务器证书申请文件，其中server是文件前缀
[root@openvpn-server 3]#./easyrsa gen-req server nopass

Note: using Easy-RSA configuration from: /etc/openvpn/easy-rsa-server/3.0.8/vars
Using SSL: openssl OpenSSL 1.1.1k  FIPS 25 Mar 2021
Generating a RSA private key
..................................................+++++
....+++++
writing new private key to '/etc/openvpn/easy-rsa-server/3/pki/easy-rsa-11911.ZIKE0B/tmp.DhRQBt'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [server]:#接受Common Name的默认
值,直接回车

Keypair and certificate request completed. Your files are:
req: /etc/openvpn/easy-rsa-server/3/pki/reqs/server.req      #生成请求文件
key: /etc/openvpn/easy-rsa-server/3/pki/private/server.key   #生成私钥文件


[root@openvpn-server 3]#tree
.
├── easyrsa
├── openssl-easyrsa.cnf
├── pki
│   ├── ca.crt
│   ├── certs_by_serial
│   ├── index.txt
│   ├── index.txt.attr
│   ├── issued
│   ├── openssl-easyrsa.cnf
│   ├── private
│   │   ├── ca.key
│   │   └── server.key          #生成私钥文件
│   ├── renewed
│   │   ├── certs_by_serial
│   │   ├── private_by_serial
│   │   └── reqs_by_serial
│   ├── reqs
│   │   └── server.req           #生成请求文件
│   ├── revoked
│   │   ├── certs_by_serial
│   │   ├── private_by_serial
│   │   └── reqs_by_serial
│   ├── safessl-easyrsa.cnf
│   └── serial
├── vars
└── x509-types
    ├── ca
    ├── client
    ├── code-signing
    ├── COMMON
    ├── email
    ├── kdc
    ├── server
    └── serverClient

14 directories, 20 files

```

### 3.4**签发服务端证书**

#### **3.4.1** **查看颁发证书命令用法**

```
[root@openvpn-serve ~]#cd /etc/openvpn/easy-rsa-server/3

[root@openvpn-server 3]#./easyrsa help sign

Note: using Easy-RSA configuration from: /etc/openvpn/easy-rsa-server/3.0.8/vars

  sign-req <type> <filename_base>
      Sign a certificate request of the defined type. <type> must be a known
      type such as 'client', 'server', 'serverClient', or 'ca' (or a user-added type.)

      This request file must exist in the reqs/ dir and have a .req file
      extension. See import-req below for importing reqs from other sources.

```



#### 3.4.2 **颁发服务端证书**

```
#将上面server.req的申请,颁发server类型的证书
[root@openvpn-server ~]#cd /etc/openvpn/easy-rsa-server/3

[root@openvpn-server 3]#./easyrsa sign server server

Note: using Easy-RSA configuration from: /etc/openvpn/easy-rsa-server/3.0.8/vars
Using SSL: openssl OpenSSL 1.1.1k  FIPS 25 Mar 2021


You are about to sign the following certificate.
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.

Request subject, to be signed as a server certificate for 8250 days:#可以看到
vars文件指定的有效期

subject=
    commonName                = server


Type the word 'yes' to continue, or any other input to abort.
  Confirm request details: yes  #输入yes回车
Using configuration from /etc/openvpn/easy-rsa-server/3/pki/easy-rsa-11953.01ydjX/tmp.capB7b
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'server'
Certificate is to be certified until Jan 14 06:42:40 2046 GMT (8250 days)

Write out database with 1 new entries
Data Base Updated

Certificate created at: /etc/openvpn/easy-rsa-server/3/pki/issued/server.crt #生成服务器证书文件

```

#### 3.4.3 **验证结果**

```
[root@openvpn-server 3]#cd /etc/openvpn/easy-rsa-server/3
[root@openvpn-server 3]#tree pki/
pki/
├── ca.crt
├── certs_by_serial
│   └── B8A307DDCAD2E3B9A473A2CB590C0460.pem    #服务器证书文件
├── index.txt
├── index.txt.attr
├── index.txt.attr.old
├── index.txt.old
├── issued
│   └── server.crt                      #服务器证书文件
├── openssl-easyrsa.cnf
├── private
│   ├── ca.key
│   └── server.key
├── renewed
│   ├── certs_by_serial
│   ├── private_by_serial
│   └── reqs_by_serial
├── reqs
│   └── server.req
├── revoked
│   ├── certs_by_serial
│   ├── private_by_serial
│   └── reqs_by_serial
├── safessl-easyrsa.cnf
├── serial
└── serial.old

12 directories, 14 files


[root@openvpn-server 3]#cat pki/issued/server.crt
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            b8:a3:07:dd:ca:d2:e3:b9:a4:73:a2:cb:59:0c:04:60
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=Easy-RSA CA
        Validity
            Not Before: Jun 14 06:42:40 2023 GMT
            Not After : Jan 14 06:42:40 2046 GMT
        Subject: CN=server
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:b6:94:94:9d:44:6b:0f:ca:73:a7:36:c3:ec:c6:
                    26:81:c8:46:70:07:90:60:3a:e5:eb:76:9a:65:82:
                    08:38:e6:d3:69:bb:55:8a:fa:38:94:89:39:0a:b8:
                    80:9f:1a:ca:02:61:e5:c1:88:b1:f0:bd:42:85:2e:
                    f8:f4:7b:43:88:e5:e2:cc:46:ab:6f:e2:58:4c:90:
                    29:a7:67:36:79:c1:9c:5e:4d:f7:fd:45:f1:ee:7d:
                    03:4f:a4:23:b3:f2:6c:11:a8:88:0a:20:0a:0d:1c:
                    31:00:75:64:61:d3:4e:3c:5c:e4:ee:e4:5d:85:09:
                    e3:fe:3e:32:34:1a:5c:9b:5c:78:b9:9d:b6:cd:02:
                    0a:bc:e1:ec:f1:45:75:83:42:b1:e6:fd:12:90:3b:
                    c4:e7:2c:bc:6f:dd:f1:c5:5f:65:3a:0b:bd:4d:a1:
                    52:cf:d2:79:d0:86:a7:6d:5a:76:c4:ac:c2:15:9c:
                    39:f4:f4:7d:ea:02:3a:33:d0:7a:b2:bd:5f:66:52:
                    db:80:0a:a1:be:f7:6b:0c:53:a6:b6:2f:00:2f:1b:
                    16:f7:41:1d:7c:33:d3:31:58:74:c6:44:a0:50:2c:
                    ef:5b:e0:b7:70:a5:b5:42:9b:ec:14:3f:1e:c7:8a:
                    03:58:31:8a:d2:9a:e3:37:1d:ea:63:9d:34:69:bb:
                    eb:25
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Basic Constraints: 
                CA:FALSE
            X509v3 Subject Key Identifier: 
                CF:7F:0B:14:10:62:41:C8:B3:21:C0:C8:16:AD:85:C7:35:2C:F6:75
            X509v3 Authority Key Identifier: 
                keyid:55:FF:F7:55:38:33:83:6A:50:71:DA:18:0D:2F:74:AD:DE:17:2D:23
                DirName:/CN=Easy-RSA CA
                serial:74:D0:0B:AD:6B:47:F3:FA:6C:0A:8F:DD:6C:47:67:F1:FE:59:9E:14

            X509v3 Extended Key Usage: 
                TLS Web Server Authentication
            X509v3 Key Usage: 
                Digital Signature, Key Encipherment
            X509v3 Subject Alternative Name: 
                DNS:server
    Signature Algorithm: sha256WithRSAEncryption
         5a:5a:26:68:f1:23:3c:55:58:9d:d3:10:8a:2f:28:47:a5:a5:
         9d:e4:e9:9d:0f:95:9a:f8:7c:26:b8:1d:93:85:84:f9:1b:4d:
         37:a9:30:92:b9:be:6d:b7:e8:60:a6:5e:94:5d:7a:60:5b:1b:
         8b:96:cc:7f:01:6b:0a:ea:97:6f:ae:76:ef:11:d5:ae:88:e4:
         23:cc:3c:59:2e:b6:f7:bb:35:81:78:c0:61:52:b0:eb:fb:5b:
         fd:f1:80:24:43:0a:ea:cd:df:77:7d:2f:b8:11:bd:09:b5:1b:
         d9:fb:54:de:28:b9:bf:62:4d:6e:68:26:4e:22:e8:d2:ba:e8:
         3d:e6:4c:d7:9e:4e:cf:d7:20:33:5c:71:8c:a9:58:7c:ec:2b:
         ce:5c:93:d2:46:06:b6:8b:56:6b:2e:90:07:5f:9a:b0:1c:18:
         b0:b2:80:75:44:d6:b7:21:6f:e2:61:6b:6d:ad:9b:0d:c5:f7:
         a5:25:41:fa:ff:9d:34:30:0a:4b:e0:a4:d1:b4:4b:f5:7c:48:
         3c:08:90:6c:e5:cc:d5:72:af:00:18:da:ed:c4:e9:e6:57:48:
         ce:a8:de:c1:9f:5f:bf:dc:4a:99:6a:5d:84:cf:6a:d8:b4:fc:
         f0:93:36:18:a3:77:cc:33:cb:c7:5b:82:8c:7a:5c:af:bc:50:
         d5:00:67:04
-----BEGIN CERTIFICATE-----
MIIDaDCCAlCgAwIBAgIRALijB93K0uO5pHOiy1kMBGAwDQYJKoZIhvcNAQELBQAw
FjEUMBIGA1UEAwwLRWFzeS1SU0EgQ0EwHhcNMjMwNjE0MDY0MjQwWhcNNDYwMTE0
MDY0MjQwWjARMQ8wDQYDVQQDDAZzZXJ2ZXIwggEiMA0GCSqGSIb3DQEBAQUAA4IB
DwAwggEKAoIBAQC2lJSdRGsPynOnNsPsxiaByEZwB5BgOuXrdpplggg45tNpu1WK
+jiUiTkKuICfGsoCYeXBiLHwvUKFLvj0e0OI5eLMRqtv4lhMkCmnZzZ5wZxeTff9
RfHufQNPpCOz8mwRqIgKIAoNHDEAdWRh0048XOTu5F2FCeP+PjI0GlybXHi5nbbN
Agq84ezxRXWDQrHm/RKQO8TnLLxv3fHFX2U6C71NoVLP0nnQhqdtWnbErMIVnDn0
9H3qAjoz0HqyvV9mUtuACqG+92sMU6a2LwAvGxb3QR18M9MxWHTGRKBQLO9b4Ldw
pbVCm+wUPx7HigNYMYrSmuM3HepjnTRpu+slAgMBAAGjgbUwgbIwCQYDVR0TBAIw
ADAdBgNVHQ4EFgQUz38LFBBiQcizIcDIFq2FxzUs9nUwUQYDVR0jBEowSIAUVf/3
VTgzg2pQcdoYDS90rd4XLSOhGqQYMBYxFDASBgNVBAMMC0Vhc3ktUlNBIENBghR0
0Auta0fz+mwKj91sR2fx/lmeFDATBgNVHSUEDDAKBggrBgEFBQcDATALBgNVHQ8E
BAMCBaAwEQYDVR0RBAowCIIGc2VydmVyMA0GCSqGSIb3DQEBCwUAA4IBAQBaWiZo
8SM8VVid0xCKLyhHpaWd5OmdD5Wa+HwmuB2ThYT5G003qTCSub5tt+hgpl6UXXpg
WxuLlsx/AWsK6pdvrnbvEdWuiOQjzDxZLrb3uzWBeMBhUrDr+1v98YAkQwrqzd93
fS+4Eb0JtRvZ+1TeKLm/Yk1uaCZOIujSuug95kzXnk7P1yAzXHGMqVh87CvOXJPS
Rga2i1ZrLpAHX5qwHBiwsoB1RNa3IW/iYWttrZsNxfelJUH6/500MApL4KTRtEv1
fEg8CJBs5czVcq8AGNrtxOnmV0jOqN7Bn1+/3EqZal2Ez2rYtPzwkzYYo3fMM8vH
W4KMelyvvFDVAGcE
-----END CERTIFICATE-----

#证书相关文件
[root@openvpn-server 3]#cat pki/serial
serial      serial.old  
[root@openvpn-server 3]#cat pki/serial
B8A307DDCAD2E3B9A473A2CB590C0461
[root@openvpn-server 3]#cat pki/index.txt
V	460114064240Z		B8A307DDCAD2E3B9A473A2CB590C0460	unknown	/CN=server
[root@openvpn-server 3]#cat pki/serial
serial      serial.old  
[root@openvpn-server 3]#cat pki/serial.old 
b8a307ddcad2e3b9a473a2cb590c0460
```

### **3.5** **创建** **Diffie-Hellman** **密钥**

#### **3.5.1 Diffie-Hellman** **算法**

```
Diffie-Hellman 密钥交换方法，由惠特菲尔德·迪菲（Bailey Whitfield Diffie）、马丁·赫尔曼
（Martin Edward Hellman）于1976年发表。它是一种安全协议，让双方在完全没有对方任何预先信息的
条件下通过不安全信道建立起一个密钥，这个密钥一般作为“对称加密”的密钥而被双方在后续数据传输中使
用。DH数学原理是base离散对数问题。做类似功能的还有非对称加密类算法，如：RSA。其应用非常广泛，在
SSH、VPN、Https等都有应用。
wiki参考链接:
https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange
```

#### **3.5.2** **创建** **Diffie-Hellman** **密钥**

```
[root@openvpn-server 3]# cd /etc/openvpn/easy-rsa-server/3
[root@openvpn-server 3]#pwd
/etc/openvpn/easy-rsa-server/3

方法一：
[root@centos8 3]#./easyrsa gen-dh

Note: using Easy-RSA configuration from: /etc/openvpn/easy-rsa-server/3.0.8/vars
Using SSL: openssl OpenSSL 1.1.1k  FIPS 25 Mar 2021
Generating DH parameters, 2048 bit long safe prime, generator 2
This is going to take a long time

................+..................................+............................
..........................++*++*++*++*
.......#需要等待一会儿
DH parameters of size 2048 created at /etc/openvpn/easy-rsa-sever/3/pki/dh.pem


#查看生成的文件
[root@openvpn-server 3]#ll pki/dh.pem 
-rw------- 1 root root 424 Jun 14 14:53 pki/dh.pem
[root@openvpn-server 3]#cat pki/dh.pem 
-----BEGIN DH PARAMETERS-----
MIIBCAKCAQEAxinQjy4jT4uSnrTkdVI1X7Bnp3xCIw9zozkC6bJugLAgD0fpkR7D
iLFXsEWhpmD025EeBrRa6n/S1ZzENIFqRDCFac/609Nx/SwZr2k9SmeSgooR62y5
Y1ExButYD74z6s4VUnGpcLNbF+rn+0BoWI+dKNGIVNQ038ZiMOsWeoUu3/2vumFu
GrtEM+qn6qi/Jr1bR0Q8m/5/+puAqSi/2mvcBCPu2Bu0XsK/r/ZDIivBzypUnRTZ
UACNae8oFpRVdnkllKamiBzJwYSgQQu4cUpfXgBtob3Q5OVC+fFJpepO6WNn5uKM
WQ6Q0J1hnKGXQykyPdb/5FUQ2YALeGkdIwIBAg==
-----END DH PARAMETERS-----




#方法二
[root@centos8 ~]#openssl dhparam -out /etc/openvpn/dh2048.pem 2048
[root@centos8 ~]#ll /etc/openvpn/dh2048.pem 
-rw-r--r-- 1 root root 424 Aug  3 20:50 /etc/openvpn/dh2048.pem
```

### **3.6** **准备客户端证书环境**（此处服务器、客户端文件放在一起）

```
[root@openvpn-server 3]# cd /etc/openvpn/easy-rsa-server/3
[root@openvpn-server 3]#pwd
/etc/openvpn/easy-rsa-server/3
[root@openvpn-server 3]#tree
.
├── easyrsa
├── openssl-easyrsa.cnf
├── pki
│   ├── ca.crt
│   ├── certs_by_serial
│   │   └── B8A307DDCAD2E3B9A473A2CB590C0460.pem
│   ├── dh.pem
│   ├── index.txt
│   ├── index.txt.attr
│   ├── index.txt.attr.old
│   ├── index.txt.old
│   ├── issued
│   │   └── server.crt
│   ├── openssl-easyrsa.cnf
│   ├── private
│   │   ├── ca.key
│   │   └── server.key
│   ├── renewed
│   │   ├── certs_by_serial
│   │   ├── private_by_serial
│   │   └── reqs_by_serial
│   ├── reqs
│   │   └── server.req
│   ├── revoked
│   │   ├── certs_by_serial
│   │   ├── private_by_serial
│   │   └── reqs_by_serial
│   ├── safessl-easyrsa.cnf
│   ├── serial
│   └── serial.old
├── vars
└── x509-types
    ├── ca
    ├── client
    ├── code-signing
    ├── COMMON
    ├── email
    ├── kdc
    ├── server
    └── serverClient

14 directories, 26 files


#生成证书申请所需目录pki和文件，同服务器配置




```

### **3.7** **创建客户端证书申请**

```
[root@centos8 ~]#cd /etc/openvpn/easy-rsa-client/3
[root@centos8 3]#pwd
/etc/openvpn/easy-rsa-client/3

#生成客户端用户的证书申请
[root@openvpn-server 3]#./easyrsa gen-req dingbaohang nopass

Note: using Easy-RSA configuration from: /etc/openvpn/easy-rsa-server/3.0.8/vars
Using SSL: openssl OpenSSL 1.1.1k  FIPS 25 Mar 2021
Generating a RSA private key
............+++++
.................+++++
writing new private key to '/etc/openvpn/easy-rsa-server/3/pki/easy-rsa-12101.P6tXRT/tmp.beIb8b'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [dingbaohang]:#接受默认值,直接
回车

Keypair and certificate request completed. Your files are:
req: /etc/openvpn/easy-rsa-server/3/pki/reqs/dingbaohang.req  #私钥文件
key: /etc/openvpn/easy-rsa-server/3/pki/private/dingbaohang.key  #证书申请文件

#生成两个新文件
 [root@openvpn-server 3]#tree
.
├── easyrsa
├── openssl-easyrsa.cnf
├── pki
│   ├── ca.crt
│   ├── certs_by_serial
│   │   └── B8A307DDCAD2E3B9A473A2CB590C0460.pem
│   ├── dh.pem
│   ├── index.txt
│   ├── index.txt.attr
│   ├── index.txt.attr.old
│   ├── index.txt.old
│   ├── issued
│   │   └── server.crt
│   ├── openssl-easyrsa.cnf
│   ├── private
│   │   ├── ca.key
│   │   ├── dingbaohang.key
│   │   └── server.key
│   ├── renewed
│   │   ├── certs_by_serial
│   │   ├── private_by_serial
│   │   └── reqs_by_serial
│   ├── reqs
│   │   ├── dingbaohang.req
│   │   └── server.req
│   ├── revoked
│   │   ├── certs_by_serial
│   │   ├── private_by_serial
│   │   └── reqs_by_serial
│   ├── safessl-easyrsa.cnf
│   ├── serial
│   └── serial.old
├── vars
└── x509-types
    ├── ca
    ├── client
    ├── code-signing
    ├── COMMON
    ├── email
    ├── kdc
    ├── server
    └── serverClient

14 directories, 28 files


```

### **3.8** **签发客户端证书**

```
[root@openvpn-server 3]#pwd
/etc/openvpn/easy-rsa-server/3

#将客户端证书请求文件复制到CA的工作目录
[root@openvpn-server 3]#./easyrsa import-req /etc/openvpn/easy-rsaclient/3/pki/reqs/dingbaohang.rep dingbaohang

Note: using Easy-RSA configuration from: /etc/openvpn/easy-rsa-server/3.0.8/vars
Using SSL: openssl OpenSSL 1.1.1k  FIPS 25 Mar 2021

Easy-RSA error:

The input file does not appear to be a certificate request. Aborting import.
File Path: /etc/openvpn/easy-rsaclient/3/pki/reqs/dingbaohang.rep

[root@openvpn-server 3]#ll pki/reqs/dingbaohang.req /etc/openvpn/easy-rsa-server/3/pki/reqs/dingbaohang.req 
-rw------- 1 root root 895 Jun 14 15:05 /etc/openvpn/easy-rsa-server/3/pki/reqs/dingbaohang.req
-rw------- 1 root root 895 Jun 14 15:05 pki/reqs/dingbaohang.req

#修改给客户端颁发的证书的有效期
[root@openvpn-server 3]#vim vars

#建议修改给客户端颁发证书的有效期,可适当减少,比如:90天
#set_var EASYRSA_CERT_EXPIRE   825 
#将上面行修改为下面
set_var EASYRSA_CERT_EXPIRE 90

#签发客户端证书
[root@openvpn-server 3]#./easyrsa sign client dingbaohang

Note: using Easy-RSA configuration from: /etc/openvpn/easy-rsa-server/3.0.8/vars
Using SSL: openssl OpenSSL 1.1.1k  FIPS 25 Mar 2021


You are about to sign the following certificate.
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.

Request subject, to be signed as a client certificate for 90 days:

subject=
    commonName                = dingbaohang


Type the word 'yes' to continue, or any other input to abort.
  Confirm request details: yes
Using configuration from /etc/openvpn/easy-rsa-server/3/pki/easy-rsa-12249.VWSlmU/tmp.5zGS4z
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'dingbaohang'
Certificate is to be certified until Sep 12 07:23:26 2023 GMT (90 days)

Write out database with 1 new entries
Data Base Updated

Certificate created at: /etc/openvpn/easy-rsa-server/3/pki/issued/dingbaohang.crt  #证书文件 

[root@openvpn-server 3]#tree pki/
pki/
├── ca.crt
├── certs_by_serial
│   ├── B8A307DDCAD2E3B9A473A2CB590C0460.pem
│   └── BBD575D412666BE1B578CACB25323067.pem
├── dh.pem
├── index.txt
├── index.txt.attr
├── index.txt.attr.old
├── index.txt.old
├── issued
│   ├── dingbaohang.crt    #生成客户端证书
│   └── server.crt
├── openssl-easyrsa.cnf
├── private
│   ├── ca.key
│   ├── dingbaohang.key
│   └── server.key
├── renewed
│   ├── certs_by_serial
│   ├── private_by_serial
│   └── reqs_by_serial
├── reqs
│   ├── dingbaohang.req
│   └── server.req
├── revoked
│   ├── certs_by_serial
│   ├── private_by_serial
│   └── reqs_by_serial
├── safessl-easyrsa.cnf
├── serial
└── serial.old

12 directories, 19 files

[root@openvpn-server 3]#cat pki/index.txt
V	460114064240Z		B8A307DDCAD2E3B9A473A2CB590C0460	unknown	/CN=server
V	230912072326Z		BBD575D412666BE1B578CACB25323067	unknown	/CN=dingbaohang

[root@openvpn-server 3]#ll pki/issued/
total 16
-rw------- 1 root root 4505 Jun 14 15:23 dingbaohang.crt
-rw------- 1 root root 4608 Jun 14 14:42 server.crt
[root@openvpn-server 3]#ll pki/certs_by_serial/
total 16
-rw------- 1 root root 4608 Jun 14 14:42 B8A307DDCAD2E3B9A473A2CB590C0460.pem
-rw------- 1 root root 4505 Jun 14 15:23 BBD575D412666BE1B578CACB25323067.pem
[root@openvpn-server 3]#cat pki/issued/dingbaohang.crt 
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            bb:d5:75:d4:12:66:6b:e1:b5:78:ca:cb:25:32:30:67
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=Easy-RSA CA
        Validity
            Not Before: Jun 14 07:23:26 2023 GMT
            Not After : Sep 12 07:23:26 2023 GMT
        Subject: CN=dingbaohang
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:98:de:9a:0b:94:98:51:0b:87:35:b6:b1:18:fe:
                    4e:e5:0e:09:e6:ea:a5:87:7e:62:c9:0c:c7:8b:95:
                    bd:78:21:04:e6:7d:7c:09:2a:62:72:aa:eb:b6:7c:
                    03:c9:f4:73:95:10:02:e8:0f:99:38:14:9e:8b:c5:
                    5f:95:8f:a7:10:cb:4b:e2:dc:43:a1:7b:d0:d9:4a:
                    4d:c8:1a:9b:27:e7:19:f1:30:ac:e3:2f:23:97:ea:
                    8c:b0:94:ef:ca:c4:14:92:30:ca:01:6e:a6:ae:78:
                    1d:5a:e2:7f:de:fb:9f:c5:06:09:38:02:30:d3:c7:
                    a7:be:d3:8b:7f:c1:dc:64:bc:8a:eb:0d:fb:54:8e:
                    89:fe:f2:6b:62:7b:de:f4:47:66:e7:1e:17:dc:5b:
                    c6:42:0e:50:99:dc:71:03:cd:be:5d:5b:1f:18:1a:
                    d1:4f:e7:8f:bb:0a:27:b3:ea:d1:dd:62:39:32:4b:
                    28:91:01:ae:c6:f5:dd:30:95:b5:0d:1d:1a:1d:97:
                    7a:36:e4:77:78:3e:07:2d:3c:7f:5d:93:51:b4:88:
                    86:c3:ab:67:70:6b:02:00:8f:ad:6e:d6:80:97:a4:
                    d1:3c:e4:c9:db:64:53:6a:67:c7:4e:1a:b5:71:87:
                    0d:8b:b4:df:a9:9c:47:a9:2c:ea:dd:4e:5f:55:31:
                    29:4b
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Basic Constraints: 
                CA:FALSE
            X509v3 Subject Key Identifier: 
                89:A5:57:59:E5:76:6D:EA:E6:07:84:CC:57:68:C4:4F:C7:78:59:17
            X509v3 Authority Key Identifier: 
                keyid:55:FF:F7:55:38:33:83:6A:50:71:DA:18:0D:2F:74:AD:DE:17:2D:23
                DirName:/CN=Easy-RSA CA
                serial:74:D0:0B:AD:6B:47:F3:FA:6C:0A:8F:DD:6C:47:67:F1:FE:59:9E:14

            X509v3 Extended Key Usage: 
                TLS Web Client Authentication
            X509v3 Key Usage: 
                Digital Signature
    Signature Algorithm: sha256WithRSAEncryption
         a7:b0:a4:f2:ee:40:6b:0a:a6:84:02:2e:3c:04:0e:9c:5b:1c:
         aa:a9:8a:99:db:44:ec:65:5e:9b:0b:6e:1c:06:86:80:f0:8f:
         31:ff:40:55:7c:6b:e8:de:16:34:83:d8:b9:5e:55:44:b8:eb:
         37:84:e0:b0:5c:ce:cd:1e:0b:37:78:38:36:04:34:ab:86:2f:
         03:bf:94:0b:24:24:c0:93:5c:4b:c4:16:06:fb:57:24:31:ab:
         20:02:35:ff:5e:29:8c:20:48:df:9d:15:b0:bb:11:eb:f2:07:
         79:3e:a4:c5:d1:44:14:53:00:8f:f2:62:e5:cc:b5:5d:f6:9b:
         4f:4f:d6:f8:35:52:07:24:b0:20:b7:5c:f3:8e:f3:c1:1e:e0:
         ef:ff:78:f5:69:d3:d8:02:22:92:3e:65:ff:09:d6:98:5e:b7:
         15:cb:7f:d2:b8:e1:84:79:1b:75:ee:ec:e7:4e:79:d6:72:c3:
         64:f6:21:6e:f8:7b:86:dd:46:07:b6:24:0a:0c:b6:c2:b5:35:
         7b:3c:78:ca:42:b8:c6:bd:79:1b:79:a2:80:b7:51:39:37:11:
         ab:12:d9:c2:3b:78:27:fd:75:e3:6e:7e:cd:5d:6b:83:f7:ef:
         03:74:ab:1c:f2:a9:3c:05:1c:b6:53:ea:ad:3a:7b:ff:74:5a:
         95:bb:15:e1
-----BEGIN CERTIFICATE-----
MIIDWjCCAkKgAwIBAgIRALvVddQSZmvhtXjKyyUyMGcwDQYJKoZIhvcNAQELBQAw
FjEUMBIGA1UEAwwLRWFzeS1SU0EgQ0EwHhcNMjMwNjE0MDcyMzI2WhcNMjMwOTEy
MDcyMzI2WjAWMRQwEgYDVQQDDAtkaW5nYmFvaGFuZzCCASIwDQYJKoZIhvcNAQEB
BQADggEPADCCAQoCggEBAJjemguUmFELhzW2sRj+TuUOCebqpYd+YskMx4uVvXgh
BOZ9fAkqYnKq67Z8A8n0c5UQAugPmTgUnovFX5WPpxDLS+LcQ6F70NlKTcgamyfn
GfEwrOMvI5fqjLCU78rEFJIwygFupq54HVrif977n8UGCTgCMNPHp77Ti3/B3GS8
iusN+1SOif7ya2J73vRHZuceF9xbxkIOUJnccQPNvl1bHxga0U/nj7sKJ7Pq0d1i
OTJLKJEBrsb13TCVtQ0dGh2Xejbkd3g+By08f12TUbSIhsOrZ3BrAgCPrW7WgJek
0TzkydtkU2pnx04atXGHDYu036mcR6ks6t1OX1UxKUsCAwEAAaOBojCBnzAJBgNV
HRMEAjAAMB0GA1UdDgQWBBSJpVdZ5XZt6uYHhMxXaMRPx3hZFzBRBgNVHSMESjBI
gBRV//dVODODalBx2hgNL3St3hctI6EapBgwFjEUMBIGA1UEAwwLRWFzeS1SU0Eg
Q0GCFHTQC61rR/P6bAqP3WxHZ/H+WZ4UMBMGA1UdJQQMMAoGCCsGAQUFBwMCMAsG
A1UdDwQEAwIHgDANBgkqhkiG9w0BAQsFAAOCAQEAp7Ck8u5AawqmhAIuPAQOnFsc
qqmKmdtE7GVemwtuHAaGgPCPMf9AVXxr6N4WNIPYuV5VRLjrN4TgsFzOzR4LN3g4
NgQ0q4YvA7+UCyQkwJNcS8QWBvtXJDGrIAI1/14pjCBI350VsLsR6/IHeT6kxdFE
FFMAj/Ji5cy1XfabT0/W+DVSBySwILdc847zwR7g7/949WnT2AIikj5l/wnWmF63
Fct/0rjhhHkbde7s50551nLDZPYhbvh7ht1GB7YkCgy2wrU1ezx4ykK4xr15G3mi
gLdROTcRqxLZwjt4J/11425+zV1rg/fvA3SrHPKpPAUctlPqrTp7/3RalbsV4Q==
-----END CERTIFICATE-----

```

如果需要颁发的客户端证书较多,可以使用下面脚本实现客户端证书的批量颁发

**客户端证书自动颁发脚本**

```
[root@centos8 ~]#cat openvpn-user-crt.sh 
#!/bin/bash
read -p "请输入用户的姓名拼音(如:${NAME}): " NAME
cd /etc/openvpn/easy-rsa-client/3
./easyrsa gen-req ${NAME} nopass <<EOF
EOF
cd /etc/openvpn/easy-rsa-server/3
./easyrsa import-req /etc/openvpn/easy-rsa-client/3/pki/reqs/${NAME}.req ${NAME}
./easyrsa sign client ${NAME} <<EOF
yes
EOF
```

### **3.9**将CA和服务器证书相关文件复制到服务器相应的目录

```
[root@openvpn-server 3]#mkdir /etc/openvpn/certs
[root@openvpn-server 3]#cp /etc/openvpn/easy-rsa-server/3/pki/ca.crt /etc/openvpn/certs/
[root@openvpn-server 3]#cp /etc/openvpn/easy-rsa-server/3/pki/issued/server.crt /etc/openvpn/certs/

[root@openvpn-server 3]#cp /etc/openvpn/easy-rsa-server/3/pki/reqs/server.req /etc/openvpn/certs/
[root@openvpn-server 3]#cp /etc/openvpn/easy-rsa-server/3/pki/dh.pem /etc/openvpn/certs/
[root@openvpn-server 3]#ll /etc/openvpn/certs/
total 20
-rw------- 1 root root 1204 Jun 14 15:30 ca.crt
-rw------- 1 root root  424 Jun 14 15:32 dh.pem
-rw------- 1 root root 4608 Jun 14 15:31 server.crt
-rw------- 1 root root  887 Jun 14 15:32 server.req
```

### **3.10** **将客户端私钥与证书相关文件复制到服务器相关的目录**

```
[root@openvpn-server 3]#find /etc/openvpn/ -name "dingbaohang.key" -o -name "dingbaohang.crt" -o -name ca.crt
/etc/openvpn/easy-rsa-server/3.0.8/pki/private/dingbaohang.key
/etc/openvpn/easy-rsa-server/3.0.8/pki/issued/dingbaohang.crt
/etc/openvpn/easy-rsa-server/3.0.8/pki/ca.crt
/etc/openvpn/certs/ca.crt

[root@openvpn-server 3]#find /etc/openvpn/ \( -name "dingbaohang.key" -o -name "dingbaohang.crt" -o -name ca.crt \) -exec cp {} /etc/openvpn/client/dingbaohang \;

[root@openvpn-server 3]#ll /etc/openvpn/client/dingbaohang/
total 16
-rw------- 1 root root 1204 Jun 14 15:45 ca.crt
-rw------- 1 root root 4505 Jun 14 15:45 dingbaohang.crt
-rw------- 1 root root 1704 Jun 14 15:45 dingbaohang.key



find /etc/openvpn/ \( -name "dingbaohang.key" -o -name "dingbaohang.crt" -o -name ca.crt \) -exec cp {} /etc/openvpn/client/dingbaohang \;
```

## **4** 、**准备** **OpenVPN** **服务器配置文件**

### **4.1** **服务器端配置文件说明**

```
#server.conf文件中以#或;开头的行都为注释

[root@openvpn-server ~]#grep '^[a-Z].*' /etc/openvpn/server.conf

port 1194
proto tcp
dev tun
ca /etc/openvpn/certs/ca.crt
cert /etc/openvpn/certs/server.crt
key /etc/openvpn/certs/server.key  # This file should be kept secret
dh /etc/openvpn/certs/dh.pem
server 10.8.0.0 255.255.255.0
push "route 172.30.0.0 255.255.255.0"
keepalive 10 120
cipher AES-256-CBC
compress lz4-v2
push "compress lz4-v2"
max-clients 2048
user openvpn
group openvpn
status /var/log/openvpn/openvpn-status.log
log-append /var/log/openvpn/openvpn.log
verb 3
mute 20

#准备目志相关目录
[root@openvpn-server ~]#vim /etc/openvpn/server.conf
[root@openvpn-server ~]#getent passwd openvpn
openvpn:x:220:988:OpenVPN:/etc/openvpn:/sbin/nologin
[root@openvpn-server ~]#mkdir /var/lo
local/ lock/  log/   
[root@openvpn-server ~]#mkdir /var/log/openvpn
[root@openvpn-server ~]#chown openvpn.openvpn /var/log/openvpn
[root@openvpn-server ~]#ll -d /var/log/openvpn
drwxr-xr-x 2 openvpn openvpn 6 Jun 14 16:31 /var/log/openvpn
```

## **5**、 **准备** **iptables** **规则和内核参数**

```
#在服务器开启ip_forward转发功能
[root@centos8 ~]#echo net.ipv4.ip_forward = 1 >> /etc/sysctl.conf 
[root@centos8 ~]#sysctl -p
net.ipv4.ip_forward = 1


#添加SNAT规则
[root@openvpn-server sysctl.d]#vim /etc/rc.local
#添加以下内容
[root@openvpn-server sysctl.d]#cat /etc/rc.local |grep '^[a-z].*'
touch /var/lock/subsys/local
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -j MASQUERADE

[root@openvpn-server sysctl.d]#/etc/rc.d/rc.local
[root@openvpn-server sysctl.d]#iptables -nvL
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

[root@openvpn-server sysctl.d]#iptables -vnL -t nat
Chain PREROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain POSTROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 MASQUERADE  all  --  *      *       10.8.0.0/24          0.0.0.0/0           

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination  
```

## **6、启动** **OpenVPN** **服务**

```
#默认8版本没有unit文件，可从7上拷出
[root@openvpn-server sysctl.d]#rpm -ql openvpn|grep systemd
/usr/lib/systemd/system/openvpn-client@.service
/usr/lib/systemd/system/openvpn-server@.service
/usr/lib/systemd/system/openvpn@.service
/usr/share/doc/openvpn/README.systemd


[root@openvpn-server sysctl.d]#vim /usr/lib/systemd/system/openvpn@.service
[root@openvpn-server sysctl.d]#cat /usr/lib/systemd/system/openvpn@.service
[Unit]
Description=OpenVPN Robust And Highly Flexible Tunneling Application On %I
After=network.target
[Service]
Type=notify
PrivateTmp=true
ExecStart=/usr/sbin/openvpn --cd /etc/openvpn/ --config %i.conf
[Install]
WantedBy=multi-user.target

[root@openvpn-server sysctl.d]#systemctl daemon-reload
[root@openvpn-server sysctl.d]#systemctl enable --now openvpn@server



#开启用户密码认证
 22 script-security 3
 23 auth-user-pass-verify /etc/openvpn/checkpsw.sh via-env
 24 username-as-common-name


注：set paste 保留格式
```

## 7、开启用户密码认证

```
server端服务端配置
[root@openvpn-server openvpn]#vim /etc/openvpn/server.conf

22 script-security 3  #允许使用自定义脚本
 23 auth-user-pass-verify /etc/openvpn/checkpsw.sh via-env  #指定自定义脚本路径
 24 username-as-common-name  #开启用户密码认证


#添加认证脚本
[root@openvpn-server ]#vim /etc/openvpn/checkpsw.sh  
 #!/bin/sh
###########################################################
# checkpsw.sh (C) 2004 Mathias Sundman <mathias@openvpn.se>
#
# This script will authenticate OpenVPN users against
# a plain text file. The passfile should simply contain
# one row per user with the username first followed by
# one or more space(s) or tab(s) and then the password.

PASSFILE="/etc/openvpn/psw-file"
LOG_FILE="/var/log/openvpn-password.log"
TIME_STAMP=`date "+%Y-%m-%d %T"`

###########################################################

if [ ! -r "${PASSFILE}" ]; then
  echo "${TIME_STAMP}: Could not open password file \"${PASSFILE}\" for reading." >> ${LOG_FILE}
  exit 1
fi

CORRECT_PASSWORD=`awk '!/^;/&&!/^#/&&$1=="'${username}'"{print $2;exit}' ${PASSFILE}`

if [ "${CORRECT_PASSWORD}" = "" ]; then 
  echo "${TIME_STAMP}: User does not exist: username=\"${username}\", password=\"${password}\"." >> ${LOG_FILE}
  exit 1
fi

if [ "${password}" = "${CORRECT_PASSWORD}" ]; then 
  echo "${TIME_STAMP}: Successful authentication: username=\"${username}\"." >> ${LOG_FILE}
  exit 0
fi

echo "${TIME_STAMP}: Incorrect password: username=\"${username}\", password=\"${password}\"." >> ${LOG_FILE}
exit 1


#添加执行权限
[root@openvpn-server openvpn]#chmod +x checkpsw.sh 
 
 
 #客户端配置文件client.ovpn添加以下内容
 
 auth-user-pass
 
```

# pgsql实现



## 1、编译安装pgsql

```
 #官方安装步骤
Short Version
./configure
make -j $CPU
su                                                                                                                                                              
make install
adduser postgres
mkdir /usr/local/pgsql/data
chown postgres /usr/local/pgsql/data
su - postgres
/usr/local/pgsql/bin/initdb -D /usr/local/pgsql/data
/usr/local/pgsql/bin/pg_ctl -D /usr/local/pgsql/data -l logfile start
/usr/local/pgsql/bin/createdb test
/usr/local/pgsql/bin/psql test
```

### #编译安装步骤

```
#安装相关依赖
[root@pgsql postgresql-14.2]# 
yum -y install gcc make readline-devel zlib-devel

#编译安装
./configure --prefix=/apps/pgsql
make -j 2 world
make install-world

#创建pgsql账号密码
useradd -s /bin/bash -m -d /home/postgres postgres
echo -e '123456\n123456' |passwd postgres

[root@pgsql postgresql-14.2]# useradd -s /bin/bash -m -d /home/postgres postgres
[root@pgsql postgresql-14.2]# echo -e '123456\n123456' |passwd postgres
Changing password for user postgres.
New password: BAD PASSWORD: The password is shorter than 8 characters
Retype new password: passwd: all authentication tokens updated successfully.

#创建数据目录并授权
mkdir -pv /pgsql/data/
chown -R postgres.postgres /pgsql/data/

#切换账号
su - postgres

#设置环境变量
[root@pgsql ~]# vim /etc/profile.d/pgslq.sh

#!/bin/bash
#
#********************************************************************
#Author:            dingbaohang
#QQ:                904748581
#Date:              2023-07-18
#FileName：         /etc/profile.d/pgslq.sh
#URL:               www.dingbh.top
#Description：      The test script
#Copyright (C):     2023 All rights reserved
#********************************************************************
export PGHOME=/apps/pgsql
export PATH=$PGHOME/bin/:$PATH
export PGDATA=/pgsql/data
export PGUSER=postgres
export MANPATH=/apps/pgsql/share/man:$MANPATH  

#生效环境变量
. /etc/profile.d/pgslq.sh

#初始化数据库
initdb -D /pgsql/data

#启动数据库并生成日志
pg_ctl -l logfile start


#创建开机自启动服务：方法1
cp /usr/local/src/postgresql-14.2/contrib/start-scripts/linux /etc/init.d/postgresql
chmod +x /etc/init.d/postgresql

#修改启动文件
/etc/init.d/postgresql
# Installation prefix
prefix=/apps/pgsql

# Data directory
PGDATA="/pgsql/data"

#设置开机自启动
[root@pgsql ~]# vim /etc/rc.local 

#ubuntu设置开机启动
#!/bin/bash
#/etc/init.d/postgresql start  
su - postgres -c "/apps/pgsql/bin/pg_ctl -l logfile start"

#设置启动service文件，方法二：
[root@pgsql ~]# vim /lib/systemd/system/postgresql.service
[Unit]                                                                                                        Description=PostgreSQL database server
After=network.target

[Service]
User=postgres
Group=postgres

ExecStart=/apps/pgsql/bin/postmaster -D /pgsql/data/
ExecReload=/bin/kill -HUP

[Install]
WantedBy=multi-user.target     

#查看service文件
[root@pgsql ~]# ll /lib/systemd/system/postgresql.service 
-rw-r--r-- 1 root root 389 Jul 18 22:54 /lib/systemd/system/postgresql.service

#生效配置
[root@pgsql ~]# systemctl daemon-reload 

#设置自启动
[root@pgsql ~]# systemctl enable --now postgresql.service

```



**pgsql启动service文件**

```
[root@postgers system]# vim postgresql@.service


[Unit]
Description=PostgreSQL Cluster %i
AssertPathExists=/etc/postgresql/%I/postgresql.conf
RequiresMountsFor=/etc/postgresql/%I /var/lib/postgresql/%I
PartOf=postgresql.service
ReloadPropagatedFrom=postgresql.service
Before=postgresql.service
After=network.target

[Service]
Type=forking
ExecStart=-/usr/bin/pg_ctlcluster --skip-systemctl-redirect %i start
TimeoutStartSec=0
ExecStop=/usr/bin/pg_ctlcluster --skip-systemctl-redirect -m fast %i stop
TimeoutStopSec=1h
ExecReload=/usr/bin/pg_ctlcluster --skip-systemctl-redirect %i reload
PIDFile=/run/postgresql/%i.pid
SyslogIdentifier=postgresql@%i
OOMScoreAdjust=-900

[Install]
WantedBy=multi-user.target
```



## 2、实现远程连接数据库

```
#主机添加远程认账权限
[root@pgsql ~]# vim /pgsql/data/pg_hba.conf

92 # IPv6 local connections:
 93 host    all             all             ::1/128                 trust
 94 host    all             all              0.0.0.0/0              md5

#client连接
[postgres@postgers ~]$ apt -y install postgresql
[postgres@postgers ~]$ psql -h 192.168.188.88 postgres
```



## 3、查看创建的数据库文件对应的表

```
#数据库表目录结构及oid
postgres=# select oid,datname from pg_database;
  oid  |  datname  
-------+-----------
 13975 | postgres
 16387 | db1
     1 | template1
 13974 | template0
 16388 | db2
(5 rows)

#系统中路径
[root@pgsql ~]# ls /pgsql/data/base/
1  13974  13975  16387  16388

#\l相当于show databases
postgres=# \l
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres

#显示数据库详细信息
postgres=# \l+
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |                       | 8561 kB | pg_default | default administrative connection database
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +| 8401 kB | pg_default | unmodifiable empty database
           |          |          |             |             | postgres=CTc/postgres |         |            | 
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +| 8553 kB | pg_default | default template for new databases
           |          |          |             |             | postgres=CTc/postgres |         |            | 

#利用scheme进行表分类
postgres=# create schema n75_sch;
CREATE SCHEMA

postgres=# create table n75_sch.t1(id int);
CREATE TABLE

postgres=# create schema n77_sch;
CREATE SCHEMA

postgres=# create table n77_sch.t1(id int);
CREATE TABLE

postgres=# \dt n75_sch.t1;
 n75_sch | t1   | table | postgres

postgres=# \dt n77_sch.t1;
 n77_sch | t1   | table | postgres

```

## 4、管理表

```
#创建数据库
postgres=# create database testdb;
CREATE DATABASE
#切换数据库testdb
postgres=# \c testdb;
You are now connected to database "testdb" as user "postgres".

#切换数据库
postgres=# \c testdb 
You are now connected to database "testdb" as user "postgres".
#创建表
testdb=# create table tb1(id serial primary key,name text);
CREATE TABLE
#创建数据
testdb=# insert into tb1(name) select (md5(random()::text)) from generate_series (2,10);
INSERT 0 9

testdb=# create table tb2(id serial primary key,name text);
CREATE TABLE
testdb=# insert into tb2(name) select (md5(random()::text)) from generate_series (2,10);
INSERT 0 9

#查看表内容
testdb=# select * from tb1;
 id |               name               
----+----------------------------------
  1 | e63cf7cba0cc397de638ba1d1b9f51c2
  2 | a7dd1dabb91657a68643633b92752777
  3 | 84a74c6f735a14ffc6324a22e93e5913
  4 | f8710e7a9fc245f567dccffe5b82a096
  5 | c1ebc2581942618bbad6ec7f5131c95d
  6 | 19d4fc4c81bbb679389c98eb5fc7c6c4
  7 | fdcd6825e6e35ca0e218dc7da0f1f689
  8 | eadeacd462dce74dc178ac23e90b0cef
  9 | 9c9bc16fcf7b0967c70e91d76efccd5e
(9 rows)

#\dt相当与show tables；
testdb=# \dt tb1;
        List of relations
 Schema | Name | Type  |  Owner   
--------+------+-------+----------
 public | tb1  | table | postgres
(1 row)
#创建并复制表结构
testdb=# create table tb2 (like tb1);
CREATE TABLE

#\dt查看表结构，相当于show tables
testdb=# \dt tb2;
        List of relations
 Schema | Name | Type  |  Owner   
--------+------+-------+----------
 public | tb2  | table | postgres
(1 row)
#查看tb2表内容
testdb=# select * from tb2;
 id | name 
----+------
(0 rows)
#查看所有表信息
testdb=# \dt
 public | tb1  | table | postgres
 public | tb2  | table | postgres
#表详情
testdb=# \dt+
 public | tb1  | table | postgres | permanent   | heap          | 16 kB | 
 public | tb2  | table | postgres | permanent   | heap          | 16 kB | 
 
 #查看表结构
 testdb=# \d tb1;
 id     | integer |           | not null | nextval('tb1_id_seq'::regclass)
 name   | text    |           |          | 

#查看表id及对应文件
testdb=# select * from pg_stat_all_tables where relname='tb1';
-[ RECORD 1 ]-------+-------
relid               | 16394
schemaname          | public
relname             | tb1
seq_scan            | 2
seq_tup_read        | 9
idx_scan            | 0
idx_tup_fetch       | 0
n_tup_ins           | 9
n_tup_upd           | 0
n_tup_del           | 0
n_tup_hot_upd       | 0
n_live_tup          | 9
n_dead_tup          | 0
n_mod_since_analyze | 9
n_ins_since_vacuum  | 9
last_vacuum         | 
last_autovacuum     | 
last_analyze        | 
last_autoanalyze    | 
vacuum_count        | 0
autovacuum_count    | 0
analyze_count       | 0
autoanalyze_count   | 0

testdb=# drop table tb1;
DROP TABLE
testdb=# select * from pg_stat_all_tables where relname='tb1';
(0 rows)

[root@postgers 16392]# ls
112        13583      16393  2579      2606_fsm  2613      2651  2669      2689      2830      2995      3381      3501      3601      3764_vm  4157  549
113        13584      16394  2600      2606_vm   2615      2652  2670      2690      2831      2996      3394      3502      3601_fsm  3766     4158  6102
1247       13584_fsm  16398  2600_fsm  2607      2615_fsm  2653  2673      2691      2832      3079      3394_fsm  3503      3601_vm   3767     4159  6104
1247_fsm   13584_vm   16399  2600_vm   2607_fsm  2615_vm   2654  2673_fsm  2692      2833      3079_fsm  3394_vm   3534      3602      3997     4160  6106
1247_vm    13587      16400  2601      2607_vm   2616      2655  2674      2693      2834      3079_vm   3395      3541      3602_fsm  4143     4163  6110
1249       13588      16402  2601_fsm  2608      2616_fsm  2656  2674_fsm  2696      2835      3080      3429      3541_fsm  3602_vm   4144     4164  6111
1249_fsm   13589      16403  2601_vm   2608_fsm  2616_vm   2657  2675      2699      2836      3081      3430      3541_vm   3603      4145     4165  6112
1249_vm    13589_fsm  16407  2602      2608_vm   2617      2658  2678      2701      2836_fsm  3085      3431      3542      3603_fsm  4146     4166  6113
1255       13589_vm   16408  2602_fsm  2609      2617_fsm  2659  2679      2702      2836_vm   3118      3433      3574      3603_vm   4147     4167  6117
1255_fsm   13592      16409  2602_vm   2609_fsm  2617_vm   2660  2680      2703      2837      3119      3439      3575      3604      4148     4168  6175
1255_vm    13593      174    2603      2609_vm   2618      2661  2681      2704      2838      3164      3440      3576      3605      4149     4169  6176
1259       13594      175    2603_fsm  2610      2618_fsm  2662  2682      2753      2838_fsm  3256      3455      3596      3606      4150     4170  826
1259_fsm   13594_fsm  2187   2603_vm   2610_fsm  2618_vm   2663  2683      2753_fsm  2838_vm   3257      3456      3597      3607      4151     4171  827
1259_vm    13594_vm   2224   2604      2610_vm   2619      2664  2684      2753_vm   2839      3258      3456_fsm  3598      3608      4152     4172  828
13579      13597      2228   2605      2611      2619_fsm  2665  2685      2754      2840      3350      3456_vm   3599      3609      4153     4173  pg_filenode.map
13579_fsm  13598      2328   2605_fsm  2612      2619_vm   2666  2686      2755      2840_fsm  3351      3466      3600      3712      4154     4174  pg_internal.init
13579_vm   1417       2336   2605_vm   2612_fsm  2620      2667  2687      2756      2840_vm   3379      3467      3600_fsm  3764      4155     5002  PG_VERSION
13582      1418       2337   2606      2612_vm   2650      2668  2688      2757      2841      3380      3468      3600_vm   3764_fsm  4156     548

#确认磁盘文件是否存在
[root@postgers 16392]# find -name 16394

#以列展示，相当与 /G；
testdb=# \x
Expanded display is on.

#查看指定表对应的文件
testdb=# select * from pg_relation_filepath('tb2');
-[ RECORD 1 ]--------+-----------------
pg_relation_filepath | base/16392/16403


#查看当前库所有表的统计信息
[root@postgers ~]# su - postgres 
[postgres@postgers ~]$ psql
psql (14.8 (Ubuntu 14.8-0ubuntu0.22.04.1))
Type "help" for help.

postgres=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------+----------+----------+-------------+-------------+-----------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 testdb    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
(4 rows)

Did not find any relations.
postgres=# \c testdb 
You are now connected to database "testdb" as user "postgres".
testdb=# \dt
        List of relations
 Schema | Name | Type  |  Owner   
--------+------+-------+----------
 public | tb1  | table | postgres
 public | tb2  | table | postgres
(2 rows)

testdb=# select * from pg_stat_all_tables where relname='tb1';
testdb=# \x
Expanded display is on.
testdb=# select * from pg_stat_all_tables where relname='tb1';
-[ RECORD 1 ]-------+-------
relid               | 16413
schemaname          | public
relname             | tb1
seq_scan            | 0
seq_tup_read        | 0
idx_scan            | 
idx_tup_fetch       | 
n_tup_ins           | 1
n_tup_upd           | 0
n_tup_del           | 0
n_tup_hot_upd       | 0
n_live_tup          | 1
n_dead_tup          | 0
n_mod_since_analyze | 1
n_ins_since_vacuum  | 1
last_vacuum         | 
last_autovacuum     | 
last_analyze        | 
last_autoanalyze    | 
vacuum_count        | 0
autovacuum_count    | 0
analyze_count       | 0
autoanalyze_count   | 0

testdb=# select * from pg_stat_all_tables where relname='tb2';
-[ RECORD 1 ]-------+-------
relid               | 16403
schemaname          | public
relname             | tb2
seq_scan            | 1
seq_tup_read        | 0
idx_scan            | 0
idx_tup_fetch       | 0
n_tup_ins           | 9
n_tup_upd           | 0
n_tup_del           | 0
n_tup_hot_upd       | 0
n_live_tup          | 9
n_dead_tup          | 0
n_mod_since_analyze | 9
n_ins_since_vacuum  | 9
last_vacuum         | 
last_autovacuum     | 
last_analyze        | 
last_autoanalyze    | 
vacuum_count        | 0
autovacuum_count    | 0
analyze_count       | 0
autoanalyze_count   | 0

#查看表relid
testdb=# select relid from pg_stat_all_tables where relname='tb1';
-[ RECORD 1 ]
relid | 16413

testdb=# select relid from pg_stat_all_tables where relname='tb2';
-[ RECORD 1 ]
relid | 16403

#查看数据库的名
testdb=# select oid,datname from pg_database where datname='testdb';
-[ RECORD 1 ]---
oid     | 16392
datname | testdb

#对应磁盘文件
[root@postgers ]# ls /var/lib/postgresql/14/main/base/
1  13760  13761  16392
```

## 5、表的CRUD

SQL的CRUD,  insert，update，delect，select四条语句

```
#查看数据类型
testdb=# select typname from pg_type;

                typname                 
----------------------------------------
 bool
 bytea
 char
 name
 int8
 int2
 int2vector
 int4
 regproc
 text
 oid
 tid
 xid
 cid
 oidvector
 pg_type
 pg_attribute
 pg_proc
 pg_class
 json
 xml

实例：

testdb=# create table tb3(id serial,name varchar(10));
CREATE TABLE

testdb=# \d
             List of relations
 Schema |    Name    |   Type   |  Owner   
--------+------------+----------+----------
 public | tb1        | table    | postgres
 public | tb1_id_seq | sequence | postgres
 public | tb2        | table    | postgres
 public | tb2_id_seq | sequence | postgres
 public | tb3        | table    | postgres
 public | tb3_id_seq | sequence | postgres
(6 rows)

查看当前数据库中所以表，相当于show tables；
testdb=# \dt;
        List of relations
 Schema | Name | Type  |  Owner   
--------+------+-------+----------
 public | tb1  | table | postgres
 public | tb2  | table | postgres
 public | tb3  | table | postgres
(3 rows)

#查看tb3表结构，相当于desc tb3；
testdb=# \dt tb3;
        List of relations
 Schema | Name | Type  |  Owner   
--------+------+-------+----------
 public | tb3  | table | postgres
(1 row)

#插入数据
testdb=# insert into tb3 (name)values('ding');
INSERT 0 1
testdb=# insert into tb3 (name)values('bao');
INSERT 0 1

testdb=# select * from tb3;
 id | name 
----+------
  1 | shi
  2 | bao
(2 rows)


#update一条数据
testdb=# update tb3 set name='shi' where id=1;
UPDATE 1
testdb=# update tb3 set name='hang' where id=2;
UPDATE 1

testdb=# select * from tb3;
 id | name 
----+------
  1 | shi
  2 | hang
(2 rows)

#delete表数据
testdb=# delete from tb3 where id=3;
DELETE 1
testdb=# select * from tb3;
 id | name 
----+------
  1 | shi
(1 row)

#清空表
testdb=# truncate table tb3;  //table可省略truncate tb3 ，与mysql不同点
TRUNCATE TABLE
testdb=# select * from tb3;
 id | name 
----+------
(0 rows)


实例：
testdb=# select generate_series(3,6);
 generate_series 
-----------------
               3
               4
               5
               6
(4 rows)


#插入100000条记录时间3758.300 ms
testdb=# create table tb3(id serial primary key,name text);
CREATE TABLE

testdb=# insert into tb3(name) select (md5(random()::text)) from generate_series (1,1000000);
INSERT 0 1000000

testdb=# \timing on
Timing is on.

testdb=# insert into tb3(name) select (md5(random()::text)) from generate_series (1,1000000);
INSERT 0 1000000
Time: 3758.300 ms (00:03.758)

```

## 6、索引管理

```
#创建表
testdb=# create table tb1(id int,info text,crt_time timestamp);
CREATE TABLE
Time: 8.054 ms

#插入数据
testdb=# insert into tb1 select generate_series (1,1000000),(md5(random()::text)),clock_timestamp();
INSERT 0 1000000
Time: 1968.414 ms (00:01.968)
testdb=# selc

#查看表内容
testdb=# select * from tb1;

   id    |               info               |          crt_time          
---------+----------------------------------+----------------------------
       1 | 359211e93d725c42c94a1c769130157b | 2023-07-23 02:50:18.560665
       2 | 443154fb6e327aef2f0680c2aaa9222b | 2023-07-23 02:50:18.560935
       3 | 2b13957f3f845c76f2f35f9adf070fed | 2023-07-23 02:50:18.560939
       4 | 60b217d1ba9648095d9059290c90c72c | 2023-07-23 02:50:18.56094
       5 | c40eef29b287a5e64e2c6325c6740064 | 2023-07-23 02:50:18.560942
       6 | 304a3bffa76172e8747b3aeaa01f7176 | 2023-07-23 02:50:18.560943
       7 | 9904065cca9d747c7d723575a02f70dd | 2023-07-23 02:50:18.560945
       8 | d82e3b7a376dbf5cf59c3dab726618d2 | 2023-07-23 02:50:18.560946
       9 | 7f8871da63b11dd33ec16b00c270f771 | 2023-07-23 02:50:18.560948
      10 | 19a590e956919231646f6a559bb5a25c | 2023-07-23 02:50:18.560949
      11 | 1223c6114e14eb44626662c6085666a9 | 2023-07-23 02:50:18.56095
      12 | 6af0efec96a10d8443eaa96e4d928814 | 2023-07-23 02:50:18.560952
      13 | 971ed122b7965250529c1a346e58b57e | 2023-07-23 02:50:18.560953
      14 | a6d06dcecb4c85f5c761cd0952944fa8 | 2023-07-23 02:50:18.560955
      15 | 796261191e4ab122a4e33280990cc25b | 2023-07-23 02:50:18.560956
      16 | 835d5ac05c6ee8a1d4fe6e196864cff2 | 2023-07-23 02:50:18.560958
      17 | 1a0b5c641c07ef633b96d595ddf41eb1 | 2023-07-23 02:50:18.560959
      18 | ab0a99c8c702141d4ea40e3fd8e5777e | 2023-07-23 02:50:18.56096
      19 | 0b89ce2eac322922da1a464b6a76916e | 2023-07-23 02:50:18.560962
      20 | 29f25dc9af38006b1b86f2b0b1677e2c | 2023-07-23 02:50:18.560963
      21 | f03b3d3915d9a4a65a3b2c390553d99a | 2023-07-23 02:50:18.560965
      22 | a0d130632a451420ff51a55761ccab06 | 2023-07-23 02:50:18.560966
      23 | 44e82f8cc6d5c5e0651d1bb823c57e4a | 2023-07-23 02:50:18.560968
      24 | 1a33db7857beec0c31dac6cd5e37b78a | 2023-07-23 02:50:18.560969
      25 | 5eb293b03c8a364c8ade47563bdf7345 | 2023-07-23 02:50:18.56097
      26 | 76dc61ef40c42e784ea2c7a71babdbdc | 2023-07-23 02:50:18.560972
      27 | d1e71b9eafde16a8e0c8049bcbcd88bc | 2023-07-23 02:50:18.560973
      28 | aa870c4a51413d03c27bee95cb73e003 | 2023-07-23 02:50:18.560975
      29 | f145dd8ba68d582ee3009e1f54e3a5e9 | 2023-07-23 02:50:18.560976
      30 | ba39359ab53d60e41a41573f21d4e9b5 | 2023-07-23 02:50:18.560978
      31 | f01f45654ca7429334b40215cd147767 | 2023-07-23 02:50:18.560979
      32 | 1118a1370c032f99ad4c1c9124d03fef | 2023-07-23 02:50:18.560981
      33 | 431dc362c7bf32234ae43c6b1f1c552f | 2023-07-23 02:50:18.560982
      34 | c331e2d170c643402440a0fdd8f6fe33 | 2023-07-23 02:50:18.560983
:

#创建索引
testdb=# create index  idx_tb1 on tb1(id);
CREATE INDEX
Time: 282.265 ms
testdb=# \d tb1;
                           Table "public.tb1"
  Column  |            Type             | Collation | Nullable | Default 
----------+-----------------------------+-----------+----------+---------
 id       | integer                     |           |          | 
 info     | text                        |           |          | 
 crt_time | timestamp without time zone |           |          | 
Indexes:
    "idx_tb1" btree (id)


#查询条件是否为索引列
testdb=# explain analyze select * from tb1 where id=9999;
                                                  QUERY PLAN                                                  
--------------------------------------------------------------------------------------------------------------
 Index Scan using idx_tb1 on tb1  (cost=0.42..8.44 rows=1 width=45) (actual time=0.034..0.035 rows=1 loops=1)
   Index Cond: (id = 9999)
 Planning Time: 0.173 ms
 Execution Time: 0.048 ms
(4 rows)

Time: 0.874 ms

#全盘扫描
testdb=# explain analyze select * from tb1 where info= 'ab0a99c8c702141d4ea40e3fd8e5777e';
                                                    QUERY PLAN                                                     
-------------------------------------------------------------------------------------------------------------------
 Gather  (cost=1000.00..15554.43 rows=1 width=45) (actual time=0.330..41.140 rows=1 loops=1)
   Workers Planned: 2
   Workers Launched: 2
   ->  Parallel Seq Scan on tb1  (cost=0.00..14554.33 rows=1 width=45) (actual time=21.906..34.424 rows=0 loops=3)
         Filter: (info = 'ab0a99c8c702141d4ea40e3fd8e5777e'::text)
         Rows Removed by Filter: 333333
 Planning Time: 0.063 ms
 Execution Time: 41.162 ms
(8 rows)

Time: 41.701 ms

#关闭索引，使用全盘扫描
testdb=# set enable_indexscan =off;
SET
Time: 0.838 ms
testdb=# explain analyze select * from tb1 where id=9999;
                                                   QUERY PLAN                                                   
----------------------------------------------------------------------------------------------------------------
 Bitmap Heap Scan on tb1  (cost=4.43..8.45 rows=1 width=45) (actual time=0.042..0.042 rows=1 loops=1)
   Recheck Cond: (id = 9999)
   Heap Blocks: exact=1
   ->  Bitmap Index Scan on idx_tb1  (cost=0.00..4.43 rows=1 width=0) (actual time=0.013..0.014 rows=1 loops=1)
         Index Cond: (id = 9999)
 Planning Time: 0.108 ms
 Execution Time: 0.062 ms
(7 rows)

Time: 0.625 ms
#开启索引
testdb=# set enable_indexscan =on;
SET
Time: 0.328 ms

#删除索引
testdb=# drop index idx_tb1;
DROP INDEX
Time: 4.781 ms
```



## 7、表空间

```
#列出所有表空间
testdb=# \db
       List of tablespaces
    Name    |  Owner   | Location 
------------+----------+----------
 pg_default | postgres | 
 pg_global  | postgres | 
(2 rows)

testdb=# \db+
                                  List of tablespaces
    Name    |  Owner   | Location | Access privileges | Options |  Size  | Description 
------------+----------+----------+-------------------+---------+--------+-------------
 pg_default | postgres |          |                   |         | 137 MB | 
 pg_global  | postgres |          |                   |         | 560 kB | 
(2 rows)

#复制表到文件中

#创建表空间
[postgres@pgsql ~]$ pwd
/home/postgres
[postgres@pgsql ~]$ mkdir ts1
[postgres@pgsql ~]$ ll
total 4
-rw------- 1 postgres postgres 2178 Jul 19 02:36 logfile
drwxrwxr-x 2 postgres postgres    6 Jul 21 16:09 ts1

#创建表空间
testdb=# create tablespace ts1 location '/home/postgres/ts1/';
CREATE TABLESPACE
Time: 4.111 ms


testdb=# select * from pg_relation_filepath('tb4');
            pg_relation_filepath             
---------------------------------------------
 pg_tblspc/16414/PG_14_202107181/16397/16415
(1 row)

Time: 0.206 ms

#判断是否与上述路径一致
[postgres@pgsql ~]$ ll /pgsql/data/pg_tblspc/16414/PG_14_202107181/16397/16415 
-rw------- 1 postgres postgres 0 Jul 21 16:23 /pgsql/data/pg_tblspc/16414/PG_14_202107181/16397/16415

#删除表空间
testdb=# drop table tb4 ;
DROP TABLE
Time: 1.102 ms

#删除表空间一定为空，若不为空则需删除存在表
testdb=# drop tablespace ts1 ;
DROP TABLESPACE
Time: 6.180 ms

```

## 8、查看系统信息

```go
#查看系统版本
testdb=# select version();
                                                 version                                                 
---------------------------------------------------------------------------------------------------------
 PostgreSQL 14.2 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 8.5.0 20210514 (Red Hat 8.5.0-18), 64-bit
(1 row)

Time: 0.269 ms

#查看启动时间
testdb=# select pg_postmaster_start_time();
   pg_postmaster_start_time    
-------------------------------
 2023-07-21 15:51:11.482102+08
(1 row)

Time: 0.249 ms

#查看加载配置文件的时间
testdb=# select pg_conf_load_time();
       pg_conf_load_time       
-------------------------------
 2023-07-21 17:09:57.111217+08
(1 row)

Time: 0.629 ms

#查看当前时间
testdb=# select now();
             now              
------------------------------
 2023-07-21 17:20:22.31219+08
(1 row)

Time: 0.814 ms

#查看当前用户
testdb=# select user;
   user   
----------
 postgres
(1 row)
testdb=# select current_user;
 current_user 
--------------
 postgres
(1 row)

Time: 0.169 ms

#查看当前连接的数据库
testdb=# select current_database();
 current_database 
------------------
 testdb
(1 row)

Time: 0.260 ms


#查看当前session客户端IP与端口
postgres=# \conninfo
You are connected to database "postgres" as user "postgres" on host "192.168.188.88" at port "5432".
postgres=# select inet_client_addr(),inet_client_port();
 inet_client_addr | inet_client_port 
------------------+------------------
 192.168.188.20   |            45208
(1 row)

#查看当前session所连接的数据库服务器的IP和端口
postgres=# select inet_server_addr(),inet_server_port();
 inet_server_addr | inet_server_port 
------------------+------------------
 192.168.188.88   |             5432
(1 row)

#查询当前session对应的后台服务时程pid
postgres=# select pg_bac

postgres=# select pg_backend_pid();
 pg_backend_pid 
----------------
          30786
(1 row)

#查看当前内置变量
postgres=# \set
AUTOCOMMIT = 'on'
COMP_KEYWORD_CASE = 'preserve-upper'
DBNAME = 'postgres'
ECHO = 'none'
ECHO_HIDDEN = 'off'
ENCODING = 'UTF8'
ERROR = 'false'
FETCH_COUNT = '0'
HIDE_TABLEAM = 'off'
HIDE_TOAST_COMPRESSION = 'off'
HISTCONTROL = 'none'
HISTSIZE = '500'
HOST = '192.168.188.88'
IGNOREEOF = '0'
LAST_ERROR_MESSAGE = 'syntax error at or near "("'
LAST_ERROR_SQLSTATE = '42601'
ON_ERROR_ROLLBACK = 'off'
ON_ERROR_STOP = 'off'
PORT = '5432'
PROMPT1 = '%/%R%x%# '
PROMPT2 = '%/%R%x%# '
PROMPT3 = '>> '
QUIET = 'off'
ROW_COUNT = '1'
SERVER_VERSION_NAME = '14.2'
SERVER_VERSION_NUM = '140002'
SHOW_CONTEXT = 'errors'
SINGLELINE = 'off
SINGLESTEP = 'off'
SQLSTATE = '00000'
USER = 'postgres'
VERBOSITY = 'default'
VERSION = 'PostgreSQL 14.8 (Ubuntu 14.8-0ubuntu0.22.04.1) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 11.3.0-1ubuntu1~22.04.1) 11.3.0, 64-bit'
VERSION_NAME = '14.8 (Ubuntu 14.8-0ubuntu0.22.04.1)'
VERSION_NUM = '140008'

#查看当前指定配置
postgres=# \h show
Command:     SHOW
Description: show the value of a run-time parameter
Syntax:
SHOW name
SHOW ALL

URL: https://www.postgresql.org/docs/14/sql-show.html
方法一
postgres=# show max_connections;
 max_connections 
-----------------
 100
(1 row)

Time: 0.199 ms
方法二
postgres=# select current_setting('max_connections');
 current_setting 
-----------------
 100
(1 row)

Time: 0.564 ms

#查看所有设置名称
postgres=# select name from pg_settings;
                  name                  
----------------------------------------
 allow_system_table_mods
 application_name
 archive_cleanup_command
 archive_command
 archive_mode
 archive_timeout
 array_nulls
 authentication_timeout
 autovacuum
 autovacuum_analyze_scale_factor
 autovacuum_analyze_threshold
 autovacuum_freeze_max_age
 autovacuum_max_workers
 autovacuum_multixact_freeze_max_age
 autovacuum_naptime
 autovacuum_vacuum_cost_delay
 autovacuum_vacuum_cost_limit
 autovacuum_vacuum_insert_scale_factor
 autovacuum_vacuum_insert_threshold
 autovacuum_vacuum_scale_factor
......
#查看当前设置名和值
postgres=# select name,setting from pg_settings;
                  name                  |            setting             
----------------------------------------+--------------------------------
 allow_system_table_mods                | off
 application_name                       | psql
 archive_cleanup_command                | 
 archive_command                        | (disabled)
 archive_mode                           | off
 archive_timeout                        | 0
 array_nulls                            | on
 authentication_timeout                 | 60
 autovacuum                             | on
 autovacuum_analyze_scale_factor        | 0.1
 autovacuum_analyze_threshold           | 50
 autovacuum_freeze_max_age              | 200000000
 autovacuum_max_workers                 | 3
 autovacuum_multixact_freeze_max_age    | 400000000
 autovacuum_naptime                     | 60
 autovacuum_vacuum_cost_delay           | 2
 autovacuum_vacuum_cost_limit           | -1
 autovacuum_vacuum_insert_scale_factor  | 0.2
 autovacuum_vacuum_insert_threshold     | 1000
 autovacuum_vacuum_scale_factor         | 0.2
 autovacuum_vacuum_threshold            | 50
 autovacuum_work_mem                    | -1
 backend_flush_after                    | 0
 backslash_quote                        | safe_encoding
 backtrace_functions                    | 
 bgwriter_delay                         | 200
 bgwriter_flush_after                   | 64
 bgwriter_lru_maxpages                  | 100
 bgwriter_lru_multiplier                | 2
 block_size                             | 8192
 bonjour                                | off
 bonjour_name                           | 
 bytea_output                           | hex
 check_function_bodies                  | on
--More--

#查看二进制选项，默认为关闭
postgres=# show archive_mode;
 archive_mode 
--------------
 off
(1 row)

Time: 0.326 ms

#导入数据库
[postgres@pgsql ~]$ psql < hellodb.sql 
CREATE DATABASE
You are now connected to database "hellodb" as user "postgres".
NOTICE:  table "classes" does not exist, skipping
DROP TABLE
CREATE TABLE
INSERT 0 8
NOTICE:  table "coc" does not exist, skipping
DROP TABLE
CREATE TABLE
INSERT 0 14
NOTICE:  table "courses" does not exist, skipping
DROP TABLE
CREATE TABLE
INSERT 0 7
NOTICE:  table "scores" does not exist, skipping
DROP TABLE
CREATE TABLE
INSERT 0 15
NOTICE:  table "students" does not exist, skipping
DROP TABLE
CREATE TABLE
INSERT 0 25
NOTICE:  table "teachers" does not exist, skipping
DROP TABLE
CREATE TABLE
INSERT 0 4
NOTICE:  table "toc" does not exist, skipping
DROP TABLE
CREATE TABLE

[postgres@pgsql ~]$ psql 
psql (14.2)
Type "help" for help.

postgres=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------+----------+----------+-------------+-------------+-----------------------
 db1       | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 db2       | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 hellodb   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 testdb    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
(7 rows)

postgres=# \c hellodb 
You are now connected to database "hellodb" as user "postgres".
hellodb=# \d
          List of relations
 Schema |   Name   | Type  |  Owner   
--------+----------+-------+----------
 public | classes  | table | postgres
 public | coc      | table | postgres
 public | courses  | table | postgres
 public | scores   | table | postgres
 public | students | table | postgres
 public | teachers | table | postgres
 public | toc      | table | postgres
(7 rows)


hellodb=# select * from students;
 stuid |     name      | age | gender | classid | teacherid 
-------+---------------+-----+--------+---------+-----------
     1 | Shi Zhongyu   |  22 | M      |       2 |         3
     2 | Shi Potian    |  22 | M      |       1 |         7
     3 | Xie Yanke     |  53 | M      |       2 |        16
     4 | Ding Dian     |  32 | M      |       4 |         4
     5 | Yu Yutong     |  26 | M      |       3 |         1
     6 | Shi Qing      |  46 | M      |       5 |          
     7 | Xi Ren        |  19 | F      |       3 |          
     8 | Lin Daiyu     |  17 | F      |       7 |          
     9 | Ren Yingying  |  20 | F      |       6 |          
    10 | Yue Lingshan  |  19 | F      |       3 |          
    11 | Yuan Chengzhi |  23 | M      |       6 |          
    12 | Wen Qingqing  |  19 | F      |       1 |          
    13 | Tian Boguang  |  33 | M      |       2 |          
    14 | Lu Wushuang   |  17 | F      |       3 |          
    15 | Duan Yu       |  19 | M      |       4 |          
    16 | Xu Zhu        |  21 | M      |       1 |          
    17 | Lin Chong     |  25 | M      |       4 |          
    18 | Hua Rong      |  23 | M      |       7 |          
    19 | Xue Baochai   |  18 | F      |       6 |          
    20 | Diao Chan     |  19 | F      |       7 |          
    21 | Huang Yueying |  22 | F      |       6 |          
    22 | Xiao Qiao     |  20 | F      |       1 |          
    23 | Ma Chao       |  23 | M      |       4 |          
    24 | Xu Xian       |  27 | M      |         |          
    25 | Sun Dasheng   | 100 | M      |         |          
(25 rows)
```

## 9、查看用户和权限

```
#查看所有用户
hellodb=# \du
                                   List of roles
 Role name |                         Attributes                         | Member of 
-----------+------------------------------------------------------------+-----------
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}

hellodb=# \du
                                   List of roles
 Role name |                         Attributes                         | Member of 
-----------+------------------------------------------------------------+-----------
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}

#查看当前用户
hellodb=# select user;
   user   
----------
 postgres
(1 row)

hellodb=# select current_user;
 current_user 
--------------
 postgres
(1 row)

#显示表、视图、序列的权限分配情况
hellodb=# \z
                              Access privileges
 Schema |   Name   | Type  | Access privileges | Column privileges | Policies 
--------+----------+-------+-------------------+-------------------+----------
 public | classes  | table |                   |                   | 
 public | coc      | table |                   |                   | 
 public | courses  | table |                   |                   | 
 public | scores   | table |                   |                   | 
 public | students | table |                   |                   | 
 public | teachers | table |                   |                   | 
 public | toc      | table |                   |                   | 
(7 rows)

hellodb=# \z students ;
                              Access privileges
 Schema |   Name   | Type  | Access privileges | Column privileges | Policies 
--------+----------+-------+-------------------+-------------------+----------
 public | students | table |                   |                   | 
(1 row)

\z: extra argument ";" ignored

#与\z功能相同
hellodb=# \dp;
                              Access privileges
 Schema |   Name   | Type  | Access privileges | Column privileges | Policies 
--------+----------+-------+-------------------+-------------------+----------
 public | classes  | table |                   |                   | 
 public | coc      | table |                   |                   | 
 public | courses  | table |                   |                   | 
 public | scores   | table |                   |                   | 
 public | students | table |                   |                   | 
 public | teachers | table |                   |                   | 
 public | toc      | table |                   |                   | 
(7 rows)

```

## 10、事务管理与锁

```
#关闭自动提交
hellodb=# \set AUTOCOMMIT off
#开启自动提交
hellodb=# \set AUTOCOMMIT on

#实例
hellodb=# \set AUTOCOMMIT off
hellodb=# insert into teachers values('8','h',30,'M');
INSERT 0 1
           
hellodb=*# select * from teachers ;
 tid |     name      | age | gender 
-----+---------------+-----+--------
   1 | Song Jiang    |  45 | M
   2 | Zhang Sanfeng |  94 | M
   3 | Miejue Shitai |  77 | F
   4 | Lin Chaoying  |  93 | F
   5 | d             |  25 | F
   6 | S             |  33 | M
   7 | p             |  33 | M
   8 | h             |  30 | M
(8 rows)


#远程连接机器
hellodb=# select * from teachers ;
 tid |     name      | age | gender 
-----+---------------+-----+--------
   1 | Song Jiang    |  45 | M
   2 | Zhang Sanfeng |  94 | M
   3 | Miejue Shitai |  77 | F
   4 | Lin Chaoying  |  93 | F
   5 | d             |  25 | F
   6 | S             |  33 | M
   7 | p             |  33 | M
(7 rows)


#rollback
hellodb=# rollback ;
WARNING:  there is no transaction in progress
ROLLBACK
hellodb=# select * from teachers ;
 tid |     name      | age | gender 
-----+---------------+-----+--------
   1 | Song Jiang    |  45 | M
   2 | Zhang Sanfeng |  94 | M
   3 | Miejue Shitai |  77 | F
   4 | Lin Chaoying  |  93 | F
   5 | d             |  25 | F
   6 | S             |  33 | M
   7 | p             |  33 | M
(7 rows)


#创建账号
postgres=# create user user1 with password '123456';
CREATE ROLE
 
postgres=# create role user2 with password '123456';
CREATE ROLE

#远程连接（user和role区别）
[root@postgers ~]# psql -h 192.168.188.88 -d hellodb -U user1
Password for user user1: 
psql (14.8 (Ubuntu 14.8-0ubuntu0.22.04.1), server 14.2)
Type "help" for help.

hellodb=> drop database hellodb ;
ERROR:  must be owner of database hellodb


hellodb=> \q

[root@postgers ~]# psql -h 192.168.188.88 -d hellodb -U user2
Password for user user2: 
psql: error: connection to server at "192.168.188.88", port 5432 failed: FATAL:  role "user2" is not permitted to log in

#创建数据库并指定所有者的用户
hellodb=# create database db3 OWNER user1;
CREATE DATABASE
hellodb=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
-----------+----------+----------+-------------+-------------+-----------------------
 db1       | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 db2       | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 db3       | user1    | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 hellodb   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 testdb    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |


#实例
创建数据库
hellodb=# create database pinxixi;
CREATE DATABASE
#连接数据库
hellodb=# \c pinxixi;
You are now connected to database "pinxixi" as user "postgres".
#创建数据库类组
pinxixi=# create schema wanrentuan;
CREATE SCHEMA
```



# pgsql体系架构

## 1、进程

```go
1、postmaster 主进程
2、bgwriter 后台写进程
3、walwriter 预写式日志
4、checkpointer 检查点操作
5、autovacuum 自动清理进程
6、pgarch 归档进程  #相当于mysql的二进制文件
7、pgstat 统计数据收集进程
8、syslogger 系统日志进程
9、startup 启动进程
10、session 会话进程


[root@pgsql ~]# ps auxf | grep ^postgres
postgres   29546  0.0  0.4 382560 17640 ?        Ss   Jul21   0:01 /apps/pgsql/bin/postgres -D /pgsql/data
postgres   29548  0.0  2.6 382676 99660 ?        Ss   Jul21   0:00  \_ postgres: checkpointer 
postgres   29549  0.0  0.1 382668  6896 ?        Ss   Jul21   0:00  \_ postgres: background writer 
postgres   29550  0.0  0.2 382560  8076 ?        Ss   Jul21   0:00  \_ postgres: walwriter 
postgres   29551  0.0  0.1 383236  6600 ?        Ss   Jul21   0:01  \_ postgres: autovacuum launcher 
postgres   29552  0.0  0.1 237584  4920 ?        Ss   Jul21   0:03  \_ postgres: stats collector 
postgres   29553  0.0  0.1 383108  5420 ?        Ss   Jul21   0:00  \_ postgres: logical replication launcher 
postgres   35581  0.0  0.2 383492  8368 ?        Ss   04:44   0:00  \_ postgres: postgres postgres 192.168.188.20(45530) idle

```

## 2、内存

```
#修改共享内存大小postgresql.conf文件，默认128M，生产要调大
postgres=# show shared_buffers ;
 shared_buffers 
----------------
 128MB
(1 row)

#观察本地内存空间
postgres=# show shared_buffers ;
 shared_buffers 
----------------
 128MB
(1 row)

postgres=# show temp_buffers ;
 temp_buffers 
--------------
 8MB
(1 row)

postgres=# show work_mem ;
 work_mem 
----------
 4MB
(1 row)

postgres=# show maintenance_work_mem ;
 maintenance_work_mem 
----------------------
 64MB
(1 row)

```

## 3、数据更新过程

```
1、先将数据库文件中的更改的数据加载至内存
2、在内存更新数据
3、将日志写入内存WAL的缓存区
4、将日志提交，将日志写入操作系统cache
5、同步日志到磁盘
6、后台写数据库的更新后的数据到操作系统cache
7、写完数据后，更新检查点checkpoint
```

## 4、pgdata相关配置文件进程路径

```

[postgres@pgsql ~]$ ls /pgsql/data -l

total 72
drwx------ 13 postgres postgres   149 Aug  2 00:33 base
drwx------  2 postgres postgres  4096 Aug  1 23:02 global
drwx------  2 postgres postgres     6 Jul 18 19:17 pg_commit_ts
drwx------  2 postgres postgres     6 Jul 18 19:17 pg_dynshmem
-rw-------  1 postgres postgres  4857 Jul 19 13:05 pg_hba.conf
-rw-------  1 postgres postgres  1636 Jul 18 19:17 pg_ident.conf
drwx------  4 postgres postgres    68 Aug  2 00:38 pg_logical
drwx------  4 postgres postgres    36 Jul 18 19:17 pg_multixact
drwx------  2 postgres postgres     6 Jul 18 19:17 pg_notify
drwx------  2 postgres postgres     6 Jul 18 19:17 pg_replslot
drwx------  2 postgres postgres     6 Jul 18 19:17 pg_serial
drwx------  2 postgres postgres     6 Jul 18 19:17 pg_snapshots
drwx------  2 postgres postgres     6 Aug  1 16:48 pg_stat
drwx------  2 postgres postgres   105 Aug  2 03:16 pg_stat_tmp
drwx------  2 postgres postgres    18 Jul 18 19:17 pg_subtrans
drwx------  2 postgres postgres     6 Jul 21 16:37 pg_tblspc
drwx------  2 postgres postgres     6 Jul 18 19:17 pg_twophase
-rw-------  1 postgres postgres     3 Jul 18 19:17 PG_VERSION
drwx------  3 postgres postgres  4096 Aug  1 23:22 pg_wal
drwx------  2 postgres postgres    18 Jul 18 19:17 pg_xact
-rw-------  1 postgres postgres    88 Jul 18 19:17 postgresql.auto.conf
-rw-------  1 postgres postgres 28770 Jul 19 13:00 postgresql.conf
-rw-------  1 postgres postgres    45 Aug  1 16:48 postmaster.opts
-rw-------  1 postgres postgres    75 Aug  1 16:48 postmaster.pid
-rw-rw-r--  1 postgres postgres  2213 Jul 18 23:02 serverlog

#进程路径及编号
[postgres@pgsql data]$ cat postmaster.opts 
/apps/pgsql/bin/postgres "-D" "/pgsql/data/"

[postgres@pgsql data]$ cat postmaster.pid
1655
/pgsql/data
1690879691
5432
/tmp
0.0.0.0
135821233         2
ready 

[postgres@pgsql data]$ cat PG_VERSION 
14

```

## 5、配置文件

```
[postgres@pgsql ~]$ vim /pgsql/data/postgresql.conf 
[postgres@pgsql ~]$ grep -A 3 '#------------------------------------------------------------------------------' /pgsql/data/postgresql.conf 

#查看修改配置
hellodb=# show port;
 port 
------
 5432
(1 row)

hellodb=# show timezone;
   TimeZone    
---------------
 Asia/Shanghai
(1 row)


hellodb=# \d pg_settings ;
               View "pg_catalog.pg_settings"
     Column      |  Type   | Collation | Nullable | Default 
-----------------+---------+-----------+----------+---------
 name            | text    |           |          | 
 setting         | text    |           |          | 
 unit            | text    |           |          | 
 category        | text    |           |          | 
 short_desc      | text    |           |          | 
 extra_desc      | text    |           |          | 
 context         | text    |           |          | 
 vartype         | text    |           |          | 
 source          | text    |           |          | 
 min_val         | text    |           |          | 
 max_val         | text    |           |          | 
 enumvals        | text[]  |           |          | 
 boot_val        | text    |           |          | 
 reset_val       | text    |           |          | 
 sourcefile      | text    |           |          | 
 sourceline      | integer |           |          | 
 pending_restart | boolean |           |          | 

\d: extra argument ";" ignored

hellodb=# select name,setting,short_desc from pg_settings ;
                  name                  |            setting             |                                                               short_desc                      
                                          
----------------------------------------+--------------------------------+-----------------------------------------------------------------------------------------------
------------------------------------------
 allow_system_table_mods                | off                            | Allows modifications of the structure of system tables.
 application_name                       | psql                           | Sets the application name to be reported in statistics and logs.
 archive_cleanup_command                |                                | Sets the shell command that will be executed at every restart point.
 archive_command                        | (disabled)                     | Sets the shell command that will be called to archive a WAL file.
 archive_mode                           | off                            | Allows archiving of WAL files using archive_command.
 archive_timeout                        | 0                              | Forces a switch to the next WAL file if a new file has not been started within N seconds.
 array_nulls                            | on                             | Enable input of NULL elements in arrays.
 authentication_timeout                 | 60                             | Sets the maximum allowed time to complete client authentication.
 autovacuum                             | on                             | Starts the autovacuum subprocess.
 autovacuum_analyze_scale_factor        | 0.1                            | Number of tuple inserts, updates, or deletes prior to analyze as a fraction of reltuples.
 autovacuum_analyze_threshold           | 50                             | Minimum number of tuple inserts, updates, or deletes prior to analyze.
 autovacuum_freeze_max_age              | 200000000                      | Age at which to autovacuum a table to prevent transaction ID wraparound.
 autovacuum_max_workers                 | 3                              | Sets the maximum number of simultaneously running autovacuum worker processes.
 autovacuum_multixact_freeze_max_age    | 400000000                      | Multixact age at which to autovacuum a table to prevent multixact wraparound.
 autovacuum_naptime                     | 60                             | Time to sleep between autovacuum runs.
 autovacuum_vacuum_cost_delay           | 2                              | Vacuum cost delay in milliseconds, for autovacuum.
 autovacuum_vacuum_cost_limit           | -1                             | Vacuum cost amount available before napping, for autovacuum.
 autovacuum_vacuum_insert_scale_factor  | 0.2                            | Number of tuple inserts prior to vacuum as a fraction of reltuples.
 autovacuum_vacuum_insert_threshold     | 1000                           | Minimum number of tuple inserts prior to vacuum, or -1 to disable insert vacuums.
 autovacuum_vacuum_scale_factor         | 0.2                            | Number of tuple updates or deletes prior to vacuum as a fraction of reltuples.
 autovacuum_vacuum_threshold            | 50                             | Minimum number of tuple updates or deletes prior to vacuum.
 autovacuum_work_mem                    | -1                             | Sets the maximum memory to be used by each autovacuum worker process.
 backend_flush_after                    | 0                              | Number of pages after which previously performed writes are flushed to disk.
 backslash_quote                        | safe_encoding                  | Sets whether "\'" is allowed in string literals.
 backtrace_functions                    |                                | Log backtrace for errors in these functions.
 bgwriter_delay                         | 200                            | Background writer sleep time between rounds.
 bgwriter_flush_after                   | 64                             | Number of pages after which previously performed writes are flushed to disk.
 bgwriter_lru_maxpages                  | 100                            | Background writer maximum number of LRU pages to flush per round.
 bgwriter_lru_multiplier                | 2                              | Multiple of the average buffer usage to free per round.
 block_size                             | 8192                           | Shows the size of a disk block.
 bonjour                                | off                            | Enables advertising the server via Bonjour.
 bonjour_name                           |                                | Sets the Bonjour service name.

#查看表对应磁盘文件路径
testdb=# \c hellodb 
You are now connected to database "hellodb" as user "postgres".
hellodb=# select pg_relation_filepath('teachers');
 pg_relation_filepath 
----------------------
 base/16418/16445
(1 row)

#查看版本
hellodb=# select version();
                                                 version                                                 
---------------------------------------------------------------------------------------------------------
 PostgreSQL 14.2 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 8.5.0 20210514 (Red Hat 8.5.0-18), 64-bit
(1 row)


```

## 6、日志文件

```
开启日志功能
[postgres@pgsql ~]$ vim /pgsql/data/postgresql.conf 

log_destination = 'csvlog' 
logging_collector = on      //添加或修改行

[postgres@pgsql ~]$ pg_ctl restart
waiting for server to shut down....2023-08-02 04:00:35.528 CST [4335] LOG:  received fast shutdown request
2023-08-02 04:00:35.529 CST [4335] LOG:  aborting any active transactions
2023-08-02 04:00:35.529 CST [4378] FATAL:  terminating connection due to administrator command
2023-08-02 04:00:35.530 CST [4335] LOG:  background worker "logical replication launcher" (PID 4342) exited with exit code 1
2023-08-02 04:00:35.530 CST [4337] LOG:  shutting down
2023-08-02 04:00:35.535 CST [4335] LOG:  database system is shut down
 done
server stopped
waiting for server to start....2023-08-02 04:00:35.638 CST [4526] LOG:  redirecting log output to logging collector process
2023-08-02 04:00:35.638 CST [4526] HINT:  Future log output will appear in directory "log".
 done
server started

#验证结果
postgres=# show logging_collector ;
 logging_collector 
-------------------
 on
(1 row)

[postgres@pgsql ~]$ ls /pgsql/data/    //log目录默认不存在
base              log           pg_hba.conf    pg_multixact  pg_serial     pg_stat_tmp  pg_twophase  pg_xact               postmaster.opts
current_logfiles  pg_commit_ts  pg_ident.conf  pg_notify     pg_snapshots  pg_subtrans  PG_VERSION   postgresql.auto.conf  postmaster.pid
global            pg_dynshmem   pg_logical     pg_replslot   pg_stat       pg_tblspc    pg_wal       postgresql.conf       serverlog

#查看日志
[postgres@pgsql ~]$ tail -f /pgsql/data/log/postgresql-2023-08-02_040035.log 
2023-08-02 04:00:35.638 CST [4526] LOG:  starting PostgreSQL 14.2 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 8.5.0 20210514 (Red Hat 8.5.0-18), 64-bit
2023-08-02 04:00:35.638 CST [4526] LOG:  listening on IPv4 address "0.0.0.0", port 5432
2023-08-02 04:00:35.640 CST [4526] LOG:  listening on Unix socket "/tmp/.s.PGSQL.5432"
2023-08-02 04:00:35.644 CST [4528] LOG:  database system was shut down at 2023-08-02 04:00:35 CST
2023-08-02 04:00:35.646 CST [4526] LOG:  database system is ready to accept connections

#查看当前二进制文件lsn号
postgres=# select pg_walfile_NAME_OFFSET(pg_current_wal_lsn());
       pg_walfile_name_offset        
-------------------------------------
 (000000010000000000000019,14041280)
(1 row)

postgres=# select pg_current_wal_lsn();
 pg_current_wal_lsn 
--------------------
 0/19D640C0
(1 row)



#查看wal文件二进制文件内容
[postgres@pgsql ~]$ ls /pgsql/data/pg_wal/ -l
total 180224
-rw------- 1 postgres postgres 16777216 Aug  2 04:32 000000010000000000000019
-rw------- 1 postgres postgres 16777216 Aug  1 16:52 00000001000000000000001A
-rw------- 1 postgres postgres 16777216 Aug  1 16:56 00000001000000000000001B
-rw------- 1 postgres postgres 16777216 Aug  1 16:56 00000001000000000000001C
-rw------- 1 postgres postgres 16777216 Aug  1 16:56 00000001000000000000001D
-rw------- 1 postgres postgres 16777216 Aug  1 16:56 00000001000000000000001E
-rw------- 1 postgres postgres 16777216 Aug  1 16:56 00000001000000000000001F
-rw------- 1 postgres postgres 16777216 Aug  1 23:13 000000010000000000000020
-rw------- 1 postgres postgres 16777216 Aug  1 23:13 000000010000000000000021
-rw------- 1 postgres postgres 16777216 Aug  1 23:13 000000010000000000000022
-rw------- 1 postgres postgres 16777216 Aug  1 23:13 000000010000000000000023
drwx------ 2 postgres postgres        6 Jul 18 19:17 archive_status

#使用pg_waldump工具查看wal文件
[postgres@pgsql pg_wal]$ pg_waldump /pgsql/data/pg_wal/000000010000000000000019 

hellodb=# insert into teachers(tid ,name,age,gender)values(8,'pan',18,'M');
INSERT 0 1

hellodb=# update teachers set age=50 where tid=6;
UPDATE 1

hellodb=# select * from teachers ;
 tid |     name      | age | gender 
-----+---------------+-----+--------
   1 | Song Jiang    |  45 | M
   2 | Zhang Sanfeng |  94 | M
   3 | Miejue Shitai |  77 | F
   4 | Lin Chaoying  |  93 | F
   5 | d             |  25 | F
   7 | p             |  33 | M
  11 | c             |  30 | M
   8 | pan           |  18 | M
   6 | S             |  50 | M
(9 rows)



[postgres@pgsql ~]$ pg_waldump /pgsql/data/pg_wal/000000010000000000000019 |tail -n 10
pg_waldump: fatal: error in WAL record at 0/19D648A8: invalid record length at 0/19D648E0: wanted 24, got 0
rmgr: Heap        len (rec/tot):     54/   654, tx:        810, lsn: 0/19D640C0, prev 0/19D64088, desc: INSERT off 14 flags 0x00, blkref #0: rel 1663/16418/16445 blk 0 FPW
rmgr: Btree       len (rec/tot):     53/   293, tx:        810, lsn: 0/19D64350, prev 0/19D640C0, desc: INSERT_LEAF off 9, blkref #0: rel 1663/16418/16449 blk 1 FPW
rmgr: Transaction len (rec/tot):     34/    34, tx:        810, lsn: 0/19D64478, prev 0/19D64350, desc: COMMIT 2023-08-02 18:42:50.772275 CST
rmgr: Standby     len (rec/tot):     50/    50, tx:          0, lsn: 0/19D644A0, prev 0/19D64478, desc: RUNNING_XACTS nextXid 811 latestCompletedXid 810 oldestRunningXid 811
rmgr: Standby     len (rec/tot):     50/    50, tx:          0, lsn: 0/19D644D8, prev 0/19D644A0, desc: RUNNING_XACTS nextXid 811 latestCompletedXid 810 oldestRunningXid 811
rmgr: XLOG        len (rec/tot):    114/   114, tx:          0, lsn: 0/19D64510, prev 0/19D644D8, desc: CHECKPOINT_ONLINE redo 0/19D644D8; tli 1; prev tli 1; fpw true; xid 0:811; oid 24614; multi 1; offset 0; oldest xid 726 in DB 1; oldest multi 1 in DB 1; oldest/newest commit timestamp xid: 0/0; oldest running xid 811; online
rmgr: Standby     len (rec/tot):     50/    50, tx:          0, lsn: 0/19D64588, prev 0/19D64510, desc: RUNNING_XACTS nextXid 811 latestCompletedXid 810 oldestRunningXid 811
rmgr: Heap        len (rec/tot):     65/   701, tx:        811, lsn: 0/19D645C0, prev 0/19D64588, desc: HOT_UPDATE off 6 xmax 811 flags 0x60 ; new off 15 xmax 0, blkref #0: rel 1663/16418/16445 blk 0 FPW
rmgr: Transaction len (rec/tot):     34/    34, tx:        811, lsn: 0/19D64880, prev 0/19D645C0, desc: COMMIT 2023-08-02 18:48:44.942660 CST
rmgr: Standby     len (rec/tot):     50/    50, tx:          0, lsn: 0/19D648A8, prev 0/19D64880, desc: RUNNING_XACTS nextXid 812 latestCompletedXid 811 oldestRunningXid 812

#二进制文件切换
hellodb=# select pg_switch_wal();
 pg_switch_wal 
---------------
 0/1C000078
(1 row)

hellodb=# select pg_walfile_NAME_OFFSET(pg_current_wal_lsn());
    pg_walfile_name_offset    
------------------------------
 (00000001000000000000001C,0)
(1 row)

[postgres@pgsql ~]$ pg_waldump /pgsql/data/pg_wal/00000001000000000000001C 
rmgr: Standby     len (rec/tot):     50/    50, tx:          0, lsn: 0/1C000028, prev 0/1B000148, desc: RUNNING_XACTS nextXid 812 latestCompletedXid 811 oldestRunningXid 812
rmgr: XLOG        len (rec/tot):     24/    24, tx:          0, lsn: 0/1C000060, prev 0/1C000028, desc: SWITCH 

```

### 1、创建恢复点

```
#事先创建恢复点,将来可用它进行还原,相当于快照
postgres=# select pg_create_restore_point( 'test-restore-point');
 pg_create_restore_point 
-------------------------
 0/30001B0
(1 row)
```

### 2、归档wal日志

```
#日志级别，日志详细程度
wal_level = replica                    # minimal, replica, or logical
                                        # (change requires restart)
hellodb=# show wal_level ;
 wal_level 
-----------
 replica
(1 row)

#查看二进制文件目录
hellodb=# select * from pg_ls_waldir() order by modification asc;
           name           |   size   |      modification      
--------------------------+----------+------------------------
 00000001000000000000001E | 16777216 | 2023-08-01 16:56:05+08
 00000001000000000000001F | 16777216 | 2023-08-01 16:56:42+08
 000000010000000000000020 | 16777216 | 2023-08-01 23:13:34+08
 000000010000000000000021 | 16777216 | 2023-08-01 23:13:35+08
 000000010000000000000022 | 16777216 | 2023-08-01 23:13:35+08
 000000010000000000000023 | 16777216 | 2023-08-01 23:13:36+08
 00000001000000000000001D | 16777216 | 2023-08-02 19:03:27+08
(7 rows)

#开启归档功能

archive_mode = on           # enables archiving; off, on, or always
                                # (change requires restart)
archive_command = '[ ! -f /archivedir/%f ] && cp %p /archivedir/%f'     //定义日志存放文件        
								# command to use to archive a logfile segment
                                # placeholders: %p = path of file to archive
                                #               %f = file name only
                                # e.g. 'test ! -f /mnt/server/archivedir/%f && cp %p /mnt/server/archivedir/%f'

[root@pgsql ~]# chown postgres. /archivedir/
[root@pgsql ~]# ll -d /archivedir
drwxr-xr-x 2 postgres postgres 6 Aug  2 19:17 /archivedir

postgres=# select pg_switch_wal();
 pg_switch_wal 
---------------
 0/21000078
(1 row)

postgres=# select pg_walfile_NAME_OFFSET(pg_current_wal_lsn());
    pg_walfile_name_offset     
-------------------------------
 (000000010000000000000022,96)
(1 row)

[root@pgsql archivedir]# ll
total 16384
-rw------- 1 postgres postgres 16777216 Aug  2 19:32 000000010000000000000021

#修改配置文件实现远程复制
#archive_command = '[ ! -f /archivedir/%f ] && cp %p /archivedir/%f'            # command to use to archive a logfile segment
archive_command = 'scp %p postgres@192.168.188.20:/archivedir/%f'               # command to use to archive a logfile segment

[root@postgers /]# mkdir /archivedir/
[root@postgers /]# chown postgres. archivedir/
[root@postgers ~]# ls -d /archivedir/ -l
drwxr-xr-x 2 postgres postgres 4096 Aug  3 14:46 /archivedir/

#.88主机插入记录并验证
[postgres@pgsql ~]$ psql < for_loop.sql 
You are now connected to database "hellodb" as user "postgres".
ERROR:  relation "emp" already exists
CREATE FUNCTION
 for_loop 
----------
 
(1 row)

  count  
---------
 4000000
(1 row)

#.20主机
[root@postgers ~]# ls /archivedir/ -l
total 81920
-rw------- 1 postgres postgres 16777216 Aug  3 14:42 00000001000000000000002A
-rw------- 1 postgres postgres 16777216 Aug  3 14:46 00000001000000000000002B
-rw------- 1 postgres postgres 16777216 Aug  3 14:46 00000001000000000000002C
-rw------- 1 postgres postgres 16777216 Aug  3 14:46 00000001000000000000002D
-rw------- 1 postgres postgres 16777216 Aug  3 14:46 00000001000000000000002E

```

## 7、备份还原

### 1、逻辑备份还原

```sh
#备份单表hellodb数据库
[root@pgsql ~]# pg_dump -f /backup/all hellodb
[root@pgsql ~]# ll /backup/
-rw-r--r-- 1 root root 83561500 Aug  3 05:08 all

#数据库还原
[root@pgsql ~]# psql -d hellodb -Upostgres -f /backup/all 
SET
SET
SET
SET
SET
 set_config 
------------
 
(1 row)

SET
SET
SET
SET
psql:/backup/all:31: ERROR:  function "for_loop" already exists with same argument types
ALTER FUNCTION
SET
SET
psql:/backup/all:48: ERROR:  relation "classes" already exists
ALTER TABLE
psql:/backup/all:61: ERROR:  relation "coc" already exists
ALTER TABLE
psql:/backup/all:73: ERROR:  relation "courses" already exists
ALTER TABLE
CREATE TABLE
ALTER TABLE
psql:/backup/all:100: ERROR:  relation "scores" already exists
ALTER TABLE
psql:/backup/all:116: ERROR:  relation "students" already exists
ALTER TABLE
psql:/backup/all:130: ERROR:  relation "teachers" already exists
ALTER TABLE
psql:/backup/all:148: ERROR:  duplicate key value violates unique constraint "classes_pkey"
DETAIL:  Key (classid)=(1) already exists.
CONTEXT:  COPY classes, line 1
psql:/backup/all:170: ERROR:  duplicate key value violates unique constraint "coc_pkey"
DETAIL:  Key (id)=(1) already exists.
CONTEXT:  COPY coc, line 1
psql:/backup/all:185: ERROR:  duplicate key value violates unique constraint "courses_pkey"
DETAIL:  Key (courseid)=(1) already exists.
CONTEXT:  COPY courses, line 1
COPY 4000000
psql:/backup/all:4000216: ERROR:  duplicate key value violates unique constraint "scores_pkey"
DETAIL:  Key (id)=(1) already exists.
CONTEXT:  COPY scores, line 1
psql:/backup/all:4000249: ERROR:  duplicate key value violates unique constraint "students_pkey"
DETAIL:  Key (stuid)=(1) already exists.
CONTEXT:  COPY students, line 1
psql:/backup/all:4000266: ERROR:  duplicate key value violates unique constraint "teachers_pkey"
DETAIL:  Key (tid)=(1) already exists.
CONTEXT:  COPY teachers, line 1
psql:/backup/all:4000274: ERROR:  multiple primary keys for table "classes" are not allowed
psql:/backup/all:4000282: ERROR:  multiple primary keys for table "coc" are not allowed
psql:/backup/all:4000290: ERROR:  multiple primary keys for table "courses" are not allowed
psql:/backup/all:4000298: ERROR:  multiple primary keys for table "scores" are not allowed
psql:/backup/all:4000306: ERROR:  multiple primary keys for table "students" are not allowed
psql:/backup/all:4000314: ERROR:  multiple primary keys for table "teachers" are not allowed

#备份还原
[root@pgsql ~]# pg_dump -Upostgres -f /backup/all 

#指定格式备份
[root@postgers ~]# pg_dump -Fc -h 192.168.188.88 -Upostgres hellodb > /backup/hellodb.dump
Password: 
[root@postgers ~]# ls /backup/
hellodb1  hellodb2  hellodb.dump
[root@postgers ~]# file /backup/hellodb.dump 
/backup/hellodb.dump: PostgreSQL custom database dump - v1.14-0

#数据备份还原
[root@postgers ~]# pg_restore -h192.168.188.88 -Upostgres -d hellodb2  /backup/hellodb.dump 
Password: 

postgres=# create database hellodb2;
CREATE DATABASE
postgres=# \t
Tuples only is on.
postgres=# \l
 db1       | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 db2       | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 hdb       | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 hellodb   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 hellodb2  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 pinxixi   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 testdb    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 

postgres=# \c hellodb2
You are now connected to database "hellodb2" as user "postgres".
hellodb2=# \dt
 public | classes  | table | postgres
 public | coc      | table | postgres
 public | courses  | table | postgres
 public | emp      | table | postgres
 public | scores   | table | postgres
 public | students | table | postgres
 public | teachers | table | postgres

#查看备份文件
[root@postgers ~]# pg_restore -l /backup/hellodb.dump 
;
; Archive created at 2023-08-06 10:14:08 UTC
;     dbname: hellodb
;     TOC Entries: 25
;     Compression: -1
;     Dump Version: 1.14-0
;     Format: CUSTOM
;     Integer: 4 bytes
;     Offset: 8 bytes
;     Dumped from database version: 14.2
;     Dumped by pg_dump version: 14.8 (Ubuntu 14.8-0ubuntu0.22.04.1)
;
;
; Selected TOC Entries:
;
216; 1255 24608 FUNCTION public for_loop() postgres
209; 1259 16419 TABLE public classes postgres
210; 1259 16425 TABLE public coc postgres
211; 1259 16430 TABLE public courses postgres
215; 1259 24614 TABLE public emp postgres
212; 1259 16435 TABLE public scores postgres
213; 1259 16440 TABLE public students postgres
214; 1259 16445 TABLE public teachers postgres
3562; 0 16419 TABLE DATA public classes postgres
3563; 0 16425 TABLE DATA public coc postgres
3564; 0 16430 TABLE DATA public courses postgres
3568; 0 24614 TABLE DATA public emp postgres
3565; 0 16435 TABLE DATA public scores postgres
3566; 0 16440 TABLE DATA public students postgres
3567; 0 16445 TABLE DATA public teachers postgres
3412; 2606 16424 CONSTRAINT public classes classes_pkey postgres
3414; 2606 16429 CONSTRAINT public coc coc_pkey postgres
3416; 2606 16434 CONSTRAINT public courses courses_pkey postgres
3418; 2606 16439 CONSTRAINT public scores scores_pkey postgres
3420; 2606 16444 CONSTRAINT public students students_pkey postgres
3422; 2606 16450 CONSTRAINT public teachers teachers_pkey postgres


#完全备份
[root@postgers ~]# pg_dumpall -h192.168.188.88 -Upostgres -f /backup/all.sql
Password: 
Password: 
Password: 
Password: 
Password: 
Password: 
Password: 
Password: 
Password: 
Password: 
[root@postgers ~]# psql -h192.168.188.88 -Upostgres  < /backup/all.sql
Password for user postgres: 
SET
SET
SET
ERROR:  role "ding" already exists
ALTER ROLE
ERROR:  role "postgres" already exists
....






#copy备份工具使用，文件导入导出
备份表
hellodb=# copy students to '/tmp/students.csv' with csv;
COPY 25

[root@pgsql ~]# cat /tmp/students.csv 
1,Shi Zhongyu,22,M,2,3
2,Shi Potian,22,M,1,7
3,Xie Yanke,53,M,2,16
4,Ding Dian,32,M,4,4
5,Yu Yutong,26,M,3,1
6,Shi Qing,46,M,5,
7,Xi Ren,19,F,3,
8,Lin Daiyu,17,F,7,
9,Ren Yingying,20,F,6,
10,Yue Lingshan,19,F,3,
11,Yuan Chengzhi,23,M,6,
12,Wen Qingqing,19,F,1,
13,Tian Boguang,33,M,2,
14,Lu Wushuang,17,F,3,
15,Duan Yu,19,M,4,
16,Xu Zhu,21,M,1,
17,Lin Chong,25,M,4,
18,Hua Rong,23,M,7,
19,Xue Baochai,18,F,6,
20,Diao Chan,19,F,7,
21,Huang Yueying,22,F,6,
22,Xiao Qiao,20,F,1,
23,Ma Chao,23,M,4,
24,Xu Xian,27,M,,
25,Sun Dasheng,100,M,

#备份指定表
hellodb=# copy students(stuid,name) to '/tmp/students1.csv' with csv;
COPY 25

[root@pgsql ~]# cat /tmp/students1.csv 
1,Shi Zhongyu
2,Shi Potian
3,Xie Yanke
4,Ding Dian
5,Yu Yutong
6,Shi Qing
7,Xi Ren
8,Lin Daiyu
9,Ren Yingying
10,Yue Lingshan
11,Yuan Chengzhi
12,Wen Qingqing
13,Tian Boguang
14,Lu Wushuang
15,Duan Yu
16,Xu Zhu
17,Lin Chong
18,Hua Rong
19,Xue Baochai
20,Diao Chan
21,Huang Yueying
22,Xiao Qiao
23,Ma Chao
24,Xu Xian
25,Sun Dasheng

#还原数据，前提表存在
hellodb=# copy students from '/tmp/students.csv'with csv;
COPY 25
hellodb=# select * from students ;
     1 | Shi Zhongyu   |  22 | M      |       2 |         3
     2 | Shi Potian    |  22 | M      |       1 |         7
     3 | Xie Yanke     |  53 | M      |       2 |        16
     4 | Ding Dian     |  32 | M      |       4 |         4
     5 | Yu Yutong     |  26 | M      |       3 |         1
     6 | Shi Qing      |  46 | M      |       5 |          
     7 | Xi Ren        |  19 | F      |       3 |          
     8 | Lin Daiyu     |  17 | F      |       7 |          
     9 | Ren Yingying  |  20 | F      |       6 |          
    10 | Yue Lingshan  |  19 | F      |       3 |          
    11 | Yuan Chengzhi |  23 | M      |       6 |          
    12 | Wen Qingqing  |  19 | F      |       1 |          
    13 | Tian Boguang  |  33 | M      |       2 |          
    14 | Lu Wushuang   |  17 | F      |       3 |          
    15 | Duan Yu       |  19 | M      |       4 |          
    16 | Xu Zhu        |  21 | M      |       1 |          
    17 | Lin Chong     |  25 | M      |       4 |          
    18 | Hua Rong      |  23 | M      |       7 |          
    19 | Xue Baochai   |  18 | F      |       6 |          
    20 | Diao Chan     |  19 | F      |       7 |          
    21 | Huang Yueying |  22 | F      |       6 |          
    22 | Xiao Qiao     |  20 | F      |       1 |          
    23 | Ma Chao       |  23 | M      |       4 |          
    24 | Xu Xian       |  27 | M      |         |          
    25 | Sun Dasheng   | 100 | M      |         |          
```

### 2、流复制（热）

```
1、pg_basebackup实现完全备份和还原
#修改配置文件/pgsql/data/pg_hba.conf开启流复制
[root@pgsql ~]# vim /pgsql/data/pg_hba.conf

95 # Allow replication connections from localhost, by a user with the
 96 # replication privilege.
 97 local   replication     all                                     trust
 98 host    replication     all             0.0.0.0/0               md5      //基于MD5认证添加此行
 
 
 2、完全备份
 #创建备份目录，必须为空
 [root@postgers ~]# mkdir -p /pgsql/backup/

#备份
[root@postgers ~]# pg_basebackup -D /pgsql/backup/ -Ft -Pv -Upostgres -h192.168.188.88 -p5432 -R
Password: 
pg_basebackup: initiating base backup, waiting for checkpoint to complete
pg_basebackup: checkpoint completed
pg_basebackup: write-ahead log start point: 0/51000028 on timeline 1
pg_basebackup: starting background WAL receiver
pg_basebackup: created temporary replication slot "pg_basebackup_17919"
584929/584929 kB (100%), 1/1 tablespace                                         
pg_basebackup: write-ahead log end point: 0/51000138
pg_basebackup: waiting for background process to finish streaming ...
pg_basebackup: syncing data to disk ...
pg_basebackup: renaming backup_manifest.tmp to backup_manifest
pg_basebackup: base backup completed

[root@postgers ~]# ll /pgsql/backup/
total 601772
drwxr-xr-x 2 root root      4096 Aug 10 03:00 ./
drwxr-xr-x 3 root root      4096 Aug 10 02:50 ../
-rw------- 1 root root    447163 Aug 10 03:00 backup_manifest
-rw------- 1 root root 598968832 Aug 10 03:00 base.tar
-rw------- 1 root root  16778752 Aug 10 03:00 pg_wal.tar

[root@postgers ~]# file /pgsql/backup/*
/pgsql/backup/backup_manifest: JSON data
/pgsql/backup/base.tar:        POSIX tar archive
/pgsql/backup/pg_wal.tar:      POSIX tar archive


3、备份还原
#创建归档日志目录并修改权限
[root@postgers ~]# mkdir -p /archivedir/

[root@postgers ~]# chmod postgres.postgres /archivedir/

#还原数据
#根据实际数据路径选择

方法1[root@postgers main]# tar xf  /pgsql/backup/base.tar -C $PGDATA/

方法2[root@postgers main]# tar xf /pgsql/backup/base.tar -C  /var/lib/postgresql/14/main
[root@postgers ~]# ls /var/lib/postgresql/14/main/ -l
total 140
-rw-------  1 postgres postgres   227 Aug  3 07:11 backup_label.old
drwx------ 12 postgres postgres  4096 Aug  3 04:34 base
drwx------  2 postgres postgres  4096 Aug 10 05:34 global
drwx------  2 postgres postgres  4096 Aug  3 06:59 log
drwx------  2 postgres postgres  4096 Jul 18 11:17 pg_commit_ts
drwx------  2 postgres postgres  4096 Jul 18 11:17 pg_dynshmem
-rw-------  1 postgres postgres  4925 Aug  3 06:56 pg_hba.conf
-rw-------  1 postgres postgres  1681 Aug  1 19:44 pg_ident.conf
drwx------  4 postgres postgres  4096 Aug  3 07:11 pg_logical
drwx------  4 postgres postgres  4096 Jul 18 11:17 pg_multixact
drwx------  2 postgres postgres  4096 Jul 18 11:17 pg_notify
drwx------  2 postgres postgres  4096 Aug  3 07:11 pg_replslot
drwx------  2 postgres postgres  4096 Jul 18 11:17 pg_serial
drwx------  2 postgres postgres  4096 Jul 18 11:17 pg_snapshots
drwx------  2 postgres postgres  4096 Aug  3 06:59 pg_stat
drwx------  2 postgres postgres  4096 Aug  3 07:11 pg_stat_tmp
drwx------  2 postgres postgres  4096 Jul 18 11:17 pg_subtrans
drwx------  2 postgres postgres  4096 Jul 21 08:37 pg_tblspc
drwx------  2 postgres postgres  4096 Jul 18 11:17 pg_twophase
-rw-------  1 postgres postgres     3 Jul 18 11:17 PG_VERSION
drwx------  3 postgres postgres  4096 Aug 10 05:34 pg_wal
drwx------  2 postgres postgres  4096 Jul 18 11:17 pg_xact
-rw-------  1 postgres postgres   329 Aug 10 03:00 postgresql.auto.conf
-rw-------  1 postgres postgres 28927 Aug  2 12:02 postgresql.conf
-rw-------  1 postgres postgres   130 Aug 10 05:34 postmaster.opts
-rw-------  1 postgres postgres   109 Aug 10 05:34 postmaster.pid
-rw-rw-r--  1 postgres postgres  2213 Jul 18 15:02 serverlog
-rw-------  1 postgres postgres     0 Aug 10 03:00 standby.signal
-rw-------  1 postgres postgres     0 Aug  3 07:11 tablespace_map.old


[root@postgers main]# tar xf /pgsql/backup/pg_wal.tar -C /archivedir/
[root@postgers main]# ls /archivedir/
000000010000000000000051

#修改配置文件，添加以下两行
[root@postgers main]# vim /etc/postgresql/14/main/postgresql.conf 

restore_command = 'cp /archivedir/%f %p'               
recovery_target = 'immediate'  

#重启服务
[postgres@postgers ~]$ systemctl restart  postgresql
postgresql@14-main.service  postgresql.service          

#验证数据
[postgres@postgers ~]$ psql
psql (14.8 (Ubuntu 14.8-0ubuntu0.22.04.1))
Type "help" for help.

postgres=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------+----------+----------+-------------+-------------+-----------------------
 db1       | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 db2       | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 hdb       | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 hellodb   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 hellodb2  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 pinxixi   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 testdb    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
(10 rows)
```

### 3、实例：故障还原

```
1、故障模拟
#创建备份目录，必须为空
 [root@postgers ~]# mkdir -p /pgsql/backup/

#备份
[root@postgers ~]# pg_basebackup -D /pgsql/backup/ -Ft -Pv -Upostgres -h192.168.188.88 -p5432 -R
Password: 
pg_basebackup: initiating base backup, waiting for checkpoint to complete
pg_basebackup: checkpoint completed
pg_basebackup: write-ahead log start point: 0/51000028 on timeline 1
pg_basebackup: starting background WAL receiver
pg_basebackup: created temporary replication slot "pg_basebackup_17919"
584929/584929 kB (100%), 1/1 tablespace                                         
pg_basebackup: write-ahead log end point: 0/51000138
pg_basebackup: waiting for background process to finish streaming ...
pg_basebackup: syncing data to disk ...
pg_basebackup: renaming backup_manifest.tmp to backup_manifest
pg_basebackup: base backup completed

[root@postgers ~]# ll /pgsql/backup/
total 601772
drwxr-xr-x 2 root root      4096 Aug 10 03:00 ./
drwxr-xr-x 3 root root      4096 Aug 10 02:50 ../
-rw------- 1 root root    447163 Aug 10 03:00 backup_manifest
-rw------- 1 root root 598968832 Aug 10 03:00 base.tar
-rw------- 1 root root  16778752 Aug 10 03:00 pg_wal.tar

[root@postgers ~]# file /pgsql/backup/*
/pgsql/backup/backup_manifest: JSON data
/pgsql/backup/base.tar:        POSIX tar archive
/pgsql/backup/pg_wal.tar:      POSIX tar archive

#插入数据
hellodb=# insert INTO teachers values (10,'action',48,'F');
INSERT 0 1

hellodb=# select pg_walfile_name(pg_current_wal_lsn());
     pg_walfile_name      
--------------------------
 000000010000000000000054
(1 row)

#模拟数据库删除
hellodb=# \c db1
You are now connected to database "db1" as user "postgres".
db1=# drop database hellodb;
DROP DATABASE


#发现故障，停止用户访问，查看当前日志
hellodb=# select pg_walfile_name(pg_current_wal_lsn());
     pg_walfile_name      
--------------------------
 000000010000000000000054
(1 row)

#查看当前事务
db1=# select txid_current();
 txid_current 
--------------
          968
(1 row)


2、故障还原：
#pg服务器上切换日志

db1=# select pg_switch_wal();
 pg_switch_wal 
---------------
 0/54001B50
(1 row)

db1=# select pg_walfile_name(pg_current_wal_lsn());
     pg_walfile_name      
--------------------------
 000000010000000000000055
(1 row)

#要在还原的服务器停止服务，准备还原
[root@pgsql ~]# su - postgres 
Last login: Thu Aug  3 14:59:36 CST 2023 on pts/4
[postgres@pgsql ~]$ pg_ctl stop -D $PGDATA
waiting for server to shut down.... done
server stopped
[postgres@pgsql ~]$ rm -rf /pgsql/data/*

#数据还原，修改配置文件/pgsql/data/postgresql.conf

[root@pgsql ~]# vim /pgsql/data/postgresql.conf 

259 restore_command = 'cp /archivedir/%f %p'        # command to use to restore an archived logfile segment

278 recovery_target_xid = '967' # the transaction ID up to which recovery will proceed 


#重启数据库服务
[postgres@pgsql ~]$ pg_ctl restart
waiting for server to shut down.... done
server stopped
waiting for server to start....2023-08-03 17:47:56.989 CST [23001] LOG:  redirecting log output to logging collector process
2023-08-03 17:47:56.989 CST [23001] HINT:  Future log output will appear in directory "log".
 done
server started

#验证结果
postgres=# \c hellodb
You are now connected to database "hellodb" as user "postgres".
hellodb=# select * from teachers ;
 tid |     name      | age | gender 
-----+---------------+-----+--------
   1 | Song Jiang    |  45 | M
   2 | Zhang Sanfeng |  94 | M
   3 | Miejue Shitai |  77 | F
   4 | Lin Chaoying  |  93 | F
   5 | d             |  25 | F
   7 | p             |  33 | M
  11 | c             |  30 | M
   8 | pan           |  18 | M
   6 | S             |  50 | M
  10 | action        |  48 | F
(9 rows)


#恢复到正常模式
postgres=# select pg_wal_replay_resume();
 pg_wal_replay_resume 
----------------------
 
(1 row)

#验证数据是否还原
hellodb=# insert into teachers values(10,'end',8,'M');
INSERT 0 1
hellodb=# select * from teachers ;
 tid |     name      | age | gender 
-----+---------------+-----+--------
   1 | Song Jiang    |  45 | M
   2 | Zhang Sanfeng |  94 | M
   3 | Miejue Shitai |  77 | F
   4 | Lin Chaoying  |  93 | F
   5 | d             |  25 | F
   7 | p             |  33 | M
  11 | c             |  30 | M
   8 | pan           |  18 | M
   6 | S             |  50 | M
  10 | end           |   8 | M
(10 rows)

```

# 日志文件

```
[root@pgsql ~]# rpm -qc rsyslog    //只看配置文件
/etc/logrotate.d/syslog
/etc/rsyslog.conf
/etc/sysconfig/rsyslog

```

