查看修改网络配置命令：   vi /etc/sysconfig/network-scripts/ifcfg-ens160
（1。删除udd       2。改静态IP   3.ONBOOT=yes//开机自动加载文件    4。添加IP、掩码、网关）
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static//配置静态IP
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens160
DEVICE=ens160
ONBOOT=yes//开机自动加载文件
IPADDR=192.168.188.100//IP地址
NETMASK=255.255.255.0//子网掩码
GATEWAY=192.168.188.2//网关
DNS1=114.114.114.114//dns服务


关闭防火墙：systemctl stop firewalld

永久关闭防火墙:systemctl disable firewalld

重启网卡：nmcli connection reload

timedatectl//时钟及时区查看

PS1="\[\e[1;36m\][\u@\h \W]\$"//改变文件字体界面

更改通配符颜色：PS1="[\[\e[01;37m\]\t][\[\e[02;34m\]\u\[\e[37m\]@\h \[\e[32m\]\w\[\e[m\]]\\$ "


常用命令
   50  cd /root转到root目录
   51  ps -x查看进程
   52  cd /proc/查看所有活动进程
   53  ll查看详情
   54  pwd当前路径
   55  echo $$查看当前活动编号
   56  cd 1869转到活动编号路径
   57  ll查看详情
   58  history 常看历史命令



二级目录：
[root@localhost 1869]# ll /
total 16
lrwxrwxrwx.   1 root root    7 Nov  3  2020 bin -> usr/bin
dr-xr-xr-x.   6 root root 4096 Sep 13 18:12 boot
drwxr-xr-x.   2 root root    6 Sep 13 18:08 data
drwxr-xr-x.  20 root root 3160 Nov 16 12:13 dev
drwxr-xr-x.  79 root root 8192 Nov 16 13:15 etc
drwxr-xr-x.   4 root root   31 Nov 16 13:07 home
lrwxrwxrwx.   1 root root    7 Nov  3  2020 lib -> usr/lib
lrwxrwxrwx.   1 root root    9 Nov  3  2020 lib64 -> usr/lib64
drwxr-xr-x.   2 root root    6 Nov  3  2020 media
drwxr-xr-x.   3 root root   18 Sep 13 18:09 mnt
drwxr-xr-x.   2 root root    6 Nov  3  2020 opt
dr-xr-xr-x. 153 root root    0 Nov 16 05:53 proc
dr-xr-x---.   4 root root  162 Nov 16 13:08 root
drwxr-xr-x.  22 root root  620 Nov 16 12:08 run
lrwxrwxrwx.   1 root root    8 Nov  3  2020 sbin -> usr/sbin
drwxr-xr-x.   2 root root    6 Nov  3  2020 srv
dr-xr-xr-x.  13 root root    0 Nov 16 05:53 sys
drwxrwxrwt.   9 root root  161 Nov 16 13:28 tmp
drwxr-xr-x.  12 root root  144 Sep 13 18:08 usr
drwxr-xr-x.  20 root root  278 Sep 13 18:12 var//不会被清除，常放些日志文件



  299  rm -rf shiren  删除文件及文件夹
  300  ll
  301  touch jingyesi  创建文件
  302  ll
  303  stat jingyesi 改变文件属性
  304  history 

********解决字体乱码问题
c/windows/fonts //字体文件存放位置
cd /usr/share/zabbix/assets/fonts//web前端字体调用路径
mv ARIALUNI.ttf graphfont.ttf//重命名并替换原字体


[root@localhost bak]# mysql -uroot -p//数据库密码123456


 准备前安装包： yum install lrzsz vim net-tools tree psmisc bash-completion dos2unix -y//安装命令工具包
             yum -y install git//git工具包
安装依赖包和组件：yum -y install net-snmp net-snmp-devel curl curl-devel libxml2 libxml2-devel libevent-devel.x86_64 javacc.noarch  javacc-javadoc.noarch javacc-maven-plugin.noarch javacc*

如果你没有安装wget，也可以用下面命令：

curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-vault-8.5.2111.repo

 

      运行 yum clearn all；yum makecache 生成缓存






  433  dnf install zabbix-server-mysql zabbix-web-mysql zabbix-apache-conf zabbix-agent
  434  rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/8/x86_64/zabbix-release-5.0-1.el8.noarch.rpm//安装yum源
  478  sed -i '3c baseurl=http://mirrors.aliyun.com/zabbix/zabbix/5.0/rhel/8/$basearch/' /etc/yum.repos.d/zabbix.repo//更改阿里源

  436  mysql -uroot -p//登录root账户数据库

  437  password

  438  mysql> create database zabbix character set utf8 collate utf8_bin;//创建zabbix数据库连接
  439  mysql> create user zabbix@localhost identified by 'password';
  440  mysql> grant all privileges on zabbix.* to zabbix@localhost;
  441  mysql> set global log_bin_trust_function_creators = 1;

  442  yum install -y httpd mariadb-server mariadb php php-mysql php-gd libjpeg* php-ldap php-odbc php-pear php-xml php-xmlrpc php-mhash//安装php服务
  443  [root@test ~]# rpm -qa httpd mariadb php//php服务
  444  mariadb-5.5.68-1.el7.x86_64
  445  httpd-2.4.6-97.el7.centos.x86_64
  446  rpm -qa httpd mariadb php
  447  ll
  448  systemctl restart mariadb
  449  systemctl status httpd
  450  cat /etc/redhat-release
  451  yum -y install git
  452  rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
  453  [root@localhost ~]# firewall-cmd --permanent --query-port=55555/tcp
  454  yes
  455  firewall-cmd --permanent --query-port=55555/tcp
  456  systemctl start firewalld
  457  firewall-cmd --permanent --add-port=55555/tcp
  458  systemctl restart sshd
  459  systemctl restart firewalld.service
  460  firewall-cmd --permanent --query-port=55555/tcp
  461  firewall-cmd --permanent --add-port=80/tcp
  462  firewall-cmd --permanent --add-port=3000/tcp
  463  firewall-cmd --permanent --add-port=3306/tcp
  464  firewall-cmd --permanent --add-port=4505/tcp
  465  firewall-cmd --permanent --add-port=4506/tcp
  466  firewall-cmd --permanent --add-port=10050/tcp
  467  firewall-cmd --permanent --add-port=10051/tcp
  468  firewall-cmd --permanent --add-port=25/tcp     //不开SSL
  469  firewall-cmd --permanent --add-port=465/tcp    //开SSL
  470  [root@localhost ~]# firewall-cmd --zone=public --list-ports
  471  55555/tcp 4505/tcp 4506/tcp 10051/tcp 10050/tcp 3000/tcp 3306/tcp 80/tcp 465/tcp
  472  firewall-cmd --zone=public --list-ports
  473  systemctl disable --now firewalld.service
  474  setenforce 0
  475  sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config


  479  dnf -y install zabbix-server-mysql zabbix-web-mysql zabbix-apache-conf zabbix-agent
  480  history
  481  	
  482  dnf install -y @httpd @mariadb
  483  systemctl enable --now httpd mariadb
  484  mysql_secure_installation
  485  mysql
  486  mysql -uroot -p  //此处密码password
  487  echo "[mysqld]
skip-name-resolve" >> /etc/my.cnf
  488  systemctl restart mysqld
  489  mysql -uroot -p
  490  zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix
  491  sed -i 's/# DBPassword=/DBPassword=password/' /etc/zabbix/zabbix_server.conf
  492  sed -i '$a php_value[date.timezone] = Asia/Shanghai' /etc/php-fpm.d/zabbix.conf
  493  systemctl restart zabbix-server zabbix-agent httpd php-fpm
  494  systemctl enable zabbix-server zabbix-agent httpd php-fpm
  495  ss -ntulp//查看开启端口
  497  yum install glibc-langpack-zh.x86_64//下载中文语言包
          *********apt-get install language-pack-zh-hans//Ubuntu安装中文语言包 
  498  localectl list-locales | grep zh//选择中文语言包
  499  history

解决界面乱码： 
    618  cd /var/www/html/assets/fonts
  620  mv DejaVuSans.ttf DejaVuSans.ttf.bak
  622  mv ARIALUNI.ttf DejaVuSans.ttf
  625  systemctl restart httpd
  626  systemctl restart zabbix-server


mysql zabbix -e 'show tables'//查看mysql库文件

/etc/apache2/conf-available/zabbix.conf//改变zabbix配置时区




ss -ant | grep ESTAB | wc -l 查看并统计端口状态 输出行数值

mail.163授权密码：OBDIENXJGQYIWSVK
athpbrmhqgqnbbde


hexdump -C ****//文件以二进制方式显示
yum -y install dos2unix //二进制转换工具

#dos2unix **** win转unix
#unix2dos **** unix转win


[root@basenode boot]# echo "this is `hostname`"//先执行单引号的命令，然后被echo引用
this is basenode