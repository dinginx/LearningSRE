# Docker容器

```
KVM 单机
openstack 多主机VM管理平台

docker 单机容器管理
kubernetes 多主机容器管理平台
```

# 1、docker介绍

## 1.1、docker简介

```
镜像：模板，文件，静态文件，只占磁盘空间,国际标准
容器：镜像副本，实例，进程，多实例，占内存，占磁盘,国际标准
```

docker 

c1  /var/log/test.log 

c2 /var/log/test.log



wordpress 

nginx/apache 容器

php 容器

mysql 容器



ansible  = docker



ansible-playbook  playbook.yml    ==  docker-compose 

docker run mysql

docker run php

docker run nginx



容器内安装docker客户端 docker

bash sh



```
 -H tcp://0.0.0.0:2375
 
 "hosts": ["uninx://var/run/docker.sock",tcp://0.0.0.:2375]"
```



ext4 xfs



```
mkfs.ext4 /dev/sda3   
mkfs.ext4 /dev/sda4

mount /dev/sda3 /data
mount /dev/sda4 /data
ls /data 
```

## docker容器优化

### 使用阿里云镜像加速

```sh
1. 安装／升级Docker客户端
推荐安装1.10.0以上版本的Docker客户端，参考文档 docker-ce
    
2. 配置镜像加速器
修改daemon配置文件/etc/docker/daemon.json来使用加速器
mkdir -p /etc/docker

sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://bgixdd17.mirror.aliyuncs.com/"]
}
EOF

3.加载并重启服务
systemctl daemon-reload
systemctl restart docker


4.查看是否生效
root@ubuntu2204:~# docker info

......

 Registry Mirrors:
  https://bgixdd17.mirror.aliyuncs.com/
 Live Restore Enabled: false
```

### 修改存储引擎

默认Overlay2（推荐）

```sh
root@ubuntu2204:~# cat /lib/systemd/system/docker.service 

[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target docker.socket firewalld.service containerd.service time-set.target
Wants=network-online.target containerd.service
Requires=docker.socket

[Service]
Type=notify
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --data-root=/data/docker  #添加指定目录选项
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutStartSec=0
RestartSec=2
Restart=always
StartLimitBurst=3
StartLimitInterval=60s
LimitNPROC=infinity
LimitCORE=infinity
TasksMax=infinity
Delegate=yes
KillMode=process
OOMScoreAdjust=-500

[Install]
WantedBy=multi-user.target
```

### 打开远程端口

修改service文件连接

```sh
1、添加-H fd:// 标记
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H fd:// --containerd=/run/containerd/containerd.sock

ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H fd:// --containerd=/run/containerd/containerd.sock --data-root=/data/docker

2、修改service文件
root@ubuntu2204:~# cat /lib/systemd/system/docker.service |grep -Ev '^#|^$' 
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target docker.socket firewalld.service containerd.service time-set.target
Wants=network-online.target containerd.service
Requires=docker.socket

[Service]
Type=notify
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H fd:// --containerd=/run/containerd/containerd.sock --data-root=/data/docker   #修改此行内容，添加fd:// 标记
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutStartSec=0
RestartSec=2
Restart=always
StartLimitBurst=3
StartLimitInterval=60s
LimitNPROC=infinity
LimitCORE=infinity
TasksMax=infinity
Delegate=yes
KillMode=process
OOMScoreAdjust=-500

[Install]
WantedBy=multi-user.target

3.加载并重启服务
systemctl daemon-reload
systemctl restart docker


4、远程主机进行连接
```

### 开启活动重启 (重启docker-daemon 不关闭容器

```sh
root@ubuntu2204:~# cat /etc/docker/daemon.json 
{
  "registry-mirrors": [
	  "https://bgixdd17.mirror.aliyuncs.com"
  ],
  "live-restore": true    #添加此行内容
}
```

### Docker优化实例：

```
root@ubuntu2204:~# cat /etc/docker/daemon.json 
{
  "registry-mirrors": [												#设置镜像代理
	  "https://bgixdd17.mirror.aliyuncs.com"，
	  "https://hub-mirror.c.163.com"
	  "https://docker.mirrors.ustc.edu.cn"
  ],
  "host": ["unix://var/run/docker.sock","tcp://0.0.0.0:2375"],		#远程连接选项
  "insecure-registies": ["harbor.dingbh.top"],						#私有仓库使用
  "exec-opts": ["native.cgroupdriver=systemd"],						#设置启动方式，默认systemd（推荐）
  "graph": ["/data/docker"],										#设置docker目录路径
  "max-concurrent-downloads": 10,									#设置最大下载数
  "max-concurrent-uploads": 5,										#设置最大上传数
  "log-opts": {														#log文件的最大文件大小及文件数
  	"max-size": "300m",
	"max-file": "2"
  },
  "live-restore": true												#设置重启docker-daemon 不关闭容器
}
```

### 下载镜像

```sh
#拉去镜像
[root@Rocky8 ~]# docker pull busybox
Using default tag: latest
latest: Pulling from library/busybox
7b2699543f22: Pull complete 
Digest: sha256:650fd573e056b679a5110a70aabeb01e26b76e545ec4b9c70a9523f2dfaf18c6
Status: Downloaded newer image for busybox:latest
docker.io/library/busybox:latest

[root@Rocky8 ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
ubuntu        latest    ca2b0f26964c   2 weeks ago     77.9MB
nginx         latest    92b11f67642b   4 weeks ago     187MB
alpine        latest    05455a08881e   7 weeks ago     7.38MB
busybox       latest    ba5dc23f65d4   10 months ago   4.26MB
hello-world   latest    d2c94e258dcb   10 months ago   13.3kB
```



```sh
[root@Rocky8 ~]# docker image ls --format "{{.Repository}}:{{.Tag}}"
ubuntu:latest
nginx:latest
alpine:latest
busybox:latest
hello-world:latest
```

镜像导出：

```sh
#方法1: 使用image ID导出镜像,在导入后的镜像没有REPOSITORY和TAG,显示为<none>
[root@ubuntu1804 ~]#docker save `docker images -qa` -o all.tar

#方法2:将所有镜像导入到一个文件中,此方法导入后可以看REPOSITORY和TAG
[root@ubuntu1804 ~]#docker save `docker images | awk 'NR!=1{print $1":"$2}'` -o 
all.tar
#方法3:将所有镜像导入到一个文件中,此方法导入后可以看REPOSITORY和TAG
[root@centos8 ~]#docker image save `docker image ls --format "{{.Repository}}:
{{.Tag}}"` -o all.tar


[root@Rocky8 ~]# docker save `docker image ls --format "{{.Repository}}:{{.Tag}}"` -o all.tar

```

镜像导入

```sh
root@ubuntu2204:~# docker load -i all1.tar 
bb01bd7e32b5: Loading layer [==================================================>]  7.618MB/7.618MB
Loaded image ID: sha256:92b11f67642b62bbb98e7e49169c346b30e20cd3c1c034d31087e46924b9312e
Loaded image ID: sha256:673d0a8531eb903c884fc8d770893db1eb27e552632a1d75ac49b7b5b46aac04
Loaded image ID: sha256:5e2b554c1c45d22c9d1aa836828828e320a26011b76c08631ac896cbc3625e3e
Loaded image ID: sha256:d2c94e258dcb3c5ac2798d32e1249e42ef01cba4841c2234249495f87264ac5a
Loaded image ID: sha256:9b957e098315598ae58c7a62bcc140ed086bbe619b38c31a52b21abbd5eb21e2
Loaded image ID: sha256:d04d353a611be4e0c344e76af6a6d57a82be87a9c633cf9f1228be94bfc10744
```

查看名称和版本

```sh
[root@Rocky8 ~]# docker image ls --format "{{.Repository}}:{{.Tag}}"
wordpress:php8.3-fpm-alpine
ubuntu:latest
alpine:latest
busybox:latest
alpine:3.18.0
hello-world:latest
nginx:latest
wordpress:latest
grafana/grafana:latest

```

镜像删除

```
docker rmi 命令可以删除本地镜像
```

格式

```sh
docker rmi [OPTIONS] IMAGE [IMAGE...]
docker image rm [OPTIONS] IMAGE [IMAGE...]
#选项:
-f, --force     Force removal of the image
    --no-prune   Do not delete untagged parents
```

## 1.4、**容器操作基础命令**

**容器生命周期**

![image-20240321211649890](Docker容器/image-20240321211649890.png)

**created --> running --> 两个步骤可用docker run替换**

### 1.4.2、显示容器信息

#### 1.4.2.1、显示当前存在容器

**格式**

```
[root@Rocky8 ~]# docker ps -h
Flag shorthand -h has been deprecated, please use --help

Usage:  docker ps [OPTIONS]

List containers

Aliases:
  docker container ls, docker container list, docker container ps, docker ps

Options:
  -a, --all             Show all containers (default shows just running)
  -f, --filter filter   Filter output based on conditions provided
      --format string   Format output using a custom template:
                        'table':            Print output in table format with column headers (default)
                        'table TEMPLATE':   Print output in table format using the given Go template
                        'json':             Print in JSON format
                        'TEMPLATE':         Print output using the given Go template.
                        Refer to https://docs.docker.com/go/formatting/ for more information about formatting output with templates
  -n, --last int        Show n last created containers (includes all states) (default -1)
  -l, --latest          Show the latest created container (includes all states)
      --no-trunc        Don't truncate output
  -q, --quiet           Only display container IDs
  -s, --size            Display total file sizes
  
  --format 按格式输出
```

**docker ps  --format命令中，你可以使用不同的占位符来指定要在输出中现实的容器信息**

```sh
#以下是一些常用的占位符
{{.ID}}：容器的id
{{.Image}}：容器使用的映像名称
{{.Command}}：容器的启动命令
{{.CreatedAt}}：容器的创建时间
{{.RunningFor}}：容器的运行时间
{{.Ports}}:容器的端口映射信息
{{.Status}}：容器的状态
{{.Size}}：容器的大小
{{.Names}}：容器的名称
{{.Label}}:容器的标签
```

**范例：**

```
#按时间顺序，列出启动时间最早的三个容器
[root@Rocky8 ~]# docker ps -a --format "{{.ID}}\t{{.Names}}\t{{.CreatedAt}}\t{{.Status}}"|sort -k 3|head -3
6fcb8ee840e7	hungry_goldwasser	2024-03-19 16:17:39 +0800 CST	Exited (0) 2 days ago
85f4904be390	stupefied_lehmann	2024-03-19 16:23:12 +0800 CST	Exited (0) 2 days ago
43543741a41d	beautiful_bartik	2024-03-19 16:24:43 +0800 CST	Exited (0) 2 days ago


[root@Rocky8 ~]# docker ps -a --format "{{.Names}}\t{{.ID}}\t{{.CreatedAt}}\t------>>>{{.Status}}"|sort -k 3|head -3
mysql	53ce8bf67e94	2024-03-22 16:14:05 +0800 CST	------>>>Up 41 minutes
n1	a80be6591ee4	2024-03-22 16:29:29 +0800 CST	------>>>Exited (0) 26 minutes ago
web01	3eccf911b1be	2024-03-22 16:33:52 +0800 CST	------>>>Exited (0) 19 minutes ago
```

**查看所有容器**

```
#查看所有容器
[root@Rocky8 ~]# docker ps -a
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS          PORTS                 NAMES
53ce8bf67e94   mysql:8.0.29-oracle   "docker-entrypoint.s…"   12 minutes ago   Up 12 minutes   3306/tcp, 33060/tcp   mysql

#查看所有退出的容器
[root@Rocky8 ~]# docker ps -f 'status=exited'
CONTAINER ID   IMAGE     COMMAND     CREATED          STATUS                      PORTS     NAMES
a80be6591ee4   alpine    "/bin/sh"   30 seconds ago   Exited (0) 29 seconds ago             n1
```

#### 1.4.2.2、查看容器的进程

**格式**

```
[root@Rocky8 ~]# docker top -h
Flag shorthand -h has been deprecated, please use --help

Usage:  docker top CONTAINER [ps OPTIONS]

Display the running processes of a container

Aliases:
  docker container top, docker top
```

**查看运行容器的进程**

```
#查看mysql进程
[root@Rocky8 ~]# docker top mysql 
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
systemd+            16470               16449               0                   16:14               ?                   00:00:05            mysqld

#查看nginx进程
[root@Rocky8 ~]# docker top  web01 
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                20110               20081               0                   17:05               ?                   00:00:00            nginx: master process nginx -g daemon off;
101                 20149               20110               0                   17:05               ?                   00:00:00            nginx: worker process
101                 20150               20110               0                   17:05               ?                   00:00:00            nginx: worker process

#查看alpine进程
[root@Rocky8 ~]# docker run -d alpine /bin/sh -c 'i=1;while true;do echo 
> hello$i;let i++;sleep 1;done'
af51b8eb5eb7160c3eb4e30d534fb4df20f6595f56606afc0ee482cf300a97f0
[root@Rocky8 ~]# docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS              PORTS                 NAMES
af51b8eb5eb7   alpine                "/bin/sh -c 'i=1;whi…"   3 seconds ago    Up 3 seconds                              elegant_williamson
3eccf911b1be   nginx:1.20.0          "/docker-entrypoint.…"   32 minutes ago   Up About a minute   80/tcp                web01
53ce8bf67e94   mysql:8.0.29-oracle   "docker-entrypoint.s…"   52 minutes ago   Up 52 minutes       3306/tcp, 33060/tcp   mysql
[root@Rocky8 ~]# docker top elegant_williamson 
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                20355               20334               0                   17:06               ?                   00:00:00            /bin/sh -c i=1;while true;do echo hello$i;let i++;sleep 1;done
root                20513               20355               0                   17:06               ?                   00:00:00            sleep 1

```

#### **1.4.2.3** **查看容器资源使用情况**

格式

```
[root@Rocky8 ~]# docker stats -h
Flag shorthand -h has been deprecated, please use --help

Usage:  docker stats [OPTIONS] [CONTAINER...]

Display a live stream of container(s) resource usage statistics

Aliases:
  docker container stats, docker stats

Options:
  -a, --all             Show all containers (default shows just running)
      --format string   Format output using a custom template:
                        'table':            Print output in table format with column headers (default)
                        'table TEMPLATE':   Print output in table format using the given Go template
                        'json':             Print in JSON format
                        'TEMPLATE':         Print output using the given Go template.
                        Refer to https://docs.docker.com/go/formatting/ for more information about formatting output with templates
      --no-stream       Disable streaming stats and only pull the first result
      --no-trunc        Do not truncate output

```

**范例：**

```
#不加具体容器选项默认显示所有正在运行中的容器
[root@Rocky8 ~]# docker stats 

CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT     MEM %     NET I/O       BLOCK I/O       PIDS
3eccf911b1be   web01     0.00%     3.543MiB / 1.694GiB   0.20%     936B / 0B     459kB / 0B      3
53ce8bf67e94   mysql     0.18%     409.6MiB / 1.694GiB   23.62%    1.37kB / 0B   233MB / 570MB   38

#单独显示web01容器资源使用信息
[root@Rocky8 ~]# docker stats web01 

CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT     MEM %     NET I/O     BLOCK I/O    PIDS
3eccf911b1be   web01     0.00%     3.543MiB / 1.694GiB   0.20%     866B / 0B   459kB / 0B   3

#单独显示mysql容器资源使用信息
[root@Rocky8 ~]# docker stats mysql 

CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT     MEM %     NET I/O       BLOCK I/O       PIDS
53ce8bf67e94   mysql     0.02%     409.6MiB / 1.694GiB   23.62%    1.37kB / 0B   233MB / 570MB   38

#限制内存使用大小

```

#### **1.4.2.4** **查看容器的详细信息**

**inspect选项，显示详细信息**

**格式**

```
[root@Rocky8 ~]# docker inspect -h
Flag shorthand -h has been deprecated, please use --help

Usage:  docker inspect [OPTIONS] NAME|ID [NAME|ID...]

Return low-level information on Docker objects

Options:
  -f, --format string   Format output using a custom template:
                        'json':             Print in JSON format
                        'TEMPLATE':         Print output using the given Go template.
                        Refer to https://docs.docker.com/go/formatting/ for more information about formatting output with templates
  -s, --size            Display total file sizes if the type is container
      --type string     Return JSON for specified type

```

**范例**

```sh
#显示容器特定信息字段
[root@Rocky8 ~]# docker inspect elasticsearch --format "{{.Name}}\t{{.I}}{{.NetworkSettings.Networks.bridge.IPAddress}}"
/elasticsearch   6096507e440a056aa493952f31d862bad184a9ce2ce04298eb77c77c77eaf73f   172.17.0.3

#显示mysql容器特定信息字段
[root@Rocky8 ~]# docker inspect mysql --format "{{.ID}} {{.State.Status}}  {{.NetworkSettings.Networks.bridge.IPAddress}}"
53ce8bf67e94568d97535982008e0e54209b1e5331bc5cf168cd5dc9d9aaf7a3 running  172.17.0.2
```









--name选项指定名称

```sh
[root@Rocky8 ~]# docker run -d  --name dinginx01 nginx
440878fd3b057cb0c5eb5f4debd23e5119466c001a5c1f42b824c0b0f95a5c7d
[root@Rocky8 ~]# docker ps 
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
440878fd3b05   nginx     "/docker-entrypoint.…"   8 seconds ago   Up 6 seconds   80/tcp    dinginx01
```



-it 选项 容器交互访问

```sh
[root@Rocky8 ~]# docker run -it  --name nginx05  nginx bash
root@bbcf598e5d0c:/# ls
bin  boot  dev	docker-entrypoint.d  docker-entrypoint.sh  etc	home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@bbcf598e5d0c:/# cat /etc/os-release 
PRETTY_NAME="Debian GNU/Linux 11 (bullseye)"
NAME="Debian GNU/Linux"
VERSION_ID="11"
VERSION="11 (bullseye)"
VERSION_CODENAME=bullseye
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
root@bbcf598e5d0c:/# 
```

-d选项 后台启动

```sh
[root@Rocky8 ~]# docker run -d nginx
5907a2f1575bea3d9a0ed7c79474fc3fa5ff639c7dcb351305ca749f543996cd
```

-h选项指定主机名

```sh
[root@Rocky8 ~]# docker run -it -h alpine.dingbh.top alpine:3.18.0 
/ # hostname
alpine.dingbh.top
/ # 
```

--rm选项 四种不同的policy no、on-failure、always、unless一次性测试容器，执行完成后自动删除

```sh
[root@Rocky8 ~]# docker run  --rm --name ddd alpine:3.18.0 sleep 5
```

--restart always选项。随着宿主机启动服务开启启动

```sh
[root@Rocky8 ~]# docker run --name web01 -d --restart always nginx
5c84745eda20f14ebad8fe72a5a75cebeb320d71e6e67773ce803c0ded406ca0

#验证
[root@Rocky8 ~]# reboot
[root@Rocky8 ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED             STATUS          PORTS     NAMES
5c84745eda20   nginx     "/docker-entrypoint.…"   About an hour ago   Up 44 seconds   80/tcp    web01
```



```sh

```



### **1.4.3** **删除容器**

**docker rm 可以删除容器，即使容器正在运行当中，也可以被强制删除掉**

**格式**

```
docker rm [OPTIONS] CONTAINER [CONTAINER...]
docker container rm [OPTIONS] CONTAINER [CONTAINER...]
#选项:  
-f, --force     Force the removal of a running container (uses SIGKILL)
-v, --volumes   Remove the volumes associated with the container
#删除停止的容器
docker container prune [OPTIONS]
Options:
      --filter filter   Provide filter values (e.g. 'until=<timestamp>')
      -f, --force           Do not prompt for confirmation
```

**范例：**

```
[root@Rocky8 ~]# docker ps -a

......

43543741a41d   c3c92cc3dcb1          "docker-entrypoint.s…"   2 days ago     Exited (0) 2 days ago                   beautiful_bartik
85f4904be390   628efc1ca5f9          "docker-entrypoint.s…"   2 days ago     Exited (0) 2 days ago                   stupefied_lehmann
6fcb8ee840e7   nginx                 "/docker-entrypoint.…"   2 days ago     Exited (0) 2 days ago                   hungry_goldwasser
#删除指定容器
[root@Rocky8 ~]# docker rm 6fcb8ee840e7
6fcb8ee840e7

#删除所有容器
[root@Rocky8 ~]# docker rm -f `docker ps -a -q`

#删除指定状态的容器，删除所有exited状态的容器
[root@Rocky8 ~]# docker rm -f `docker ps -a -f status=exited -q`
[root@Rocky8 ~]# docker ps -a
CONTAINER ID   IMAGE           COMMAND                  CREATED        STATUS        PORTS     NAMES
5c84745eda20   nginx           "/docker-entrypoint.…"   12 hours ago   Up 11 hours   80/tcp    web01
e7c67502b46f   alpine:3.18.0   "ll"                     13 hours ago   Created                 affectionate_booth

#使用命令删除exited状态的容器
[root@Rocky8 ~]# docker ps -a
CONTAINER ID   IMAGE           COMMAND                  CREATED          STATUS                      PORTS     NAMES
ac3ca8e4d400   alpine:3.18.0   "cat /etc/os-release"    6 seconds ago    Exited (0) 5 seconds ago              pensive_nash
dacdd2a68249   alpine:3.18.0   "ls"                     18 seconds ago   Exited (0) 17 seconds ago             hopeful_sinoussi
5c84745eda20   nginx           "/docker-entrypoint.…"   12 hours ago     Up 11 hours                 80/tcp    web01
e7c67502b46f   alpine:3.18.0   "ll"                     13 hours ago     Created                               affectionate_booth

#命令删除prune，删除已经退出的容器
[root@Rocky8 ~]# docker container prune 
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
Deleted Containers:
ac3ca8e4d40072b24b1526285ebaaf5a20c71820132fd980a3a96852578c313a
dacdd2a682499df8826dba6c5ddb3b1b15a931c329045cf48065b1494767d976
e7c67502b46f9bf52663a31bedfd6f2e651760c48a1e615257955c6b1306f627

Total reclaimed space: 0B
[root@Rocky8 ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED        STATUS        PORTS     NAMES
5c84745eda20   nginx     "/docker-entrypoint.…"   12 hours ago   Up 11 hours   80/tcp    web01
```

#### 面试：删除所有退出的容器

```
方法一：
[root@Rocky8 ~]# docker rm `docker ps -qf "status=exited"`

方法二：
[root@Rocky8 ~]# docker ps -f status=exited -q|xargs docker rm -f

方法三：
[root@Rocky8 ~]# docker container prune
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
Deleted Containers:
3eccf911b1be7054159753aeca07a10f73413bf445c8f689403b11b37feeb498

Total reclaimed space: 1.093kB
或加 -f选项，非交互方式删除
```

### 1.4.4、容器的停止和启动

**docker start|stop|restart|pause|unpause 容器ID**

批量正常启动或关闭所有容器

```
docker start $(docker ps -a -q)  
docker stop $(docker ps -a -q)
```

范例：

```
#查看所有容器状态
[root@Rocky8 ~]# docker ps -a
CONTAINER ID   IMAGE           COMMAND                  CREATED          STATUS                      PORTS     NAMES
775219757ba1   alpine:3.18.0   "/bin/sh"                20 minutes ago   Exited (0) 20 minutes ago             web02
5c84745eda20   nginx           "/docker-entrypoint.…"   12 hours ago     Exited (0) 18 minutes ago             web01
#开启所有容器
[root@Rocky8 ~]# docker start $(docker ps -a -q)
775219757ba1
5c84745eda20
[root@Rocky8 ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED        STATUS         PORTS     NAMES
5c84745eda20   nginx     "/docker-entrypoint.…"   12 hours ago   Up 3 seconds   80/tcp    web01
#停止所有容器
[root@Rocky8 ~]# docker stop $(docker ps -aq)
775219757ba1
5c84745eda20

[root@Rocky8 ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

范例: 启动并进入容器

```
[root@Rocky8 ~]# docker run --name=u1 -it ubuntu bash
root@56504cb7a652:/# ls
bin  boot  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@56504cb7a652:/# exit
exit
```

范例: 暂停和恢复容器

```sh
[root@Rocky8 ~]# docker run -d --name web03 nginx
b1a9d392592cab284f20a4ca507b9ffd4956758e70352ae655d40c828605b04a
[root@Rocky8 ~]# docker ps 
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
b1a9d392592c   nginx     "/docker-entrypoint.…"   6 seconds ago   Up 4 seconds   80/tcp    web03
[root@Rocky8 ~]# docker top web03 
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                12748               12728               0                   09:41               ?                   00:00:00            nginx: master process nginx -g daemon off;
101                 12790               12748               0                   09:41               ?                   00:00:00            nginx: worker process
101                 12791               12748               0                   09:41               ?                   00:00:00            nginx: worker process
[root@Rocky8 ~]#  ps aux|grep nginx
root       12748  0.0  0.3   8856  5360 ?        Ss   09:41   0:00 nginx: master process nginx -g daemon off;
101        12790  0.0  0.1   9276  2444 ?        S    09:41   0:00 nginx: worker process
101        12791  0.0  0.1   9276  2444 ?        S    09:41   0:00 nginx: worker process
root       12844  0.0  0.0 221940  1096 pts/1    S+   09:41   0:00 grep --color=auto nginx

#挂起web03容器
[root@Rocky8 ~]# docker pause web03 
web03
[root@Rocky8 ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS                   PORTS     NAMES
b1a9d392592c   nginx     "/docker-entrypoint.…"   44 seconds ago   Up 43 seconds (Paused)   80/tcp    web03
[root@Rocky8 ~]# ps aux|grep nginx
root       12748  0.0  0.3   8856  5360 ?        Ds   09:41   0:00 nginx: master process nginx -g daemon off;
101        12790  0.0  0.1   9276  2444 ?        D    09:41   0:00 nginx: worker process
101        12791  0.0  0.1   9276  2444 ?        D    09:41   0:00 nginx: worker process
root       12937  0.0  0.0 221940  1148 pts/1    S+   09:42   0:00 grep --color=auto nginx

#取消web03容器挂起
[root@Rocky8 ~]# docker unpause web03 
web03

[root@Rocky8 ~]# ps aux| > grep nginx
root       12748  0.0  0.3   8856  5360 ?        Ss   09:41   0:00 nginx: master process nginx -g daemon off;
101        12790  0.0  0.1   9276  2444 ?        S    09:41   0:00 nginx: worker process
101        12791  0.0  0.1   9276  2444 ?        S    09:41   0:00 nginx: worker process
root       13060  0.0  0.0 221940  1188 pts/1    S+   09:42   0:00 grep --color=auto nginx
```

### 1.4.5、**给正在运行的容器发信号**

**docker kill 可以给容器发信号,默认号SIGKILL,即9信号**

**格式**

```
docker kill [OPTIONS] CONTAINER [CONTAINER...]
#选项:
-s, --signal string   Signal to send to the container (default "KILL")
```

**范例：**

```
[root@Rocky8 ~]# docker top  web01 
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                28815               28795               0                   18:33               ?                   00:00:00            nginx: master process nginx -g daemon off;
101                 29058               28815               0                   18:33               ?                   00:00:00            nginx: worker process
101                 29059               28815               0                   18:33               ?                   00:00:00            nginx: worker process
[root@Rocky8 ~]# docker kill -s 1 web01 
web01
[root@Rocky8 ~]# docker top  web01 
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                28815               28795               0                   18:33               ?                   00:00:00            nginx: master process nginx -g daemon off;
101                 29170               28815               0                   18:34               ?                   00:00:00            nginx: worker process
101                 29171               28815               0                   18:34               ?                   00:00:00            nginx: worker process
```

### **1.4.6** **进入正在运行的容器**

#### **1.4.6.1** 使用attach命令

docker attach 容器名，attach 类似于vnc，操作会在同一个容器的多个会话界面同步显示，所有使用此方式进入容器的操作都是同步显示的，且使用exit退出后容器自动关闭，**不推荐使用**，需要进入到有shell环境的容器

**格式**

```
docker attach [OPTIONS] CONTAINER

```

**范例：**

```
#进入正在运行的容器
[root@Rocky8 ~]# docker run  -it   --name a3 alpine
/ # ls
bin    etc    lib    mnt    proc   run    srv    tmp    var
dev    home   media  opt    root   sbin   sys    usr
/ # 

#窗口会保持同步
[root@Rocky8 ~]# docker attach a3 
/ # ls
bin    etc    lib    mnt    proc   run    srv    tmp    var
dev    home   media  opt    root   sbin   sys    usr
/ # 
```

#### **1.4.6.2** 使用exec命令

在运行中的容器启动新进程,可以执行单次命令，以及进入容器测试环境使用此方式，使用exit退出,但容器还在运行，**此为推荐方式**

**格式:** 

```
[root@Rocky8 ~]# docker exec -h
Flag shorthand -h has been deprecated, please use --help

Usage:  docker exec [OPTIONS] CONTAINER COMMAND [ARG...]

Execute a command in a running container

Aliases:
  docker container exec, docker exec

Options:
  -d, --detach               Detached mode: run command in the background
      --detach-keys string   Override the key sequence for detaching a container
  -e, --env list             Set environment variables
      --env-file list        Read in a file of environment variables
  -i, --interactive          Keep STDIN open even if not attached
      --privileged           Give extended privileges to the command
  -t, --tty                  Allocate a pseudo-TTY
  -u, --user string          Username or UID (format: "<name|uid>[:<group|gid>]")
  -w, --workdir string       Working directory inside the container
```

范例

```
[root@Rocky8 ~]# docker run  -itd ubuntu
0f6def8c24e6b86626f6dce8c56273644289e243a1d156ba37e96008b9d993d0
[root@Rocky8 ~]# docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED         STATUS         PORTS                 NAMES
0f6def8c24e6   ubuntu                "/bin/bash"              2 seconds ago   Up 2 seconds                         happy_swirles
45d3816b4c8b   nginx                 "/docker-entrypoint.…"   45 hours ago    Up 45 hours    80/tcp                web01
53ce8bf67e94   mysql:8.0.29-oracle   "docker-entrypoint.s…"   47 hours ago    Up 45 hours    3306/tcp, 33060/tcp   mysql


#执行一次性命令
[root@Rocky8 ~]# docker exec  happy_swirles cat /etc/os-release
PRETTY_NAME="Ubuntu 22.04.4 LTS"
NAME="Ubuntu"
VERSION_ID="22.04"
VERSION="22.04.4 LTS (Jammy Jellyfish)"
VERSION_CODENAME=jammy
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=jammy

#进入容器，执行命令，exit退出但容器不停止
[root@Rocky8 ~]# docker exec -it happy_swirles bash
root@0f6def8c24e6:/# uname -r 
4.18.0-513.5.1.el8_9.x86_64
root@0f6def8c24e6:/# exit
exit
[root@Rocky8 ~]# docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED         STATUS         PORTS                 NAMES
0f6def8c24e6   ubuntu                "/bin/bash"              2 minutes ago   Up 2 minutes                         happy_swirles
45d3816b4c8b   nginx                 "/docker-entrypoint.…"   45 hours ago    Up 45 hours    80/tcp                web01
53ce8bf67e94   mysql:8.0.29-oracle   "docker-entrypoint.s…"   47 hours ago    Up 45 hours    3306/tcp, 33060/tcp   mysql

```

### **1.4.7** **暴露所有容器端口**

容器启动后,默认处于预定义的NAT网络中,所以外部网络的主机无法直接访问容器中网络服务**docker run -P** 可以将事先容器预定义的所有端口映射宿主机的网卡的随机端口，默认从32768开始使用随机端口时,当停止容器后再启动可能会导致端口发生变化

```
-P , --publish-all= true | false默认为false
#示例:
docker run -P docker.io/nginx  #映射容器所有暴露端口至随机本地端口
```

**docker port** 可以查看容器的端口映射关系

**格式**

```
docker port CONTAINER [PRIVATE_PORT[/PROTO]]
```

**范例**

```
#创建web01容器
[root@Rocky8 ~]# docker run -P --name web01 -d nginx
244c87519cb8333ea056cd1fec86457de776e470b8591f94ec21f47b44eeb9d2
[root@Rocky8 ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                     NAMES
244c87519cb8   nginx     "/docker-entrypoint.…"   3 seconds ago   Up 2 seconds   0.0.0.0:32768->80/tcp, :::32768->80/tcp   web01

[root@Rocky8 ~]# docker port  web01 
80/tcp -> 0.0.0.0:32768
80/tcp -> [::]:32768


#创建web02、暴露端口80端口容器
[root@Rocky8 ~]# docker run -p 80:80 -d --name web02 nginx   #80(宿主机端口):80（容器端口）
e2c8e3cbc3a2b3e817584863c08babfc31742cfb6a369473eb799b566a6a2e92
[root@Rocky8 ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                     NAMES
e2c8e3cbc3a2   nginx     "/docker-entrypoint.…"   4 seconds ago   Up 3 seconds   0.0.0.0:80->80/tcp, :::80->80/tcp         web02
244c87519cb8   nginx     "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes   0.0.0.0:32768->80/tcp, :::32768->80/tcp   web01

[root@Rocky8 ~]# docker port web02 
80/tcp -> 0.0.0.0:80
80/tcp -> [::]:80

#测试
[root@Rocky8 ~]# curl 127.0.0.1
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

**实质改变iptable的DNAT策略，自动生成Iptables规则**

```
[root@Rocky8 ~]# iptables -vnL -t nat
Chain PREROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    5   260 DOCKER     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain POSTROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 MASQUERADE  all  --  *      !docker0  172.17.0.0/16        0.0.0.0/0           
    0     0 MASQUERADE  tcp  --  *      *       172.17.0.2           172.17.0.2           tcp dpt:80
    0     0 MASQUERADE  tcp  --  *      *       172.17.0.3           172.17.0.3           tcp dpt:80

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DOCKER     all  --  *      *       0.0.0.0/0           !127.0.0.0/8          ADDRTYPE match dst-type LOCAL

Chain DOCKER (2 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 RETURN     all  --  docker0 *       0.0.0.0/0            0.0.0.0/0           
    4   208 DNAT       tcp  --  !docker0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:32768 to:172.17.0.2:80
    0     0 DNAT       tcp  --  !docker0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:80 to:172.17.0.3:80
```

### 1.4.8、**范例：安装wordpress应用**

```sh
#创建数据库容器
[root@Rocky8 ~]# docker run -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -e MYSQL_DATABASE=wordpress -e MYSQL_USER=wordpress -e MYSQL_PASSWORD=123456 --name mysql -d -v /data/mysql:/var/lib/mysql --restart=always mysql:8.0.29

#创建wordpress容器
[root@Rocky8 ~]# docker run -p 8080:80 -d --name wordpressserver wordpress
e240c9c5d6fb0cc24a6f7e11e010f67e791131fba4dcd449fb10bd634a7556ed

#查看端口
[root@Rocky8 ~]# docker port wordpressserver 
80/tcp -> 0.0.0.0:8080
80/tcp -> [::]:8080

#本地访问即可http://11.0.1.8:8080
```

### 1.4.9、**查看容器的日志**

docker logs 可以查看容器中运行的进程在控制台输出的日志信息，**实质是终端窗口的标准输出**

**格式**

```sh
docker logs [OPTIONS] CONTAINER
选项:
--details       Show extra details provided to logs
-f, --follow     Follow log output
--since string   Show logs since timestamp (e.g. 2013-01-02T13:23:37) or relative (e.g. 42m for   42 minutes)
--tail string   Number of lines to show from the end of the logs (default "all")
-t, --timestamps     Show timestamps
--until string   Show logs before a timestamp (e.g. 2013-01-02T13:23:37) or relative (e.g. 42m     for 42 minutes)
```

**范例：查看容器日志**

```sh
[root@Rocky8 ~]# docker run --name test01 busybox /bin/sh -c 'i=1;while true;do echo hello$i;let i++;sleep 2;done'
hello1
hello2
hello3
hello4
hello5
hello6
hello7
hello8
hello9
hello10
hello11
hello12
hello13
......

#查看logs,持续跟踪
[root@Rocky8 ~]# docker logs -f test01 
hello1
hello2
hello3
hello4
hello5
hello6
hello7
hello8
hello9
hello10
hello11
hello12
hello13
......

#显示时间
[root@Rocky8 ~]# docker logs --tail 3  -t test01 
2024-03-26T05:46:46.421342864Z hello128
2024-03-26T05:46:48.429061439Z hello129
2024-03-26T05:46:50.432971832Z hello130
```

查看httpd日志

```
[root@Rocky8 ~]# docker run -d -p 80:80 --name web02 httpd
[root@Rocky8 ~]# docker logs --f web01 
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress this message
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress this message
[Tue Mar 26 14:53:53.847040 2024] [mpm_event:notice] [pid 1:tid 140057967406400] AH00489: Apache/2.4.52 (Unix) configured -- resuming normal operations
[Tue Mar 26 14:53:53.847187 2024] [core:notice] [pid 1:tid 140057967406400] AH00094: Command line: 'httpd -D FOREGROUND'
11.0.1.99 - - [26/Mar/2024:14:54:06 +0000] "GET / HTTP/1.1" 200 45
11.0.1.99 - - [26/Mar/2024:14:54:07 +0000] "GET /favicon.ico HTTP/1.1" 404 196
...
```

### 1.4.10、传递运行命令

```sh
[root@Rocky8 ~]# docker run -d -P --name web01 nginx
890c054bd3074dd5e903cf5100afb520f848ecc562ce76c3078b0f40e35e7cdb
[root@Rocky8 ~]# docker top web01 
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                3789                3768                0                   22:26               ?                   00:00:00            nginx: master process nginx -g daemon off;
101                 3835                3789                0                   22:26               ?                   00:00:00            nginx: worker process
101                 3836                3789                0                   22:26               ?                   00:00:00            nginx: worker process



[root@Rocky8 ~]# docker run -d -P --name web02 --rm  nginx tail -f /etc/os-release
46eb7ea7675da757e59871920961b702bb9bc9feb5cc974c815b5c113b4b61ac
[root@Rocky8 ~]# docker top web02
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                4001                3982                0                   22:28               ?                   00:00:00            tail -f /etc/os-release

```

### 1.4.11、修改容器的host文件

容器会自动将容器的ID加入自已的/etc/hosts文件中，并解析成容器的IP

**--rm选项容器结束即删除**

```sh
[root@Rocky8 ~]# docker run -it centos bash
Unable to find image 'centos:latest' locally
latest: Pulling from library/centos
a1d0c7532777: Pull complete 
Digest: sha256:a27fd8080b517143cbbbab9dfb7c8571c40d67d534bbdee55bd6c473f432b177
Status: Downloaded newer image for centos:latest
[root@3ebd0bfb0a65 /]# ls  
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@3ebd0bfb0a65 /]# cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.2	3ebd0bfb0a65
[root@3ebd0bfb0a65 /]# hostname
3ebd0bfb0a65
[root@3ebd0bfb0a65 /]# ping 3ebd0bfb0a65
PING 3ebd0bfb0a65 (172.17.0.2) 56(84) bytes of data.
64 bytes from 3ebd0bfb0a65 (172.17.0.2): icmp_seq=1 ttl=64 time=0.071 ms
64 bytes from 3ebd0bfb0a65 (172.17.0.2): icmp_seq=2 ttl=64 time=0.098 ms
64 bytes from 3ebd0bfb0a65 (172.17.0.2): icmp_seq=3 ttl=64 time=0.097 ms
64 bytes from 3ebd0bfb0a65 (172.17.0.2): icmp_seq=4 ttl=64 time=0.098 ms
64 bytes from 3ebd0bfb0a65 (172.17.0.2): icmp_seq=5 ttl=64 time=0.096 ms
^C
--- 3ebd0bfb0a65 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4076ms
rtt min/avg/max/mdev = 0.071/0.092/0.098/0.010 ms
[root@3ebd0bfb0a65 /]# exit
exit

```

**范例: 修改容器的 hosts文件**

```sh
[root@Rocky8 ~]# docker run --name test01 --add-host "www.dingbh.com:1.1.1.1" --add-host "www.dingbh.top:2.2.2.2" --rm alpine cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
1.1.1.1	www.dingbh.com
2.2.2.2	www.dingbh.top
172.17.0.2	d64b6e3a5707
```

### 1.4.12、指定容器dns

容器的dns服务器，默认采用宿主机的dns 地址，可以用下面方式指定其它的DNS地址将dns地址配置在宿主机

在容器启动时加选项 --dns=x.x.x.x在/etc/docker/daemon.json 文件中指定

范例: 容器的DNS默认从宿主机的DNS获取

```sh
[root@Rocky8 ~]# cat /etc/resolv.conf 
# Generated by NetworkManager
search dingbh.top
nameserver 223.6.6.6
nameserver 11.0.1.7


[root@Rocky8 ~]# docker run --name web01 --rm nginx cat /etc/resolv.conf
# Generated by NetworkManager
search dingbh.top
nameserver 223.6.6.6
nameserver 11.0.1.7
```

**指定dns地址**

```sh
[root@Rocky8 ~]# docker run --name test01 --dns 1.1.1.1 --dns 2.2.2.2  --rm alpine cat /etc/resolv.conf
search dingbh.top
nameserver 1.1.1.1
nameserver 2.2.2.2
```

**指定domain名**

```sh
[root@Rocky8 ~]# docker run --name test01 --dns 1.1.1.1 --dns 2.2.2.2  --dns-search a.com --rm alpine cat /etc/resolv.conf
search a.com
nameserver 1.1.1.1
nameserver 2.2.2.2
```

**范例: 配置文件指定DNS和搜索domain名**

```sh
[root@ubuntu1804 ~]#vim /etc/docker/daemon.json 
[root@ubuntu1804 ~]#cat /etc/docker/daemon.json 
{
  "storage-driver": "overlay2",
  "registry-mirrors": ["https://si7y70hh.mirror.aliyuncs.com"],
   "dns" : [  "114.114.114.114", "119.29.29.29"],
   "dns-search": [ "magedu.com", "wang.org"]
}
[root@ubuntu1804 ~]#systemctl restart docker
[root@ubuntu1804 ~]#docker run -it --rm   centos bash
[root@7a2d8fac6f6b /]# cat /etc/resolv.conf 
search magedu.com wang.org
nameserver 114.114.114.114
nameserver 119.29.29.29
[root@7a2d8fac6f6b /]# exit
exit
#用--dns指定优先级更高
[root@ubuntu1804 ~]#docker run -it --rm --dns 8.8.8.8 --dns 8.8.4.4 centos bash
[root@80ffe3547b87 /]# cat /etc/resolv.conf 
search magedu.com wang.org
nameserver 8.8.8.8
nameserver 8.8.4.4
[root@80ffe3547b87 /]# exit
exit
```

### 1.4.13、容器内和宿主机之间复制文件（面试）

```sh
docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH
Options:
  -a, --archive       Archive mode (copy all uid/gid information)
  -L, --follow-link   Always follow symbol link in SRC_PATH
```

范例：

```sh
[root@Rocky8 ~]# docker run -d --name test01 alpine  sleep 1000
48cd0fc085ae066f51668ab1b65dcd72d199f99ca35fe9325ba64c2ee46299d7
[root@Rocky8 ~]# docker ps
CONTAINER ID   IMAGE     COMMAND        CREATED         STATUS         PORTS     NAMES
48cd0fc085ae   alpine    "sleep 1000"   5 seconds ago   Up 4 seconds             test01

[root@Rocky8 ~]# docker exec -it test01 ls -l
total 12
drwxr-xr-x    2 root     root          4096 Jan 26 17:53 bin
drwxr-xr-x    5 root     root           340 Mar 26 15:12 dev
drwxr-xr-x    1 root     root            66 Mar 26 15:12 etc
drwxr-xr-x    2 root     root             6 Jan 26 17:53 home
drwxr-xr-x    7 root     root          4096 Jan 26 17:53 lib
drwxr-xr-x    5 root     root            44 Jan 26 17:53 media
drwxr-xr-x    2 root     root             6 Jan 26 17:53 mnt
drwxr-xr-x    2 root     root             6 Jan 26 17:53 opt
dr-xr-xr-x  179 root     root             0 Mar 26 15:12 proc
drwx------    2 root     root             6 Jan 26 17:53 root
drwxr-xr-x    2 root     root             6 Jan 26 17:53 run
drwxr-xr-x    2 root     root          4096 Jan 26 17:53 sbin
drwxr-xr-x    2 root     root             6 Jan 26 17:53 srv
dr-xr-xr-x   13 root     root             0 Mar 26 15:12 sys
drwxrwxrwt    2 root     root             6 Jan 26 17:53 tmp
drwxr-xr-x    7 root     root            66 Jan 26 17:53 usr
drwxr-xr-x   12 root     root           137 Jan 26 17:53 var


#从宿主机拷贝文件到容器
[root@Rocky8 ~]# docker cp /etc/issue  test01:/
Successfully copied 2.05kB to test01:/
[root@Rocky8 ~]# docker exec -it test01 ls -l
total 16

......

-rw-r--r--    1 root     root            23 Nov  3 03:10 issue

......

#拷贝文件从容器至宿主机
[root@Rocky8 ~]# docker cp test01:/issue .
Successfully copied 2.05kB to /root/.
[root@Rocky8 ~]# ll
total 2023220
-rw-------  1 root root 1325489152 Mar 19 17:05 allnew.tar
-rw-------  1 root root  746110976 Mar 22 17:31 all.tar
-rw-------. 1 root root       1463 Mar 19 04:43 anaconda-ks.cfg
-rwxr-xr-x  1 root root     161471 Mar 19 06:47 dingbh-v3.sh
-rw-r--r--  1 root root         23 Nov  3 11:10 issue
-rwxr-xr-x  1 root root       2307 Mar 21 23:18 set_docker_proxy.sh
```

### 1.4.14、**使用** **systemd** **控制容器运行**

```sh
[root@ubuntu1804 ~]#cat /lib/systemd/system/hello.service
[Unit] 
Description=Hello World 
After=docker.service 
Requires=docker.service 
[Service] 
TimeoutStartSec=0
ExecStartPre=-/usr/bin/docker kill busybox-hello
ExecStartPre=-/usr/bin/docker rm busybox-hello
ExecStartPre=/usr/bin/docker pull busybox
ExecStart=/usr/bin/docker run --name busybox-hello busybox /bin/sh -c "while 
true; do echo Hello World; sleep 1; done"
ExecStop=/usr/bin/docker kill busybox-hello
[Install] 
WantedBy=multi-user.target
[root@ubuntu1804 ~]#systemctl daemon-reload 
[root@ubuntu1804 ~]#systemctl enable --now hello.service
```



```sh
[root@Rocky8 ~]# docker system -h
Flag shorthand -h has been deprecated, please use --help

Usage:  docker system COMMAND

Manage Docker

Commands:
  df          Show docker disk usage
  events      Get real time events from the server
  info        Display system-wide information
  prune       Remove unused data

Run 'docker system COMMAND --help' for more information on a command.

```



### 1.4.15、 传递环境变量

有些容器运行时，需要传递变量，可以使用 -e <参数> 或 --env-file <参数文件> 实现

范例: 传递变量创建MySQL

变量参考链接: https://hub.docker.com/_/mysql

```sh
#使用命令选项-e传递参数
[root@Rocky8 ~]# docker run -d -p 3306:3306 --env-file=env.txt  --name mysql01  mysql:8.0.29-oracle 


#使用文件传递参数，创建数据库
[root@Rocky8 ~]# cat env.txt 
MYSQL_ROOT_PASSWORD=123456
MYSQL_DATABASE=wordpress
MYSQL_USE=wordpress
MYSQL_PASSWORD=123456

[root@Rocky8 ~]# docker run -d -p 3306:3306 --env-file env.txt --name mysql02 mysql:8.0.29-oracle 
d4e994efe96400bff9f8e562168cfce20bf04f6c0ad9d1d8e7aee6d8ea01225f

#查看数据库文件是否创建成功
[root@Rocky8 ~]# mysql -h11.0.1.8 -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.29 MySQL Community Server - GPL

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
| mysql              |
| performance_schema |
| sys                |
| wordpress          |
+--------------------+
5 rows in set (0.00 sec)

mysql> 

```

## 1.5 实战案例: 利用 docker 快速部署自动化运维平台

![image-20240327113410525](Docker容器/image-20240327113410525.png)

### **1.5.1** **项目说明**

Spug 面向中小型企业设计的轻量级无 Agent 的自动化运维平台，整合了主机管理、主机批量执行、主机在线终端、文件在线上传下载、应用发布部署、在线任务计划、配置中心、监控、报警等一系列功能

```
特性
批量执行: 主机命令在线批量执行
在线终端: 主机支持浏览器在线终端登录
文件管理: 主机文件在线上传下载
任务计划: 灵活的在线任务计划
发布部署: 支持自定义发布部署流程
配置中心: 支持 KV、文本、json 等格式的配置
监控中心: 支持站点、端口、进程、自定义等监控
报警中心: 支持短信、邮件、钉钉、微信等报警方式
优雅美观: 基于 Ant Design 的 UI 界面
开源免费: 前后端代码完全开源

#相关链接
官网地址: https://www.spug.dev/
使用文档：https://www.spug.dev/docs/about-spug/
gitee链接: https://gitee.com/openspug/spug
```

### **1.5.2** **部署过程**

官方说明:

```
https://www.spug.dev/docs/install-docker/
```

#### **1.5.2.1** 安装docker

略

#### **1.5.2.2** **拉取镜像**

```
[root@Rocky8 ~]# docker pull registry.aliyuncs.com/openspug/spug
```

#### **1.5.2.3** **启动容器**

```
[root@Rocky8 ~]# docker run -d --restart=always --name=spug -p 80:80 registry.aliyuncs.com/openspug/spug

# 持久化存储启动命令：
# mydata指的是本地磁盘路径，也可以是其他目录，但需要保证映射的本地磁盘路径已经存在，/data是容
器内代码和数据初始化存储的路径
$ docker run -d --restart=always --name=spug -p 80:80 -v /mydata/:/data 
registry.aliyuncs.com/openspug/spug
```

#### **1.5.2.4** **初始化**

以下操作会创建一个用户名为 admin 密码为 123456 的管理员账户，可自行替换管理员账户。

```sh
[root@Rocky8 ~]# docker exec -it spug init_spug admin 123456
/usr/local/lib/python3.6/site-packages/OpenSSL/_util.py:6: CryptographyDeprecationWarning: Python 3.6 is no longer supported by the Python core team. Therefore, support for it is deprecated in cryptography. The next release of cryptography will remove support for Python 3.6.
  from cryptography.hazmat.bindings.openssl.binding import Binding
Migrations for 'account':
  data/spug/spug_api/apps/account/migrations/0001_initial.py
    - Create model History
    - Create model Role
    - Create model User
    - Add field created_by to role
Migrations for 'alarm':
  data/spug/spug_api/apps/alarm/migrations/0001_initial.py
    - Create model Alarm
    - Create model Group
    - Create model Contact
Migrations for 'config':
  data/spug/spug_api/apps/config/migrations/0001_initial.py
    - Create model Service
    - Create model Environment
    - Create model ConfigHistory
    - Create model Config
Migrations for 'exec':
  data/spug/spug_api/apps/exec/migrations/0001_initial.py
    - Create model Transfer
    - Create model ExecTemplate
    - Create model ExecHistory
Migrations for 'home':
  data/spug/spug_api/apps/home/migrations/0001_initial.py
    - Create model Navigation
    - Create model Notice
Migrations for 'host':
  data/spug/spug_api/apps/host/migrations/0001_initial.py
    - Create model Host
    - Create model HostExtend
    - Create model Group
Migrations for 'monitor':
  data/spug/spug_api/apps/monitor/migrations/0001_initial.py
    - Create model Detection
Migrations for 'notify':
  data/spug/spug_api/apps/notify/migrations/0001_initial.py
    - Create model Notify
Migrations for 'schedule':
  data/spug/spug_api/apps/schedule/migrations/0001_initial.py
    - Create model History
    - Create model Task
Migrations for 'setting':
  data/spug/spug_api/apps/setting/migrations/0001_initial.py
    - Create model Setting
    - Create model UserSetting
Migrations for 'app':
  data/spug/spug_api/apps/app/migrations/0001_initial.py
    - Create model App
    - Create model Deploy
    - Create model DeployExtend1
    - Create model DeployExtend2
Migrations for 'repository':
  data/spug/spug_api/apps/repository/migrations/0001_initial.py
    - Create model Repository
Migrations for 'deploy':
  data/spug/spug_api/apps/deploy/migrations/0001_initial.py
    - Create model DeployRequest
Operations to perform:
  Apply all migrations: account, alarm, app, config, deploy, exec, home, host, monitor, notify, repository, schedule, setting
Running migrations:
  Applying account.0001_initial... OK
  Applying alarm.0001_initial... OK
  Applying config.0001_initial... OK
  Applying app.0001_initial... OK
  Applying repository.0001_initial... OK
  Applying deploy.0001_initial... OK
  Applying exec.0001_initial... OK
  Applying home.0001_initial... OK
  Applying host.0001_initial... OK
  Applying monitor.0001_initial... OK
  Applying notify.0001_initial... OK
  Applying schedule.0001_initial... OK
  Applying setting.0001_initial... OK
初始化/更新成功
/usr/local/lib/python3.6/site-packages/OpenSSL/_util.py:6: CryptographyDeprecationWarning: Python 3.6 is no longer supported by the Python core team. Therefore, support for it is deprecated in cryptography. The next release of cryptography will remove support for Python 3.6.
  from cryptography.hazmat.bindings.openssl.binding import Binding
创建用户成功
```

#### **1.5.2.5** **访问测试**

在浏览器中输入 http://localhost:80 访问。

```
用户名： admin 密码： 123456
```

![image-20240327114322097](Docker容器/image-20240327114322097.png)

![image-20240327114355108](Docker容器/image-20240327114355108.png)



## 1.6、查看docker run启动参数命令

```sh
#安装工具
方法一：pip
[root@Rocky8 ~]# yum -y install python3-pip

[root@Rocky8 ~]# pip3 install runlike

方法二：by docker
[root@Rocky8 ~]# alias runlike="docker run --rm -v /var/run/docker.sock:/var/lib/docker.sock assaflavie/runlike"

#创建测试容器
[root@Rocky8 ~]# docker run  -e MYSQL_ROOT_PASSWORD=123456 -e MYSQL_DATABASE=wordpress -e MYSQL_USER=wordpress -e MYSQL_PASSWORD=123456 --name mysql -d -v /data/mysql:/var/lib/mysql --restart=always mysql:8.0.29-oracle

#测试
[root@Rocky8 ~]# runlike -p mysql
docker run --name=mysql \
	--hostname=53ce8bf67e94 \
	--mac-address=02:42:ac:11:00:02 \
	--env=MYSQL_USER=wordpress \
	--env=MYSQL_PASSWORD=123456 \
	--env=MYSQL_ROOT_PASSWORD=123456 \
	--env=MYSQL_DATABASE=wordpress \
	--volume=/data/mysql:/var/lib/mysql \
	--expose=3306 \
	--expose=33060 \
	--restart=always \
	--runtime=runc \
	--detach=true \
	mysql:8.0.29-oracle \
	mysqld
```

**实战案例:修改已经创建的容器的端口映射关系**

```yaml
#查看容器ID
[root@Rocky8 ~]# docker inspect g1 --format "{{.ID}}"
2963f97a3dd873811bec9c45b33111d1a28b88e01e02076dcfb14df10c9ffe24

#查看容器目录
[root@Rocky8 ~]# find /data/docker/ -name 2963f97a3dd873811bec9c45b33111d1a28b88e01e02076dcfb14df10c9ffe24
/data/docker/containers/2963f97a3dd873811bec9c45b33111d1a28b88e01e02076dcfb14df10c9ffe24
/data/docker/image/overlay2/layerdb/mounts/2963f97a3dd873811bec9c45b33111d1a28b88e01e02076dcfb14df10c9ffe24

[root@Rocky8 ~]# ls /data/docker/containers/2963f97a3dd873811bec9c45b33111d1a28b88e01e02076dcfb14df10c9ffe24
2963f97a3dd873811bec9c45b33111d1a28b88e01e02076dcfb14df10c9ffe24-json.log  config.v2.json   hostname  mounts       resolv.conf.hash
checkpoints                                                                hostconfig.json  hosts     resolv.conf

#停止docker服务
[root@Rocky8 ~]# systemctl stop docker.service 

#修改端口信息文件内容
#PortBindings后80/tcp对应的是容器内部的80端口，HostPort对应的是映射到宿主机的端口80 修改此处为8000
[root@Rocky8 ~]# cat /data/docker/containers/2963f97a3dd873811bec9c45b33111d1a28b88e01e02076dcfb14df10c9ffe24/hostconfig.json 
{"Binds":null,"ContainerIDFile":"","LogConfig":{"Type":"json-file","Config":{}},"NetworkMode":"default","PortBindings":{"3000/tcp":[{"HostIp":"","HostPort":"3333]},"RestartPolicy":{"Name":"no","MaximumRetryCount":0},"AutoRemove":false,"VolumeDriver":"","VolumesFrom":null,"ConsoleSize":[34,169],"CapAdd":null,"CapDrop":null,"CgroupnsMode":"host","Dns":[],"DnsOptions":[],"DnsSearch":[],"ExtraHosts":null,"GroupAdd":null,"IpcMode":"private","Cgroup":"","Links":null,"OomScoreAdj":0,"PidMode":"","Privileged":false,"PublishAllPorts":false,"ReadonlyRootfs":false,"SecurityOpt":null,"UTSMode":"","UsernsMode":"","ShmSize":67108864,"Runtime":"runc","Isolation":"","CpuShares":0,"Memory":0,"NanoCpus":0,"CgroupParent":"","BlkioWeight":0,"BlkioWeightDevice":[],"BlkioDeviceReadBps":[],"BlkioDeviceWriteBps":[],"BlkioDeviceReadIOps":[],"BlkioDeviceWriteIOps":[],"CpuPeriod":0,"CpuQuota":0,"CpuRealtimePeriod":0,"CpuRealtimeRuntime":0,"CpusetCpus":"","CpusetMems":"","Devices":[],"DeviceCgroupRules":null,"DeviceRequests":null,"MemoryReservation":0,"MemorySwap":0,"MemorySwappiness":null,"OomKillDisable":false,"PidsLimit":null,"Ulimits":[],"CpuCount":0,"CpuPercent":0,"IOMaximumIOps":0,"IOMaximumBandwidth":0,"MaskedPaths":["/proc/asound","/proc/acpi","/proc/kcore","/proc/keys","/proc/latency_stats","/proc/timer_list","/proc/timer_stats","/proc/sched_debug","/proc/scsi","/sys/firmware","/sys/devices/virtual/powercap"],"ReadonlyPaths":["/proc/bus","/proc/fs","/proc/irq","/proc/sys","/proc/sysrq-trigger"]}

#开启docker服务
[root@Rocky8 ~]# systemctl start docker.service 

#启动容器查看端口是否修改成功
[root@Rocky8 ~]# docker start g1 
g1
[root@Rocky8 ~]# docker ps
CONTAINER ID   IMAGE             COMMAND                  CREATED             STATUS          PORTS                                                  NAMES
653bbda2995c   mysql:8.0.29      "docker-entrypoint.s…"   About an hour ago   Up 23 seconds   0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp   mysql
2963f97a3dd8   grafana/grafana   "/run.sh"                2 hours ago         Up 2 seconds    0.0.0.0:3333->3000/tcp, :::3333->3000/tcp              g1

#测试
浏览器访问成功http://11.0.1.8:3333/login
```



# 2、※Docker 管理容器和镜像

## **2.1 Docker** **镜像说明**

### **2.1.1 Docker** **镜像中有没有内核**

```
从镜像大小上面来说，一个比较小的镜像只有1MB多点或几MB，而内核文件需要几十MB， 因此镜像里面是没有内核的，镜像在被启动为容器后将直接使用宿主机的内核，而镜像本身则只提供相应的rootfs，即系统正常运行所必须的用户空间的文件系统，比如: /dev/，/proc，/bin，/etc等目录，容器当中/boot目录是空的，而/boot当中保存的就是与内核相关的文件和目录。
```

### **2.1.2** **为什么没有内核**

```
由于容器启动和运行过程中是直接使用了宿主机的内核，不会直接调用物理硬件，所以也不会涉及到硬件驱动，因此也无需容器内拥有自已的内核和驱动。而如果使用虚拟机技术，对应每个虚拟机都有自已独立的内核
```

### 2.1.3、**容器中的程序后台运行会导致此容器启动后立即退出**

```
Docker容器如果希望启动后能持续运行,就必须有一个能前台持续运行的进程，如果在容器中启动传统的服务，如:httpd,php-fpm等均为后台进程模式运行,就导致 docker 在前台没有运行的应用,这样的容器启动后会立即退出。所以一般会将服务程序以前台方式运行，对于有一些可能不知道怎么实现前台运行的程序,只需要在你启动的该程序之后添加类似于 tail ，top 这种可以前台运行的程序即可. 比较常用的方法，如 tail -f /etc/hosts 。
```

范例

```
#httpd
ENTRYPOINT [ "/usr/sbin/apache2" ]  
CMD ["-D", "FOREGROUND"]  
#nginx
ENTRYPOINT [ "/usr/sbin/nginx", "-g", "daemon off;" ]  
#用脚本运行容器
cat run_haproxy.sh 
#!/bin/bash
haproxy -f /etc/haproxy/haproxy.cfg
tail -f /etc/hosts
tail -n1 Dockerfile 
CMD ["run_haproxy.sh"] 
```

### 2.1.4 docker 镜像生命周期

![image-20240327135831481](Docker容器/image-20240327135831481.png)



### **2.1.5** **制作镜像方式**

Docker 镜像制作类似于虚拟机的镜像（模版）制作，即按照公司的实际业务需求将需要安装的软件、相关配置等基础环境配置完成，然后将其做成镜像，最后再批量从镜像批量生成容器实例，这样可以极大的简化相同环境的部署工作.

Docker的镜像制作分为手动制作（基于容器）和自动制作(基于DockerFile)，企业通常都是基于Dockerfile制作镜像

```
docker commit #通过修改现有容器,将之手动构建为镜像

docker build  #通过Dockerfile文件,批量构建为镜像
```

## 2.2 将现有容器通过 docker commit 手动构建镜像

### **2.2.1** **基于容器手动制作镜像步骤**（不推荐）

**docker commit** **格式**

```sh
[root@Rocky8 ~]# docker commit -h
Flag shorthand -h has been deprecated, please use --help

Usage:  docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

Create a new image from a container's changes

Aliases:
  docker container commit, docker commit

Options:
  -a, --author string    Author (e.g., "John Hannibal Smith <hannibal@a-team.com>")
  -c, --change list      Apply Dockerfile instruction to the created image
  -m, --message string   Commit message
  -p, --pause            Pause container during commit (default true)


#说明:
制作镜像和CONTAINER状态无关,停止状态也可以制作镜像
如果没有指定[REPOSITORY[:TAG]],REPOSITORY和TAG都为<none>
提交的时候标记TAG号: 生产当中常用，后期可以根据TAG标记创建不同版本的镜像以及创建不同版本的容器
```

**基于容器手动制作镜像步骤具体如下**:

```sh
1. 下载一个系统的官方基础镜像，如: CentOS 或 Ubuntu 
2. 基于基础镜像启动一个容器,并进入到容器 
3. 在容器里面做配置操作
	安装基础命令
	配置运行环境
	安装服务和配置服务
	放业务程序代码
4. 提交为一个新镜像 docker commit
5. 基于自己的的镜像创建容器并测试访问
```

### **2.2.2** **实战案例**:基于 **busybox** **制作** **httpd** **镜像**

```sh
1、运行系统容器，进入容器定制，手动操作
[root@Rocky8 ~]# docker run -it --name busybox busybox:latest sh

/ # httpd -h
httpd: option requires an argument -- 'h'
BusyBox v1.36.1 (2023-05-18 22:34:17 UTC) multi-call binary.

Usage: httpd [-ifv[v]] [-c CONFFILE] [-p [IP:]PORT] [-u USER[:GRP]] [-r REALM] [-h HOME]
or httpd -d/-e/-m STRING

Listen for incoming HTTP requests

	-i		Inetd mode
	-f		Run in foreground
	-v[v]		Verbose
	-p [IP:]PORT	Bind to IP:PORT (default *:80)
	-u USER[:GRP]	Set uid/gid after binding to port
	-r REALM	Authentication Realm for Basic Authentication
	-h HOME		Home directory (default .)
	-c FILE		Configuration file (default {/etc,HOME}/httpd.conf)
	-m STRING	MD5 crypt STRING
	-e STRING	HTML encode STRING
	-d STRING	URL decode STRING
/ # mkdir -p /data/html
/ # exit

2、生成镜像:
#格式1
[root@ubuntu1804 ~]#docker commit -a "wang<root@wangxiaochun.com>" -c 'CMD 
/bin/httpd -fv -h /data/html' -c "EXPOSE 80" b1 httpd-busybox:v1.0
#格式2
[root@ubuntu1804 ~]#docker commit -a "wang<root@wangxiaochun.com>" -c 'CMD 
["/bin/httpd", "-f", "-v","-h", "/data/html"]' -c "EXPOSE 80" b1 httpdbusybox:v1.0

[root@Rocky8 ~]# docker images
REPOSITORY                            TAG                 IMAGE ID       CREATED          SIZE
httpdbusybox                          v1.00               e0685578d089   9 seconds ago    4.26MB

[root@Rocky8 ~]# docker run -it --name b1 busybox:v1.0

[root@Rocky8 ~]# docker images
REPOSITORY                            TAG                 IMAGE ID       CREATED         SIZE
busybox                               v1.0                9d6bbf19378c   7 minutes ago   4.26MB
```

## **2.3** **利用DockerFile文件执行docker build自动构建镜像**

### 2.3.1 Dockfile 使用详解

#### **2.3.1.1 Dockerfile** **介绍**

```sh
DockerFile 是一种被Docker程序解释执行的脚本，由一条条的命令组成的，每条命令对应linux下面的一条命令，Docker程序将这些DockerFile指令再翻译成真正的linux命令，其有自己的书写方式和支持的命令，Docker程序读取DockerFile并根据指令生成Docker镜像，相比手动制作镜像的方式，DockerFile更能直观的展示镜像是怎么产生的，有了DockerFile，当后期有额外的需求时，只要在之前的DockerFile添加或者修改响应的命令即可重新生成新的Docker镜像，避免了重复手动制作镜像的麻烦,类似与shell脚本一样,可以方便高效的制作镜像。

Docker守护程序 Dockerfile 逐一运行指令，如有必要，将每个指令的结果提交到新镜像，然后最终输出新镜像的ID。Docker守护程序将自动清理之前发送的上下文

请注意，每条指令都是独立运行的，并会导致创建新镜像，比如 RUN cd /tmp 对下一条指令不会有任何影响。

Docker将尽可能重用中间镜像层（缓存），以显著加速 docker build 命令的执行过程，这由 Using cache 控制台输出中的消息指示
```

#### 2.3.1.2 Dockerfile 镜像制作和使用流程

![image-20240328101151354](Docker容器/image-20240328101151354.png)

#### 2.3.1.3 Dockerfile文件的制作镜像的分层结构

![image-20240328101214074](Docker容器/image-20240328101214074.png)



范例: 

```sh
#按照业务类型或系统类型等方式划分创建目录环境，方便后期镜像比较多的时候进行分类

[root@Rocky8 ~]# mkdir /data/dockerfile/{web/{nginx,apache,tomcat,jdk},system/{centos,ubuntu,alpine,debian}} -p

[root@Rocky8 ~]# tree /data/dockerfile/
/data/dockerfile/
├── system
│   ├── alpine
│   ├── centos
│   ├── debian
│   └── ubuntu
└── web
    ├── apache
    ├── jdk
    ├── nginx
    └── tomcat

10 directories, 0 files
```

#### 2.3.1.4 Dockerfile 文件格式

Dockerfile 是一个有特定语法格式的文本文件

dockerfile 官方说明: https://docs.docker.com/engine/reference/builder/

帮助: man 5 dockerfile 

**Dockerfile 文件说明**

```
Dockerfile 文件说明
	每一行以Dockerfile的指令开头，指令不区分大小写，但是惯例使用大写
	使用 # 开始作为注释
	每一行只支持一条指令，每条指令可以携带多个参数
	指令按文件的顺序从上至下进行执行
	每个指令的执行会生成一个新的镜像层，为了减少分层和镜像大小，尽可能将多条指令合并成一条指令
	制作镜像一般可能需要反复多次，每次执行dockfile都按顺序执行，从头开始，已经执行过的指令
	已经缓存，不需要再执行，如果后续有一行新的指令没执行过，其往后的指令将会重新执行，所以
	为加速镜像制作，将最常变化的内容放下dockerfile的文件的后面
```

#### **2.3.1.5 Dockerfile** 相关指令

dockerfile 文件中的常见指令:

```
ADD
COPY
ENV
EXPOSE
FROM
LABEL
STOPSIGNAL
USER
VOLUME
WORKDIR
```

![image-20240328101600579](Docker容器/image-20240328101600579.png)



##### **2.3.1.5.1 FROM:** **指定基础镜像**

定制镜像，需要先有一个基础镜像，在这个基础镜像上进行定制

```sh
FROM** 就是指定基础镜像，此指令通常必需放在Dockerfile文件第一个非注释行。后续的指令都是运行于此基准镜像所提供的运行环境

基础镜像可以是任何可用镜像文件，默认情况下，docker build会在docker主机上查找指定的镜像文件，在其不存在时，则会从Docker Hub Registry上拉取所需的镜像文件.如果找不到指定的镜像文件，docker build会返回一个错误信息
```

**如何选择合适的镜像呢？**

```
对于不同的软件官方都提供了相关的docker镜像，比如: nginx、redis、mysql、httpd、tomcat等服务类的镜像，也有操作系统类，如: centos、ubuntu、debian等。建议使用官方镜像，比较安全。
```

**格式：**

```
FROM [--platform=<platform>] <image> [AS <name>]
FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]
FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]
#说明:  
--platform 指定镜像的平台，比如: linux/amd64, linux/arm64, or windows/amd64 tag 和 digest是可选项，如果不指定，默认为latest
```

**说明: 关于scratch 镜像**

```
FROM scratch
参考链接:
https://hub.docker.com/_/scratch?tab=description
https://docs.docker.com/develop/develop-images/baseimages/
该镜像是一个空的镜像，可以用于构建busybox等超小镜像，可以说是真正的从零开始构建属于自己的镜像
该镜像在构建基础镜像（例如debian和busybox）或超最小镜像（仅包含一个二进制文件及其所需内容，例
如:hello-world）的上下文中最有用
```

**范例：**

```
FROM scratch #所有镜像的起源镜像，相当于Object类
FROM ubuntu
FROM ubuntu:bionic
FROM debian:buster-slim 
```

##### **2.3.1.5.2 LABEL:** **指定镜像元数据**

可以指定镜像元数据，如: 镜像作者等

https://github.com/gliderlabs/docker-alpine/blob/df6f51eceabaeee4e2f04187403e0e816ca8736e/versions/library-3.2/Dockerfile

```sh
格式:
LABEL <key>=<value> <key>=<value> <key>=<value> ...

LABEL "com.example.vendor"="ACME Incorporated"
LABEL com.example.label-with-value="foo"
LABEL version="1.0"
LABEL description="This text illustrates \
that label-values can span multiple lines."
```

一个镜像可以有多个label ,还可以写在一行中,即多标签写法,可以减少镜像的的大小

范例: 多标签写法

```
老版作者添加 #MAINTAINER ding<ding@qq.com> 

#一行格式
LABEL multi.label1="value1" multi.label2="value2" other="value3"
#多行格式
LABEL multi.label1="value1" \
     multi.label2="value2" \
      other="value3"
```

docker inspect 命令可以查看LABEL

**范例:** 

```yaml
"Labels": {
    "com.example.vendor": "ACME Incorporated"
    "com.example.label-with-value": "foo",
    "version": "1.0",
    "description": "This text illustrates that label-values can span multiple 
lines.",
    "multi.label1": "value1",
    "multi.label2": "value2",
    "other": "value3"
},
```

**MAINTAINER:** **指定维护者信息**

此指令已过时，用LABEL代替

```
MAINTAINER <name>
```

**范例:** 

```sh
MAINTAINER wangxiaochun <root@wangxiaochun.com>
#用LABEL代替
LABEL maintainer="wangxiaochun <root@wangxiaochun.com>"
```

##### **2.3.1.5.3 RUN:执行shell命令**

**RUN** 指令用来在**构建镜像阶段**需要执行 FROM 指定镜像所支持的Shell命令。

通常各种基础镜像一般都支持丰富的shell命令

注意: RUN 可以写多个，每一个RUN指令都会建立一个镜像层，所以尽可能合并成一条指令,比如将多个shell命令通过 && 连接一起成为在一条指令

每个RUN都是独立运行的,和前一个RUN无关

```sh
#shell 格式: 相当于 /bin/sh -c <命令> 此种形式支持环境变量
RUN <命令> 

#exec 格式: 此种形式不支持环境变量,注意:是双引号,不能是单引号
RUN ["executable","param1","param2"...] 

#exec格式可以指定其它shell
RUN ["/bin/bash","-c","echo hello wang"]
```

**说明:**

```sh
shell格式中，<command>通常是一个shell命令，且以"/bin/sh -c”来运行它，这意味着此进程在容器中的PID不为1，不能接收Unix信号，因此，当使用docker stop <container>命令停止容器时，此进程接收不到SIGTERM信号

exec格式中的参数是一个JSON格式的数组，其中<executable>为要运行的命令，后面的<paramN>为传递给命令的选项或参数;然而，此种格式指定的命令不会以"/bin/sh -c"来发起，因此常见的shell操作如变量替换以及通配符(?,*等)替换将不会进行;不过，如果要运行的命令依赖于此shell特性的话，可以将其替换
为类似下面的格式。
RUN ["/bin/bash", "-c", "<executable>", "<param1>"]
```

**范例:** 

```sh
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
RUN ["/bin/bash", "-c", "echo hello world"]
RUN yum -y install epel-release \
     && yum -y install nginx \
     && rm -rf /usr/share/nginx/html/*
     && echo "<h1> docker test nginx </h1>" > /usr/share/nginx/html/index.html
```

```sh
alpine软件管理

#修改国内源
/ # sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/' /etc/apk/repositories
/ # apk update
fetch https://mirrors.aliyun.com/alpine/v3.18/main/x86_64/APKINDEX.tar.gz
fetch https://mirrors.aliyun.com/alpine/v3.18/community/x86_64/APKINDEX.tar.gz
v3.18.6-114-ge2fe3df7380 [https://mirrors.aliyun.com/alpine/v3.18/main]
v3.18.6-112-g494bcb94505 [https://mirrors.aliyun.com/alpine/v3.18/community]
OK: 20079 distinct packages available

/ # apk --help
apk-tools 2.14.0, compiled for x86_64.

usage: apk [<OPTIONS>...] COMMAND [<ARGUMENTS>...]

Package installation and removal:
  add        Add packages to WORLD and commit changes
  del        Remove packages from WORLD and commit changes

System maintenance:
  fix        Fix, reinstall or upgrade packages without modifying WORLD
  update     Update repository indexes
  upgrade    Install upgrades available from repositories
  cache      Manage the local package cache

Querying package information:
  info       Give detailed information about packages or repositories
  list       List packages matching a pattern or other criteria
  dot        Render dependencies as graphviz graphs
  policy     Show repository policy for packages
  search     Search for packages by name or description

Repository maintenance:
  index      Create repository index file from packages
  fetch      Download packages from repositories to a local directory
  manifest   Show checksums of package contents
  verify     Verify package integrity and signature

Miscellaneous:
  audit      Audit system for changes
  stats      Show statistics about repositories and installations
  version    Compare package versions or perform tests on version strings

This apk has coffee making abilities.
For more information: man 8 apk


#安装常用软件
apk update && apk  --no-cache add iotop gcc libgcc libc-dev libcurl libc-utils pcre-dev zlib-dev libnfs make pcre pcre2 zip unzip net-tools telnet pstree wget libevent libevent-dev iproute2
```

**范例：基于alpine:3.18.0生成自定义镜像**

```sh
#基于alpine:3.18.0生成自定义镜像
[root@ubuntu2004 /data/dockerfile/base/alpine]#cat Dockerfile 
FROM alpine:3.18.0
LABEL maintainer="dingbaohang <root@dingbh.top>" version=1.0
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/' /etc/apk/repositories && apk update && apk add tzdata && ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo "Asia/Shanghai" > /etc/timezone && apk --no-cache add iotop gcc libgcc libc-dev libcurl libc-utils pcre-dev zlib-dev libnfs make

[root@ubuntu2004 ~]#tree /data/dockerfile/base/alpine/
/data/dockerfile/base/alpine/
├── Dockerfile

0 directories, 2 files

#创建镜像（切记末端添加路径）
[root@ubuntu2004 ~]#docker build -t alpine-init:v1.0 .

#验证镜像是否生成
[root@ubuntu2004 ~]#docker images
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
alpine-init   v1.0      c35bfb0e7734   11 minutes ago   205MB    #自定义镜像已生成
alpine        3.18.0    5e2b554c1c45   10 months ago    7.33MB
busybox       latest    beae173ccac6   2 years ago      1.24MB
hello-world   latest    feb5d9fea6a5   2 years ago      13.3kB

#创建镜像
[root@ubuntu2004 ~]#docker run -it --name a1 sh
Unable to find image 'sh:latest' locally
^C
[root@ubuntu2004 /data/dockerfile/base/alpine]#docker run -it --name a1 alpine-init:v1.0 sh
/ # cat /etc/timezone 
Asia/Shanghai
/ # 
```

##### **2.3.1.5.4 ENV:** **设置环境变量**

ENV 可以定义环境变量和值，会被后续指令(如:ENV,ADD,COPY,RUN等)通过$KEY或${KEY}进行引用，并在容器运行时保持

```sh
#变量赋值格式1
ENV <key> <value>   #此格式只能对一个key赋值,<key>之后的所有内容均会被视作其<value>的组成部分
#变量赋值格式2
ENV <key1>=<value1> <key2>=<value2> \  #此格式可以支持多个key赋值,定义多个变量建议使用,减少镜像层
 <key3>=<value3> ...
 
#如果<value>中包含空格，可以以反斜线\进行转义，也可通过对<value>加引号进行标识;另外，反斜线也可用于续行
#只使用一次变量
RUN <key>=<value> <command>
    
#引用变量
RUN $key .....
#变量支持高级赋值格式
${key:-word}
${kye:+word}
```

如果运行容器时如果需要修改变量,可以执行下面通过基于 exec 机制实现

**注意: 下面方式只影响容器运行时环境,而不影响构建镜像的过程,即只能覆盖docker run时的环境变量,而不会影响docker build时环境变量的值**

```
docker run -e|--env <key>=<value>
#说明
-e, --env list   #Set environment variables
    --env-file filename     #Read in a file of environment variables
```

示例: 两种格式功能相同

```sh
#格式1
ENV myName="John Doe" myDog=Rex\ The\ Dog \
    myCat=fluffy
#格式2
ENV myName John Doe
ENV myDog Rex The Dog
ENV myCat fluffy
```

**范例**

```sh
#创建dockerfile文件
[root@ubuntu2004 /data/dockerfile/base/alpine]#cat Dockerfile 
FROM alpine:3.18.0
LABEL maintainer="dingbaohang <root@dingbh.top>" version=1.0
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/' /etc/apk/repositories && apk update && apk add tzdata && ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo "Asia/Shanghai" > /etc/timezone && apk --no-cache add iotop gcc libgcc libc-dev libcurl libc-utils pcre-dev zlib-dev libnfs make
ENV version=1.0 name=ding		#添加环境变量

#基于Dockerfile文件生成镜像
[root@ubuntu2004 /data/dockerfile/base/alpine]#docker build -t alpine-init:v1.1 .
[+] Building 0.1s (6/6) FINISHED                                                                                                                          docker:default
 => [internal] load build definition from Dockerfile                                                                                                                0.0s
 => => transferring dockerfile: 453B                                                                                                                                0.0s
 => [internal] load metadata for docker.io/library/alpine:3.18.0                                                                                                    0.0s
 => [internal] load .dockerignore                                                                                                                                   0.0s
 => => transferring context: 2B                                                                                                                                     0.0s
 => [1/2] FROM docker.io/library/alpine:3.18.0                                                                                                                      0.0s
 => CACHED [2/2] RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/' /etc/apk/repositories && apk update && apk add tzdata && ln -s /usr/share/zoneinfo/Asia  0.0s
 => exporting to image                                                                                                                                              0.0s
 => => exporting layers                                                                                                                                             0.0s
 => => writing image sha256:73d08a0307301ad28bc233caee65a6710e09248ed4c1fc288abd3c35ab035e76                                                                        0.0s
 => => naming to docker.io/library/alpine-init:v1.1                                                                                                                 0.0s
#查看新生成镜像
[root@ubuntu2004 /data/dockerfile/base/alpine]#docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
alpine-init   v1.1      73d08a030730   14 hours ago    205MB
alpine-init   v1.0      c5bb40541811   14 hours ago    205MB
alpine        3.18.0    5e2b554c1c45   10 months ago   7.33MB
busybox       latest    beae173ccac6   2 years ago     1.24MB
hello-world   latest    feb5d9fea6a5   2 years ago     13.3kB

#基于镜像创建容器，并查看内置环境变量
[root@ubuntu2004 /data/dockerfile/base/alpine]#docker run alpine-init:v1.1 env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=2c95ecf5a09e
version=1.0
name=ding
HOME=/root
 
#手动修改环境变量
[root@ubuntu2004 /data/dockerfile/base/alpine]#docker run  -e version=2.0 -e name=hang alpine-init:v1.1 env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=479320e5cd41
version=2.0
name=hang
HOME=/root
```

##### 2.3.1.5.5 COPY: 复制文本

复制本地宿主机的到容器中的 。

```
COPY [--chown=<user>:<group>] <src>... <dest>
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"] #路径中有空白字符时,建议使用此格式
```

说明: 

```
可以是多个,可以使用通配符，通配符规则满足Go的filepath.Match 规则filepath.Match 参考链接:         https://golang.org/pkg/path/filepath/#Match

必须是build上下文中的路径(为 Dockerfile 所在目录的相对路径），**不能是其父目录中的文件**

如果是目录，则其内部文件或子目录会被递归复制，**但目录自身不会被复制**

如果指定了多个, 或在中使用了通配符，则必须是一个目 录，**且必须以** **/** **结尾**

可以是绝对路径或者是 WORKDIR 指定的相对路径

使用 COPY 指令，源文件的各种元数据都会保留。比如读、写、执行权限、文件变更时间等

如果事先不存在，它将会被自动创建，这包括其父目录路径,即递归创建目录
```



```sh
[root@ubuntu2004 /data/dockerfile/base/ubuntu]#cat test.sh 
#! /bin/sh

cat > /test.log <<EOF

name:${name:-wang}

EOF
[root@ubuntu2004 /data/dockerfile/base/ubuntu]#chmod +x test.sh
[root@ubuntu2004 /data/dockerfile/base/ubuntu]#cat Dockerfile 
FROM ubuntu:22.04
LABEL maintainer="dingbaohang <root@dingbh.top>" version=1.0
COPY test.sh /test2.sh
RUN /test2.sh

[root@ubuntu2004 /data/dockerfile/base/ubuntu]#docker build -t ubuntu:v1.0 .

[root@ubuntu2004 /data/dockerfile/base/ubuntu]#docker run -it --name u1 --rm ubuntu:v1.0 cat /test.log

name:wang


[root@ubuntu2004 /data/dockerfile/base/ubuntu]#docker run -it -e name=ding --name u1 --rm ubuntu:v1.0 cat /test.log

name:ding

```



##### 2.3.1.5.6 ADD: 复制和解包文件

该命令可认为是增强版的COPY，不仅支持COPY，还支持自动解压缩。可以将复制指定的 到容器中的

```
ADD [--chown=<user>:<group>] <src>... <dest>
ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]
```

说明: 

```
可以是Dockerfile所在目录的一个相对路径；也可是一个 URL；还可是一个 tar 文件（自动解压）

可以是绝对路径或者是 WORKDIR 指定的相对路径

如果是目录，只复制目录中的内容，而非目录本身

如果是一个 URL ，下载后的文件权限自动设置为 600 

如果为URL且不以/结尾，则指定的文件将被下载并直接被创建为,如果以 / 结尾，则文件名URL指定的文件将被直接下载并保存为/< filename>

如果是一个本地文件系统上的打包文件,如: gz, bz2 ,xz ，它将被解包 ，其行为类似于"tar -x"命令,但是通过URL获取到的tar文件将不会自动展开

如果有多个，或其间接或直接使用了通配符，则必须是一个以/结尾的目录路径;如果不以/结尾，则其被视作一个普通文件，的内容将被直接写入到
```

**范例:** 

```sh
#样例
ADD test relativeDir/          # adds "test" to `WORKDIR`/relativeDir/
ADD test /absoluteDir/         # adds "test" to /absoluteDir/
ADD --chown=55:mygroup files* /somedir/
ADD --chown=bin files* /somedir/
ADD --chown=1 files* /somedir/
ADD --chown=10:11 files* /somedir/
ADD ubuntu-xenial-core-cloudimg-amd64-root.tar.gz

#实验步骤
[root@Rocky8 /data/dockerfile/base/alpine]# ll
total 2432
-rw-r--r-- 1 root root      95 Mar 27 14:13 Dockerfile
-rw-r--r-- 1 root root 2484499 Mar 28  2024 rootfs.tar.gz
[root@Rocky8 /data/dockerfile/base/alpine]# echo $OLDPWD
/data/dockerfile/base

[root@Rocky8 /data/dockerfile/base/alpine]# vim Dockerfile 

  1 FROM alpine:3.18.0                                                                                                                                                   
  2 LABEL maintainer="dingbaohang <root@dingbh.top>" version=1.0
  3 ADD rootfs.tar.gz /tmp

[root@Rocky8 /data/dockerfile/base/alpine]# docker build -t a1:v1.0 .
[+] Building 0.3s (7/7) FINISHED                                                                                                                          docker:default
 => [internal] load build definition from Dockerfile                                                                                                                0.0s
 => => transferring dockerfile: 141B                                                                                                                                0.0s
 => [internal] load metadata for docker.io/library/alpine:3.18.0                                                                                                    0.0s
 => [internal] load .dockerignore                                                                                                                                   0.0s
 => => transferring context: 2B                                                                                                                                     0.0s
 => [internal] load build context                                                                                                                                   0.0s
 => => transferring context: 37B                                                                                                                                    0.0s
 => [1/2] FROM docker.io/library/alpine:3.18.0                                                                                                                      0.0s
 => [2/2] ADD rootfs.tar.gz /tmp                                                                                                                                    0.2s
 => exporting to image                                                                                                                                              0.0s
 => => exporting layers                                                                                                                                             0.0s
 => => writing image sha256:e9877782a88ee5d3755a918d5bab4ec40c9b58fa4e430cd552aecb1bc8de2f9a                                                                        0.0s
 => => naming to docker.io/library/a1:v1.0                                                                                                                          0.0s
        
[root@Rocky8 /data/dockerfile/base/alpine]# docker run -it --rm --name a1 a1:v1.0 ls /tmp
bin      dev      etc      home     lib      linuxrc  media    mnt      proc     root     run      sbin     sys      tmp      usr      var

```

##### 2.3.1.5.7 CMD: 容器启动命令(一般后跟前台命令)

![image-20240329220514007](Docker容器/image-20240329220514007.png)

一个容器中需要持续运行的进程一般只有一个,CMD 用来指定启动容器时默认执行的一个命令，且其运行结束后,容器也会停止,所以一般CMD 指定的命令为持续运行且为前台命令。

```
如果docker run没有指定任何的执行命令或者dockerfile里面也没有ENTRYPOINT命令，那么开启容器时就会使用执行CMD指定的默认的命令。

前面介绍过的 RUN 命令是在构建镜像时执行的命令,注意二者的不同之处。

每个 Dockerfile 只能有一条 CMD 命令。如指定了多条，只有最后一条被执行。

如果用户启动容器时用 docker run xxx 指定运行的命令，则会覆盖 CMD 指定的命令。
```

##### **范例：基于alpine-init：v1.0创建nginx镜像**

```
#准备dockerfile目录
[root@ubuntu2004 ~]# mkdir /data/dockerfile/web/nginx/ -p
[root@ubuntu2004 ~]# tree /data/dockerfile/web/
/data/dockerfile/web/
├── apache
├── nginx
│   ├── Dockerfile
│   ├── index.html
│   ├── nginx-1.24.0.tar.gz
│   └── nginx.conf
└── tomecat

3 directories, 4 files

#创建nginx访问页及nginx配置文件和nginx二进制安装包
[root@ubuntu2004 ~]# cat  /data/dockerfile/web/nginx/index.html 
<h1>welcome to dingbh website</h1>

[root@ubuntu2004 ~]# grep -Ev ".*+#|^$"  /data/dockerfile/web/nginx/nginx.conf 
worker_processes  auto;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  localhost;
        location / {					#添加访问路径
            root   /data/nginx/html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}

#创建Dockerfile文件
[root@ubuntu2004 ~]# cat  /data/dockerfile/web/nginx/Dockerfile 
FROM alpine-init:v1.0
LABEL maintainer="dingbaohang <root@dingbh.top>" version="1.0"
ADD nginx-1.24.0.tar.gz /usr/local/src/
RUN cd /usr/local/src/nginx-1.24.0 && ./configure --prefix=/apps/nginx/ && make -j 2 && make install && ln -s /apps/nginx/sbin/nginx /usr/bin/ && addgroup -g 2019 -S nginx && adduser -s /sbin/nologin -S -D -u 2019 -G nginx nginx
COPY nginx.conf /apps/nginx/conf/nginx.conf
ADD index.html /data/nginx/html/index.html
RUN chown -R nginx.nginx /apps/nginx/
#EXPOSE 80 443
CMD ["nginx","-g","daemon off;"]

#创建镜像
[root@ubuntu2004 ~]# docker build -t mynginx:v1.0 .

#查看生成的镜像
[root@ubuntu2004 ~]# docker image ls mynginx:v1.0 
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
mynginx      v1.0      0d29561a9912   29 minutes ago   229MB

#基于mynginx:V1.0镜像创建容器
[root@ubuntu2004 ~]# docker run  -d -p 80:80 --name web01 mynginx:v1.0 

#查看容器
[root@ubuntu2004 ~]# docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                               NAMES
7e7718fc3cf6   mynginx:v1.0   "nginx -g 'daemon of…"   14 minutes ago   Up 14 minutes   0.0.0.0:80->80/tcp, :::80->80/tcp   web01

#访问测试
[root@Rocky8 ~]# curl 11.0.1.100
<h1>welcome to dingbh website</h1>

[root@Rocky8 ~]# curl -I 11.0.1.100 
HTTP/1.1 200 OK
Server: nginx/1.24.0
Date: Mon, 01 Apr 2024 05:27:31 GMT
Content-Type: text/html
Content-Length: 35
Last-Modified: Mon, 01 Apr 2024 04:47:45 GMT
Connection: keep-alive
ETag: "660a3c71-23"
Accept-Ranges: bytes
```

![image-20240401132707685](Docker容器/image-20240401132707685.png)

##### **2.3.1.5.8 ENTRYPOINT:** **入口点**

功能类似于CMD，配置容器启动后执行的命令及参数

```
# 使用 exec 执行
ENTRYPOINT ["executable", "param1", "param2"...]

# shell中执行
ENTRYPOINT command param1 param2	
```

**docker常用命令from、label、env、run、copy、add（基础镜像-->定制镜像）、cmd（镜像-->容器）**

```
ENTRYPOINT 不能被 docker run 提供的参数覆盖，而是追加,即如果docker run 命令有参数，那么参数全部都会作为ENTRYPOINT的参数

如果docker run 后面没有额外参数，但是dockerfile中有CMD命令（即上面CMD的第三种用法），即Dockerfile中即有CMD也有ENTRYPOINT,那么CMD的全部内容会作为ENTRYPOINT的参数

如果docker run 后面有额外参数，同时Dockerfile中即有CMD也有ENTRYPOINT,那么docker run后面的参数覆盖掉CMD参数内容,最终作为ENTRYPOINT的参数

可以通过docker run --entrypoint string 参数在运行时替换,注意string不要加空格
使用CMD要在运行时重新写命令本身,然后在后面才能追加运行参数，ENTRYPOINT则可以运行时无需重写命令就可以直接接受新参数

每个 Dockerfile 中只能有一个 ENTRYPOINT，当指定多个时，只有最后一个生效

通常会利用ENTRYPOINT指令配合脚本,可以为CMD指令提供环境配置
```

**特点：**

```sh
CMD 作为容器启动默认进程，docker run image xxx 此xxx替换CMD
ENTRTPOINT 作为容器启动默认进程，docker run image xxx 此xxx将不作为替换，而追加到entrypoint后面作为参数，如果cmd也存在，CMD将成为entrypoint参数

一般ENTORYPONT主要用于定制不同容器进行初始化容器运行环境
ENTRYPOINT和CMD组合，ENTORYPONT主要用于定制不同容器进行初始化容器运行环境,ENTORYPONT是一个脚本最后一句exec $@, CMD负责定义容器启动进程
```

**范例：ENTRYPOINT创建定制容器**

```sh
#创建dockerfile文件
[root@ubuntu2004 /data/dockerfile/web/nginx-env]# cat Dockerfile 
FROM mynginx:v1.0
LABEL maintainer="dingbaohang <root@dingbh.top>" version="1.0"
ENV HOST=www.a.com PORT=8080
RUN mkdir -p /data/website && mkdir -p /apps/nginx/conf/conf.d/ && chown -R nginx.nginx /apps/nginx
COPY index.html /data/website
RUN cat > /apps/nginx/conf/conf.d/mysite.conf <<EOF
server {
	listen $PORT;
	server_name $HOST;
	root /data/website/;

}
EOF
CMD ["-g","daemon off;"]
#EXPOSE 80 443
ENTRYPOINT ["nginx"]

#创建镜像
[root@ubuntu2004 /data/dockerfile/web/nginx-env]# docker build -t nginx-alpine-entryponit:v1.0 .

#生成容器
[root@ubuntu2004 /data/dockerfile/web/nginx-env]# docker run -d -p 88:80 --name web02 nginx-alpine-entryponit:v1.0 

#查看容器
[root@ubuntu2004 /data/dockerfile/web/nginx-env]# docker ps
CONTAINER ID   IMAGE                          COMMAND                  CREATED              STATUS              PORTS                               NAMES
bc9587a7711a   nginx-alpine-entryponit:v1.0   "nginx -g 'daemon of…"   About a minute ago   Up About a minute   0.0.0.0:88->80/tcp, :::88->80/tcp   web02
```



# 面试题

```
docker 打镜像时需不需要对容器内操作系统进行内核参数优化？
Dockerfile有哪些指令?
docker 有哪些常用指令？
两个服务器装了docker，双方的容器之间如何实现通信,有几种方式，不同主机的两个容器之间能
够通信吗，怎么实现?
docker 的网络模式有几种？
docker版本号是多少？
docker容器中如何对外映射内部的端口？
如何找到某个docker容器的输出日志的宿主机位置？
docker的dockerfile中的expose端口暴露与命令执行暴露的区别？
docker查看镜像？
docker如何实现容器互联？
docker使用到那些技术？ 分别有什么作用？
docker run运行一个容器，端口映射为8000，一段时间之后忘记当初运行这个容器的命令，请问如何修改端口的映射？
dockerfile有哪些常见的指令？ CMD和entrypoint有什么区别？ADD和COPY有什么区别？
dockerfile CMD,RUN区别？
dockerfile优化？
```

docker 私有仓库  harbor 
