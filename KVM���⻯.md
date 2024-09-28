# KVM虚拟化技术

# 1、虚拟化基础

## 1.1、**什么是虚拟化**

```shell
虚拟化（Virtualization）是一种资源分配和管理技术，是将计算机的各种实体资源,比如CPU、内存、磁
盘空间、网络适配器等，进行抽象转换后虚拟的设备,可以实现灵活地分割、组合为一个或多个计算机配
置环境，并还支持重新分割、重新组合，以达到最大化合理利用物理资源的目的。
参考资料: https://www.vmware.com/cn/solutions/virtualization.html
```

## 1.2、虚拟化的优势

虚拟化可以提高 IT 敏捷性、灵活性和可扩展性，同时大幅节约成本。更高的工作负载移动性、更高的性能和资源可用性、自动化运维 - 这些都是虚拟化的优势，虚拟化技术可以使 IT 部门更轻松地进行管理以及降低拥有成本和运维成本。其优势包括：

```
资源超分,如实际的物理内存只有128G,可以给虚拟机分配200G内存
降低资金成本和运维成本
最大限度减少或消除停机
提高 IT 部门的工作效率和响应能力
加快应用和资源的调配速度
提高业务连续性和灾难恢复能力
简化数据中心管理
减少资源,比如IP和端口的冲突
支持较旧的硬件和操作系统及应用
```

虚拟化类型

```
1、服务器虚拟化

	服务器虚拟化支持在单个物理服务器上运行多个操作系统,每个操作系统作为虚拟机独立运行。

2、网络虚拟化

	通过软件定义网络(Software Defined Network，SDN)，即网络的创建不再依赖于物理设备，如公有云厂商支持用户自己通过配置界面创建新的网络，在 KVM、docker、kubernetes、openstack等虚拟化技术中都使用到了网络虚拟化。

3、桌面虚拟化

	将桌面部署为托管的服务,使 IT 组织能够更快地响应不断变化的工作场所需求和新出现的机会。还可以将虚拟化桌面和应用快速、轻松地交付给分支机构、外包和远程员工以及使用 iPad 和 Android 平板电脑的移动员工。
Citrix 思杰公司在云计算虚拟化、虚拟桌面和远程接入技术领域的处于优势地位。

4、应用虚拟化

	将软件应用通过网络实现虚拟化，比如 office 365,钉钉,企业微信
	
5、存储虚拟化

	将存储用软件虚拟化实现, 如 SAN/NAS(NFS/Samba)/GlusterFS/ceph等
	
6、库虚拟化

在Linux上使用 wine来运行Windows 程序，在Mac系统使用CrossOver来运行Windows程序

7、容器虚技术

当前比较火的虚拟化技术，典型代表: Docker、Containerd、Podman、Linux Container(LXC)、Pouch
```

## **1.3**、虚拟机

### 1.3.1、虚拟机

虚拟计算机称为“虚拟机”(VM,Virtual Machine)，它是一种严密隔离且内含操作系统和应用的软件容器。每个虚拟机都是完全独立的。通过将多台虚拟机放置在一台物理计算机上，可仅在一台物理服务器或“主机”上运行多个操作系统和应用，名为“hypervisor”的精简软件层可将虚拟机与主机分离开来，并根据需要为每个虚拟机动态分配计算资源。

### 1.3.2、**虚拟机的主要特性**

虚拟机具有以下特征，这些特征可提供多项优势

```
分区
	可在一台物理机上运行多个操作系统
	可在虚拟机之间分配系统资源。

隔离
	可在硬件级别进行故障和安全隔离。
	可利用高级资源控制功能保持性能

封装
	可将虚拟机的完整状态保存到文件中。
	移动和复制虚拟机就像移动和复制文件一样轻松

独立于硬件
	可将任意虚拟机调配或迁移到任意物理服务器上
	安装系统不会受硬件兼容性的影响
```

## 1.4、Hypervisor类型

### **1.4.1 Hypervisor** **介绍**

![image-20240112220624617](KVM虚拟化/image-20240112220624617.png)

```
Hypervisor是一种运行在基础物理服务器和操作系统之间的中间软件层，其可以允许多个操作系统和应用共享底层的内存、CPU、磁盘等物理硬件，也可叫做VMM（ virtual machine monitor），即虚拟机监视器。

Hypervisor是所有虚拟化技术的核心，非中断地支持多工作负载迁移的能力是Hypervisor的基本功能，当服务器启动并执行Hypervisor时，它会给每一台虚拟机分配适量的内存、CPU、网络和磁盘，并加载所有虚拟机的客户操作系统。

Hypervisor 允许多种操作系统在相同的物理系统中运行

Hypervisor 控制硬件并向来宾操作系统提供访问底层硬件的途径

Hypervisor 向来宾操作系统提供虚拟化的硬件
```

**X86 CPU** **的保护环**

![image-20240112220709330](KVM虚拟化/image-20240112220709330.png)

**注意**：CPU为了保证程序代码执行的安全性、多用户的独立性、保护OS的正常运行，提出了CPU执行状态的概念。这样能够限制不同程序之间的访问能力，避免一个程序获取另一个程序的内存数据造成数据混乱，同时也避免了程序错误的操作物理硬件。一般CPU都会划分为 用户态 和内核态 ，x86的CPU架构更是细分为了Ring3~0四种状态。

![image-20240112220732074](KVM虚拟化/image-20240112220732074.png)

```
Ring3用户态(User Mode)：运行在用户态的程序代码需要受到CPU的检查，用户态程序代码只能访问内存页表项中规定能被用户态程序代码访问的页面虚拟地址(受限的内存访问)，而且还只能访问任务描述符TSS中的I/O Permission Bitmap中规定能被用户态程序代码访问的端口。甚至不能直接访问外围硬件设备、不能抢占CPU。所有的应用程序都运行在用户态上。当运行在用户态的Application需要调用只能被核心态代码直接访问的硬件设备时，CPU会通过特别的接口去调用核心态的代码，以此来实现Application对硬件设备的调用。如果用户态的Application直接调用硬件设备时，就会被Host OS捕捉到并触发异常，弹出警告窗口。
```

```
Ring0 核心态(Kernel Mode)：是Host OS Kernel运行的模式，运行在核心态的代码可以无限制的对系统内存、设备驱动程序、网卡接口、显卡接口等外围硬件设备进行访问。只有Host OS能够无限制的访问磁盘、键盘等外围硬件设备的数据，但是首先需要在Host OS上安装驱动程序
```

### 1.4.2、Hypervisor分类

1974年Gerald J.Popek和Robert P.Goldberg的文章“Formal Requirements forVirtualizable Third 

Generation Architectures" 将Hypervisor分为两类:

![image-20240112220902632](KVM虚拟化/image-20240112220902632.png)

![image-20240112220910977](KVM虚拟化/image-20240112220910977.png)

**两种类型：**

类型1：裸金属机器

直接运行到物理机的Hypervisor上，这种架构搭建的虚拟化环境称为裸机虚拟化环境(Bare-Metal Hardware

```
KVM
XEN
vmware esxi
rhev hypervisor
Hyper-v Server
#Redhat将KVM划分到类型I即裸机型：
https://www.redhat.com/zh/topics/virtualization/what-is-KVM
```

类型2：宿主型，即需要运行在具有虚拟化功能的操作系统上的Hypervisor，构建的是主机虚拟化环境(Hosted 

Virtualization)

```
vmware workstation
Microsoft Hyper-V
VirtualBox
paralles desktop #Mac系统最强虚拟机技术
```



# 2、KVM架构和部署

## **2.1 KVM** 概述

**红帽** **kvm** **介绍**

```
https://www.redhat.com/zh/topics/virtualization/what-is-KVM
```

**RedHat虚拟化指南**

```
https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_Linux/7/html/virtualization_getting_started_guide/index
```

**RedHat创建虚拟机数量限制：**

```
https://access.redhat.com/articles/rhel-kvm-limits
```

### 2.1.1、KVM介绍

```
KVM（ Kernel-based Virtual Machine）是一个完整的虚拟化解决方案，适用于包含虚拟化扩展（Intel VT或AMD-V）的x86硬件上的Linux。目前也支持ARM等其它硬件平台. 它由可加载的内核模块kvm.ko组成，它提供核心虚拟化基础架构和处理器特定模块，kvm-intel.ko或kvm-amd.ko，KVM的用户空间组件包含在QEMU1.3后续版本中,KVM目前已成为学术界的主流 VMM (virtual machine monitor，虚拟机
监视器，也称为 hypervisor)之一KVM是开源软件，可运行多个运行未修改的Linux或Windows映像的虚拟机依赖于HVM；Intel VT-x, AMD AMD-V

官网： https://www.Linux-kvm.org
```

![image-20240112221435340](KVM虚拟化/image-20240112221435340.png)

### 2.1.2 KVM架构

```
KVM 是基于虚拟化扩展（Intel VT 或者 AMD-V）的 X86 硬件的开源的 Linux 原生的全虚拟化解决方案。KVM 中，虚拟机被实现为常规的 Linux 进程，由标准 Linux 调度程序进行调度；虚机的每个虚拟CPU 被实现为一个常规的 Linux 进程。这使得 KVM 能够使用 Linux 内核的已有功能。但是，KVM 本身不执行任何硬件模拟，需要客户空间程序通过 /dev/kvm 接口设置一个客户机虚拟服务器的地址空间，向它提供模拟的 I/O，并将它的视频显示映射回宿主的显示屏。目前这个应用程序使用QEMU。
```

#### **2.1.2.1 KVM体系结构**

![image-20240112102305244](KVM虚拟化/image-20240112102305244.png)

```
KVM： 初始化CPU硬件,打开虚拟机模式,负责CPU,内存,中断控制器,时钟. 由内核模块kvm_xxx.ko实现，工作于hypervisor，设备/dev/kvm，是一个字符设备，在用户空间可通过ioctl()系统调用来完成VM创建、启动，为VM分配内存、读写VCPU的寄存器、向VCPU注入中断、时钟等管理功能
QEMU进程：工作于用户空间，主要用于实现模拟IO设备,如显卡，网卡，硬盘等， qemu-kvm进程：工作于用户空间，用于实现一个虚拟机实例
Libvirt：提供统一API，守护进程libvirtd和相关工具，如:virsh，virt-manager等
```

#### **2.1.2.2 KVM** **模块载入后的系统的运行模式**

![image-20240112102426655](KVM虚拟化/image-20240112102426655.png)

```
内核模式：HostOS执行I/O类操作，或其它的特殊指令的操作；称作内核模式
用户模式：GuestOS的I/O类操作
来宾模式：GuestOS非I/O类操作，称作虚拟机的用户模式更贴切
```

### **2.1.3 KVM** 集中管理与控制

KVM是运行在单机的系统，需要其它软件实现跨主机的统一的管理。常见的虚拟化管理平台如下：

http://www.Linux-kvm.org/page/Management_Tools

![image-20240112102517642](KVM虚拟化/image-20240112102517642.png)

```
oVirt
功能强大，是Redhat虚拟化管理平台RHEV的开源版本。
http://www.ovirt.org/
WebVirtMgr
https://www.webvirtmgr.net
virt-manager的Web模式的替代品
OpenStack
最主流的开源虚拟化管理平台
Proxmox virtualization environment
简称PVE，是一个开源免费的基于Linux的企业级虚拟化方案，功能不输专业收费的VMware。是一
个完整的企业虚拟化开源平台。借助内置的Web界面，您可以轻松管理VM和容器，软件定义的存
储和网络，高可用性集群以及单个解决方案上的多个开箱即用工具。
```

## 2.2、宿主机环境准备

KVM需要宿主机CPU必须支持虚拟化功能，因此如果是在vmware workstation上使用虚拟机做宿主机，那么必须要在虚拟机配置界面的处理器选项中开启虚拟机化功能。

### 2.2.1 CPU开启虚拟化

注意： 给宿主机的CPU和内存分配足够多的配置

![image-20240112221549307](KVM虚拟化/image-20240112221549307.png)



### **2.2.2**、验证开启虚拟化

```
grep -Em 1  "vmx|svm" /proc/cpuinfo 
#Intel CPU 对应 vmx
#AMD CPU 对应 svm
```

范例: 验证是否开启虚拟化支持

```
[root@ubuntu2004 ~]#grep -Em 1 'vmx|svm' /proc/cpuinfo 
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm constant_tsc rep_good nopl tsc_reliable nonstop_tsc cpuid extd_apicid pni pclmulqdq ssse3 fma cx16 sse4_1 sse4_2 x2apic movbe popcnt aes xsave avx f16c rdrand hypervisor lahf_lm svm extapic cr8_legacy abm sse4a misalignsse 3dnowprefetch osvw topoext ssbd ibpb vmmcall fsgsbase bmi1 avx2 smep bmi2 rdseed adx smap clflushopt clwb sha_ni xsaveopt xsavec xgetbv1 xsaves clzero wbnoinvd arat npt svm_lock nrip_save vmcb_clean flushbyasid decodeassists umip rdpid overflow_recov succor


[root@Rocky8 ~]#grep -Em 1 "vmx|svm" /proc/cpuinfo 
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm constant_tsc rep_good nopl tsc_reliable nonstop_tsc cpuid extd_apicid pni pclmulqdq ssse3 fma cx16 sse4_1 sse4_2 x2apic movbe popcnt aes xsave avx f16c rdrand hypervisor lahf_lm svm extapic cr8_legacy abm sse4a misalignsse 3dnowprefetch osvw topoext ssbd ibpb vmmcall fsgsbase bmi1 avx2 smep bmi2 rdseed adx smap clflushopt clwb sha_ni xsaveopt xsavec xgetbv1 xsaves clzero wbnoinvd arat npt svm_lock nrip_save vmcb_clean flushbyasid decodeassists umip rdpid overflow_recov succor
```

范例: 查看AMD主机的内核模块

```
[root@Rocky8 ~]#lscpu |grep svm
Flags:               fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm constant_tsc rep_good nopl tsc_reliable nonstop_tsc cpuid extd_apicid pni pclmulqdq ssse3 fma cx16 sse4_1 sse4_2 x2apic movbe popcnt aes xsave avx f16c rdrand hypervisor lahf_lm svm extapic cr8_legacy abm sse4a misalignsse 3dnowprefetch osvw topoext ssbd ibpb vmmcall fsgsbase bmi1 avx2 smep bmi2 rdseed adx smap clflushopt clwb sha_ni xsaveopt xsavec xgetbv1 xsaves clzero wbnoinvd arat npt svm_lock nrip_save vmcb_clean flushbyasid decodeassists umip rdpid overflow_recov succor

[root@ubuntu2004 ~]#lscpu |grep svm
Flags:                              fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm constant_tsc rep_good nopl tsc_reliable nonstop_tsc cpuid extd_apicid pni pclmulqdq ssse3 fma cx16 sse4_1 sse4_2 x2apic movbe popcnt aes xsave avx f16c rdrand hypervisor lahf_lm svm extapic cr8_legacy abm sse4a misalignsse 3dnowprefetch osvw topoext ssbd ibpb vmmcall fsgsbase bmi1 avx2 smep bmi2 rdseed adx smap clflushopt clwb sha_ni xsaveopt xsavec xgetbv1 xsaves clzero wbnoinvd arat npt svm_lock nrip_save vmcb_clean flushbyasid decodeassists umip rdpid overflow_recov succor




[root@ubuntu2004 ~]#lsmod |grep kvm
kvm_amd                98304  5
ccp                    86016  1 kvm_amd
kvm                   663552  1 kvm_amd

[root@ubuntu2004 ~]#ll /dev/kvm 
crw-rw---- 1 root kvm 10, 232 Jan 12 13:02 /dev/kvm

```

## **2.3**、安装KVM工具包

### **2.3.1 KVM** 相关工具包介绍

```
1、qemu-kvm: 为kvm提供底层仿真支持
2、libvirt-daemon: libvirtd守护进程，管理虚拟机
3、libvirt-client: 用户端软件，提供客户端管理命令
4、libvirt-daemon-driver-qemu: libvirtd连接qemu的驱动

5、libvirt: 使用最多的KVM虚拟化管理工具和应用程序接口，即通过libvirt调用KVM创建虚拟机，
libvirt是KVM通用的访问API，其不但能管理KVM，还能管理VMware、Xen、Hyper-V、
virtualBox等虚拟化方案。

6、virt-manager: 图形界面管理工具,其底层也是调用libvirt API来完成对虚拟机的操作，包括虚拟机的创建、删除、启动、停止以及一些简单的监控功能等。
7、virt-install: 虚拟机命令行安装工具
8、virsh: 命令行工具是基于 libvirt API 创建的命令行工具，它可以作为图形化的 virt-manager 应用的备选工具。virsh 命令可以被用来创建虚拟化任务管理脚本，如安装、启动和停止虚拟机
9、virt-viewer: 通过 VNC 和 SPICE 协议显示虚拟机器图形控制台的最小工具。该工具在其同名软件包
中：virtviewer

10、cockpit: CentOS8 专门提供的基于Web的虚拟机管理界面
```

### 2.3.2 libvirt包功能

#### 2.3.2.1 libvirt介绍

```
libvirt 程序包是一个与虚拟机监控程序相独立的虚拟化应用程序接口，它可以与操作系统的一系列虚拟化性能进行交互


libvirt 程序包提供：
	一个稳定的通用层来安全地管理主机上的虚拟机。
	一个管理本地系统和连网主机的通用接口。

在虚拟机监控程序支持的情况下，部署、创建、修改、监测、控制、迁移以及停止虚拟机操作都需要这些API。尽管 libvirt 可同时访问多个主机，但 API 只限于单节点操作

libvirt 程序包被设计为用来构建高级管理工具和应用程序，例如 virt-manager 与 virsh 命令行管理工具。libvirt 主要的功能是管理单节点主机，并提供 API 来列举、监测和使用管理节点上的可用资源，其中包括CPU、内存、储存、网络和非一致性内存访问（NUMA）分区。管理工具可以位于独立于主机的物理机上，并通过安全协议和主机进行交流
```

#### 2.3.2.2 libvirt结构图

![image-20240112222603663](KVM虚拟化/image-20240112222603663.png)

### **2.3.3**、安装KVM相关包

#### 2.3.3.1 Ubuntu安装KVM

Ubuntu 18.04：

官方文档: https://ubuntu.com/server/docs/virtualization-libvirt

```
[root@ubuntu2204 ~]#apt -y install cpu-checker
[root@ubuntu2004 ~]#apt -y install cpu-checker
#如果CPU不支持会如下提示
[root@ubuntu2004 ~]#kvm-ok
INFO: Your CPU does not support KVM extensions
KVM acceleration can NOT be used
# apt install qemu-kvm virt-manager libvirt-daemon-system
# kvm-ok #验证是否支持kvm,只有Ubuntu支持,CentOS 不支持
#如果CPU支持，如下提示
INFO: /dev/kvm exists
KVM acceleration can be used
```

范例: ubuntu 安装KVM工具（默认会自启动），centos需要手动启动

```
[root@ubuntu2004 ~]#apt update
[root@ubuntu2204 ~]#apt -y install qemu-kvm virt-manager libvirt-daemon-system
[root@ubuntu2004 ~]#apt -y install qemu-kvm virt-manager libvirt-daemon-system
[root@ubuntu1804 ~]#apt -y install qemu-kvm virt-manager libvirt-daemon-system
[root@ubuntu2004 ~]#ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:6a:18:a5 brd ff:ff:ff:ff:ff:ff
    inet 192.168.188.100/24 brd 192.168.188.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe6a:18a5/64 scope link 
       valid_lft forever preferred_lft forever
3: virbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 52:54:00:c4:f1:47 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever

[root@ubuntu2004 ~]#apt -y install bridge-utils

[root@ubuntu2004 ~]#brctl show
bridge name	bridge id		STP enabled	interfaces
virbr0		8000.525400c4f147	yes		virbr0-nic
							vnet0

#如果没有开启CPU虚拟化功能会提示以下信息
[root@ubuntu1804 ~]#kvm-ok
INFO: Your CPU does not support KVM extensions
KVM acceleration can NOT be used

#添加CPU的虚拟化支持再执行
[root@ubuntu2004 ~]#kvm-ok 
INFO: /dev/kvm exists
KVM acceleration can be used
```

#### **2.3.3.2 CentOS** **安装** **KVM**

使用虚拟化，需要至少 qemu-kvm 和 qemu-img(安装qemu-kvm会自动安装) 软件建议安装：

yum install qemu-kvm libvirt virt-manager virt-install 

范例: CentOS 8 安装 KVM相关工具

```
[root@Rocky8 ~]#yum -y install qemu-kvm libvirt  virt-manager virt-install virt-viewer


[root@Rocky8 ~]#systemctl enable --now libvirtd

[root@Rocky8 ~]#ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:da:06:57 brd ff:ff:ff:ff:ff:ff
    altname enp3s0
    altname ens160
    inet 192.168.188.8/24 brd 192.168.188.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:feda:657/64 scope link 
       valid_lft forever preferred_lft forever
3: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 52:54:00:4c:01:ee brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
4: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr0 state DOWN group default qlen 1000
    link/ether 52:54:00:4c:01:ee brd ff:ff:ff:ff:ff:ff
[root@Rocky8 ~]#brctl show
bridge name	bridge id		STP enabled	interfaces
virbr0		8000.5254004c01ee	yes		virbr0-nic

```

范例: CentOS 8 还提供基于Web的虚拟机管理方式

```
[root@Rocky8 ~]#yum -y install cockpit cockpit-machines

[root@Rocky8 ~]#systemctl enable --now cockpit

访问：https://192.168.188.8:9090
```

![image-20240113134721947](KVM虚拟化/image-20240113134721947.png)



![image-20240113134737968](KVM虚拟化/image-20240113134737968.png)



创建虚拟机（提前准备镜像文件）

![image-20240113134821727](KVM虚拟化/image-20240113134821727.png)

![image-20240113135132802](KVM虚拟化/image-20240113135132802.png)



![image-20240113135643121](KVM虚拟化/image-20240113135643121.png)



### 2.3.4、图形化工具 virt-manager

范例: CentOS 上管理工具 virt-manager

```
[root@Rocky8 ~]#virt-manager 

#报错解决方案
[root@Rocky8 ~]#Unable to init server: Could not connect: Connection refused
Unable to init server: Could not connect: Connection refused
Unable to init server: Could not connect: Connection refused

(virt-manager:10412): Gtk-WARNING **: 13:59:06.511: cannot open display:


解决方法：
[root@Rocky8 ~]#export DISPLAY=192.168.188.99:0.0
#再次启动
[root@Rocky8 ~]#virt-manager
```

![image-20240113140107377](KVM虚拟化/image-20240113140107377.png)

选择手动镜像文件

![image-20240113140640384](KVM虚拟化/image-20240113140640384.png)

![image-20240113140719768](KVM虚拟化/image-20240113140719768.png)

选择内存大小及cpus

![image-20240113140731340](KVM虚拟化/image-20240113140731340.png)

选择硬盘大小

![image-20240113140807428](KVM虚拟化/image-20240113140807428.png)

设置主机名和定制化操作（可选），无定制需求finish即可

![image-20240113140842831](KVM虚拟化/image-20240113140842831.png)

安装完成

![image-20240113140611761](KVM虚拟化/image-20240113140611761.png)

选择语言，默认即可（English）

![image-20240113140221115](KVM虚拟化/image-20240113140221115.png)

![image-20240113141121129](KVM虚拟化/image-20240113141121129.png)

配置时区

![image-20240113141152706](KVM虚拟化/image-20240113141152706.png)

配置硬盘分区

![image-20240113141240359](KVM虚拟化/image-20240113141240359.png)

开启网卡

![image-20240113141336938](KVM虚拟化/image-20240113141336938.png)

设置密码及创建用户

![image-20240113141420796](KVM虚拟化/image-20240113141420796.png)

![image-20240113141440434](KVM虚拟化/image-20240113141440434.png)

等待进度条完成，reboot

![image-20240113150641992](KVM虚拟化/image-20240113150641992.png)

安装完成

![image-20240113150951017](KVM虚拟化/image-20240113150951017.png)

### **2.3.5** 默认网络配置

安装完虚拟工具后,会自动生成一个 virbr1 网卡,类似于Vmware workstation 生成的VMnet8 网卡,充当虚拟机的 NAT 网卡



查看内核：

```
[root@Rocky8 ~]#grep -i kvm /boot/config-4.18.0-425.3.1.el8.x86_64 
CONFIG_KVM_GUEST=y
CONFIG_PTP_1588_CLOCK_KVM=m
CONFIG_DRM_I915_GVT_KVMGT=m
CONFIG_HAVE_KVM=y
CONFIG_HAVE_KVM_PFNCACHE=y
CONFIG_HAVE_KVM_IRQCHIP=y
CONFIG_HAVE_KVM_IRQFD=y
CONFIG_HAVE_KVM_IRQ_ROUTING=y
CONFIG_HAVE_KVM_DIRTY_RING=y
CONFIG_HAVE_KVM_EVENTFD=y
CONFIG_KVM_MMIO=y
CONFIG_KVM_ASYNC_PF=y
CONFIG_HAVE_KVM_MSI=y
CONFIG_HAVE_KVM_CPU_RELAX_INTERCEPT=y
CONFIG_KVM_VFIO=y
CONFIG_KVM_GENERIC_DIRTYLOG_READ_PROTECT=y
CONFIG_KVM_COMPAT=y
CONFIG_HAVE_KVM_IRQ_BYPASS=y
CONFIG_HAVE_KVM_NO_POLL=y
CONFIG_HAVE_KVM_PM_NOTIFIER=y
CONFIG_KVM=m   				#以模块方式提供
CONFIG_KVM_INTEL=m
CONFIG_X86_SGX_KVM=y
CONFIG_KVM_AMD=m
CONFIG_KVM_AMD_SEV=y
# CONFIG_KVM_XEN is not set
CONFIG_KVM_EXTERNAL_WRITE_TRACKING=y


[root@Rocky8 ~]#modinfo kvm
filename:       /lib/modules/4.18.0-425.3.1.el8.x86_64/kernel/arch/x86/kvm/kvm.ko.xz
license:        GPL
author:         Qumranet
rhelversion:    8.7
srcversion:     9647C2F614358F30249C0C6
depends:        irqbypass
intree:         Y
name:           kvm
vermagic:       4.18.0-425.3.1.el8.x86_64 SMP mod_unload modversions 
sig_id:         PKCS#7
signer:         Rocky kernel signing key
sig_key:        54:B0:44:38:9B:6E:F7:43:B2:FD:8F:3B:8B:D3:69:22:26:2A:BE:87
sig_hashalgo:   sha256
signature:      1A:D8:1E:FD:2E:23:25:E5:F2:4F:E2:E6:D4:2C:7C:89:80:FB:AE:41:
		7C:3C:A9:A0:75:FF:ED:2D:F0:01:51:73:A5:29:55:5F:21:A4:3C:F6:
		D6:BA:F4:3B:5B:65:11:D2:CA:DA:2D:E8:37:3A:F6:8F:96:52:35:12:
		B8:6F:AB:76:4E:38:18:11:47:4D:2F:7E:9A:11:E9:EA:E0:69:0E:FF:
		4E:33:A7:9A:5B:9A:CE:1A:44:B1:0C:2B:4B:9F:75:AD:C7:D4:4F:C3:
		DF:A6:AF:9D:F2:4B:11:70:0C:E8:3B:A7:D7:DE:08:9F:48:6C:27:A0:
		1F:79:AD:4A:07:67:D3:94:58:5B:48:75:8E:45:69:0F:C0:D1:E0:3D:
		E1:D6:F9:D6:D8:F6:3C:E5:A5:9E:C9:B9:6B:93:AA:08:F2:77:66:09:
		21:51:D9:29:C2:05:84:EB:22:F0:17:0C:0C:89:6E:5B:B7:48:F3:63:
		EC:73:D0:EE:E6:ED:96:52:B5:66:BF:FE:D6:74:B0:80:FC:0F:59:39:
		80:80:A3:56:C3:BA:08:78:1D:59:E2:46:40:0B:55:C1:4B:9D:BC:C3:
		4C:0C:ED:B6:D4:DD:86:37:C6:F0:DB:DA:D5:5C:D5:6B:93:81:DB:6C:
		B1:FF:24:F8:EC:78:19:F0:DB:A5:85:40:A0:2D:44:F5:29:60:90:88:
		FC:B0:9C:57:25:D6:5F:9F:20:3A:DD:DE:E5:A2:58:A7:2B:FF:DF:FC:
		29:9B:E0:CC:05:75:D2:34:4B:2D:A7:8C:5D:00:F5:A4:63:12:97:CC:
		D5:F8:62:A4:D9:00:B1:B5:21:B2:32:D6:6B:6A:CB:5D:65:8B:99:44:
		36:12:49:84:E8:4A:5D:ED:C0:C2:EE:98:CA:A6:94:CC:9B:91:2E:CE:
		4B:A4:45:DD:E3:7B:CE:F2:E6:A5:D1:97:C1:74:51:17:1C:5C:6F:5E:
		84:38:E7:34:2D:93:A9:35:16:7C:3F:9A:81:0A:03:30:7C:16:D5:A2:
		17:85:89:C2
parm:           tdp_mmu:bool
parm:           mmio_caching:bool
parm:           nx_huge_pages:bool
parm:           nx_huge_pages_recovery_ratio:uint
parm:           nx_huge_pages_recovery_period_ms:uint
parm:           flush_on_reuse:bool
parm:           ignore_msrs:bool
parm:           report_ignored_msrs:bool
parm:           min_timer_period_us:uint
parm:           kvmclock_periodic_sync:bool
parm:           tsc_tolerance_ppm:uint
parm:           lapic_timer_advance_ns:int
parm:           vector_hashing:bool
parm:           enable_vmware_backdoor:bool
parm:           force_emulation_prefix:bool
parm:           pi_inject_timer:bint
parm:           enable_pmu:bool
parm:           eager_page_split:bool
parm:           halt_poll_ns:uint
parm:           halt_poll_ns_grow:uint
parm:           halt_poll_ns_grow_start:uint
parm:           halt_poll_ns_shrink:uint

```

![image-20240112210851828](KVM虚拟化/image-20240112210851828.png)

# 3、创建虚拟机

## 3.1、同2.3.4节（图形化工具 virt-manager）

## **3.2** 使用virt-install创建虚拟机

虽然使用virt-manager 可以方便的管理虚拟机,但如果需要批量进行虚拟机的创建管理,命令行工具virt-install 更加方便和适合。

### **3.2.1 virt-install** 使用说明

```
# virt-install --help
usage: virt-install --name NAME --ram RAM STORAGE INSTALL [options]
使用 '--option=?' 或者 '--option help' 查看可用子选项
有关示例及完整选项语法，请查看 man page。
使用指定安装介质新建虚拟机
optional arguments:
-h, --help show this help message and exit
--version show program's version number and exit
--connect URI 使用 libvirt URI 连接到 hypervisor
通用选项:
-n NAME, --name NAME 客户端虚拟机的名称
--memory MEMORY 配置虚拟机内存分配
例如：
--memory 1024 (in MiB) 
--memory 512,maxmemory=1024
--vcpus VCPUS 为虚拟机配置的 vcpus 数。
例如：
--vcpus 5
--vcpus 5,maxcpus=10,cpuset=1-4,6,8
--vcpus sockets=2,cores=4,threads=2
--cpu CPU CPU 型号及功能。
例如：
--cpu coreduo,+x2apic
--cpu host
--metadata METADATA 配置虚拟机元数据
例如：
--metadata name=foo,title="My pretty title",uuid=...
--metadata description="My nice long description"
安装方法选项:
--cdrom CDROM 光驱安装介质
-l LOCATION, --location LOCATION 安装源
例如：
nfs:host:/path
http://host/path
ftp://host/path
--pxe 使用 PXE 协议从网络引导
--import 在磁盘映像中构建虚拟机
--livecd 将光驱介质视为 Live CD
-x EXTRA_ARGS, --extra-args EXTRA_ARGS 附加到使用 --location 引导的内核的参数
--initrd-inject INITRD_INJECT 使用 --location 为 initrd 的 root 添加给定文件
--os-variant DISTRO_VARIANT 在其中安装 OS 变体的虚拟机,比
如:'fedora18'、'rhel6'、'winxp' 等等
--boot BOOT 配置虚拟机引导设置。
例如：
--boot hd,cdrom,menu=on
--boot init=/sbin/init (for containers)
--idmap IDMAP 为 LXC 容器启用用户名称空间。
例如：
--idmap uid_start=0,uid_target=1000,uid_count=10
设备选项:
--disk DISK 使用不同选项指定存储。例如：
--disk size=10 (new 10GiB image in default location)
--disk /my/existing/disk,cache=none
--disk device=cdrom,bus=scsi
--disk=?
-w NETWORK, --network NETWORK 配置虚拟机网络接口。
例如：
--network bridge=myvirbr1
--network network=my_libvirt_virtual_net
--network network=mynet,model=virtio,mac=00:11...
--network none
--network help
--graphics GRAPHICS 配置虚拟机显示设置。
例如：
--graphics vnc
--graphics spice,port=5901,tlsport=5902
--graphics none
--graphics vnc,password=foobar,port=5910,keymap=ja
--controller CONTROLLER 配置虚拟机控制程序设备。
例如：
--controller type=usb,model=ich9-ehci1
--input INPUT 配置虚拟机输入设备。
例如：
--input tablet
--input keyboard,bus=usb
--serial SERIAL 配置虚拟机串口设备
--parallel PARALLEL 配置虚拟机并口设备
--channel CHANNEL 配置虚拟机沟通频道
--console CONSOLE 配置虚拟机与主机之间的文本控制台连接
--hostdev HOSTDEV 将物理 USB/PCI/etc 主机设备配置为与虚拟机共享
--filesystem FILESYSTEM 将主机目录传递给虚拟机。
例如：
--filesystem /my/source/dir,/dir/in/guest
--filesystem template_name,/,type=template
--sound [SOUND] 配置虚拟机声音设备模拟
--watchdog WATCHDOG 配置虚拟机 watchdog 设备
--video VIDEO 配置虚拟机视频硬件。
--smartcard SMARTCARD 配置虚拟机智能卡设备。例如：--smartcard mode=passthrough
--redirdev REDIRDEV 配置虚拟机重定向设备。例如：--redirdev 
usb,type=tcp,server=192.168.1.1:4000
--memballoon MEMBALLOON 配置虚拟机 memballoon 设备。例如：--memballoon model=virtio
--tpm TPM 配置虚拟机 TPM 设备。例如：--tpm /dev/tpm
--rng RNG 配置虚拟机 RNG 设备。例如：--rng /dev/random
--panic PANIC 配置虚拟机 panic 设备。例如：--panic default
虚拟机配置选项:
--security SECURITY 设定域安全驱动器配置。
--numatune NUMATUNE 为域进程调整 NUMA 策略。
--memtune MEMTUNE 为域进程调整内粗策略。
--blkiotune BLKIOTUNE为域进程调整 blkio 策略。
--memorybacking MEMORYBACKING 为域进程设置内存后备策略。例如：
--memorybacking hugepages=on
--features FEATURES 设置域 <features> XML。
例如：
--features acpi=off
--features apic=on,eoi=on
--clock CLOCK 设置域 <clock> XML。例如：--clock 
offset=localtime,rtc_tickpolicy=catchup
--pm PM 配置 VM 电源管理功能
--events EVENTS 配置 VM 生命周期管理策略
--resource RESOURCE 配置 VM 资源分区（cgroups）
虚拟化平台选项:
-v, --hvm 客户端应该是一个全虚拟客户端
-p, --paravirt 这个客户端一个是一个半虚拟客户端
--container 这台虚拟机需要一个容器客户端
--virt-type HV_TYPE 要使用的管理程序名称(kvm、qemu、xen等等)
--arch ARCH 模拟的 CPU 构架
--machine MACHINE 要模拟的机器类型
其它选项:
--autostart 引导主机时自动启动域。
--wait WAIT 等待安装完成的分钟数。
--noautoconsole 不要自动尝试连接到客户端控制台
--noreboot 完成安装后不要引导虚拟机。
--print-xml [XMLONLY] 输出所生成域 XML，而不是创建虚拟机。
--dry-run 完成安装步骤，但不要创建设备或者定义虚拟机。
--check CHECK 启用或禁用验证检查。例如：
--check path_in_use=off
--check all=off
-q, --quiet 禁止无错误输出
-d, --debug 输入故障排除信息
```

### **3.2.2 virt-install** **命令创建虚拟机**

查看支持的系统版本

```
#查看支持的系统版本
[root@ubuntu2004 ~]#osinfo-query os|grep centos
 centos-stream8       | CentOS Stream 8                                    | 8        | http://centos.org/centos-stream/8       
 centos5.0            | CentOS 5.0                                         | 5.0      | http://centos.org/centos/5.0            
 centos5.1            | CentOS 5.1                                         | 5.1      | http://centos.org/centos/5.1            
 centos5.10           | CentOS 5.10                                        | 5.10     | http://centos.org/centos/5.10           
 centos5.11           | CentOS 5.11                                        | 5.11     | http://centos.org/centos/5.11           
 centos5.2            | CentOS 5.2                                         | 5.2      | http://centos.org/centos/5.2            
 centos5.3            | CentOS 5.3                                         | 5.3      | http://centos.org/centos/5.3            
 centos5.4            | CentOS 5.4                                         | 5.4      | http://centos.org/centos/5.4            
 centos5.5            | CentOS 5.5                                         | 5.5      | http://centos.org/centos/5.5            
 centos5.6            | CentOS 5.6                                         | 5.6      | http://centos.org/centos/5.6            
 centos5.7            | CentOS 5.7                                         | 5.7      | http://centos.org/centos/5.7            
 centos5.8            | CentOS 5.8                                         | 5.8      | http://centos.org/centos/5.8            
 centos5.9            | CentOS 5.9                                         | 5.9      | http://centos.org/centos/5.9            
 centos6.0            | CentOS 6.0                                         | 6.0      | http://centos.org/centos/6.0            
 centos6.1            | CentOS 6.1                                         | 6.1      | http://centos.org/centos/6.1            
 centos6.10           | CentOS 6.10                                        | 6.10     | http://centos.org/centos/6.10           
 centos6.2            | CentOS 6.2                                         | 6.2      | http://centos.org/centos/6.2            
 centos6.3            | CentOS 6.3                                         | 6.3      | http://centos.org/centos/6.3            
 centos6.4            | CentOS 6.4                                         | 6.4      | http://centos.org/centos/6.4            
 centos6.5            | CentOS 6.5                                         | 6.5      | http://centos.org/centos/6.5            
 centos6.6            | CentOS 6.6                                         | 6.6      | http://centos.org/centos/6.6            
 centos6.7            | CentOS 6.7                                         | 6.7      | http://centos.org/centos/6.7            
 centos6.8            | CentOS 6.8                                         | 6.8      | http://centos.org/centos/6.8            
 centos6.9            | CentOS 6.9                                         | 6.9      | http://centos.org/centos/6.9            
 centos7.0            | CentOS 7                                           | 7        | http://centos.org/centos/7.0            
 centos8              | CentOS 8                                           | 8        | http://centos.org/centos/8 
```

#### **3.2.2.1** **利用** **qemu-img** 命令创建虚拟磁盘(可选）

**注意: qemu-img create一定要确认对应路径下没有此文件，如果存在将覆盖原文件**

```

[root@ubuntu2004 ~]#qemu-img create -f qcow2 /var/lib/libvirt/images/rocky8.qcow2 10G
Formatting '/var/lib/libvirt/images/rocky8.qcow2', fmt=qcow2 size=10737418240 cluster_size=65536 lazy_refcounts=off refcount_bits=16

#观察文件虚拟磁盘大小,比较用virt-manager创建的虚拟机磁盘文件大小
[root@ubuntu2004 ~]#ls /var/lib/libvirt/images/ -hl
total 1.7G
-rw------- 1 libvirt-qemu kvm   21G Jan 14 11:18 centos7.0.qcow2
-rw-r--r-- 1 root         root 193K Jan 18 13:11 rocky8.qcow2
```

#### 3.2.2.2利用osinfo-query命令查看支持的OS版本

```
#列出支持OS系统名称
#方法1
[root@ubuntu2204 ~]#virt-install --osinfo list  #不推荐，新版本已不适用
#方法2
[root@ubuntu2004 ~]#apt install libosinfo-bin
[root@ubuntu2004 ~]#osinfo-query os |grep -i rhel

#查看支持的OS
[root@ubuntu2004 ~]#osinfo-query os|grep centos
 centos-stream8       | CentOS Stream 8                                    | 8        | http://centos.org/centos-stream/8       
 centos5.0            | CentOS 5.0                                         | 5.0      | http://centos.org/centos/5.0            
 centos5.1            | CentOS 5.1                                         | 5.1      | http://centos.org/centos/5.1            
 centos5.10           | CentOS 5.10                                        | 5.10     | http://centos.org/centos/5.10           
 centos5.11           | CentOS 5.11                                        | 5.11     | http://centos.org/centos/5.11           
 centos5.2            | CentOS 5.2                                         | 5.2      | http://centos.org/centos/5.2            
 centos5.3            | CentOS 5.3                                         | 5.3      | http://centos.org/centos/5.3            
 centos5.4            | CentOS 5.4                                         | 5.4      | http://centos.org/centos/5.4            
 centos5.5            | CentOS 5.5                                         | 5.5      | http://centos.org/centos/5.5            
 centos5.6            | CentOS 5.6                                         | 5.6      | http://centos.org/centos/5.6            
 centos5.7            | CentOS 5.7                                         | 5.7      | http://centos.org/centos/5.7            
 centos5.8            | CentOS 5.8                                         | 5.8      | http://centos.org/centos/5.8            
 centos5.9            | CentOS 5.9                                         | 5.9      | http://centos.org/centos/5.9            
 centos6.0            | CentOS 6.0                                         | 6.0      | http://centos.org/centos/6.0            
 centos6.1            | CentOS 6.1                                         | 6.1      | http://centos.org/centos/6.1            
 centos6.10           | CentOS 6.10                                        | 6.10     | http://centos.org/centos/6.10           
 centos6.2            | CentOS 6.2                                         | 6.2      | http://centos.org/centos/6.2            
 centos6.3            | CentOS 6.3                                         | 6.3      | http://centos.org/centos/6.3            
 centos6.4            | CentOS 6.4                                         | 6.4      | http://centos.org/centos/6.4            
 centos6.5            | CentOS 6.5                                         | 6.5      | http://centos.org/centos/6.5            
 centos6.6            | CentOS 6.6                                         | 6.6      | http://centos.org/centos/6.6            
 centos6.7            | CentOS 6.7                                         | 6.7      | http://centos.org/centos/6.7            
 centos6.8            | CentOS 6.8                                         | 6.8      | http://centos.org/centos/6.8            
 centos6.9            | CentOS 6.9                                         | 6.9      | http://centos.org/centos/6.9            
 centos7.0            | CentOS 7                                           | 7        | http://centos.org/centos/7.0            
 centos8              | CentOS 8                                           | 8        | http://centos.org/centos/8 
```

#### **3.2.2.3** **创建虚拟机使用光盘启动并手动安装**

范例：无需事先创建磁盘文件，直接创建虚拟机

```
[root@ubuntu2004 /var/lib/libvirt/images]#virt-install --virt-type kvm --os-variant=centos7.0 --name centos7 --ram 1024 --vcpus 2 --cdrom=/data/isos/CentOS-7-x86_64-Minimal-2207-02.iso --disk path=/var/lib/libvirt/images/centos7.qcow2,size=10,format=qcow2,bus=virtio --network network=default --graphics vnc,listen=0.0.0.0 --noautoconsole

Starting install...
Allocating 'centos7.qcow2'                                                                                                                     |  10 GB  00:00:00     
Domain installation still in progress. You can reconnect to 
the console to complete the installation process.
[root@ubuntu2004 /var/lib/libvirt/images]#ll
total 2872828
drwx--x--x 2 root         root        4096 Jan 20 05:47 ./
drwxr-xr-x 7 root         root        4096 Jan 20 02:34 ../
-rw------- 1 libvirt-qemu kvm  10739318784 Jan 20 05:47 centos7.qcow2
-rw-r--r-- 1 libvirt-qemu kvm   2939944960 Jan 20 05:44 rocky8.qcow2

[root@ubuntu2004 /var/lib/libvirt/images]#pstree -p|grep qemu
           |-qemu-system-x86(28818)-+-{qemu-system-x86}(28820)
           |                        |-{qemu-system-x86}(28830)
           |                        |-{qemu-system-x86}(28831)
           |                        |-{qemu-system-x86}(28832)
           |                        |-{qemu-system-x86}(28834)
           |                        `-{qemu-system-x86}(28835)
           |-qemu-system-x86(29366)-+-{qemu-system-x86}(29368)
           |                        |-{qemu-system-x86}(29379)
           |                        |-{qemu-system-x86}(29380)
           |                        |-{qemu-system-x86}(29381)
           |                        |-{qemu-system-x86}(29383)
           |                        |-{qemu-system-x86}(29384)
           |                        `-{qemu-system-x86}(29443)
```

范例: 事先手动创建磁盘文件，再创建虚拟机

```
#创建磁盘文件
[root@ubuntu2004 ~]#qemu-img create -f qcow2 /var/lib/libvirt/images/rocky8.qcow2 10G
qemu-img create -f qcow2 /var/lib/libvirt/images/rocky8.qcow2 10G

#命令行创建虚拟机
[root@ubuntu2004 /var/lib/libvirt/images]# virt-install --virt-type kvm --os-variant=centos8 --name rocky8 --ram 1024 --vcpus 2 --cdrom=/data/isos/Rocky-8.9-x86_64-minimal.iso --disk path=/var/lib/libvirt/images/rocky8.qcow2 --network network=default --graphics vnc,listen=0.0.0.0 --noautoconsole
WARNING  Requested memory 1024 MiB is less than the recommended 1536 MiB for OS centos8

Starting install...
Domain installation still in progress. You can reconnect to 
the console to complete the installation process.

#创建默认NAT模式的虚拟机,并不自动打开virt-viewer连接console,需要手动打开virt-manager 连
接,并手动安装系统




```

#### **3.2.2.4** 验证宿主机进程

```
[root@ubuntu2004 /var/lib/libvirt/images]#ps aux|grep qemu
libvirt+   28818 90.4 14.7 4287652 1195732 ?     Sl   05:34   7:48 /usr/bin/qemu-system-x86_64 -name guest=rocky8,debug-threads=on -S -object secret,id=masterKey0,format=raw,file=/var/lib/libvirt/qemu/domain-9-rocky8/master-key.aes -machine pc-q35-4.2,accel=kvm,usb=off,dump-guest-core=off -cpu EPYC-Rome,x2apic=on,tsc-deadline=on,hypervisor=on,tsc-adjust=on,arch-capabilities=on,xsaves=on,virt-ssbd=on,rdctl-no=on,skip-l1dfl-vmentry=on,mds-no=on,pschange-mc-no=on,perfctr-core=off,xsaveerptr=off,amd-stibp=off -m 1024 -overcommit mem-lock=off -smp 2,sockets=2,cores=1,threads=1 -uuid 4bbf1b41-88fb-4edc-a241-e6b957148f88 -no-user-config -nodefaults -chardev socket,id=charmonitor,fd=31,server,nowait -mon chardev=charmonitor,id=monitor,mode=control -rtc base=utc,driftfix=slew -global kvm-pit.lost_tick_policy=delay -no-hpet -no-reboot -global ICH9-LPC.disable_s3=1 -global ICH9-LPC.disable_s4=1 -boot strict=on -device pcie-root-port,port=0x10,chassis=1,id=pci.1,bus=pcie.0,multifunction=on,addr=0x2 -device pcie-root-port,port=0x11,chassis=2,id=pci.2,bus=pcie.0,addr=0x2.0x1 -device pcie-root-port,port=0x12,chassis=3,id=pci.3,bus=pcie.0,addr=0x2.0x2 -device pcie-root-port,port=0x13,chassis=4,id=pci.4,bus=pcie.0,addr=0x2.0x3 -device pcie-root-port,port=0x14,chassis=5,id=pci.5,bus=pcie.0,addr=0x2.0x4 -device pcie-root-port,port=0x15,chassis=6,id=pci.6,bus=pcie.0,addr=0x2.0x5 -device pcie-root-port,port=0x16,chassis=7,id=pci.7,bus=pcie.0,addr=0x2.0x6 -device qemu-xhci,p2=15,p3=15,id=usb,bus=pci.2,addr=0x0 -device virtio-serial-pci,id=virtio-serial0,bus=pci.3,addr=0x0 -blockdev {"driver":"file","filename":"/var/lib/libvirt/images/rocky8.qcow2","node-name":"libvirt-2-storage","auto-read-only":true,"discard":"unmap"} -blockdev {"node-name":"libvirt-2-format","read-only":false,"driver":"qcow2","file":"libvirt-2-storage","backing":null} -device virtio-blk-pci,scsi=off,bus=pci.4,addr=0x0,drive=libvirt-2-format,id=virtio-disk0,bootindex=2 -blockdev {"driver":"file","filename":"/data/isos/Rocky-8.9-x86_64-minimal.iso","node-name":"libvirt-1-storage","auto-read-only":true,"discard":"unmap"} -blockdev {"node-name":"libvirt-1-format","read-only":true,"driver":"raw","file":"libvirt-1-storage"} -device ide-cd,bus=ide.0,drive=libvirt-1-format,id=sata0-0-0,bootindex=1 -netdev tap,fd=33,id=hostnet0,vhost=on,vhostfd=34 -device virtio-net-pci,netdev=hostnet0,id=net0,mac=52:54:00:15:1b:a6,bus=pci.1,addr=0x0 -chardev pty,id=charserial0 -device isa-serial,chardev=charserial0,id=serial0 -chardev socket,id=charchannel0,fd=35,server,nowait -device virtserialport,bus=virtio-serial0.0,nr=1,chardev=charchannel0,id=channel0,name=org.qemu.guest_agent.0 -device usb-tablet,id=input0,bus=usb.0,port=1 -vnc 0.0.0.0:0 -device qxl-vga,id=video0,ram_size=67108864,vram_size=67108864,vram64_size_mb=0,vgamem_mb=16,max_outputs=1,bus=pcie.0,addr=0x1 -device virtio-balloon-pci,id=balloon0,bus=pci.5,addr=0x0 -object rng-random,id=objrng0,filename=/dev/urandom -device virtio-rng-pci,rng=objrng0,id=rng0,bus=pci.6,addr=0x0 -sandbox on,obsolete=deny,elevateprivileges=deny,spawn=deny,resourcecontrol=deny -msg timestamp=on
root       29158  0.0  0.0   6300   720 pts/5    S+   05:42   0:00 grep --color=auto qemu


[root@ubuntu2004 /var/lib/libvirt/images]#pstree -p|grep qemu
           |-qemu-system-x86(28818)-+-{qemu-system-x86}(28820)
           |                        |-{qemu-system-x86}(28830)
           |                        |-{qemu-system-x86}(28831)
           |                        |-{qemu-system-x86}(28832)
           |                        |-{qemu-system-x86}(28834)
           |                        |-{qemu-system-x86}(28835)
           |                        |-{qemu-system-x86}(29192)
           |                        |-{qemu-system-x86}(29193)
           |                        |-{qemu-system-x86}(29194)
           |                        `-{qemu-system-x86}(29195)



#一键创建磁盘文件及虚拟机器
[root@ubuntu2004 /var/lib/libvirt/images]#virt-install --virt-type kvm --os-variant=centos7.0 --name centos7 --ram 1024 --vcpus 2 --cdrom=/data/isos/CentOS-7-x86_64-Minimal-2207-02.iso --disk path=/var/lib/libvirt/images/centos7.qcow2,size=10,format=qcow2,bus=virtio --network network=default --graphics vnc,listen=0.0.0.0 --noautoconsole

Starting install...
Allocating 'centos7.qcow2'                                                                                                                     |  10 GB  00:00:00     
Domain installation still in progress. You can reconnect to 
the console to complete the installation process.
[root@ubuntu2004 /var/lib/libvirt/images]#ll
total 2872828
drwx--x--x 2 root         root        4096 Jan 20 05:47 ./
drwxr-xr-x 7 root         root        4096 Jan 20 02:34 ../
-rw------- 1 libvirt-qemu kvm  10739318784 Jan 20 05:47 centos7.qcow2
-rw-r--r-- 1 libvirt-qemu kvm   2939944960 Jan 20 05:44 rocky8.qcow2

```

## **3.3** **基于现有虚拟机磁盘为模版创建新的虚拟机**

## **3.3.1** 利用virt-manager实现

### **3.3.1.1** 导入镜盘文件实现

```
#基于已经安装好虚拟机磁盘文件,创建新的磁盘文件
[root@centos8 ~]#cp -a /var/lib/libvirt/images/centos7.qcow2 
/var/lib/libvirt/images/centos7-2.qcow2
```

![image-20240120135914571](KVM虚拟化/image-20240120135914571.png)

![image-20240120135926492](KVM虚拟化/image-20240120135926492.png)

![image-20240120140606952](KVM虚拟化/image-20240120140606952.png)

![image-20240120140630362](KVM虚拟化/image-20240120140630362.png)



![image-20240120140657383](KVM虚拟化/image-20240120140657383.png)

![image-20240120140706892](KVM虚拟化/image-20240120140706892.png)

![image-20240120140721365](KVM虚拟化/image-20240120140721365.png)

### **3.3.1.2 Clone**

![image-20240120140040851](KVM虚拟化/image-20240120140040851.png)

![image-20240120140126160](KVM虚拟化/image-20240120140126160.png)

##  **3.3.2** **利用virt-install实现**

```
[root@ubuntu2004 /var/lib/libvirt/images]#pwd
/var/lib/libvirt/images
[root@ubuntu2004 /var/lib/libvirt/images]#cp centos7.qcow2 centos7-2.qcow2 


#Ubuntu 必须使用--os-variant 选项指定OS版本
[root@ubuntu2004 /var/lib/libvirt/images]#virt-install --virt-type kvm --os-variant=centos7.0 --name centos7-2 --ram 1024 --vcpu 1 --disk bus=virtio,path=/var/lib/libvirt/images/centos7-2.qcow2 --network network=default,model=virtio --graphics vnc,listen=0.0.0.0 --noautoconsole --boot hd

Starting install...
Domain creation completed.


#CentOS 可以不使用--os-variant 选项指定OS版本
[root@centos8 images]#virt-install --virt-type kvm --name centos8-2 --ram 2048 --
vcpus 2 --disk bus=virtio,path=/var/lib/libvirt/images/centos8-2.qcow2 --network 
network=default,model=virtio --graphics vnc,listen=0.0.0.0 --noautoconsole --
autostart --boot hd
WARNING No operating system detected, VM performance may suffer. Specify an OS 
with --os-variant for optimal results.
Starting install...
Domain creation completed.


#运行工具,可以看到下面出现新的虚拟机
[root@centos8 images]#virt-manager
```

![image-20240120141541485](KVM虚拟化/image-20240120141541485.png)

## **3.3.3** 利用virt-clone克隆实现

```
#基于已有的虚拟机克隆生成新的虚拟机
[root@ubuntu2004 ~]#virt-clone -o rocky8 -n rocky8-3 -f /var/lib/libvirt/images/rocky8-3.qcow2 
-o rocky8 #指已存在的虚拟机的名称
-n rocky8-3 #新虚拟机的名称
-f /var/lib/libvirt/images/rocky8-3.qcow2 #新虚拟机磁盘文件路径，此文件自动生成，不需要事先创建


实例：
[root@ubuntu2004 /var/lib/libvirt/images]#virt-clone -o rocky8  -f /var/lib/libvirt/images/rocky8-3.qcow2 -n rocky8-3
Allocating 'rocky8-3.qcow2'                                                                                                                    |  10 GB  00:00:04     

Clone 'rocky8-3' created successfully.

[root@ubuntu2004 /var/lib/libvirt/images]#ls -hl
total 5.5G
-rw------- 1 libvirt-qemu kvm   11G Jan 20 06:08 centos7-2.qcow2
-rw------- 1 libvirt-qemu kvm   11G Jan 20 05:47 centos7.qcow2
-rw------- 1 root         root 2.7G Jan 20 06:20 rocky8-3.qcow2 #查看新虚拟文件已创建
-rw-r--r-- 1 root         root 2.8G Jan 20 05:51 rocky8.qcow2
```

```
#脚本实现批量创建虚拟机，需要提前创建磁盘文件模板
[root@ubuntu2004 /var/lib/libvirt/images]#bash /root/clone.sh 
 Id   Name               State
-----------------------------------
 12   centos7-2          running
 -    centos7            shut off
 -    centos7-template   shut off
 -    rocky8             shut off
 -    rocky8-3           shut off
 -    rocky8-template    shut off

1) 克隆集群	 3) 启动集群	 5) 集群重启	 7) 单节点关机	 9) 克隆单机	11) 创建快照	13) 恢复快照
2) 删除集群	 4) 关闭集群	 6) 单节点启动	 8) 单节点重启	10) 单机删除	12) 删除快照	14) 退出
请按要求输入操作选项:1
克隆集群
模板清单:
<-------------------->
centos7-template
rocky8-template
<-------------------->
请输入克隆模板: 

```

# 4、管理虚拟机

## 4.1 使用半虚拟化驱动 virtio

### 4.1.1 半虚拟化驱动virtio的工作原理

为了提高内存、硬盘、网络的性能，需要支持半虚拟化

![image-20240120152916759](KVM虚拟化/image-20240120152916759.png)

![image-20240120152928241](KVM虚拟化/image-20240120152928241.png)

```
virtio 是一种 I/O 半虚拟化解决方案，是一套通用 I/O 设备虚拟化的程序，是对半虚拟化 Hypervisor 中的一组通用 I/O 设备的抽象，提供了一套上层应用与各 Hypervisor 虚拟化设备（KVM，Xen，VMware等）之间的通信框架和编程接口，减少跨平台所带来的兼容性问题，大大提高驱动程序开发效率，Windows 系统需要单独安装virtio驱 动，Linux系统自带virtio驱动。
```

![image-20240120152947974](KVM虚拟化/image-20240120152947974.png)

```
实现IO虚拟化主要有三种方式：全虚拟化、半虚拟化和透传，全虚拟化Guest OS不会感知到自己是虚拟机，也无需修改Guest OS，但是它的效率比较低，半虚拟化Guest OS知道自己是虚拟机，通过Frontend/Backend驱动模拟实现IO虚拟化，透传就是直接分配物理设备给VM用，但是需要解决单个硬件在多个虚拟机共享使用的问题。
```

**性能比较**

```
https://www.ilsistemista.net/index.php/virtualization/42-kvm-virtioparavirtualized-drivers-why-they-matter.html?start=2
```

![image-20240120153401043](KVM虚拟化/image-20240120153401043.png)

![image-20240120153410242](KVM虚拟化/image-20240120153410242.png)

### **4.1.2** 半虚拟化设备统一接口virtio

![image-20240120153440357](KVM虚拟化/image-20240120153440357.png)

```
通过统一的接口virtio以支持的多种硬件设备

​	不同的虚拟设备和不同的虚拟机可以有不同的前端驱动

​	不同的硬件设备可以有不同的后端驱动

​	两者之间的交互遵循virtio的标准
```

范例：创建windos2008 KVM虚拟机(磁盘不低于20G)

```
virt-install \
--virt-type=kvm \
--name win2008r2 \
--ram 4096 \
--vcpus=4 \
--os-variant=win2k8r2 \
--cdrom=/data/isos/windows_server_2008_r2_x64_dvd_617598.iso \
--network=bridge=virbr0,model=virtio \
--graphics vnc,listen=0.0.0.0 --noautoconsole \
--disk path=/var/lib/libvirt/images/win2008r2.qcow2,size=20,bus=virtio,format=qcow2 \
--disk path=/data/isos/virtio-win-0.1.189.iso,device=cdrom


#可以临时加载光盘ISO镜像
virsh attach-disk win2016 /data/isos/virtio-win-0.1.141.iso hda --type cdrom --mode readonly
```

命令行安装虚拟机

```
[root@ubuntu2004 /data/isos]#rz -E
rz waiting to receive.
[root@ubuntu2004 /data/isos]#ls
CentOS-7-x86_64-Minimal-2207-02.iso  Rocky-8.9-x86_64-minimal.iso  virtio-win-0.1.141.iso  virtio-win-0.1.189.iso  windows_server_2008_r2_x64_dvd_617598.iso
[root@ubuntu2004 /data/isos]#virt-install \
> --virt-type=kvm \
> --name win2008r2 \
> --ram 4096 \
> --vcpus=4 \
> --os-variant=win2k8r2 \
> --cdrom=/data/isos/windows_server_2008_r2_x64_dvd_617598.iso \
> --network=bridge=virbr0,model=virtio \
> --graphics vnc,listen=0.0.0.0 --noautoconsole \
> --disk path=/var/lib/libvirt/images/win2008r2.qcow2,size=20,bus=virtio,format=qcow2 \
> --disk path=/data/isos/virtio-win-0.1.189.iso,device=cdrom

Starting install...
Allocating 'win2008r2.qcow2'                                                                                                                   |  20 GB  00:00:00     
Domain installation still in progress. You can reconnect to 
the console to complete the installation process.


[root@ubuntu2004 ~]#virt-manager 
```

![image-20240121213137809](KVM虚拟化/image-20240121213137809.png)

![image-20240121213201006](KVM虚拟化/image-20240121213201006.png)

![image-20240121213213499](KVM虚拟化/image-20240121213213499.png)

![image-20240121213220453](KVM虚拟化/image-20240121213220453.png)

![image-20240121213226738](KVM虚拟化/image-20240121213226738.png)

![image-20240121213245792](KVM虚拟化/image-20240121213245792.png)

![image-20240121213252917](KVM虚拟化/image-20240121213252917.png)

![image-20240121213301500](KVM虚拟化/image-20240121213301500.png)

![image-20240121214019705](KVM虚拟化/image-20240121214019705.png)

![](KVM虚拟化/image-20240121213307393.png)

![image-20240121214043317](KVM虚拟化/image-20240121214043317.png)

![image-20240121214104190](KVM虚拟化/image-20240121214104190.png)

![image-20240121212750459](KVM虚拟化/image-20240121212750459.png)

![image-20240121212809598](KVM虚拟化/image-20240121212809598.png)

![image-20240121214229200](KVM虚拟化/image-20240121214229200.png)

![image-20240121214237533](KVM虚拟化/image-20240121214237533.png)

![image-20240121214143339](KVM虚拟化/image-20240121214143339.png)



### **4.1.5.4** 验证 Windows virtio 驱动





### **4.1.5.7** **生成Windows Server镜像模版**

利用sysprep工具,清除个性信息,下次Windows开机时, 会自动生成初始化个性信息

![image-20240123111129404](KVM虚拟化/image-20240123111129404.png)



![image-20240123111521336](KVM虚拟化/image-20240123111521336.png)



![image-20240123111543287](KVM虚拟化/image-20240123111543287.png)

**下次开机需要重新初始化**

范例：windos -server2008(不支持virt-clone的方式创建)

```
virt-install \
--virt-type kvm \
--name win2008r2-template.qcow2 \
--memory 3072 \
--vcpus=2 \
--os-variant=win2k8r2 \
--disk path=/var/lib/libvirt/images/win2008r2-template.qcow2,format=qcow2,bus=virtio \
--network bridge=virbr0,model=virtio --graphics vnc,listen=0.0.0.0 --noautoconsole --autostart --boot hd


[root@ubuntu2004 /var/lib/libvirt/images]# cp win2008r2.qcow2 win2008r2-template.qcow2
[root@ubuntu2004 /var/lib/libvirt/images]#virt-install \
> --virt-type kvm \
> --name win2008r2-template.qcow2 \
> --memory 3072 \
> --vcpus=2 \
> --os-variant=win2k8r2 \
> --disk path=/var/lib/libvirt/images/win2008r2-template.qcow2,format=qcow2,bus=virtio \
> --network bridge=virbr0,model=virtio --graphics vnc,listen=0.0.0.0 --noautoconsole --autostart --boot hd

Starting install...
Domain creation completed.

```

## **4.2 libvirt** **架构**

![image-20240123124038563](KVM虚拟化/image-20240123124038563.png)

**注意:如果libvirtd服务意外关闭，将导致相关工具，如:virt-manager等无法和虚拟机连接，但虚拟机仍会正常运行**

范例: libvirtd 服务功能

```
[root@ubuntu2004 ~]#virsh list
 Id   Name                       State
------------------------------------------
 17   win2008r2                  running
 18   win2008r2-template.qcow2   running

[root@ubuntu2004 ~]#systemctl status libvirtd
● libvirtd.service - Virtualization daemon
     Loaded: loaded (/lib/systemd/system/libvirtd.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2024-01-20 08:04:24 UTC; 2 days ago
TriggeredBy: ● libvirtd.socket
             ● libvirtd-ro.socket
             ● libvirtd-admin.socket
       Docs: man:libvirtd(8)
             https://libvirt.org
   Main PID: 991 (libvirtd)
      Tasks: 20 (limit: 32768)
     Memory: 22.3M
     CGroup: /system.slice/libvirtd.service
             ├─ 991 /usr/sbin/libvirtd
             ├─1154 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/lib/libvirt/libvirt_leaseshelper
             └─1155 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/lib/libvirt/libvirt_leaseshelper

Jan 23 04:23:07 ubuntu2004 dnsmasq-dhcp[1154]: DHCPREQUEST(virbr0) 192.168.122.200 52:54:00:6e:94:dd
Jan 23 04:23:07 ubuntu2004 dnsmasq-dhcp[1154]: DHCPACK(virbr0) 192.168.122.200 52:54:00:6e:94:dd WIN-VO5VOCU0T20
Jan 23 04:23:10 ubuntu2004 dnsmasq-dhcp[1154]: DHCPINFORM(virbr0) 192.168.122.200 52:54:00:6e:94:dd
Jan 23 04:23:10 ubuntu2004 dnsmasq-dhcp[1154]: DHCPACK(virbr0) 192.168.122.200 52:54:00:6e:94:dd WIN-VO5VOCU0T20
Jan 23 04:23:15 ubuntu2004 dnsmasq-dhcp[1154]: DHCPREQUEST(virbr0) 192.168.122.104 52:54:00:0d:d2:55
Jan 23 04:23:15 ubuntu2004 dnsmasq-dhcp[1154]: DHCPACK(virbr0) 192.168.122.104 52:54:00:0d:d2:55 WIN-JU0QC868CMR
Jan 23 04:23:18 ubuntu2004 dnsmasq-dhcp[1154]: DHCPREQUEST(virbr0) 192.168.122.104 52:54:00:0d:d2:55
Jan 23 04:23:18 ubuntu2004 dnsmasq-dhcp[1154]: DHCPACK(virbr0) 192.168.122.104 52:54:00:0d:d2:55 WIN-JU0QC868CMR
Jan 23 04:23:21 ubuntu2004 dnsmasq-dhcp[1154]: DHCPINFORM(virbr0) 192.168.122.104 52:54:00:0d:d2:55
Jan 23 04:23:21 ubuntu2004 dnsmasq-dhcp[1154]: DHCPACK(virbr0) 192.168.122.104 52:54:00:0d:d2:55 WIN-JU0QC868CMR

[root@ubuntu2004 ~]#systemctl stop libvirtd libvirtd.socket libvirtdadmin.socket libvirtd-ro.socket

[root@ubuntu2004 ~]#virt-manager 
[root@ubuntu2004 ~]#systemctl status libvirtd
● libvirtd.service - Virtualization daemon
     Loaded: loaded (/lib/systemd/system/libvirtd.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Tue 2024-01-23 04:47:25 UTC; 45s ago
TriggeredBy: ● libvirtd.socket
             ● libvirtd-ro.socket
             ● libvirtd-admin.socket
       Docs: man:libvirtd(8)
             https://libvirt.org
    Process: 991 ExecStart=/usr/sbin/libvirtd $libvirtd_opts (code=exited, status=0/SUCCESS)
   Main PID: 991 (code=exited, status=0/SUCCESS)
      Tasks: 2 (limit: 32768)
     Memory: 14.0M
     CGroup: /system.slice/libvirtd.service
             ├─1154 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/lib/libvirt/libvirt_leaseshelper
             └─1155 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/lib/libvirt/libvirt_leaseshelper

Jan 23 04:23:10 ubuntu2004 dnsmasq-dhcp[1154]: DHCPACK(virbr0) 192.168.122.200 52:54:00:6e:94:dd WIN-VO5VOCU0T20
Jan 23 04:23:15 ubuntu2004 dnsmasq-dhcp[1154]: DHCPREQUEST(virbr0) 192.168.122.104 52:54:00:0d:d2:55
Jan 23 04:23:15 ubuntu2004 dnsmasq-dhcp[1154]: DHCPACK(virbr0) 192.168.122.104 52:54:00:0d:d2:55 WIN-JU0QC868CMR
Jan 23 04:23:18 ubuntu2004 dnsmasq-dhcp[1154]: DHCPREQUEST(virbr0) 192.168.122.104 52:54:00:0d:d2:55
Jan 23 04:23:18 ubuntu2004 dnsmasq-dhcp[1154]: DHCPACK(virbr0) 192.168.122.104 52:54:00:0d:d2:55 WIN-JU0QC868CMR
Jan 23 04:23:21 ubuntu2004 dnsmasq-dhcp[1154]: DHCPINFORM(virbr0) 192.168.122.104 52:54:00:0d:d2:55
Jan 23 04:23:21 ubuntu2004 dnsmasq-dhcp[1154]: DHCPACK(virbr0) 192.168.122.104 52:54:00:0d:d2:55 WIN-JU0QC868CMR
Jan 23 04:47:25 ubuntu2004 systemd[1]: Stopping Virtualization daemon...
Jan 23 04:47:25 ubuntu2004 systemd[1]: libvirtd.service: Succeeded.
Jan 23 04:47:25 ubuntu2004 systemd[1]: Stopped Virtualization daemon.


#virt-manager工具也无法连接虚拟机
#如果再次重新运行virt-manage，会自动激活libvirtd服务
#但是systemctl stop libvirtd.socket libvirtd-admin.socket libvirtd-ro.socket 
libvirtd.service，将无法自动激活libvirtd.service

[root@ubuntu2004 ~]#systemctl start libvirtd.socket 

[root@ubuntu2004 ~]#virt-manager   #虚拟机管理工具无法连接
[root@ubuntu2004 ~]#systemctl status libvirtd
● libvirtd.service - Virtualization daemon
     Loaded: loaded (/lib/systemd/system/libvirtd.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2024-01-23 04:48:40 UTC; 5s ago
TriggeredBy: ● libvirtd.socket
             ● libvirtd-ro.socket
             ● libvirtd-admin.socket
       Docs: man:libvirtd(8)
             https://libvirt.org
   Main PID: 44082 (libvirtd)
      Tasks: 20 (limit: 32768)
     Memory: 43.4M
     CGroup: /system.slice/libvirtd.service
             ├─ 1154 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/lib/libvirt/libvirt_leaseshelper
             ├─ 1155 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/lib/libvirt/libvirt_leaseshelper
             └─44082 /usr/sbin/libvirtd

Jan 23 04:48:40 ubuntu2004 systemd[1]: libvirtd.service: Found left-over process 1155 (dnsmasq) in control group while starting unit. Ignoring.
Jan 23 04:48:40 ubuntu2004 systemd[1]: This usually indicates unclean termination of a previous run, or service implementation deficiencies.
Jan 23 04:48:40 ubuntu2004 systemd[1]: Starting Virtualization daemon...
Jan 23 04:48:40 ubuntu2004 systemd[1]: Started Virtualization daemon.
Jan 23 04:48:40 ubuntu2004 dnsmasq[1154]: read /etc/hosts - 7 addresses
Jan 23 04:48:40 ubuntu2004 dnsmasq[1154]: read /var/lib/libvirt/dnsmasq/default.addnhosts - 0 addresses
Jan 23 04:48:40 ubuntu2004 dnsmasq-dhcp[1154]: read /var/lib/libvirt/dnsmasq/default.hostsfile
Jan 23 04:48:40 ubuntu2004 libvirtd[44082]: libvirt version: 6.0.0, package: 0ubuntu8.16 (Marc Deslauriers <marc.deslauriers@ubuntu.com> Wed, 20 Apr 2022 11:31:12 -0400)
Jan 23 04:48:40 ubuntu2004 libvirtd[44082]: hostname: ubuntu2004
Jan 23 04:48:40 ubuntu2004 libvirtd[44082]: operation failed: pool 'default' already exists with uuid 44843129-a452-4e40-8a2a-cfb7b246b20b
[root@ubuntu2004 ~]#virsh list
 Id   Name                       State
------------------------------------------
 17   win2008r2                  running
 18   win2008r2-template.qcow2   running

#virt-manager工具也恢复连接虚拟机
```

![image-20240131141358478](KVM虚拟化/image-20240131141358478.png)



# 5、存储管理

## 5.1、存储模式

**基于文件系统的存储**

```
dir: Filesystem Directory 需要有挂载点的文件系统
fs: Pre-Formatted Block Device 无需挂载的文件系统,如:位于SAN存储的文件系统,可支持多个主机同时访问,而本地文件系统不支持
netfs: Network Exported Directory 网络文件系统,比如:NFS,SAMBA等
```

**基于设备的存储**

```
无需文件系统,性能更好,但可管理性差,无法实现快照
	Disk: Physical Disk Device
	Iscsi: isCSI Target
	logical:LVM Volume Group
```

## **5.2** **虚拟磁盘类型**

```
1、固定Fixed
	在配置时，指定磁盘大小
	不管在虚拟磁盘上实际存储多少数据，都将占用相同大小宿主机的磁盘空间
2、动态Dynamic
	初始空间占用小
	随着空间的使用逐渐增长到最大容量，但是只根据需求使用更多的空间
3、差异Differencing
	因为创建是差异磁盘，所以只保存变更的数据
	例如，将操作系统安装在父盘，然后创建差异化磁盘来执行进一步配置
```

### **5.3** **虚拟镜像文件格式**

镜像文件储存在主机文件系统中。它可以储存在本地文件系统中，如 ext4 或 xfs；或网络文件系统中，如 NFS 。例如 libguestfs 这样的工具，能管理、备份及监控文件。KVM 上的磁盘镜像格式包括：

```
1、raw
此为默认磁盘格式,但并是一种真正的磁盘格式，而是代表虚拟机所使用的原始镜像,它并不存储元数据，因此可作为保证虚拟机兼容性的候选方案。然而，也正因为它不存储元数据，因此不支持某些高级特往，比如快照和压缩等格式简单，容易转换为其他的格式。
如果主机文件系统允许，raw 文件可以是预分配（pre-allocated）或 稀疏（sparse）。稀疏文件根据需求分配主机磁盘空间，因此它是一种精简配置形式（thin provisioning）。预分配文件的所有空间需要被预先分配，但它比稀疏文件性能好。当对磁盘 I/O 性能要求非常高，而且通常不需要通过网络传输镜像文件时，可以使用 raw文件
优点 : 性能好
缺点 : 空间占用大,功能较少,生产不推荐使用
2、cow : copy-on-write格式，昙花一现

3、qcow : QEMU早期的copy-on-write格式，过渡性方案
4、qcow2
qcow2 镜像文件提供许多高级磁盘镜像特征，如快照、压缩及加密。它们可以用来代表通过模板镜像创建的虚拟机。因为只有虚拟机写入的扇区部分才会分配在镜像中，所以 qcow2 文件的网络传输效率较高。RHEL 7.0 及更新版本支持 qcow2 v3 镜像文件格式
按需进行分配磁盘空间，不管文件系统是否支持支持快照
支持zlib的磁盘压缩
支持AES的加密
优点 : 空间节约,功能丰富
缺点 : 性能较差,生产推荐使用
5、vmdk( Virtual Machine Disk ): VMware环境当中默认使用的磁盘格式
6、vhd \ vhdx ( Virtual Hard Disk ):微软默认采用的文件格式
7、vdi : VirtualBox 采用的文件格式
```

查看支持的格式

```
qemu-img --help
```

范例: 查看KVM支持的磁盘格式

```
[root@ubuntu2004 ~]#qemu-img --help |grep -i support
    supported. 'b' is ignored.
    name=value format. Use -o ? for an overview of the options supported by the
  '-h' with or without a command shows this help and lists the supported formats
Supported formats: blkdebug blklogwrites blkreplay blkverify bochs cloop copy-on-read dmg file ftp ftps host_cdrom host_device http https iscsi iser luks nbd null-aio null-co nvme parallels qcow qcow2 qed quorum raw rbd replication sheepdog ssh throttle vdi vhdx vmdk vpc vvfat
```

## **5.4** **使用qemu-img管理虚拟磁盘文件**

### **5.4.1 qemu-img概述**

qemu-img 是一个功能强大的磁盘镜像管理工具

```
查看帮助: qemu-img --help
```

qemu-img 包括以下子命令

```
check #检查完整性
create #创建镜像
commit #提交更改
compare #比较
convert #转换
info #获得信息
map #映射
snapshot #快照管理
rebase #在已有的镜像的基础上创建新的镜像
resize #调整大小
amend #修订镜像格式选项
```

### **5.4.2** **创建虚拟磁盘文件**

```
qemu-img create [--object objectdef] [-q] [-f fmt] [-b backing_file] [-F backing_fmt] [-u] [-o options] filename [size]
```

#### **5.4.2.1** **创建默认稀疏格式的磁盘**

```
#默认为raw格式稀疏文件
[root@ubuntu2004 ~]#qemu-img create vm1.img 1g
Formatting 'vm1.img', fmt=raw size=1073741824

#查看文件类型
[root@ubuntu2004 ~]#file vm1.img 
vm1.img: data

#查看文件大小
[root@ubuntu2004 ~]#ll vm1.img -h
-rw-r--r-- 1 root root 1.0G Jan 31 14:27 vm1.img

#查看文件实际大小
[root@ubuntu2004 ~]#du -sh vm1.img 
4.0K	vm1.img

#显示问价信息
[root@ubuntu2004 ~]#qemu-img info vm1.img 
image: vm1.img
file format: raw
virtual size: 1 GiB (1073741824 bytes)
disk size: 4 KiB

```

#### **5.4.2.2** 查看不同磁盘文件格式支持选项

```
[root@ubuntu2004 ~]#qemu-img create -f raw -o ?
Supported raw options:
  size=<size>            - Virtual disk size

The protocol level may support further options.
Specify the target filename to include those options.
[root@ubuntu2004 ~]#qemu-img create -f qcow2 -o ?
Supported qcow2 options:
  backing_file=<str>     - File name of a base image
  backing_fmt=<str>      - Image format of the base image
  cluster_size=<size>    - qcow2 cluster size
  compat=<str>           - Compatibility level (v2 [0.10] or v3 [1.1])
  data_file=<str>        - File name of an external data file
  data_file_raw=<bool (on/off)> - The external data file must stay valid as a raw image
  encrypt.cipher-alg=<str> - Name of encryption cipher algorithm
  encrypt.cipher-mode=<str> - Name of encryption cipher mode
  encrypt.format=<str>   - Encrypt the image, format choices: 'aes', 'luks'
  encrypt.hash-alg=<str> - Name of encryption hash algorithm
  encrypt.iter-time=<num> - Time to spend in PBKDF in milliseconds
  encrypt.ivgen-alg=<str> - Name of IV generator algorithm
  encrypt.ivgen-hash-alg=<str> - Name of IV generator hash algorithm
  encrypt.key-secret=<str> - ID of secret providing qcow AES key or LUKS passphrase
  encryption=<bool (on/off)> - Encrypt the image with format 'aes'. (Deprecated in favor of encrypt.format=aes)
  lazy_refcounts=<bool (on/off)> - Postpone refcount updates
  preallocation=<str>    - Preallocation mode (allowed values: off, metadata, falloc, full)
  refcount_bits=<num>    - Width of a reference count entry in bits
  size=<size>            - Virtual disk size

The protocol level may support further options.
Specify the target filename to include those options.
```

#### **5.4.2.3 qcow2 格式选项**

```
backing_file #指定后端镜像文件
backing_fmt #设置后端镜像的镜像格式
cluster_size #设置镜像中的簇大小，取值在512到2M之间，默认值为64K
preallocation #设置镜像文件空间的预分配模式
encryption #用于设置加密
```

#### **5.4.2.5 **创建raw格式非稀疏文件

```
[root@ubuntu2004 ~]#dd if=/dev/zero of=/data/vm2.img bs=1M count=1024
1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 3.80443 s, 282 MB/s

[root@ubuntu2004 ~]#ll -h /data/vm2.img 
-rw-r--r-- 1 root root 1.0G Jan 31 14:37 /data/vm2.img

[root@ubuntu2004 ~]#du -sh /data/vm2.img 
1.1G	/data/vm2.img

[root@ubuntu2004 ~]#qemu-img info /data/vm2.img 
image: /data/vm2.img
file format: raw
virtual size: 1 GiB (1073741824 bytes)
disk size: 1 GiB
```

#### **5.4.2.4** **创建raw格式稀疏文件**

```
[root@ubuntu2004 ~]#dd if=/dev/zero of=/data/vm3.img bs=1M count=0 seek=1024
0+0 records in
0+0 records out
0 bytes copied, 0.000140816 s, 0.0 kB/s

[root@ubuntu2004 ~]#ll -h /data/vm3.img 
-rw-r--r-- 1 root root 1.0G Jan 31 14:34 /data/vm3.img

[root@ubuntu2004 ~]#du -sh /data/vm3.img 
0	/data/vm3.img

[root@ubuntu2004 ~]#qemu-img info /data/vm3.img 
image: /data/vm3.img
file format: raw
virtual size: 1 GiB (1073741824 bytes)
disk size: 0 B
```

#### **5.4.2.6 raw文件复制的格式控制**

```
#复制非稀疏文件,默认也为非稀疏文件 
[root@ubuntu2004 ~]#cp /data/vm2.img /data/vm2.img.bak
[root@ubuntu2004 ~]#du -sh /data/vm2.img.bak 
1.1G	/data/vm2.img.bak
[root@ubuntu2004 ~]#ll /data/vm2.img.bak 
-rw-r--r-- 1 root root 1073741824 Jan 31 14:40 /data/vm2.img.bak
[root@ubuntu2004 ~]#qemu-img info /data/vm2.img.bak 
image: /data/vm2.img.bak
file format: raw
virtual size: 1 GiB (1073741824 bytes)
disk size: 1 GiB

#复制稀疏文件,默认仍为稀疏文件
[root@ubuntu2004 ~]#cp /data/vm3.img /data/vm3.img.bak
[root@ubuntu2004 ~]#ll -h /data/vm3.img.bak
-rw-r--r-- 1 root root 1.0G Jan 31 14:42 /data/vm3.img.bak
[root@ubuntu2004 ~]#du -sh /data/vm3.img.bak
0	/data/vm3.img.bak
[root@ubuntu2004 ~]#qemu-img info /data/vm3.img.bak
image: /data/vm3.img.bak
file format: raw
virtual size: 1 GiB (1073741824 bytes)
disk size: 0 B

#指定将非稀疏文件复制为稀疏格式格式
[root@ubuntu2004 ~]#cp --sparse=always /data/vm2.img /data/vm2.img.bak
[root@ubuntu2004 ~]#ll -h /data/vm2.img.bak 
-rw-r--r-- 1 root root 1.0G Feb  6 13:50 /data/vm2.img.bak
[root@ubuntu2004 ~]#du -h /data/vm2.img.bak 
0	/data/vm2.img.bak
[root@ubuntu2004 ~]#qemu-img info /data/vm2.img.bak 
image: /data/vm2.img.bak
file format: raw
virtual size: 1 GiB (1073741824 bytes)
disk size: 0 B

#指定将稀疏文件复制为非稀疏格式格式
[root@ubuntu2004 ~]#cp --sparse=never /data/vm1.img /data/vm1.img.bak
[root@ubuntu2004 ~]#ll -h /data/vm1.img.bak 
-rw-r--r-- 1 root root 1.0G Feb  6 13:53 /data/vm1.img.bak
[root@ubuntu2004 ~]#du -h /data/vm1.img.bak 
1.1G	/data/vm1.img.bak
[root@ubuntu2004 ~]#qemu-img info /data/vm1.img.bak 
image: /data/vm1.img.bak
file format: raw
virtual size: 1 GiB (1073741824 bytes)
disk size: 1 GiB

[root@ubuntu2004 ~]#qemu-img info /var/lib/libvirt/images/rocky8.qcow2 
image: /var/lib/libvirt/images/rocky8.qcow2
file format: qcow2
virtual size: 10 GiB (10737418240 bytes)
disk size: 2.74 GiB
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
    refcount bits: 16
    corrupt: false

http://11.0.1.21/zabbix/api_jsonrpc.php
```

### 5.4.3、检查虚拟磁盘

```
[root@ubuntu2004 ~]#virsh list --alll
error: command 'list' doesn't support option --alll
[root@ubuntu2004 ~]#virsh list --all
 Id   Name                       State
-------------------------------------------
 -    rocky8-template            shut off
 -    win2008r2-template.qcow2   shut off
 -    win2012r2                  shut off

[root@ubuntu2004 ~]#qemu-img check /var/lib/libvirt/images/rocky8-template.qcow2 
No errors were found on the image.
43887/163840 = 26.79% allocated, 0.02% fragmented, 0.00% compressed clusters
Image end offset: 2877227008

[root@ubuntu2004 ~]#qemu-img check /var/lib/libvirt/images/win2008r2-template.qcow2 
No errors were found on the image.
327680/327680 = 100.00% allocated, 0.00% fragmented, 0.00% compressed clusters
Image end offset: 21478375424
```

### 5.4.4、磁盘预分配策略

raw 文件的预分配策略和文件系统是否支持有关,而qcow2则无关预分配策略

```
off
此为缺省策略，即不使用预分配策略,预分配后的虚拟磁盘占用空间很小,不属于稀疏映像类型生成的磁盘文件占用空间很小相当于房地产开发商拆迁，只圈地但不禁止旧房的使用
metadata
只预分配元数据(metadata)，预分配后的磁盘文件属于稀疏映像类型,相当于vmware中的磁盘置备选项: Thin Provision(精简配置)生成的磁盘文件为稀疏格式,实际占用的空间比off策略稍大一些相当于房地产开发商拆迁，只圈地但不禁止旧房的使用，占一小块地建了售楼处
falloc
分配文件的块并标识它们的状态为未初始化，即只分配空间,但不置零. 预分配后的虚拟磁盘属于非稀疏映像类型,相对full模式来说，创建虚拟磁盘的速度要快很多,相当于vmware中的磁盘置备选项: 厚置备延迟置零生成的磁盘文件实际占用的空间和分配的空间相同大小相当于房地产开发商拆迁，只圈地且禁止旧房的使用，但旧房还存在
full
分配所有磁盘空间并置零，预分配后的虚拟磁盘属于非稀疏映像类型,创建最慢,相当于vmware中的磁盘置备选项: 厚置备置零生成的磁盘文件实际占用的空间和分配的空间相同大小相当于房地产开发商拆迁，只圈地且完全清除旧房
```

范例: 创建预分配置策略

```
[root@ubuntu2004 ~]#qemu-img create -f qcow2 test1.qcow2 1g
Formatting 'test1.qcow2', fmt=qcow2 size=1073741824 cluster_size=65536 lazy_refcounts=off refcount_bits=16
[root@ubuntu2004 ~]#qemu-img info test1.qcow2
image: test1.qcow2
file format: qcow2
virtual size: 1 GiB (1073741824 bytes)
disk size: 196 KiB
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
    refcount bits: 16
    corrupt: false
[root@ubuntu2004 ~]#du -sh test1.qcow2
196K	test1.qcow2

#指定关闭预分配      
[root@ubuntu2004 ~]#qemu-img create -f qcow2 test2.qcow2 1g  -o preallocation=off  #创建速度最快
Formatting 'test2.qcow2', fmt=qcow2 size=1073741824 cluster_size=65536 preallocation=off lazy_refcounts=off refcount_bits=16
[root@ubuntu2004 ~]#ll -h test2.qcow2 
-rw-r--r-- 1 root root 193K Feb  6 14:05 test2.qcow2
[root@ubuntu2004 ~]#du -sh test2.qcow2 
196K	test2.qcow2
[root@ubuntu2004 ~]#qemu-img info test2.qcow2 
image: test2.qcow2
file format: qcow2
virtual size: 1 GiB (1073741824 bytes)
disk size: 196 KiB
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
    refcount bits: 16
    corrupt: false

#指定预分配metadata
[root@ubuntu2004 ~]#qemu-img create -f qcow2 test3.qcow2 1g -o preallocation=metadata
Formatting 'test3.qcow2', fmt=qcow2 size=1073741824 cluster_size=65536 preallocation=metadata lazy_refcounts=off refcount_bits=16
[root@ubuntu2004 ~]#ll -h test3.qcow2 
-rw-r--r-- 1 root root 1.1G Feb  6 14:08 test3.qcow2
[root@ubuntu2004 ~]#du -sh test3.qcow2 
324K	test3.qcow2
[root@ubuntu2004 ~]#qemu-img info test3.qcow2 
image: test3.qcow2
file format: qcow2
virtual size: 1 GiB (1073741824 bytes)
disk size: 324 KiB
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
    refcount bits: 16
    corrupt: false
    
#指定预分配falloc
[root@ubuntu2004 ~]#qemu-img create -f qcow2 test4.qcow2 1g -o preallocation=falloc
Formatting 'test4.qcow2', fmt=qcow2 size=1073741824 cluster_size=65536 preallocation=falloc lazy_refcounts=off refcount_bits=16
[root@ubuntu2004 ~]#ll -h test4.qcow2 
-rw-r--r-- 1 root root 1.1G Feb  6 14:10 test4.qcow2
[root@ubuntu2004 ~]#du -sh test4.qcow2 
1.1G	test4.qcow2
[root@ubuntu2004 ~]#qemu-img info test4.qcow2 
image: test4.qcow2
file format: qcow2
virtual size: 1 GiB (1073741824 bytes)
disk size: 1 GiB
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
    refcount bits: 16
    corrupt: false

#指定预分配full
[root@ubuntu2004 ~]#qemu-img create -f qcow2 test5.qcow2 1g -o preallocation=full   #生产建议使用，速度稍慢
Formatting 'test5.qcow2', fmt=qcow2 size=1073741824 cluster_size=65536 preallocation=full lazy_refcounts=off refcount_bits=16
[root@ubuntu2004 ~]#ll -h test5.qcow2 
-rw-r--r-- 1 root root 1.1G Feb  6 14:11 test5.qcow2
[root@ubuntu2004 ~]#du -sh test5.qcow2 
1.1G	test5.qcow2
[root@ubuntu2004 ~]#qemu-img info test5.qcow2 
image: test5.qcow2
file format: qcow2
virtual size: 1 GiB (1073741824 bytes)
disk size: 1 GiB
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
    refcount bits: 16
    corrupt: false
    
#查看真实大小
[root@ubuntu2004 ~]#du -sh test*
196K	test1.qcow2
196K	test2.qcow2
1.1G	test3.qcow2
1.1G	test4.qcow2
1.1G	test5.qcow2
```

### **5.4.5**、 虚拟磁盘格式转换

qemu-img 可以将不同格式的虚拟磁盘文件进行格式转化

```shell

#转化为qcow2格式
[root@ubuntu2004 /var/lib/libvirt/images]#qemu-img convert -f vmdk -O qcow2 centos6.vmdk centos6.qcow2
[root@ubuntu2004 /var/lib/libvirt/images]#ll
total 21137184	
drwx--x--x 2 root         root        4096 Feb  6 14:23 ./
drwxr-xr-x 7 root         root        4096 Feb  6 04:02 ../
-rw-r--r-- 1 root         root  1587478528 Feb  6 14:23 centos6.qcow2
-rw-r--r-- 1 root         root  1606221824 Feb  6 14:18 centos6.vmdk
-rw------- 1 root         root  2877227008 Jan 21 14:43 rocky8-template.qcow2
-rw------- 1 libvirt-qemu kvm  21478375424 Feb  6 13:26 win2008r2-template.qcow2
-rw-r--r-- 1 root         root 21474836480 Feb  6 13:27 win2k12r2.qcow2

[root@ubuntu2004 /var/lib/libvirt/images]#qemu-img info centos6.qcow2 
image: centos6.qcow2
file format: qcow2
virtual size: 200 GiB (214748364800 bytes)
disk size: 1.48 GiB
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
    refcount bits: 16
    corrupt: false

[root@ubuntu2004 ~]# virt-install --virt-type kvm --os-variant=centos6.0 --name centos6.0 --ram 1024 --vcpus 1 --disk bus=virtio,path=/var/lib/libvirt/images/centos6.qcow2 --network network=default,model=virtio --graphics vnc,listen=0.0.0.0 --noautoconsole --autostart --boot hd   #创建新虚拟机

[root@ubuntu2004 ~]#virsh list --all
 Id   Name                       State
-------------------------------------------
 3    centos6.0                  running
 -    rocky8-template            shut off
 -    win2008r2-template.qcow2   shut off
 -    win2012r2                  shut off
 
#查看硬盘大小
[root@ubuntu2004 /var/lib/libvirt/images]#qemu-img info centos6.qcow2 
image: centos6.qcow2
file format: qcow2
virtual size: 205 GiB (220117073920 bytes)
disk size: 1.48 GiB
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
    refcount bits: 16
    corrupt: false
       
#硬盘扩容5G
[root@ubuntu2004 /var/lib/libvirt/images]#qemu-img resize centos6.qcow2 +5G
Image resized.
[root@ubuntu2004 /var/lib/libvirt/images]#qemu-img info centos6.qcow2 
image: centos6.qcow2
file format: qcow2
virtual size: 210 GiB (225485783040 bytes)   #查看已增加5G
disk size: 1.48 GiB
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
    refcount bits: 16
    corrupt: false
```

### **5.4.6**、调整虚拟磁盘大小

虚拟磁盘文件创建后,还可以调整虚拟磁盘大小

语法格式

```
qemu-img resize [--shrink] filename [+l -]size



注：
操作之前，一定要做好数据备份
增加文件大小后，需要在客户机中使用fdisk、parted等分区工具进行相应的操作才能真正让客户机使用到增加后的镜像空间。
缩小镜像之前，要在客户机中保证里面的文件系统有空余空间，否则会数据丢失。另外xfs文件系统不支持缩减
qcow2不支持缩小镜像的操作
```

#### 范例: 扩展虚拟磁盘

```
#查看原磁盘大小及分区
[root@localhost ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0          11:0    1 1024M  0 rom  
vda         252:0    0   10G  0 disk 
?𼰿𺲀vda1      252:1    0    1G  0 part /boot
?𺰿𺲀vda2      252:2    0    9G  0 part 
  ?𼰿𺲀rl-root 253:0    0    8G  0 lvm  /
  ?𺰿𺲀rl-swap 253:1    0    1G  0 lvm  [SWAP]

#为镜像文件增加10G空间
[root@ubuntu2004 /var/lib/libvirt/images]#qemu-img resize Rocky8-template.qcow2 +10G

#远程ssh连接
[root@ubuntu2004 /var/lib/libvirt/images]#ssh 192.168.122.27
root@192.168.122.27's password: 
Last login: Thu Feb  8 17:21:18 2024 from 192.168.122.1

[root@localhost ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0          11:0    1 1024M  0 rom  
vda         252:0    0   20G  0 disk        #已扩展10G
?𼰿𺲀vda1      252:1    0    1G  0 part /boot
?𺰿𺲀vda2      252:2    0    9G  0 part 
  ?𼰿𺲀rl-root 253:0    0    8G  0 lvm  /
  ?𺰿𺲀rl-swap 253:1    0    1G  0 lvm  [SWAP]



范例：qemu-img resize 扩展空间后，在虚拟机内还需执行如下操作才能最终扩容
#添加新分区
[root@localhost ~]# fdisk /dev/vda

Welcome to fdisk (util-linux 2.32.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): p		#查看现有分区信息
Disk /dev/vda: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x4ae4e2c2

Device     Boot   Start      End  Sectors Size Id Type
/dev/vda1  *       2048  2099199  2097152   1G 83 Linux
/dev/vda2       2099200 20971519 18872320   9G 8e Linux LVM

Command (m for help): n
Partition type
   p   primary (2 primary, 0 extended, 2 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (3,4, default 3): 
First sector (20971520-41943039, default 20971520): 
Last sector, +sectors or +size{K,M,G,T,P} (20971520-41943039, default 41943039): 

Created a new partition 3 of type 'Linux' and of size 10 GiB.

Command (m for help): p
Disk /dev/vda: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x4ae4e2c2

Device     Boot    Start      End  Sectors Size Id Type
/dev/vda1  *        2048  2099199  2097152   1G 83 Linux
/dev/vda2        2099200 20971519 18872320   9G 8e Linux LVM
/dev/vda3       20971520 41943039 20971520  10G 83 Linux

Command (m for help): t
Partition number (1-3, default 3): 3
Hex code (type L to list all codes): 8e

Changed type of partition 'Linux' to 'Linux LVM'.

Command (m for help): p
Disk /dev/vda: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x4ae4e2c2

Device     Boot    Start      End  Sectors Size Id Type
/dev/vda1  *        2048  2099199  2097152   1G 83 Linux
/dev/vda2        2099200 20971519 18872320   9G 8e Linux LVM
/dev/vda3       20971520 41943039 20971520  10G 8e Linux LVM

Command (m for help): w
The partition table has been altered.
Syncing disks.

[root@localhost ~]# lsblk    #若没有显示新分区，则执行partprobe命令
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0          11:0    1 1024M  0 rom  
vda         252:0    0   20G  0 disk 
?𼰿𺲀vda1      252:1    0    1G  0 part /boot
?𼰿𺲀vda2      252:2    0    9G  0 part 
?𶐠?𼰿𺲀rl-root 253:0    0    8G  0 lvm  /
?𶐠?𺰿𺲀rl-swap 253:1    0    1G  0 lvm  [SWAP]
?𺰿𺲀vda3      252:3    0   10G  0 part 

#（选择执行）
[root@localhost ~]# partprobe  #若没有显示新分区，则执行partprobe命令
```

```
注意： pv→vg→lv
范例：qemu-img resize 扩展空间后，在虚拟机内还需执行如下操作才能最终扩容
#利用前面qemu-img resize扩容的空间来扩容分区
fdisk /dev/vda
d 删除旧的20G的/dev/vda2
n 重新创建/dev/vda2,容量使用所有空间，/dev/vda2扩容到30G
t 指定8e 类型
w 保存退出
#如果是逻辑卷，执行下面操作
#当前物理卷空间并没有扩容，需要执行下面操作扩容PV
pvresize  --setphysicalvolumesize 29G /dev/vda2
#扩容逻辑卷和文件系统
lvextend -r -l  +100%free /dev/rl/root
#如果是分区不是逻辑卷，则执行如下操作
#如果是xfs文件系统
xfs_growfs /
#如果是ext
resize2fs /dev/vda2
```

#扩容

```
#查看现有物理卷
[root@localhost ~]# pvs
  PV         VG Fmt  Attr PSize  PFree
  /dev/vda2  rl lvm2 a--  <9.00g    0 

#创建新物理卷
[root@localhost ~]# pvcreate /dev/vda3 
  Physical volume "/dev/vda3" successfully created.
[root@localhost ~]# pvs
  PV         VG Fmt  Attr PSize  PFree 
  /dev/vda2  rl lvm2 a--  <9.00g     0 
  /dev/vda3     lvm2 ---  10.00g 10.00g

#查看现有逻辑卷组
[root@localhost ~]# vgs
  VG #PV #LV #SN Attr   VSize  VFree
  rl   1   2   0 wz--n- <9.00g    0 

#扩展逻辑卷组
[root@localhost ~]# vgextend rl /dev/vda3 
  Volume group "rl" successfully extended
[root@localhost ~]# vgdisplay 
  --- Volume group ---
  VG Name               rl
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               18.99 GiB
  PE Size               4.00 MiB
  Total PE              4862
  Alloc PE / Size       2303 / <9.00 GiB
  Free  PE / Size       2559 / <10.00 GiB
  VG UUID               16gjdm-xXW4-X7lS-PwLt-8jTo-dbTe-nR883a

#查看当前逻辑卷
[root@localhost ~]# lvs
  LV   VG Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root rl -wi-ao---- <8.00g                                                    
  swap rl -wi-ao----  1.00g                                                    

#逻辑卷扩容并生效
[root@localhost ~]# lvextend -r -l +100%free /dev/rl/root 
  Size of logical volume rl/root changed from <8.00 GiB (2047 extents) to 17.99 GiB (4606 extents).
  Logical volume rl/root successfully resized.
meta-data=/dev/mapper/rl-root    isize=512    agcount=4, agsize=524032 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=0 inobtcount=0
data     =                       bsize=4096   blocks=2096128, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 2096128 to 4716544

[root@localhost ~]# lvs
  LV   VG Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root rl -wi-ao---- 17.99g                                                    
  swap rl -wi-ao----  1.00g                                                    

#查看磁盘大小信息
[root@localhost ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0          11:0    1 1024M  0 rom  
vda         252:0    0   20G  0 disk 
?𼰿𺲀vda1      252:1    0    1G  0 part /boot
?𼰿𺲀vda2      252:2    0    9G  0 part 
?𶐠?𼰿𺲀rl-root 253:0    0   18G  0 lvm  /     #已成功扩展
?𶐠?𺰿𺲀rl-swap 253:1    0    1G  0 lvm  [SWAP]
?𺰿𺲀vda3      252:3    0   10G  0 part 
  ?𺰿𺲀rl-root 253:0    0   18G  0 lvm  /
```



```
#载入分区
[root@ubuntu2004 ~]#partprobe
```

## 5.5、磁盘快照管理

### **5.5.1** **快照Snapshot介绍**

```
磁盘快照,对磁盘数据进行快照,主要用于虚拟机备份等场合
磁盘快照分类
按快照信息保存分为:
内置快照∶快照数据和base磁盘数据放在同一个qcow2文件中
外置快照︰快照数据单独的另一个qcow2文件存放
按虚拟机状态可以分为:
关机态快照︰数据可以保证一致性
运行态快照∶数据无法保证一致性，类似与系统crash后的磁盘数据。使用是可能需要fsck等操作。
按磁盘数量可以分为:
单盘:单盘快照不涉及原子性
多盘:涉及原子性。主要分两个方面:1.是所有盘快照点相同2.所有盘要么都快照成功，要么都快照失
败。主要依赖于qemu的transaction实现
```

### **5.5.2 qemu-img管理磁盘快照**

命令格式

```
qemu-img snapshot [--object objectdef] [--image-opts] [-U] [-q] [-l | -asnapshot | -c snapshot | -d snapshot] filename


snapshot is the name of the snapshot to create, apply or delete
 -a applies a snapshot (revert disk to saved state)
 -c creates a snapshot
 -d deletes a snapshot
 -l lists all snapshots in the given image
```

范例: 

```
#查看块设备
[root@ubuntu2004 ~]#virsh domblklist Rocky8-template 
 Target   Source
---------------------------------------------------------
 vda      /var/lib/libvirt/images/Rocky8-template.qcow2
 sda      -

#查看快照,如果没有快照,则无显示信息
[root@ubuntu2004 ~]#qemu-img snapshot -l /var/lib/libvirt/images/Rocky8-template.qcow2 
Snapshot list:
ID        TAG                     VM SIZE                DATE       VM CLOCK
1         1707385798              507 MiB 2024-02-08 09:49:58   00:25:50.806
[root@ubuntu2004 ~]#qemu-img snapshot -l /var/lib/libvirt/images/win2k12r2.qcow2 
[root@ubuntu2004 ~]#

#关机才能创建快照
[root@ubuntu2004 ~]#qemu-img snapshot -c rocky8S /var/lib/libvirt/images/Rocky8-template.qcow2
[root@ubuntu2004 ~]#qemu-img snapshot -l /var/lib/libvirt/images/Rocky8-template.qcow2 
Snapshot list:
ID        TAG                     VM SIZE                DATE       VM CLOCK
1         1707385798              507 MiB 2024-02-08 09:49:58   00:25:50.806
2         rocky8S                     0 B 2024-02-08 14:45:08   00:00:00.000


#查看快照信息
[root@ubuntu2004 ~]#qemu-img info /var/lib/libvirt/images/Rocky8-template.qcow2 
image: /var/lib/libvirt/images/Rocky8-template.qcow2
file format: qcow2
virtual size: 20 GiB (21474836480 bytes)
disk size: 2.99 GiB
cluster_size: 65536
Snapshot list:
ID        TAG                     VM SIZE                DATE       VM CLOCK
1         1707385798              507 MiB 2024-02-08 09:49:58   00:25:50.806
2         rocky8S                     0 B 2024-02-08 14:45:08   00:00:00.000
Format specific information:
    compat: 1.1
    lazy refcounts: true
    refcount bits: 16
    corrupt: false


#删除文件,模拟破坏
rm -rf /*

#关机后才能还原快照修复故障
[root@centos8 ~]#qemu-img snapshot -a centos7-s1 /var/lib/libvirt/images/centos7.qcow2

#关机后才能删除快照
[root@centos8 ~]#qemu-img snapshot -d centos7-s1 /var/lib/libvirt/images/centos7.qcow2
[root@centos8 ~]#qemu-img snapshot -l /var/lib/libvirt/images/centos7.qcow2
```

### **5.5.3 virsh管理虚拟机快照**



# 6、网络管理

官方文档:

```
https://wiki.libvirt.org/page/VirtualNetworking
```

## **6.1 Linux 网桥实现**

范例: Ubuntu 配置网桥

```
[root@ubuntu2004 ~]# apt install -y bridge-utils
[root@ubuntu2004 ~]#dpkg -L bridge-utils

#三个网卡配置使用一个配置文件
root@ubuntu1804:~# cat /etc/netplan/01-netcfg.yaml 
# This file describes the network interfaces available on your system
# For more information, see netplan(5).
network:
 version: 2
 renderer: networkd
 ethernets:
   eth0:
     dhcp4: yes
   eth1:
     dhcp4: no
     dhcp6: no
   eth2:
     dhcp4: no     
 bridges:
   virbr1:
     dhcp4: no
     dhcp6: no
     addresses: [10.0.0.18/16]
     gateway4: 10.0.0.2
     nameservers:
       addresses: [223.6.6.6]
     interfaces:
      - eth1
      - eth2
#桥接配置单独一个文件
[root@ubuntu1804 netplan]#cat virbr1.yaml
network:
 version: 2
 renderer: networkd
 ethernets:
   eth1:
     dhcp4: no
     dhcp6: no
   eth2:
     dhcp4: no     
 bridges:
   virbr1:
     dhcp4: no
     dhcp6: no
     addresses: [10.0.0.10/16]
     gateway4: 10.0.0.2
     nameservers:
       addresses: [223.6.6.6]
     interfaces:
      - eth1
      - eth2

root@ubuntu1804:~# netplan apply
root@ubuntu1804:~# ifconfig virbr1
virbr1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500
       inet 10.0.0.18 netmask 255.255.0.0 broadcast 10.0.255.255
       inet6 fe80::9cbe:1dff:fe85:6601 prefixlen 64 scopeid 0x20<link>
       ether 9e:be:1d:85:66:01 txqueuelen 1000 (Ethernet)
       RX packets 158 bytes 19534 (19.5 KB)
       RX errors 0 dropped 0 overruns 0 frame 0
       TX packets 10 bytes 796 (796.0 B)
       TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0
root@ubuntu1804:~# brctl show
bridge name bridge id STP enabled interfaces
virbr1 8000.9ebe1d856601 no eth1
   eth2
```

范例: CentOS 配置网桥

```
#创建网桥
nmcli con add type bridge con-name virbr1 ifname virbr1
nmcli connection modify virbr1 ipv4.addresses 10.0.0.100/24 ipv4.method manual
nmcli con up virbr1
#加入物理网卡
nmcli con add type bridge-slave con-name virbr1-port0 ifname eth0 master virbr1
nmcli con add type bridge-slave con-name virbr1-port1 ifname eth1 master virbr1
nmcli con up virbr1-port0
nmcli con up virbr1-port1
#查看网桥配置文件
cat /etc/sysconfig/network-scripts/ifcfg-virbr1
DEVICE=virbr1
NAME=virbr1
STP=yes
TYPE=Bridge
BOOTPROTO=static
IPADDR=10.0.0.100
PREFIX=24
cat /etc/sysconfig/network-scripts/ifcfg-virbr1-port0
TYPE=Ethernet
NAME=virbr1-port0
DEVICE=eth0
ONBOOT=yes
BRIDGE=virbr1
UUID=23f41d3b-b57c-4e26-9b17-d5f02dafd12d
#安装管理软件包,注意:CentOS8取消了此包
yum install bridge-utils
#查看网桥
brctl show
ip link show master virbr1
bridge link show
#删除virbr1
nmcli con down virbr1
rm /etc/sysconfig/network-scripts/ifcfg-virbr1*
nmcli con reload
```

## **6.2 qemu-kvm支持的网络**

```
虚拟机的网络模式:
基于NAT ( Network Addresss Translation)的虚拟网络,此为virt-install的默认模式，相当于Vmware中的NAT模式
基于自定义网桥（Bridge ）的虚拟网络，支持虚拟机和宿主机的网卡桥接在一个网桥，从而实现外部网络访问虚拟机
用户自定义的隔离的虚拟网络，相当于Vmware中的仅主机模式
直接分配物理网络设备（包括VT-d和SR-IOV)，即桥接到宿主机的网卡，类似于Vmware中的桥接模式,性能最好

虚拟机的网卡设备:
	RTL8139、e1000、....
	virtio 生产建议使用
```

注意: 桥接物理网卡,会自动生成macvtap网卡,并且会导致本宿主机无法访问虚拟机,但其它物理机可以访问当前宿主机上面的虚拟机

范例: 查看qemu-kvm支持的网卡型号

```
#Ubuntu20.04没有此命令
[root@centos8 ~]#/usr/libexec/qemu-kvm -net nic,model=?
qemu: Supported NIC models: e1000,e1000-82540em,e1000e,rtl8139,virtio-net-pci
```

## **6.3** **默认的网络配置NAT模式**

### **6.3.1** **默认网络连接的架构图**

默认虚拟机网络配置为NAT模式,相当于vmware的NAT模式的Vmnet8

![image-20240209095752267](KVM虚拟化/image-20240209095752267.png)

### **6.3.2** **宿主机默认网络相关服务和信息**

范例：修改默认的网段信息

```
#默认网面是192.168.122.0/24，可以修改为其它网段
[root@Rocky8 ~]#vim /etc/libvirt/qemu/networks/default.xml

<!--
WARNING: THIS IS AN AUTO-GENERATED FILE. CHANGES TO IT ARE LIKELY TO BE
OVERWRITTEN AND LOST. Changes to this xml configuration should be made using:
  virsh net-edit default
or other application using the libvirt API.
-->

<network>
  <name>default</name>
  <uuid>5bee326f-904b-44e7-b2f2-b9ebb6c5ca94</uuid>
  <forward mode='nat'/>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:4c:01:ee'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>
    </dhcp>
  </ip>
</network>

[root@Rocky8 ~]#reboot


#自动更新DNSMASQ的相关文件
[root@Rocky8 ~]#cat /var/lib/libvirt/dnsmasq/default.conf 
##WARNING:  THIS IS AN AUTO-GENERATED FILE. CHANGES TO IT ARE LIKELY TO BE
##OVERWRITTEN AND LOST.  Changes to this configuration should be made using:
##    virsh net-edit default
## or other application using the libvirt API.
##
## dnsmasq conf file created by libvirt
strict-order
pid-file=/run/libvirt/network/default.pid
except-interface=lo
bind-dynamic
interface=virbr0
dhcp-range=192.168.122.2,192.168.122.254,255.255.255.0
dhcp-no-override
dhcp-authoritative
dhcp-lease-max=253
dhcp-hostsfile=/var/lib/libvirt/dnsmasq/default.hostsfile
addn-hosts=/var/lib/libvirt/dnsmasq/default.addnhosts

```

默认宿主机安装dnsmasq包指供DHCP服务

```
[root@ubuntu2004 /var/lib/libvirt/qemu]#dpkg -l dnsmasq-base
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name           Version           Architecture Description
+++-==============-=================-============-============================================
ii  dnsmasq-base   2.80-1.1ubuntu1.7 amd64        Small caching DNS proxy and DHCP/TFTP server


[root@ubuntu2004 ~]#systemctl status libvirtd
?𹠠libvirtd.service - Virtualization daemon
     Loaded: loaded (/lib/systemd/system/libvirtd.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2024-02-09 02:13:53 UTC; 8s ago
TriggeredBy: ?𹠠libvirtd.socket
             ?𹠠libvirtd-admin.socket
             ?𹠠libvirtd-ro.socket
       Docs: man:libvirtd(8)
             https://libvirt.org
   Main PID: 30780 (libvirtd)
      Tasks: 19 (limit: 32768)
     Memory: 649.1M
     CGroup: /system.slice/libvirtd.service
             ?𼰿𺲀 1169 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/lib/libvirt/libvirt_leaseshelper
             ?𼰿𺲀 1171 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/lib/libvirt/libvirt_leaseshelper
             ?𺰿𺲀30780 /usr/sbin/libvirtd

Feb 09 02:13:53 ubuntu2004 systemd[1]: Starting Virtualization daemon...
Feb 09 02:13:53 ubuntu2004 systemd[1]: Started Virtualization daemon.
Feb 09 02:13:54 ubuntu2004 dnsmasq[1169]: read /etc/hosts - 7 addresses
Feb 09 02:13:54 ubuntu2004 dnsmasq[1169]: read /var/lib/libvirt/dnsmasq/default.addnhosts - 0 addresses
Feb 09 02:13:54 ubuntu2004 dnsmasq-dhcp[1169]: read /var/lib/libvirt/dnsmasq/default.hostsfile

[root@ubuntu2004 ~]#cat /var/lib/libvirt/dnsmasq/default.conf
##WARNING:  THIS IS AN AUTO-GENERATED FILE. CHANGES TO IT ARE LIKELY TO BE
##OVERWRITTEN AND LOST.  Changes to this configuration should be made using:
##    virsh net-edit default
## or other application using the libvirt API.
##
## dnsmasq conf file created by libvirt
strict-order
user=libvirt-dnsmasq
pid-file=/run/libvirt/network/default.pid
except-interface=lo
bind-dynamic
interface=virbr0
dhcp-range=192.168.122.2,192.168.122.254,255.255.255.0
dhcp-no-override
dhcp-authoritative
dhcp-lease-max=253
dhcp-hostsfile=/var/lib/libvirt/dnsmasq/default.hostsfile
addn-hosts=/var/lib/libvirt/dnsmasq/default.addnhosts


#CentOS
[root@Rocky8 ~]#rpm -q dnsmasq 
dnsmasq-2.79-19.el8.x86_64


[root@Rocky8 ~]#cat /var/lib/libvirt/dnsmasq/default.conf 
##WARNING:  THIS IS AN AUTO-GENERATED FILE. CHANGES TO IT ARE LIKELY TO BE
##OVERWRITTEN AND LOST.  Changes to this configuration should be made using:
##    virsh net-edit default
## or other application using the libvirt API.
##
## dnsmasq conf file created by libvirt
strict-order
pid-file=/run/libvirt/network/default.pid
except-interface=lo
bind-dynamic
interface=virbr0
dhcp-range=192.168.122.2,192.168.122.254,255.255.255.0
dhcp-no-override
dhcp-authoritative
dhcp-lease-max=253
dhcp-hostsfile=/var/lib/libvirt/dnsmasq/default.hostsfile
addn-hosts=/var/lib/libvirt/dnsmasq/default.addnhosts

```

范例：查看宿主机的网桥信息

```
[root@ubuntu2004 ~]#virsh list
 Id   Name              State
---------------------------------
 18   Rocky8-template   running

[root@ubuntu2004 ~]#ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:55:69:96 brd ff:ff:ff:ff:ff:ff
    inet 11.0.1.61/24 brd 11.0.1.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe55:6996/64 scope link 
       valid_lft forever preferred_lft forever
3: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 52:54:00:c4:f1:47 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
4: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr0 state DOWN group default qlen 1000
    link/ether 52:54:00:c4:f1:47 brd ff:ff:ff:ff:ff:ff
26: virbr1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether fe:54:00:c2:ad:ca brd ff:ff:ff:ff:ff:ff
    inet6 fe80::885d:26ff:fe2d:ee90/64 scope link 
       valid_lft forever preferred_lft forever
32: vnet0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master virbr1 state UNKNOWN group default qlen 1000
    link/ether fe:54:00:c2:ad:ca brd ff:ff:ff:ff:ff:ff
    inet6 fe80::fc54:ff:fec2:adca/64 scope link 
       valid_lft forever preferred_lft forever

#查看网桥信息
[root@ubuntu2004 ~]#brctl show
bridge name	bridge id		STP enabled	interfaces
virbr0		8000.525400c4f147	yes		virbr0-nic
virbr1		8000.fe5400c2adca	no		vnet0

#查看virbr1网络的情况

[root@ubuntu2004 ~]#virt-manager 

#查看虚拟机列表
[root@ubuntu2004 ~]#virsh  list
 Id   Name              State
---------------------------------
 18   Rocky8-template   running
 19   rocky8-1          running
 20   rocky8-2          running
    
[root@ubuntu2004 ~]#brctl show
bridge name	bridge id		STP enabled	interfaces
virbr0		8000.525400c4f147	yes		virbr0-nic
virbr1		8000.fe540010bc4b	no		vnet0
							vnet1
							vnet2

#查看virbr1网络的情况
[root@ubuntu2004 ~]#ip link show master virbr1
32: vnet0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master virbr1 state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether fe:54:00:c2:ad:ca brd ff:ff:ff:ff:ff:ff
34: vnet1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master virbr1 state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether fe:54:00:10:bc:4b brd ff:ff:ff:ff:ff:ff
35: vnet2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master virbr1 state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether fe:54:00:95:59:b1 brd ff:ff:ff:ff:ff:ff


#查看所有桥接网卡信息及对应网桥
[root@ubuntu2004 ~]#bridge link show
4: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 master virbr0 state disabled priority 32 cost 100 
32: vnet0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master virbr1 state forwarding priority 32 cost 100 
34: vnet1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master virbr1 state forwarding priority 32 cost 100 
35: vnet2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master virbr1 state forwarding priority 32 cost 100


[root@ubuntu2004 ~]#virsh net-dumpxml default
<network>
  <name>default</name>
  <uuid>6e73feaa-432c-4025-8b8e-6a5adeb51313</uuid>
  <forward mode='nat'>
    <nat>
      <port start='1024' end='65535'/>
    </nat>
  </forward>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:c4:f1:47'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>
    </dhcp>
  </ip>
</network>

[root@ubuntu2004 ~]#cat /etc/libvirt/qemu/networks/default.xml
<!--
WARNING: THIS IS AN AUTO-GENERATED FILE. CHANGES TO IT ARE LIKELY TO BE
OVERWRITTEN AND LOST. Changes to this xml configuration should be made using:
  virsh net-edit default
or other application using the libvirt API.
-->

<network>
  <name>default</name>
  <uuid>6e73feaa-432c-4025-8b8e-6a5adeb51313</uuid>
  <forward mode='nat'/>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:c4:f1:47'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>
    </dhcp>
  </ip>
</network>


#查看NAT策略实现从虚拟机可以访问外部网络,反之不通,此策略由libvirtd服务启动时自动加载到NAT表
[root@ubuntu2004 ~]#iptables -nvL -t nat
Chain PREROUTING (policy ACCEPT 29 packets, 3886 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain INPUT (policy ACCEPT 2 packets, 488 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 3 packets, 196 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain POSTROUTING (policy ACCEPT 3 packets, 196 bytes)
 pkts bytes target     prot opt in     out     source               destination         
  493 42270 LIBVIRT_PRT  all  --  *      *       0.0.0.0/0            0.0.0.0/0           

Chain LIBVIRT_PRT (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 RETURN     all  --  *      *       192.168.122.0/24     224.0.0.0/24        
    0     0 RETURN     all  --  *      *       192.168.122.0/24     255.255.255.255     
    0     0 MASQUERADE  tcp  --  *      *       192.168.122.0/24    !192.168.122.0/24     masq ports: 1024-65535
    0     0 MASQUERADE  udp  --  *      *       192.168.122.0/24    !192.168.122.0/24     masq ports: 1024-65535
    0     0 MASQUERADE  all  --  *      *       192.168.122.0/24    !192.168.122.0/24    

```

### **6.3.3** **虚拟机网卡默认设置**

Ubutun20.04的虚拟机默认网卡设置

![image-20240209110630177](KVM虚拟化/image-20240209110630177.png)

Rocky8的虚拟机默认网卡设置

![image-20240209110651963](KVM虚拟化/image-20240209110651963.png)

![image-20240209110659801](KVM虚拟化/image-20240209110659801.png)

![image-20240209110719067](KVM虚拟化/image-20240209110719067.png)

![image-20240209110724562](KVM虚拟化/image-20240209110724562.png)

![image-20240209110736114](KVM虚拟化/image-20240209110736114.png)

```
[root@centos8 ~]#virsh dumpxml centos7 |sed -n '/interface/,/interface/p' 
   <interface type='network'>
     <mac address='52:54:00:95:25:fb'/>
     <source network='default' bridge='virbr1'/>
     <target dev='vnet0'/>
     <model type='virtio'/>
     <alias name='net0'/>
     <address type='pci' domain='0x0000' bus='0x00' slot='0x03'
function='0x0'/>
   </interface>
```

### **6.3.4 virsh查看虚拟机网络配置**

相当于vmware的桥接模式的Vmnet0

此方式让宿主机网络的主机可以直接访问虚拟机,性能更优

**Ubuntu22.04** **宿主机的虚拟机界面如下**

![image-20240209111417070](KVM虚拟化/image-20240209111417070.png)

**Ubuntu20.04** **宿主机的虚拟机界面如下**

选择Network source 为 Host deivce eth0: macvtap

![image-20240209111435312](KVM虚拟化/image-20240209111435312.png)

**Rocky8** **宿主机的虚拟机界面如下**

需要手动输入device name 为 eth0

![image-20240209111449208](KVM虚拟化/image-20240209111449208.png)

**CentOS 7** **界面如下**

![image-20240209111505266](KVM虚拟化/image-20240209111505266.png)

![image-20240209111511706](KVM虚拟化/image-20240209111511706.png)

```
[root@centos8 ~]#virsh domiflist centos7
Interface Type       Source     Model       MAC
-------------------------------------------------------
macvtap0   direct     eth0       virtio      52:54:00:95:25:fb
[root@centos8 ~]#virsh domifaddr centos7
 Name       MAC address         Protocol     Address
-------------------------------------------------------------------------------
```

![image-20240209111529908](KVM虚拟化/image-20240209111529908.png)

在宿主机上自动生成虚拟网卡macvtap0@eth0

```
[root@centos8 ~]#ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group 
default qlen 1000
   link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
   inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
   inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP 
group default qlen 1000
   link/ether 00:0c:29:e1:0e:53 brd ff:ff:ff:ff:ff:ff
   inet 10.0.0.8/24 brd 10.0.0.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
   inet6 fe80::20c:29ff:fee1:e53/64 scope link 
       valid_lft forever preferred_lft forever
3: virbr1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN 
group default qlen 1000
   link/ether 52:54:00:8b:be:ee brd ff:ff:ff:ff:ff:ff
   inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr1
       valid_lft forever preferred_lft forever
4: virbr1-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr1 state 
DOWN group default qlen 1000
   link/ether 52:54:00:8b:be:ee brd ff:ff:ff:ff:ff:ff
6: macvtap0@eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel 
state UP group default qlen 500
   link/ether 52:54:00:0d:8a:0e brd ff:ff:ff:ff:ff:ff
   inet6 fe80::5054:ff:fe0d:8a0e/64 scope link 
       valid_lft forever preferred_lft forever
```

## **6.5** **基于自定义网桥的虚拟网络**

### **6.5.1** **自定义网桥架构**

桥接网络可以让运行在宿主机上的虚拟机使用和宿主机相同网段的IP，并且可以从外部直接访问到虚拟机，目前企业中大部分场景都使用桥接网络。

![image-20240209111614905](KVM虚拟化/image-20240209111614905.png)

### **6.5.2** **在宿主机创建网桥**

#### **6.5.2.1 CentOS** **创建桥接网卡**

```
[root@centos8 network-scripts]#pwd
/etc/sysconfig/network-scripts
#创建网桥配置文件
[root@centos8 network-scripts]#cat ifcfg-virbr1 
TYPE=Bridge
NAME=virbr1
DEVICE=virbr1
ONBOOT=yes
BOOTPROTO=static
IPADDR=10.0.0.8
NETMASK=255.255.255.0
GATEWAY=10.0.0.2
DNS1=180.76.76.76
DNS2=223.6.6.6
#将物理网卡eth0加入网桥
[root@centos8 network-scripts]#cat ifcfg-eth0 
TYPE=Ethernet
NAME=eth0
DEVICE=eth0
ONBOOT=yes
BRIDGE=virbr1
[root@centos8 network-scripts]#nmcli connection reload
[root@centos8 network-scripts]#nmcli connection up eth0 virbr1
      
[root@centos8 ~]#ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group 
default qlen 1000
   link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
   inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
   inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master virbr1 
state UP group default qlen 1000
   link/ether 00:0c:29:44:c3:fe brd ff:ff:ff:ff:ff:ff
4: virbr1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN 
group default qlen 1000
   link/ether 52:54:00:52:f2:5c brd ff:ff:ff:ff:ff:ff
   inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr1
       valid_lft forever preferred_lft forever
5: virbr1-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr1 state 
DOWN group default qlen 1000
   link/ether 52:54:00:52:f2:5c brd ff:ff:ff:ff:ff:ff
18: virbr1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP 
group default qlen 1000
   link/ether 00:0c:29:44:c3:fe brd ff:ff:ff:ff:ff:ff
   inet 10.0.0.8/24 brd 10.0.0.255 scope global noprefixroute virbr1
       valid_lft forever preferred_lft forever
   inet6 fe80::8055:c0ff:fe4e:411e/64 scope link 
       valid_lft forever preferred_lft forever
[root@centos8 ~]#bridge link show
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master virbr1 state 
forwarding priority 32 cost 100
5: virbr1-nic: <BROADCAST,MULTICAST> mtu 1500 master virbr1 state disabled 
priority 32 cost 100
[root@centos8 ~]#ip link show master virbr1
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master virbr1 
state UP mode DEFAULT group default qlen 1000
   link/ether 00:0c:29:44:c3:fe brd ff:ff:ff:ff:ff:ff
```

#### 6.5.2.2 Ubuntu创建桥接网卡

```
[root@ubuntu2004 ~]# cat /etc/netplan/01-netcfg.yaml
# This file describes the network interfaces available on your system
# For more information, see netplan(5).
network:
 version: 2
 renderer: networkd
 ethernets:
   eth0:
     addresses: [10.0.0.100/16]
     gateway4: 10.0.0.2
     nameservers:
       addresses: [223.6.6.6]
     eth1:
     dhcp4: no
     dhcp6: no
 bridges:
   virbr1:
     dhcp4: no
     dhcp6: no
      #addresses: [10.0.0.100/16]
      #gateway4: 10.0.0.2
      #nameservers:
      # addresses: [223.6.6.6]
     interfaces:
     - eth1
```

### **6.5.3** **基于现有虚拟机镜像批量创建新虚拟机**

将前面生成的虚拟机做为模版,生成新的虚拟机

**注意:先在前面的模版虚拟机修改配置，如：关闭irewalld和SELinux,禁用NetworkManager(CentOS 7)等,安装常用包等**

```
[root@centos8 images]#pwd
/var/lib/libvirt/images
[root@centos8 images]#cp centos8.qcow2 centos8-2.qcow2
[root@centos8 images]#virt-install --virt-type kvm --os-variant centos8 --name 
centos8-2 --ram 2048 --vcpus 2 --disk 
bus=virtio,path=/var/lib/libvirt/images/centos8-2.qcow2 --network 
bridge=virbr1,model=virtio --graphics vnc,listen=0.0.0.0 --noautoconsole --
autostart --boot hd
WARNING No operating system detected, VM performance may suffer. Specify an OS 
with --os-variant for optimal results.
Starting install...
Domain creation completed.
#运行工具,可以看到下面出现新的虚拟机
[root@centos8 images]#virt-manager 
#开启的虚拟机磁盘文件所有者为qemu,关闭后为root
[root@centos8 ~]#ll /var/lib/libvirt/images
total 9977352
-rw-r--r-- 1 qemu qemu 2029518848 Sep 21 23:37 centos8-2.qcow2
-rw------- 1 root root 1929183232 Sep 21 23:33 centos8-clone.qcow2
-rw-r--r-- 1 root root 2004221952 Sep 21 23:32 centos8.qcow2
```

![image-20240209111901778](KVM虚拟化/image-20240209111901778.png)



### **6.5.4** **查看新虚拟机设置**

![image-20240209111915717](KVM虚拟化/image-20240209111915717.png)

![image-20240209111925711](KVM虚拟化/image-20240209111925711.png)

![image-20240209111932056](KVM虚拟化/image-20240209111932056.png)

![image-20240209111936654](KVM虚拟化/image-20240209111936654.png)

## 6.6、范例：内外网综合隔离实验

```shell
#命令行创建网桥
[root@ubuntu2004 ~]#brctl addbr virbr1

#up网桥
[root@ubuntu2004 ~]#ip link set virbr1 up

#查看网桥信息
[root@ubuntu2004 ~]#brctl show
bridge name	bridge id		STP enabled	interfaces
virbr0		8000.525400c4f147	yes		virbr0-nic
virbr1		8000.fe540010bc4b	no		vnet1
							            vnet2
```

### 6.6.1、rocky8-1（master）网络配置

![image-20240209095024526](KVM虚拟化/image-20240209095024526.png)

### 6.6.2、rocky8-2（slave）网络配置

![image-20240209095118758](KVM虚拟化/image-20240209095118758.png)

### 6.6.3、网络测试互ping，网桥virbr1可以互通

![image-20240209095253353](KVM虚拟化/image-20240209095253353.png)

```
[root@ubuntu2004 ~]#brctl addbr virbr1
[root@ubuntu2004 ~]#ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:55:69:96 brd ff:ff:ff:ff:ff:ff
    inet 11.0.1.61/24 brd 11.0.1.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe55:6996/64 scope link 
       valid_lft forever preferred_lft forever
3: virbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 52:54:00:c4:f1:47 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
4: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr0 state DOWN group default qlen 1000
    link/ether 52:54:00:c4:f1:47 brd ff:ff:ff:ff:ff:ff
13: vnet0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master virbr0 state UNKNOWN group default qlen 1000
    link/ether fe:54:00:9f:a8:ef brd ff:ff:ff:ff:ff:ff
    inet6 fe80::fc54:ff:fe9f:a8ef/64 scope link 
       valid_lft forever preferred_lft forever
14: macvtap0@eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 500
    link/ether 52:54:00:da:1d:83 brd ff:ff:ff:ff:ff:ff
    inet6 fe80::5054:ff:feda:1d83/64 scope link 
       valid_lft forever preferred_lft forever
15: virbr1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether de:ea:f0:1c:88:b6 brd ff:ff:ff:ff:ff:ff

#开启网桥
[root@ubuntu2004 ~]#ip link set virbr1 up
[root@ubuntu2004 ~]#ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:55:69:96 brd ff:ff:ff:ff:ff:ff
    inet 11.0.1.61/24 brd 11.0.1.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe55:6996/64 scope link 
       valid_lft forever preferred_lft forever
3: virbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 52:54:00:c4:f1:47 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
4: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr0 state DOWN group default qlen 1000
    link/ether 52:54:00:c4:f1:47 brd ff:ff:ff:ff:ff:ff
13: vnet0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master virbr0 state UNKNOWN group default qlen 1000
    link/ether fe:54:00:9f:a8:ef brd ff:ff:ff:ff:ff:ff
    inet6 fe80::fc54:ff:fe9f:a8ef/64 scope link 
       valid_lft forever preferred_lft forever
14: macvtap0@eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 500
    link/ether 52:54:00:da:1d:83 brd ff:ff:ff:ff:ff:ff
    inet6 fe80::5054:ff:feda:1d83/64 scope link 
       valid_lft forever preferred_lft forever
15: virbr1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether de:ea:f0:1c:88:b6 brd ff:ff:ff:ff:ff:ff
    inet6 fe80::dcea:f0ff:fe1c:88b6/64 scope link 
       valid_lft forever preferred_lft forever

```

# 面试：

如何实现批量创建虚拟机？





如何实现虚拟机迁移？

1、xml和qcow2文件

2、clone

3、cp和virt-install





# #windos系统一键安装

```
virt-install \
--virt-type=kvm \
--name win2012r2 \
--ram 4096 \
--vcpus=4 \
--os-variant=win2k8r2 \
--cdrom=/data/isos/cn_windows_server_2012_r2_x64_dvd_2707961.iso \
--network=bridge=virbr0,model=virtio \
--graphics vnc,listen=0.0.0.0 --noautoconsole \
--disk path=/var/lib/libvirt/images/win2k12r2.qcow2,size=20,bus=virtio,format=qcow2 \
--disk path=/data/isos/virtio-win-0.1.189.iso,device=cdrom
```

# #Linux系统一键安装

```
virt-install \
--virt-type kvm \
--os-variant=centos8 \
--name Rocky8-template --ram 1024 \
--vcpus 2 \
--cdrom=/data/isos/Rocky-8.9-x86_64-minimal.iso \
--disk path=/var/lib/libvirt/images/Rocky8-template.qcow2,size=10,format=qcow2,bus=virtio \
--network network=default \
--graphics vnc,listen=0.0.0.0 --noautoconsole
```

# #一键clone

```
[root@ubuntu2004 ~]#virt-clone -o Rocky8-template -n rocky8-1 -f /var/lib/libvirt/images/Rocky8-1.qcow2
```

# #需要提前准备qcow2文件

```
virt-install --virt-type kvm --os-variant=centos8 --name centos8-1 --ram 1024 --vcpu 1 --disk bus=virtio,path=/var/lib/libvirt/images/centos8-1.qcow2 --network network=default,model=virtio --graphics vnc,listen=0.0.0.0 --noautoconsole --boot hd
```

