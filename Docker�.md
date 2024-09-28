1.安装Docker的依赖库。

```
yum install -y yum-utils device-mapper-persistent-data lvm2
```

2.添加Docker CE的软件源信息。

```
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

3.安装Docker CE。

```
yum makecache fast
yum -y install docker-ce
```

4.启动Docker服务。

```
systemctl start docker
```

5.配置Docker的自定义镜像仓库地址。请将下面命令中的镜像仓库地址https://kqh8.mirror.aliyuncs.com替换为阿里云为您提供的专属镜像加速地址。

```
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://kqh8.mirror.aliyuncs.com"]
}
EOF
```

6.重新加载服务配置文件。

```
systemctl daemon-reload
```

7.重启Docker服务。

```
systemctl restart docker
```

8.查看Docker镜像仓库中Nginx的可用版本。

```
docker search nginx
```

命令输出如下所示。

![img](https://ucc.alicdn.com/pic/developer-ecology/y4dn6eatoa22k_5b1b8bfe1cd84a88bfb01e6c33aeb8f2.png)

9.拉取最新版的Nginx镜像。

```
docker pull nginx:latest
```

![img](https://ucc.alicdn.com/pic/developer-ecology/y4dn6eatoa22k_e67605155fec4c4ebf870f6c2eea3a71.png) 

10.查看本地镜像。

```
docker images
```

![img](https://ucc.alicdn.com/pic/developer-ecology/y4dn6eatoa22k_8b598a4d000b4dbc8d18b4c01e46a6d4.png) 

11.运行容器。

```
docker run --name nginx-test -p 8080:80 -d nginx
```

命令参数说明：

- --name nginx-test：容器名称。
- -p 8080:80： 端口进行映射，将本地8080端口映射到容器内部的80端口。
- -d nginx： 设置容器在后台一直运行。

命令输出如下所示。

![img](https://ucc.alicdn.com/pic/developer-ecology/y4dn6eatoa22k_1ca66fab94964b529ec210bc825fce30.png) 

12.在浏览器地址栏输入http://<ECS公网IP地址>:8080访问Nginx服务。