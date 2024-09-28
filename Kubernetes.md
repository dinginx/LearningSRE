# Kubernetes

# 1、Kubernetes集群示例环境部署

## 1.1、部署前提

```
#使用kubeadm部署Kubernetes集群的前提条件
    支持Kubernetes运行的Linux主机，例如Debian、RedHat及其变体等
    每主机2GB以上的内存，以及2颗以上的CPU
    各主机间能够通过网络正常通信，支持各节点位于不同的网络中
    独占的hostname、MAC地址以及product_uuid，主机名能够正常解析
    放行由Kubernetes使用到的各端口，或直接禁用iptables
    禁用各主机的上的Swap设备
    各主机时间同步
#准备代理服务，以便接入registry.k8s.io，或根据部署过程提示的方法获取相应的Image
#重要提示
     kubeadm不仅支持集群部署，还支持集群升级、卸载、更新数字证书等功能
     目前，kubeadm为各节点默认生成的SSL证书的有效期限为1年，在到期之前需要renew这些证书
```

Kubernetes集群示例环境

```
1、OS: Ubuntu 22.04 LTS
2、Docker：25.0，CGroup Driver: systemd
3、Kubernetes：v1.27
	◼ CRI: cri-dockerd, CNI: CoreOS Flannel
4、网络环境
	◼ 节点网络：11.0.1.0/16
	◼ Pod网络：10.244.0.0/16
	◼ Service网络：10.96.0.0/12
```

|  IP地址   |                     主机名                      |  角色  |
| :-------: | :---------------------------------------------: | :----: |
| 11.0.1.20 | master01,master01.dingbh.top,kubeapi.dingbh.top | Master |
| 11.0.1.21 |                                                 | Master |
| 11.0.1.22 |                                                 | Master |
| 11.0.1.30 |            node01,node01.dingbh.top             | Worker |
| 11.0.1.31 |            node02,node02.dingbh.top             | Worker |
| 11.0.1.32 |            node03,node03.dingbh.top             | Worker |

```sh
/etc/hosts

11.0.1.10 k8s-master01.dingbh.top k8s-master01 kubeapi.dingbh.top k8sapi.dingbh.top kubeapi
11.0.1.101 k8s-node01.dingbh.top k8s-node01
11.0.1.102 k8s-node02.dingbh.top k8s-node02
11.0.1.103 k8s-node03.dingbh.top k8s-node03
```

## 1.2、初始化步骤：

### 1.2.1、关闭swap

```
#注释掉swap开机挂载
root@ubuntu2204:~# cat /etc/fstab 
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/ubuntu-vg/ubuntu-lv during curtin installation
/dev/disk/by-id/dm-uuid-LVM-Q7NRoi9OPuh2siV1cA19XMstTzfDnkgIw7c025djVqYzxo82VGKQQi2PusxyHMbx / ext4 defaults 0 1
# /boot was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/fe430f55-6be5-4178-8444-48f1ba45012b /boot ext4 defaults 0 1
# /data was on /dev/ubuntu-vg/lv-0 during curtin installation
#/dev/disk/by-id/dm-uuid-LVM-Q7NRoi9OPuh2siV1cA19XMstTzfDnkgImunGc548L6MewT9xaByBZ3bA3M2AQkDL /data ext4 defaults 0 1
/dev/disk/by-id/dm-uuid-LVM-Q7NRoi9OPuh2siV1cA19XMstTzfDnkgIdgnr015bEE38wg3V6ACZBfcXpdHVY6Ct none swap sw 0 0


root@ubuntu2204:~# systemctl mask swap.target 

root@ubuntu2204:~# free -h
               total        used        free      shared  buff/cache   available
Mem:           1.9Gi       320Mi       431Mi       1.0Mi       1.2Gi       1.4Gi
Swap:             0B          0B          0B

#临时禁用
root@ubuntu2204:~# swapoff -a

```

### 1.2.2、禁用防火墙

```
root@ubuntu2204:~# apt install ufw
root@ubuntu2204:~# ufw disable
root@ubuntu2204:~# ufw status
```

## 1.3、安装程序包

### 1.3.1、安装docker

```
# step 1: 安装必要的一些系统工具
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
# step 2: 安装GPG证书
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
# Step 3: 写入软件源信息
sudo add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
# Step 4: 更新并安装Docker-CE
sudo apt-get -y update
sudo apt-get -y install docker-ce=23.0.5

# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# apt-cache madison docker-ce
#   docker-ce | 17.03.1~ce-0~ubuntu-xenial | https://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
#   docker-ce | 17.03.0~ce-0~ubuntu-xenial | https://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
# Step 2: 安装指定版本的Docker-CE: (VERSION例如上面的17.03.1~ce-0~ubuntu-xenial)
# sudo apt-get -y install docker-ce

#离线安装docker23.0.5版本
root@ubuntu2204:~# wget https://mirrors.aliyun.com/docker-ce/linux/ubuntu/dists/jammy/pool/stable/amd64/docker-ce_23.0.6-1~ubuntu.22.04~jammy_amd64.deb
root@ubuntu2204:~# dpkg -i docker-ce_23.0.6-1~ubuntu.22.04~jammy_amd64.deb
#缺少依赖包解决办法
root@ubuntu2204:~# apt install --fix-broken
root@ubuntu2204:~# docker info #检查docker版本信息

#检查
root@ubuntu2204:~# docker info
Client: Docker Engine - Community
 Version:    25.0.3
 Context:    default
 Debug Mode: false
 Plugins:
  buildx: Docker Buildx (Docker Inc.)
    Version:  v0.12.1
    Path:     /usr/libexec/docker/cli-plugins/docker-buildx
  compose: Docker Compose (Docker Inc.)
    Version:  v2.24.5
    Path:     /usr/libexec/docker/cli-plugins/docker-compose

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 23.0.6
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Using metacopy: false
  Native Overlay Diff: true
  userxattr: false
 Logging Driver: json-file
 Cgroup Driver: systemd
 Cgroup Version: 2
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: ae07eda36dd25f8a1b98dfbf587313b99c0190bb
 runc version: v1.1.12-0-g51d5e94
 init version: de40ad0
 Security Options:
  apparmor
  seccomp
   Profile: builtin
  cgroupns
 Kernel Version: 5.15.0-69-generic
 Operating System: Ubuntu 22.04.2 LTS
 OSType: linux
 Architecture: x86_64
 CPUs: 2
 Total Memory: 1.896GiB
 Name: ubuntu2204
 ID: a0c95804-d98b-4257-8894-b62eea18755f
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false
```

### 1.3.2、安装部署cri-dockerd

```
root@ubuntu2204:~# curl -LO https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.1/cri-dockerd_0.3.1.3-0.ubuntu-jammy_amd64.deb
root@ubuntu2204:~# dpkg -i cri-dockerd_0.3.1.3-0.ubuntu-jammy_amd64.deb
root@ubuntu2204:~# systemctl restart cri-docker.service
```

### 1.3.3、安装部署kubelet、kubectl、kubeadm

```shell
root@ubuntu2204:~# apt update && apt-get install -y apt-transport-https
root@ubuntu2204:~# curl -fsSL https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.27/deb/Release.key |gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

root@ubuntu2204:~# echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.27/deb/ /" |tee /etc/apt/sources.list.d/kubernetes.list

root@ubuntu2204:~# apt update && apt install -y kubelet kubeadm kubectl
```

### 1.3.4、修改配置文件

```shell
root@ubuntu2204:~# sed -in.bak  '/^ExecStart/c ExecStart=/usr/bin/cri-dockerd --container-runtime-endpoint fd:// --network-plugin=cni --cni-bin-dir=/opt/cni/bin --cni-cache-dir=/var/lib/cni/cache --cni-conf-dir=/etc/cni/net.d' /lib/systemd/system/cri-docker.service

root@ubuntu2204:~# systemctl daemon-reload && systemctl restart cri-docker.service 
```

```shell
root@ubuntu2204:~# cat >> /etc/sysconfig/kubelet <<EOF 
KUBELET_KUBEADM_ARGS="--container-runtime=remote --container-runtime-endpoint=/run/cri-dockerd.sock"
EOF
```

**至此各节点通用配置已完成**

## 1.4、主节点初始化

```shell
root@ubuntu2204:~# kubeadm config images list

[root@master1 ~]# kubeadm config images list --image-repository=registry.aliyuncs.com/google_containers

[root@master1 ~]# docker image pull registry.aliyuncs.com/google_containers/pause:3.6

[root@master1 ~]# docker image tag registry.aliyuncs.com/google_containers/pause:3.6 registry.k8s.io/pause:3.6

root@ubuntu2204:~# kubeadm config images pull --cri-socket unix:///run/cri-dockerd.sock --image-repository=registry.aliyuncs.com/google_containers
I0217 13:40:24.315922    2059 version.go:256] remote version is much newer: v1.29.2; falling back to: stable-1.27
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-apiserver:v1.27.11
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-controller-manager:v1.27.11
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-scheduler:v1.27.11
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-proxy:v1.27.11
[config/images] Pulled registry.aliyuncs.com/google_containers/pause:3.9
[config/images] Pulled registry.aliyuncs.com/google_containers/etcd:3.5.10-0
[config/images] Pulled registry.aliyuncs.com/google_containers/coredns:v1.10.1

#init主master节点

kubeadm init \        
      --control-plane-endpoint="kubeapi.dingbh.top" \
      --kubernetes-version=v1.28.1 \
      --pod-network-cidr=10.244.0.0/16 \
      --service-cidr=10.96.0.0/12 \
      --token-ttl=0 \
      --cri-socket unix:///run/cri-dockerd.sock \
      --image-repository=registry.aliyuncs.com/google_containers
      --upload-certs   #高可用需要配置此项
      
kubeadm init --control-plane-endpoint="kubeapi.dingbh.top" --kubernetes-version=v1.27.11 --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12 --token-ttl=0 --cri-socket unix:///run/cri-dockerd.sock  --image-repository=registry.aliyuncs.com/google_containers
      --upload-certs   #高可用需要配置此项
```

## 1.5、master01节点配置

```shell
#初始化master01节点    
[root@master01 ~]# kubeadm init --control-plane-endpoint="kubeapi.dingbh.top" --kubernetes-version=v1.28.7 --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12 --token-ttl=0 --cri-socket unix:///run/cri-dockerd.sock  --image-repository=registry.aliyuncs.com/google_containers

#显示以下信息即成功

	. . . . . .

[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join kubeapi.dingbh.top:6443 --token jb1934.kupzgtw89u39yuv3 \
	--discovery-token-ca-cert-hash sha256:014f5b366e86cc3e435ef2ee869233383748ead1d244da15a65a1788726c9a83 \
	--control-plane 

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join kubeapi.dingbh.top:6443 --token jb1934.kupzgtw89u39yuv3 \
	--discovery-token-ca-cert-hash sha256:014f5b366e86cc3e435ef2ee869233383748ead1d244da15a65a1788726c9a83 
	
[root@master01 ~]# mkdir .kube
[root@master01 ~]# cp /etc/kubernetes/admin.conf .kube/config

#显示信息证明配置成功
root@master01:~# kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://kubeapi.dingbh.top:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED


#查看隐藏配置信息
root@master01:~# kubectl config view --raw
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJVFdBM29CMW8vRTh3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TkRBeU1UY3dPVFV3TURoYUZ3MHpOREF5TVRRd09UVTFNRGhhTUJVeApFekFSQmdOVkJBTVRDbXQxWW1WeWJtVjBaWE13Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLCkFvSUJBUURuYnNLakZqbS9yVGNRSy9iWjdTVG5FMHAvWW16b3h1bHNnUWJNRjdFSW1sdE9iNkxwOFlZaEhQa2QKNWI2elVNbE42Z2w5YjdEZGZKbnJxSFhJbCtBemxTNHRua0l6U3FDTEZjbkZBR2puMkpxOTN3YUZ1VkhLaHc2UwptdnNIdWhhUi9VOWNEdWFFeXZ5Q3ZZUFNHOUwxNEg2aWRnM3c1TXpFZllRTzFKdEl6ZDZFaC9IQkpFVVpMK2xECjFaM2RGYlN0K3pZK1VraDMzbnhrQnZ5NUc3M2E0VjgyOG9PR0pNUlJWZUpkenNYTmpTdVpPUWhwT01rUlJSYTIKM3JpSWx2YlgxRC9xaXRwZlU1aGM0aFplT2kwL0hsZEVDQlIxbEJuNmVvS2E3VWFSVXByUGcyVWJaY0lUOWdUUQpFUlY4ZzA1R1cwbEIxUjdYNXEwRGRhTSs2Tlh0QWdNQkFBR2pXVEJYTUE0R0ExVWREd0VCL3dRRUF3SUNwREFQCkJnTlZIUk1CQWY4RUJUQURBUUgvTUIwR0ExVWREZ1FXQkJRVS9MdTViM1ZCMUlCa2ZqSHFDL05HUjFwdkNUQVYKQmdOVkhSRUVEakFNZ2dwcmRXSmxjbTVsZEdWek1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQmFsVzFZTjN6dwpwVTIxcnlIMTBsdlY4VVdHdlRESnFjMmhoWDIyYTVVbUhQYk5FTThUTDBabG91blEwRndhdmd1VWhQRk10MDdDCkRLUTNlM1BWbVBxTFdpS1VndGRpZ05YWVhTOXYzZm5kcjc4M0pSMnVIOGdIUU9HbXI3a09URVROWlNlN2xYUksKbzVLMm5xeHV1RUZsaUR5ekFjeFZGZnB3eVFaNGlvWVZwZy9NUi9UVUEvRExKY1Z1eHlHNUMxV2g3NStuaXZTZQpuTjZpSGVrczFQbW83c2g3Z1FhaVl2dmNDUk5jYjErTHlqeVZxdUIxVWc2bGdaUGtXbUc2SDlJSzc2OGpVUDg2CmJFRzI0bnhSWktkQTQxVDZiREx5NkcrdUhIY3JDazk0dTQzWU4wZ2ZRNFpnL2JDU3cweUYvWnhTNWFNbkdESm0KVkRJY3ZRTDgrVU44Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    server: https://kubeapi.dingbh.top:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURJVENDQWdtZ0F3SUJBZ0lJZTNBU3dld25pdnN3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TkRBeU1UY3dPVFV3TURoYUZ3MHlOVEF5TVRZd09UVTFNVEJhTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGRHVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXYrL3ljeGxYRUIwcXNYZkwKUHRlTmhFK2x3TjhZQkQvVmpYQ3BwYWkrd3RMNDU4SUZ4eUswZm9Kb081eXo3bWlvR3lkempmUTZEbDBqYUpabQpWcnJVNDNaUmxTUGRFaVZDOGRXeWI3MHYwZ1lrVkJPclJiWkU4RzNlVWtpTFQ2NUt4YnQ2UXRpZXE4RktOTHFLCjMwK3F4SHYxN2ZFbTdhZFhSK0VWWmV1YzU3eTZiZy9BaFNrdGRMdHQyUmhjWThMaURTT3ZuRHA3clJBc0g2ZWgKb2FvM2l0bHd3RHcyb0Z2NU11VmJYTkxhYWhPZmZTNkI1ZkFwS2tuaUd1aGg3bEdQV2lTRW9icUk4ak1ic3A2Qgp2eW52NEdXVVdPdnJiZ2I3R2ZjRjgyemFobVhwQVJMa1J5RHNxRUl6SVpSaXdHdG1KQWZuT2dkNDluajNXdXg5CkRRdEhPd0lEQVFBQm8xWXdWREFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0RBWURWUjBUQVFIL0JBSXdBREFmQmdOVkhTTUVHREFXZ0JRVS9MdTViM1ZCMUlCa2ZqSHFDL05HUjFwdgpDVEFOQmdrcWhraUc5dzBCQVFzRkFBT0NBUUVBSUVkOSszZzV0Znp2aDZYREs0WVdsd1FSUlR1RVVsWDdFeTRXCjJYWFdWRDRFclNpVFRIWi9CdURObHRiQ0F4Qm5xTnFyenFUdW5jZVpSVEM3c3p3N2J0Y3RONUJsbWdydHlmQ00KMzdIck1XVmw5ZWp2VGp2dGRvb0FHZU03WEFYd1ZlYm5xVW9YbjNQMURrejRIRiswZVBIT1dnL3M2ZWNKUzBrQgo3SS8rTkxHbHkydll6M05kN2xDdkNuclBYODA5dTlncE5QUy9nYWFYVWF3MHlJdHl2alBKeVZIRG1jL3M4SVVHCnYreUZJTFN6MDJ1b00ybE8vMFBDNWYwcUdIeWtpZldLaWtIOEJ5ajY1RHFIT244NEZtUEtNZzlPdE5pOEVvZW8KUkJ2dTYySjdNSVdISkgrRUZFczNiVUxpLzhINXpaeUE3WFUwZC9NOUZKZlJUUmMxc3c9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcFFJQkFBS0NBUUVBdisveWN4bFhFQjBxc1hmTFB0ZU5oRStsd044WUJEL1ZqWENwcGFpK3d0TDQ1OElGCnh5SzBmb0pvTzV5ejdtaW9HeWR6amZRNkRsMGphSlptVnJyVTQzWlJsU1BkRWlWQzhkV3liNzB2MGdZa1ZCT3IKUmJaRThHM2VVa2lMVDY1S3hidDZRdGllcThGS05McUszMCtxeEh2MTdmRW03YWRYUitFVlpldWM1N3k2YmcvQQpoU2t0ZEx0dDJSaGNZOExpRFNPdm5EcDdyUkFzSDZlaG9hbzNpdGx3d0R3Mm9GdjVNdVZiWE5MYWFoT2ZmUzZCCjVmQXBLa25pR3VoaDdsR1BXaVNFb2JxSThqTWJzcDZCdnludjRHV1VXT3ZyYmdiN0dmY0Y4MnphaG1YcEFSTGsKUnlEc3FFSXpJWlJpd0d0bUpBZm5PZ2Q0OW5qM1d1eDlEUXRIT3dJREFRQUJBb0lCQVFDVDkwdm1kWU9MMWFCagpIZUhldVZmTXg0WVYwU2IzcDZwZkZwcUNManVaZ0FFSTBhZVlvK1dYbmJyek1kUDRtRS80VndxYlJjczM1WTkxCmduUzBxZVdTMFZVRmI1ZHU5TERPTkpzUnp6dzlZaVRCZUVSU1FyaVV6VzVWNnIwVEdWOWxYaWxidlJsZGVXNVcKaGZTblFRaDUzc0p4dzdYcUFsYWtBQjU1NnhDVUNMRHhOdVgwWldiRTlGRXIzWlg3WDZDT0RSVFJQQVl4NHU3RQoybDhVSHZHU1hGY204V1pXV2pxVHpFYXpNQ0tDeE4zbDgwdGpqYWNwS0ZFQ0lTamNXTFRESGZtQzBmV0dwQWdHCjVDTXUwS0VnSzc1S0t1dFlzNzBUMSt2R2FnVXNYUzgzSHVDRjhpTGxhclM1b0wxZjhBTmFMS1lxVi9YUyt0eWMKQkhPVEFnR3hBb0dCQU04QkpnOVhVQ29jSGo4enpDQUVxTXV4T3NEOWpEWmVVaWdBUlFnM1lqaWZTazlhelB6Ngoxd2l0djh5b1p1R2tPMXg4L21qRHZTYXlPOUZDNjJCUFZGUU9ha2VqcGpzMW40SEs2VFBZWENEWndsais4clZtClJFeG93VTB2dUV3N1RJdlNPdkhrdEloRlgzNHU2eDlSU05tTlRNTEwyR2xCZ09IOXdVdU5aTmVqQW9HQkFPMWQKMkN0b3FKZU1pNVF0WTNRZWF5ZVZXSjZjUG92WnppNTcyaVBvc1U5U1huUktlRjFZUm14YU5HTFNmREFNbVRNVQp4aTBJVDg2TnJ2emZWYWlaK0ZEcTUwNVh5eEJCZEs4ZHI2WXZKRER5V3VjMWxoZkc5VXc2TTRUL2tWOW9EaDlaClRmV0MvdXpzaEhXOWpvSjRIekYvbHd4Z25TQTRkdUNHb0drS2JxdUpBb0dCQUl3RVllTVVreE80cXpqaTUxcHMKN0dQREFGUFQ3SWhGNmxwR1BHdGZVdCtyMXc2d0EySEdwYmFvRjhqdHk1RXFqa0IwQjMyRGtLamcvQmNDcGNVaQpRNDhMcFJYSitzR245cmxMTDh3eXQ3WTh0UG1Qa1VuajM2U3pZclM3NTNRZzFCanJYWldLdGhaS25sbnlrSFZCCmZQNHlWazZZcFV3azAzR0pPN3JIN0FMekFvR0JBTFFnWS92RkJ6TkNKMlhCNzlmZGdZbGZUd25HSGJ5NUF6ckoKSitmYUhEdktrUDBFeWtSanljVk55clUzYjZCeTN3c2hkNllTeWFWQVpZdGwyN1gvYWEzV0pNT1I0Zm9yQWVZbAp5L1o3amFaTE0rcURBcXgyZ2cxZm1Wc1RLZmxSbUczM3NpNWVReC85K0NMMVB5RUpFUHJhRFFFS2pCSTRaeFhKCmRBOWlLRXVwQW9HQVFJdUp0VFhrMjhRKy9EUGZLVHU3T25VZzAwTWxPdHovWldWci9pS0hLT3dlOGdTK2oxTUIKUlpGR0cvQnkyeHJZaEVpVmNUL1FiM3Q1cXRGZ1FFbzc5UmxCampTcENiYVRxMENKVmF2b2NLN0RlblZBMjFuVQptS1ZpNXdFREhMTkZSSzdFaFFTK1hQQ05ZWFFxUVBkY21zbW41aURQY3pKZlN5cE0rcnJpQ0xjPQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=


#部署网络插件
root@master01:~# kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
namespace/kube-flannel created
serviceaccount/flannel created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created

#查看节点
root@k8s-master01:~# kubectl get pods -n kube-flannel
NAME                    READY   STATUS    RESTARTS      AGE
kube-flannel-ds-26f97   1/1     Running   1 (16m ago)   35m
kube-flannel-ds-2bvs6   1/1     Running   1 (16m ago)   27m
kube-flannel-ds-nlkdj   1/1     Running   1 (16m ago)   37m
kube-flannel-ds-sqqvz   1/1     Running   1 (16m ago)   30m


#查看主节点已ready
root@k8s-master01:~# kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
k8s-master01   Ready    control-plane   40m   v1.27.11
k8s-node01     Ready    <none>          36m   v1.27.11
k8s-node02     Ready    <none>          30m   v1.27.11
k8s-node03     Ready    <none>          27m   v1.27.11

#有高可用需求需要执行（可选） 手动添加--control-plane --cri-socket unix:///run/cri-dockerd.sock
kubeadm join kubeapi.dingbh.top:6443 --token dzho7x.2x9falxbu125xyh7 --discovery-token-ca-cert-hash sha256:d20cd8989de53814971c9cf6f7e1d03f99fdd09e521013e45034445d9055322d --control-plane --cri-socket unix:///run/cri-dockerd.sock

#查看节点信息
root@k8s-master01:~# kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
k8s-master01   Ready    control-plane   38m   v1.27.11
k8s-node01     Ready    <none>          34m   v1.27.11
k8s-node02     Ready    <none>          29m   v1.27.11
k8s-node03     Ready    <none>          25m   v1.27.11
```

## 1.6、woker节点配置及添加

```
#node01、node02、node03配置
kubeadm join kubeapi.dingbh.top:6443 --token aj6ema.jbq65upadm8789gl \
--discovery-token-ca-cert-hash sha256:b2ab3e6cc016d736f22bc55e37e3e46d39dfa859500ee0a28b371729bb651bc4 \
--cri-socket unix:///run/cri-dockerd.sock
```

**kubeadm-init文件(每次初始化内容会不同，切记先进行保存)**

```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join kubeapi.dingbh.top:6443 --token dzho7x.2x9falxbu125xyh7 \
	--discovery-token-ca-cert-hash sha256:d20cd8989de53814971c9cf6f7e1d03f99fdd09e521013e45034445d9055322d \
	--control-plane 

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join kubeapi.dingbh.top:6443 --token dzho7x.2x9falxbu125xyh7 \
	--discovery-token-ca-cert-hash sha256:d20cd8989de53814971c9cf6f7e1d03f99fdd09e521013e45034445d9055322d 
```

**验证集群状态**

```
#全部节点及namespace为running状态
root@k8s-master01:~# kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
k8s-master01   Ready    control-plane   23m   v1.27.11
k8s-node01     Ready    <none>          18m   v1.27.11
k8s-node02     Ready    <none>          17m   v1.27.11
k8s-node03     Ready    <none>          17m   v1.27.11
root@k8s-master01:~# kubectl get pod -n kube-system
NAME                                   READY   STATUS    RESTARTS   AGE
coredns-7bdc4cb885-b675p               1/1     Running   0          22m
coredns-7bdc4cb885-fvvh2               1/1     Running   0          22m
etcd-k8s-master01                      1/1     Running   0          23m
kube-apiserver-k8s-master01            1/1     Running   0          23m
kube-controller-manager-k8s-master01   1/1     Running   0          23m
kube-proxy-d24vz                       1/1     Running   0          22m
kube-proxy-g6hb5                       1/1     Running   0          18m
kube-proxy-gt2vh                       1/1     Running   0          19m
kube-proxy-qpgxw                       1/1     Running   0          18m
kube-scheduler-k8s-master01            1/1     Running   0          23m
root@k8s-master01:~# kubectl get pod -n kube-flannel
NAME                    READY   STATUS    RESTARTS   AGE
kube-flannel-ds-2d87x   1/1     Running   0          18m
kube-flannel-ds-k2gm8   1/1     Running   0          18m
kube-flannel-ds-rwvx4   1/1     Running   0          22m
kube-flannel-ds-xcqfw   1/1     Running   0          19m

```

测试

## 1.7、部署Add-ons

### 1.7.1、安装openELB

```
kubectl apply -f https://raw.githubusercontent.com/openelb/openelb/master/deploy/openelb.yaml

[root@master1 ~]# kubectl apply -f https://raw.githubusercontent.com/openelb/openelb/master/deploy/openelb.yaml
namespace/openelb-system created
customresourcedefinition.apiextensions.k8s.io/bgpconfs.network.kubesphere.io created
customresourcedefinition.apiextensions.k8s.io/bgppeers.network.kubesphere.io created
customresourcedefinition.apiextensions.k8s.io/eips.network.kubesphere.io created
serviceaccount/kube-keepalived-vip created
serviceaccount/openelb-admission created
role.rbac.authorization.k8s.io/leader-election-role created
role.rbac.authorization.k8s.io/openelb-admission created
clusterrole.rbac.authorization.k8s.io/kube-keepalived-vip created
clusterrole.rbac.authorization.k8s.io/openelb-admission created
clusterrole.rbac.authorization.k8s.io/openelb-manager-role created
rolebinding.rbac.authorization.k8s.io/leader-election-rolebinding created
rolebinding.rbac.authorization.k8s.io/openelb-admission created
clusterrolebinding.rbac.authorization.k8s.io/kube-keepalived-vip created
clusterrolebinding.rbac.authorization.k8s.io/openelb-admission created
clusterrolebinding.rbac.authorization.k8s.io/openelb-manager-rolebinding created
service/openelb-admission created
deployment.apps/openelb-manager created
job.batch/openelb-admission-create created
job.batch/openelb-admission-patch created
mutatingwebhookconfiguration.admissionregistration.k8s.io/openelb-admission created
validatingwebhookconfiguration.admissionregistration.k8s.io/openelb-admission created

[root@master1 ~]# kubectl get ns
NAME              STATUS   AGE
default           Active   36m
kube-flannel      Active   18m
kube-node-lease   Active   36m
kube-public       Active   36m
kube-system       Active   36m
openelb-system    Active   37s
```

### 1.7.2、部署Ingress Nginx（按需部署）

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.7.1/deploy/static/provider/cloud/deploy.yaml

kubectl get pods -n ingress-nginx
```



### 1.7.3、部署Kuboard(按需部署)



```
 kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.7.1/deploy/static/provider/cloud/deploy.yaml
```

## 1.8、拆除整个集群

**清除节点,!!!慎用!!!，生产禁用**

```
[root@node3 ~]# kubeadm reset --cri-socket unix:///run/cri-dockerd.sock &&  rm -rf /etc/kubernetes/ /var/lib/kubelet /var/lib/dockershim /var/run/kubernetes /var/lib/cni /etc/cni/net.d /var/lib/etcd /run/flannel/
W0213 18:11:02.459288   11639 preflight.go:56] [reset] WARNING: Changes made to this host by 'kubeadm init' or 'kubeadm join' will be reverted.
[reset] Are you sure you want to proceed? [y/N]: y
[preflight] Running pre-flight checks
W0213 18:11:03.636996   11639 removeetcdmember.go:106] [reset] No kubeadm config, using etcd pod spec to get data directory
[reset] Deleted contents of the etcd data directory: /var/lib/etcd
[reset] Stopping the kubelet service
[reset] Unmounting mounted directories in "/var/lib/kubelet"
[reset] Deleting contents of directories: [/etc/kubernetes/manifests /var/lib/kubelet /etc/kubernetes/pki]
[reset] Deleting files: [/etc/kubernetes/admin.conf /etc/kubernetes/kubelet.conf /etc/kubernetes/bootstrap-kubelet.conf /etc/kubernetes/controller-manager.conf /etc/kubernetes/scheduler.conf]

The reset process does not clean CNI configuration. To do so, you must remove /etc/cni/net.d

The reset process does not reset or clean up iptables rules or IPVS tables.
If you wish to reset iptables, you must do so manually by using the "iptables" command.

If your cluster was setup to utilize IPVS, run ipvsadm --clear (or similar)
to reset your system's IPVS tables.

The reset process does not clean your kubeconfig files and you must remove them manually.
Please, check the contents of the $HOME/.kube/config file.

#拆除节点
kubeadm reset --cri-socket unix:///run/cri-dockerd.sock && rm -rf /etc/kubernetes/ /var/lib/kubelet /var/lib/dockershim /var/run/kubernetes /var/lib/cni /etc/cni/net.d /var/lib/etcd /run/flannel/
```

```
kubeadm config images list --image-repository=registry.aliyuncs.com/google_containers

docker image pull registry.aliyuncs.com/google_containers/pause:3.6

kubeadm config images pull --cri-socket unix:///run/cri-dockerd.sock --image-repository=registry.aliyuncs.com/google_containers

kubeadm init --control-plane-endpoint="kubeapi.dingbh.top" --kubernetes-version=v1.28.7 --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12 --token-ttl=0 --cri-socket unix:///run/cri-dockerd.sock  --image-repository=registry.aliyuncs.com/google_containers
```

```
#初始化worker节点
kubeadm join kubeapi.dingbh.top:6443 --token 6ztn8p.mr3sh9shty0cxmu3 --discovery-token-ca-cert-hash sha256:8f5bc41a1ba93874117940ced9e024f7bae1c719de499a30ebbb1207e068407f --cri-socket unix:///run/cri-dockerd.sock

#拆除节点
kubeadm reset --cri-socket unix:///run/cri-dockerd.sock && rm -rf /etc/kubernetes/ /var/lib/kubelet /var/lib/dockershim /var/run/kubernetes /var/lib/cni /etc/cni/net.d /var/lib/etcd /run/flannel/
```

