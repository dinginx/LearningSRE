# Ansible自动化

**批量管理远程主机**

# 1、**Ansible** **介绍和架构**

```
公司计划在年底做一次大型市场促销活动，全面冲刺下交易额，为明年的上市做准备。公司要求各业务组对年底大促做准备，运维部要求所有业务容量进行三倍的扩容，并搭建出多套环境可以供开发和测试人员使用，运维老大为了在年底有所表现，要求运维部门同学尽快实现，当你接到这个任务时，有没有更快的解决方案？
```

## 1.1、Ansible 发展史

```
作者：Michael DeHaan（ Cobbler 与 Func 作者）
Ansible 的名称来自科幻小说《安德的游戏》中跨越时空的即时通信工具，使用它可以在相距数光年的距离，远程实时控制前线的舰队战斗
2012-03-09，发布0.0.1版，2015-10-17，Red Hat宣布1.5亿美元收购

官网：https://www.ansible.com/
官方文档：https://docs.ansible.com/
```

![image-20231211172921759](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231211172921759.png)

## 1.2、

# 2、**Ansible** **安装和常见模块**

## 2.1、ansible的安装

ansible的安装方法有多种

官方文档

```
https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html
https://docs.ansible.com/ansible/latest/installation_guide/index.html
```

下载

```
https://releases.ansible.com/ansible/
```

pip 下载

```
https://pypi.org/project/ansible/
```

### 2.1.1、包安装方法

```
#CentOS 的EPEL源的rpm包安装
[root@centos ~]#yum install ansible
#ubuntu 安装
[root@ubuntu ~]#apt -y install ansible
```

范例：查看ansible版本

```
[root@Ansible ~]#apt info ansible
Package: ansible
Version: 2.9.6+dfsg-1
Priority: optional
Section: universe/admin
Origin: Ubuntu
Maintainer: Ubuntu Developers <ubuntu-devel-discuss@lists.ubuntu.com>
Original-Maintainer: Harlan Lieberman-Berg <hlieberman@debian.org>
Bugs: https://bugs.launchpad.net/ubuntu/+filebug
Installed-Size: 58.0 MB
Depends: python3-cryptography, python3-jinja2, python3-yaml, python3:any, openssh-client | python3-paramiko, python3-crypto, python3-distutils, python3-dnspython, python3-httplib2, python3-netaddr
Recommends: python3-argcomplete, python3-jmespath, python3-kerberos, python3-libcloud, python3-selinux, python3-winrm, python3-xmltodict
Suggests: cowsay, sshpass
Homepage: https://www.ansible.com
Download-Size: 5,794 kB
APT-Manual-Installed: yes
APT-Sources: http://mirrors.aliyun.com/ubuntu focal/universe amd64 Packages
Description: Configuration management, deployment, and task execution system
 Ansible is a radically simple model-driven configuration management,
 multi-node deployment, and remote task execution system. Ansible works
 over SSH and does not require any software or daemons to be installed
 on remote nodes. Extension modules can be written in any language and
 are transferred to managed machines automatically.

#Rocky 8版本
[root@Rocky8 ~]#yum info ansible

Last metadata expiration check: 3:51:26 ago on Mon 11 Dec 2023 05:48:31 PM CST.
Available Packages
Name         : ansible
Version      : 8.3.0
Release      : 1.el8
Architecture : noarch
Size         : 41 M
Source       : ansible-8.3.0-1.el8.src.rpm
Repository   : epel
Summary      : Curated set of Ansible collections included in addition to ansible-core
URL          : https://ansible.com
License      : GPL-3.0-or-later AND Apache-2.0 AND BSD-2-Clause AND BSD-3-Clause AND MIT AND MPL-2.0 AND PSF-2.0
Description  : Ansible is a radically simple model-driven configuration management,
             : multi-node deployment, and remote task execution system. Ansible works
             : over SSH and does not require any software or daemons to be installed
             : on remote nodes. Extension modules can be written in any language and
             : are transferred to managed machines automatically.
             : 
             : This package provides a curated set of Ansible collections included in addition
             : to ansible-core.

```

### 2.1.2、范例: Ubuntu20.04 安装ansible

```
[root@Ansible ~]#apt update
[root@Ansible ~]#apt -y install ansible
[root@Ansible ~]#ansible --version
ansible 2.9.6
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.8.10 (default, Nov 22 2023, 10:22:35) [GCC 9.4.0]

[root@Ansible ~]#dpkg -L ansible |head
/.
/etc
/etc/ansible
/etc/ansible/ansible.cfg
/etc/ansible/hosts
/usr
/usr/bin
/usr/lib
/usr/lib/python3
/usr/lib/python3/dist-packages
```

### 2.1.3、范例：ubuntu18.04安装最新版的ansible

```
[root@ubuntu1804 ~]#apt update
[root@ubuntu1804 ~]#apt install software-properties-common
[root@ubuntu1804 ~]#apt-add-repository --yes --update ppa:ansible/ansible
[root@ubuntu1804 ~]#apt install ansible
[root@ubuntu1804 ~]# ansible --version
ansible 2.5.1
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.17 (default, Mar  8 2023, 18:40:28) [GCC 7.5.0]
```

### 2.1.4、范例: 在rocky8上通过pip3安装ansible

```
 pip 是安装Python包的管理器，类似 yum
 
[root@Rocky8 ~]# yum -y install python38 python38-pip
[root@Rocky8 ~]# pip3 install --upgrade pip -i https://pypi.douban.com/simple
[root@Rocky8 ~]# pip3 install ansible -i https://pypi.douban.com/simple/
[root@Rocky8 ~]#ansible --version
[DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.6.8 (default, Nov  8 2022, 11:32:15)
 [GCC 8.5.0 20210514 (Red Hat 8.5.0-15)]. This feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by setting 
deprecation_warnings=False in ansible.cfg.
/usr/local/lib/python3.6/site-packages/ansible/parsing/vault/__init__.py:44: CryptographyDeprecationWarning: Python 3.6 is no longer supported by the Python core team. Therefore, support for it is deprecated in cryptography. The next release of cryptography will remove support for Python 3.6.
  from cryptography.exceptions import InvalidSignature
ansible [core 2.11.12] 
  config file = None
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.6/site-packages/ansible
  ansible collection location = /root/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible
  python version = 3.6.8 (default, Nov  8 2022, 11:32:15) [GCC 8.5.0 20210514 (Red Hat 8.5.0-15)]
  jinja version = 3.0.3
  libyaml = True


注：范例: 安装默认的python3.6版本会有警报提示

[root@Rocky8 ~]#ansible-doc -l 2> /dev/null|wc -l
6141
```

### 2.1.5、范例: 在centos7上通过pip安装ansible

```
[root@centos7 ~]#yum -y install python-pip 
[root@centos7 ~]#pip install --upgrade pip
[root@centos7 ~]#pip install ansible --upgrade
[root@centos7 ~]#ansible --version
/usr/lib64/python2.7/site-packages/cryptography/__init__.py:39: 
CryptographyDeprecationWarning: Python 2 is no longer supported by the Python 
core team. Support for it is now deprecated in cryptography, and will be removed 
in a future release.
 CryptographyDeprecationWarning,
ansible 2.9.12
 config file = None
 configured module search path = [u'/root/.ansible/plugins/modules', 
u'/usr/share/ansible/plugins/modules']
 ansible python module location = /usr/lib/python2.7/site-packages/ansible
 executable location = /usr/bin/ansible
 python version = 2.7.5 (default, Apr  2 2020, 13:16:51) [GCC 4.8.5 20150623
(Red Hat 4.8.5-39)]
[root@centos7 ~]#ll /opt/etc/ansible/ansible.cfg 
-rw-r--r-- 1 wang bin 19980 Aug 11 21:34 /opt/etc/ansible/ansible.cfg
```

## 2.2、ansible相关文件

### 2.2.1、Ansible配置文件列表

```
/etc/ansible/ansible.cfg 主配置文件，配置ansible工作特性,也可以在项目的目录中创建此文件,当前目录下如果也有ansible.cfg,则此文件优先生效,建议每个项目目录下,创建独有的ansible.cfg文件
/etc/ansible/hosts 主机清单
/etc/ansible/roles/ 存放角色的目录
```

### 2.2.2、 Ansible主配置文件

Ansible 的配置文件可以放在多个不同地方,优先级从高到低顺ANSIBLE_CONFIG #环境变量,目录下的文件必须存在才能生效

```
ANSIBLE_CONFIG #环境变量,目录下的文件必须存在才能生效
./ansible.cfg   #当前目录下的ansible.cfg,一般一个项目对应一个专用配置文件,推荐使用
~/.ansible.cfg #当前用户家目录下的.ansible.cfg
/etc/ansible/ansible.cfg  #系统默认配置文件


范例：
[root@Ansible ~]#ansible --version
ansible 2.9.6
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.8.10 (default, Nov 22 2023, 10:22:35) [GCC 9.4.0]
[root@Ansible ~]#touch .ansible.cfg
[root@Ansible ~]#ansible --version
ansible 2.9.6
  config file = /root/.ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.8.10 (default, Nov 22 2023, 10:22:35) [GCC 9.4.0]
[root@Ansible ~]#cd /data/
[root@Ansible /data]#touch .ansible.cfg

```

Ansible 的默认配置文件 /etc/ansible/ansible.cfg ,其中大部分的配置内容无需进行修改

```
[defaults]
#inventory     = /etc/ansible/hosts #主机列表配置文件
#library = /usr/share/my_modules/ #库文件存放目录
#remote_tmp = $HOME/.ansible/tmp #临时py命令文件存放在远程主机目录
#local_tmp     = $HOME/.ansible/tmp #本机的临时命令执行目录
#forks         = 5   #默认并发数
#sudo_user     = root #默认sudo 用户
#ask_sudo_pass = True #每次执行ansible命令是否询问ssh密码
#ask_pass     = True   
#remote_port   = 22
#host_key_checking = False     #检查对应服务器的host_key，建议取消此行注释,实现第一次连接自动信任目标主机
#log_path=/var/log/ansible.log #日志文件，建议启用
#module_name = command   #默认模块，可以修改为shell模块
[privilege_escalation] #普通用户提权配置
#become=True
#become_method=sudo
#become_user=root
#become_ask_pass=False
```

范例: 通过环境变量ANSIBLE_CONFIG指定ansible配置文件路径

```
[root@Ansible ~]# mkdir /opt/ansible/ ;export ANSIBLE_CONFIG=/opt/ansible
[root@Ansible ~]# export ANSIBLE_CONFIG=/opt/ansible/
[root@Ansible ~]# echo $ANSIBLE_CONFIG
/opt/ansible/
[root@Ansible ~]# ansible --version
ansible 2.9.6
  config file = /opt/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.8.10 (default, Nov 22 2023, 10:22:35) [GCC 9.4.0]


#删除变量后恢复
[root@Ansible ~]# unset ANSIBLE_CONFIG
[root@Ansible ~]# ansible --version
ansible 2.9.6
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.8.10 (default, Nov 22 2023, 10:22:35) [GCC 9.4.0]
```

范例: 当前目录下的ansible的配置文件优先生效

```
[root@Ansible ~]# cp /etc/ansible/ansible.cfg /data/ansible/
[root@Ansible ~]# ansible --version
ansible 2.9.6
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.8.10 (default, Nov 22 2023, 10:22:35) [GCC 9.4.0]
#切换目录
[root@Ansible ~]# cd /data/ansible/
[root@Ansible /data/ansible]# ansible --version
ansible 2.9.6
  config file = /data/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.8.10 (default, Nov 22 2023, 10:22:35) [GCC 9.4.0]
```







```
192.168.188.100   ansible_connection=ssh  ansible_ssh_port=22  ansible_ssh_user=root ansible_ssh_password=123456   
```



```
[root@Ansible ~]#ls /etc/ansible/ -l
total 24
-rw-r--r-- 1 root root 19985 Mar  5  2020 ansible.cfg		#配置文件
-rw-r--r-- 1 root root  1104 Dec 11 09:58 hosts				#主机清单

[root@Ansible ~]#ansible --version  #查看版本
ansible 2.9.6
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.8.10 (default, Nov 22 2023, 10:22:35) [GCC 9.4.0]

#主要命令
[root@Ansible ~]#ansible
ansible             ansible-connection  ansible-doc         ansible-inventory   ansible-pull        
ansible-config      ansible-console     ansible-galaxy      ansible-playbook    ansible-vault  
```



## 2.3、Ansible相关工具

```
/usr/bin/ansible 主程序，临时命令执行工具
/usr/bin/ansible-doc 查看配置文档，模块功能查看工具,相当于man 
/usr/bin/ansible-playbook 定制自动化任务，编排剧本工具,相当于脚本
/usr/bin/ansible-pull 远程执行命令的工具
/usr/bin/ansible-vault 文件加密工具
/usr/bin/ansible-console 基于Console界面与用户交互的执行工具
/usr/bin/ansible-galaxy 下载/上传优秀代码或Roles模块的官网平台
```

### 2.3.1、利用ansible实现管理的主要方式：

```
Ansible Ad-Hoc 即利用ansible命令，主要用于临时命令使用场景
Ansible playbook 主要用于长期规划好的，大型项目的场景，需要有前期的规划过程
```

**ansible** **使用前准备**

```
ansible 相关工具大多数是通过ssh协议，实现对远程主机的配置管理、应用部署、任务执行等功能
建议：使用此工具前，先配置ansible主控端能基于密钥认证的方式联系各个被管理节点
```

#### 2.3.1.1、范例：利用sshpass批量实现基于key验证脚本1

```
[root@centos8 ~]#vim /etc/ssh/ssh_config
#修改下面一行
StrictHostKeyChecking no
[root@centos8 ~]#cat hosts.list
10.0.0.18
10.0.0.28
[root@centos8 ~]#vim push_ssh_key.sh
#!/bin/bash
rpm -q sshpass &> /dev/null || yum -y install sshpass  
[ -f /root/.ssh/id_rsa ] || ssh-keygen -f /root/.ssh/id_rsa  -P ''
export SSHPASS=magedu
while read IP;do
   sshpass -e ssh-copy-id  -o StrictHostKeyChecking=no $IP
done < hosts.list
```

#### 2.3.1.2、范例: 实现基于key验证的脚本2

```
[root@centos8 ~]#cat ssh_key.sh 
#!/bin/bash
#
IPLIST="
192.168.188.8
192.168.188.200
192.168.188.201
192.168.188.88"
rpm -q sshpass &> /dev/null || yum -y install sshpass  
[ -f /root/.ssh/id_rsa ] || ssh-keygen -f /root/.ssh/id_rsa  -P ''
export SSHPASS=123456
for IP in $IPLIST;do
   { sshpass -e ssh-copy-id -o StrictHostKeyChecking=no $IP; } &
done
wait
```

### 2.3.2、**ansible-doc**

此工具用来显示模块帮助,相当于man 

```
格式：
ansible-doc [options] [module...]
-l, --list       #列出可用模块
-s, --snippet #显示指定模块的playbook片段
```

范例: 查看帮助

```
[root@Ansible ~]# ansible-doc --help
usage: ansible-doc [-h] [--version] [-v] [-M MODULE_PATH] [--playbook-dir BASEDIR]
                   [-t {become,cache,callback,cliconf,connection,httpapi,inventory,lookup,netconf,shell,module,strategy,vars}] [-j] [-F | -l | -s | --metadata-dump]
                   [plugin [plugin ...]]

plugin documentation tool

positional arguments:
  plugin                Plugin

optional arguments:
  --metadata-dump       **For internal testing only** Dump json metadata for all plugins.
  --playbook-dir BASEDIR
                        Since this tool does not use playbooks, use this as a substitute playbook directory.This sets the relative path for many features including
                        roles/ group_vars/ etc.
  --version             show program's version number, config file location, configured module search path, module location, executable location and exit
  -F, --list_files      Show plugin names and their source files without summaries (implies --list)
  -M MODULE_PATH, --module-path MODULE_PATH
                        prepend colon-separated path(s) to module library (default=~/.ansible/plugins/modules:/usr/share/ansible/plugins/modules)
  -h, --help            show this help message and exit
  -j, --json            Change output into json format.
  -l, --list            List available plugins
  -s, --snippet         Show playbook snippet for specified plugin(s)
  -t {become,cache,callback,cliconf,connection,httpapi,inventory,lookup,netconf,shell,module,strategy,vars}, --type {become,cache,callback,cliconf,connection,httpapi,inventory,lookup,netconf,shell,module,strategy,vars}
                        Choose which plugin type (defaults to "module"). Available plugin types are : ('become', 'cache', 'callback', 'cliconf', 'connection',
                        'httpapi', 'inventory', 'lookup', 'netconf', 'shell', 'module', 'strategy', 'vars')
  -v, --verbose         verbose mode (-vvv for more, -vvvv to enable connection debugging)

See man pages for Ansible CLI options or website for tutorials https://docs.ansible.com

```

范例

```
#列出所有模块
ansible-doc -l  
#查看指定模块帮助用法
ansible-doc ping  
#查看指定模块帮助用法
ansible-doc -s  ping

[root@Ansible ~]# ansible-doc -l|wc -l
3387

范例: 查看指定的插件
[root@ansible ~]#ansible-doc -t connection -l
[root@ansible ~]#ansible-doc -t lookup -l

root@Ansible ~]# ansible all -m ping
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host 192.168.188.200 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with 
prior Ansible releases. A future Ansible release will default to using the discovered platform python for this host. See 
https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. Deprecation 
warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
192.168.188.200 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
192.168.188.201 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
192.168.188.7 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
192.168.188.8 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}



[root@Ansible ~]# ansible all -a 'date --s "20231212 14:35:36"'

```



```
[root@Ansible ~]# ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:rrNXcDGpyjNvqNvSi28Eng1d9nb4F7OdhLd872RW+ns root@Ansible.dingbh.top
The key's randomart image is:
+---[RSA 3072]----+
|           .     |
|        o +      |
|     . o o +  .  |
|    o . o = ..oo |
|   . * .S+ o  +=+|
|    o B.  . . o=+|
|     o =..   .. *|
|    .o=.+      =E|
|    +***       o=|
+----[SHA256]-----+
[root@Ansible ~]# ssh-copy-id 192.168.188.88
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@192.168.188.88's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh '192.168.188.88'"
and check to make sure that only the key(s) you wanted were added.

[root@Ansible ~]# ssh-copy-id 192.168.188.200
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@192.168.188.200's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh '192.168.188.200'"
and check to make sure that only the key(s) you wanted were added.

[root@Ansible ~]# ssh-copy-id 192.168.188.201
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@192.168.188.201's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh '192.168.188.201'"
and check to make sure that only the key(s) you wanted were added.

[root@Ansible ~]# ssh-copy-id 192.168.188.8
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@192.168.188.8's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh '192.168.188.8'"
and check to make sure that only the key(s) you wanted were added.

[root@Ansible ~]# ansible -m ping appservers
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host 192.168.188.200 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with 
prior Ansible releases. A future Ansible release will default to using the discovered platform python for this host. See 
https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. Deprecation 
warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
192.168.188.200 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
192.168.188.201 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}

```

## 2.4、常见模块

幂等性首选

### 2.4.1、Command模块(非幂等性)

功能：在远程主机执行命令，此为默认模块，可忽略 -m 选项

注意：此命令不支持 $VARNAME < > | ; & 等，可能用shell模块实现

注意：此模块不具有幂等性

常见选项

```
chdir=dir    	#执行命令前,先切换至目录dir
creates=file 	#当file不存在时才会执行
removes=file 	#当file存在时才会执行
```

范例：

```
[root@Ansible ~]# ansible-doc command
> COMMAND    (/usr/lib/python3/dist-packages/ansible/modules/commands/command.py)

        The `command' module takes the command name followed by a list of space-delimited arguments. The given command will be
        executed on all selected nodes. The command(s) will not be processed through the shell, so variables like `$HOME' and
        operations like `"<"', `">"', `"|"', `";"' and `"&"' will not work. Use the [shell] module if you need these features. To
        create `command' tasks that are easier to read than the ones using space-delimited arguments, pass parameters using the
        `args' L(task keyword,../reference_appendices/playbooks_keywords.html#task) or use `cmd' parameter. Either a free form
        command or `cmd' parameter is required, see the examples. For Windows targets, use the [win_command] module instead.

  * This module is maintained by The Ansible Core Team
  * note: This module has a corresponding action plugin.

OPTIONS (= is mandatory):

- argv
        Passes the command as a list rather than a string.
        Use `argv' to avoid quoting values that would otherwise be interpreted incorrectly (for example "user name").
        Only the string or the list form can be provided, not both.  One or the other must be provided.
        [Default: (null)]
        type: list
        version_added: 2.6

- chdir
        Change into this directory before running the command.
        [Default: (null)]
        type: path
        version_added: 0.6

- cmd
        The command to run.
        [Default: (null)]
        type: str

- creates
        A filename or (since 2.0) glob pattern. If it already exists, this step *won't* be run.
        [Default: (null)]
        type: path
:
```

实例：

```
1、#创建文件
[root@Ansible ~]# ansible 192.168.188.8 -m command -a 'touch /opt/reset.sh chdir=/opt creates=/root/reset.sh'
192.168.188.8 | SUCCESS | rc=0 >>
skipped, since /root/reset.sh exists
[root@Ansible ~]# ansible 192.168.188.8 -m command -a 'touch /opt/reset.sh chdir=/opt creates=/opt/reset.sh'
192.168.188.8 | CHANGED | rc=0 >>

#验证
[root@Rocky8 ~]#ls /opt/
reset.sh


2、#创建账号
[root@Ansible ~]# ansible 192.168.188.8 -m command -a 'useradd test'
```

### 2.4.2、Shell(非幂等性)

功能：和command相似，用shell执行命令,支持各种符号,比如:*,$, > , 相当于增强版的command模块

注意：此模块不具有幂等性,建议能不能就用此模块,最好使用专用模块

常见选项

```
chdir=dir    #执行命令前,先切换至目录dir
creates=file #当file不存在时才会执行
removes=file #当file存在时才会执行
```

范例

```
#创建账号并修改密码
[root@Ansible ~]# ansible 192.168.188.8 -m shell -a 'echo $hostname'
[root@Ansible ~]# ansible 192.168.188.8 -m shell -a 'echo 123456| passwd --stdin test'
192.168.188.8 | CHANGED | rc=0 >>
Changing password for user test.
passwd: all authentication tokens updated successfully.

[root@Rocky8 ~]#cat /etc/shadow|grep test
test:$6$uEJ2/Y8MdhCMkOJf$3H.xYEBY1/Q5KSkzutwb3Kc5BW2siVQdYysaq4F/wVSUAhy.LcKyGb0VqG6mCjDOWcdPMJmswKivbKyl4KrNo0:19703:0:99999:7:::

[root@Ansible ~]# ansible 192.168.188.8 -m shell -a 'echo 123456| passwd --stdin hang'
192.168.188.8 | CHANGED | rc=0 >>
Changing password for user hang.
passwd: all authentication tokens updated successfully.
[root@Rocky8 ~]#cat /etc/shadow|grep hang
hang:$6$nN4jCGejN/SnumRn$L1UcTTkgL9.hYXachpvMpvpNxtkty9hio8e9aZnUL7KmOcrnmkbe2k7PFhxQtr9EoOEy3Uk8C2BasDMxKT4kY.:19703:0:99999:7:::
```

### 2.4.3、Script(非幂等性)

功能：在远程主机上运行ansible服务器上的脚本(无需执行权限)

注意：此模块不具有幂等性

常见选项

```
chdir=dir    #执行命令前,先切换至目录dir
cmd          #指定ansible主机的命令
creates=file #当file不存在时才会执行
removes=file #当file存在时才会执行
```

范例：

```
[root@Ansible /data/ansible]# cat ip.sh 
#!/bin/bash

hostname -I > /opt/ip.log
hostname -I

[root@Ansible /data/ansible]# ansible all -m script -a './ip.sh'
192.168.188.201 | CHANGED => {
    "changed": true,
    "rc": 0,
    "stderr": "Shared connection to 192.168.188.201 closed.\r\n",
    "stderr_lines": [
        "Shared connection to 192.168.188.201 closed."
    ],
    "stdout": "192.168.188.201 \r\n",
    "stdout_lines": [
        "192.168.188.201 "
    ]
}
192.168.188.200 | CHANGED => {
    "changed": true,
    "rc": 0,
    "stderr": "Shared connection to 192.168.188.200 closed.\r\n",
    "stderr_lines": [
        "Shared connection to 192.168.188.200 closed."
    ],
    "stdout": "192.168.188.200 \r\n",
    "stdout_lines": [
        "192.168.188.200 "
    ]
}
192.168.188.7 | CHANGED => {
    "changed": true,
    "rc": 0,
    "stderr": "Shared connection to 192.168.188.7 closed.\r\n",
    "stderr_lines": [
        "Shared connection to 192.168.188.7 closed."
    ],
    "stdout": "192.168.188.7 \r\n",
    "stdout_lines": [
        "192.168.188.7 "
    ]
}
192.168.188.8 | CHANGED => {
    "changed": true,
    "rc": 0,
    "stderr": "Shared connection to 192.168.188.8 closed.\r\n",
    "stderr_lines": [
        "Shared connection to 192.168.188.8 closed."
    ],
    "stdout": "192.168.188.8 \r\n",
    "stdout_lines": [
        "192.168.188.8 "
    ]
}
192.168.188.88 | CHANGED => {
    "changed": true,
    "rc": 0,
    "stderr": "Shared connection to 192.168.188.88 closed.\r\n",
    "stderr_lines": [
        "Shared connection to 192.168.188.88 closed."
    ],
    "stdout": "192.168.188.88 \r\n",
    "stdout_lines": [
        "192.168.188.88 "
    ]
}

[root@dns ~]# ls /opt/
ip.log

[root@Rocky8 ~]#ls /opt/
ip.log

[root@ubuntu1804 ~]# ls /opt/       
ip.log 
......
```

### 2.4.4、Copy(幂等性)

功能：复制ansible服务器主控端或远程的本机的文件到远程主机

注意: src=file 如果是没指明路径,则为当前目录或当前目录下的files目录下的file文件

常见选项

```
src      #控制端的源文件路径
dest     #被控端的文件路径
owner    #属主
group    #属组
mode     #权限
backup   #是否备份
validate #验证成功才会执行copy
remote_src  #no是默认值,表示src文件在ansible主机,yes表示src文件在远程主机
```

范例:

```
#如目标存在，默认覆盖，此处指定先备
ansible websrvs -m copy -a "src=/root/test1.sh dest=/tmp/test2.sh   owner=wang mode=600 backup=yes"
#指定内容，直接生成目标文件    
ansible websrvs -m copy -a "content='wang 123456\nxiao 654321\n' dest=/etc/rsync.pas owner=root group=root mode=0600"
#复制/etc目录自身,注意/etc/后面没有/
ansible websrvs -m copy -a "src=/etc dest=/backup"
#复制/etc/下的文件，不包括/etc/目录自身,注意/etc/后面有/
ansible websrvs -m copy -a "src=/etc/ dest=/backup"
#复制/etc/suders,并校验语法
ansible websrvs -m copy -a "src=/etc/suders dest=/etc/sudoers.edit 
remote_src=yes validate=/usr/sbin/visudo -csf %s"
```

实例：（不会覆盖）

```
[root@Ansible /data/ansible]# ansible all -m copy -a 'src=/etc/issue dest=/opt/issue.bak mode=600 owner=test group=bin'
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host 192.168.188.200 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with 
prior Ansible releases. A future Ansible release will default to using the discovered platform python for this host. See 
https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. Deprecation 
warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
192.168.188.200 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "checksum": "ad0c3fcd470080ec3abed12de33106b206e3eef3",
    "dest": "/opt/issue.bak",
    "gid": 2,
    "group": "bin",
    "mode": "0600",
    "owner": "test",
    "path": "/opt/issue.bak",
    "size": 26,
    "state": "file",
    "uid": 1001
}
192.168.188.7 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "checksum": "ad0c3fcd470080ec3abed12de33106b206e3eef3",
    "dest": "/opt/issue.bak",
    "gid": 1,
    "group": "bin",
    "mode": "0600",
    "owner": "test",
    "path": "/opt/issue.bak",
    "size": 26,
    "state": "file",
    "uid": 1001
}
192.168.188.201 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "checksum": "ad0c3fcd470080ec3abed12de33106b206e3eef3",
    "dest": "/opt/issue.bak",
    "gid": 2,
    "group": "bin",
    "mode": "0600",
    "owner": "test",
    "path": "/opt/issue.bak",
    "size": 26,
    "state": "file",
    "uid": 1001
}
192.168.188.8 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "checksum": "ad0c3fcd470080ec3abed12de33106b206e3eef3",
    "dest": "/opt/issue.bak",
    "gid": 1,
    "group": "bin",
    "mode": "0600",
    "owner": "test",
    "path": "/opt/issue.bak",
    "size": 26,
    "state": "file",
    "uid": 1002
}
192.168.188.88 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": true,
    "checksum": "ad0c3fcd470080ec3abed12de33106b206e3eef3",
    "dest": "/opt/issue.bak",
    "gid": 1,
    "group": "bin",
    "mode": "0600",
    "owner": "test",
    "path": "/opt/issue.bak",
    "size": 26,
    "state": "file",
    "uid": 1000
}


[root@dns ~]# ls /opt/ -l
total 12
-rw-r--r-- 1 root root 15 Dec 12 19:23 ip.log
-rw-r--r-- 1 root root 15 Dec 12 19:23 ip.txt
-rw------- 1 test bin  26 Dec 12 19:50 issue.bak
[root@dns ~]# cat /opt/issue.bak 
Ubuntu 20.04.3 LTS \n \l


```

```
[root@Ansible /data/ansible]# ansible all -m copy -a 'src=/etc/issue dest=/opt/issue.bak mode=600 owner=test group=bin remote_src=yes'
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host 192.168.188.200 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with 
prior Ansible releases. A future Ansible release will default to using the discovered platform python for this host. See 
https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. Deprecation 
warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
192.168.188.200 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "checksum": "f301c352f6abca3bf480cd9297de5ce8be26a1a0",
    "dest": "/opt/issue.bak",
    "gid": 2,
    "group": "bin",
    "md5sum": "8315e643ed6a5d7c9962fc0a8ef9c11f",
    "mode": "0600",
    "owner": "test",
    "size": 26,
    "src": "/etc/issue",
    "state": "file",
    "uid": 1001
}
192.168.188.7 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "checksum": "5c76e3b565c91e21bee303f15c728c71e6b39540",
    "dest": "/opt/issue.bak",
    "gid": 1,
    "group": "bin",
    "md5sum": "f078fe086dfc22f64b5dca2e1b95de2c",
    "mode": "0600",
    "owner": "test",
    "size": 23,
    "src": "/etc/issue",
    "state": "file",
    "uid": 1001
}
192.168.188.201 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "checksum": "f301c352f6abca3bf480cd9297de5ce8be26a1a0",
    "dest": "/opt/issue.bak",
    "gid": 2,
    "group": "bin",
    "md5sum": "8315e643ed6a5d7c9962fc0a8ef9c11f",
    "mode": "0600",
    "owner": "test",
    "size": 26,
    "src": "/etc/issue",
    "state": "file",
    "uid": 1001
}
192.168.188.88 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": true,
    "checksum": "5c76e3b565c91e21bee303f15c728c71e6b39540",
    "dest": "/opt/issue.bak",
    "gid": 1,
    "group": "bin",
    "md5sum": "f078fe086dfc22f64b5dca2e1b95de2c",
    "mode": "0600",
    "owner": "test",
    "size": 23,
    "src": "/etc/issue",
    "state": "file",
    "uid": 1000
}
192.168.188.8 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": true,
    "checksum": "5c76e3b565c91e21bee303f15c728c71e6b39540",
    "dest": "/opt/issue.bak",
    "gid": 1,
    "group": "bin",
    "md5sum": "f078fe086dfc22f64b5dca2e1b95de2c",
    "mode": "0600",
    "owner": "test",
    "size": 23,
    "src": "/etc/issue",
    "state": "file",
    "uid": 1002
}


[root@dns ~]# cat /opt/issue.bak 
\S
Kernel \r on an \m

[root@ubuntu1804 ~]# cat /opt/issue.bak 
Ubuntu 18.04.5 LTS \n \l


```



### 2.4.5、Get_url模块(幂等性)

功能: 用于将文件从http、https或ftp下载到被管理机节点上

常用参数如下：

```
url   	#下载文件的URL,支持HTTP，HTTPS或FTP协议
dest  	#下载到目标路径（绝对路径），如果目标是一个目录，就用原文件名，如果目标设置了名称就用目标设置的名称
owner 	#指定属主
group 	#指定属组
mode  	#指定权限
force 	#如果yes，dest不是目录，将每次下载文件，如果内容改变替换文件。如果no，则只有在目标不存在时才会下载
checksum  	#对目标文件在下载后计算摘要，以确保其完整性
          	#示例: checksum="sha256:D98291AC[...]B6DC7B97",
                checksum="sha256:http://example.com/path/sha256sum.txt"
                
url_username 	#用于HTTP基本认证的用户名。 对于允许空密码的站点，此参数可以不使用`url_password'
url_password 	#用于HTTP基本认证的密码。 如果未指定`url_username'参数，则不会使用`url_password'参数
validate_certs 	#如果“no”，SSL证书将不会被验证。 适用于自签名证书在私有网站上使用timeout  #URL请求的超时时间,秒为单位
```

```
[root@Ansible /data/ansible]# ansible all -m get_url -a 'url=http://nginx.org/download/nginx-1.18.0.tar.gz dest=/usr/local/src/nginx.tar.gz checksum="md5:b2d33d24d89b8b1f87ff5d251aa27eb8"'

192.168.188.88 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": true,
    "checksum_dest": null,
    "checksum_src": "47b2c5ccd12e2a7088b03d629ff6b9ab18215180",
    "dest": "/usr/local/src/nginx.tar.gz",
    "elapsed": 1,
    "gid": 0,
    "group": "root",
    "md5sum": "b2d33d24d89b8b1f87ff5d251aa27eb8",
    "mode": "0644",
    "msg": "OK (1039530 bytes)",
    "owner": "root",
    "size": 1039530,
    "src": "/root/.ansible/tmp/ansible-tmp-1702382507.6903882-72625977357107/tmpax5iw01o",
    "state": "file",
    "status_code": 200,
    "uid": 0,
    "url": "http://nginx.org/download/nginx-1.18.0.tar.gz"
}
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host 192.168.188.200 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with 
prior Ansible releases. A future Ansible release will default to using the discovered platform python for this host. See 
https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. Deprecation 
warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
192.168.188.200 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "checksum_dest": null,
    "checksum_src": "47b2c5ccd12e2a7088b03d629ff6b9ab18215180",
    "dest": "/usr/local/src/nginx.tar.gz",
    "elapsed": 2,
    "gid": 0,
    "group": "root",
    "md5sum": "b2d33d24d89b8b1f87ff5d251aa27eb8",
    "mode": "0644",
    "msg": "OK (1039530 bytes)",
    "owner": "root",
    "size": 1039530,
    "src": "/root/.ansible/tmp/ansible-tmp-1702382507.694381-119046957676015/tmpzhUZu3",
    "state": "file",
    "status_code": 200,
    "uid": 0,
    "url": "http://nginx.org/download/nginx-1.18.0.tar.gz"
}
192.168.188.201 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "checksum_dest": null,
    "checksum_src": "47b2c5ccd12e2a7088b03d629ff6b9ab18215180",
    "dest": "/usr/local/src/nginx.tar.gz",
    "elapsed": 2,
    "gid": 0,
    "group": "root",
    "md5sum": "b2d33d24d89b8b1f87ff5d251aa27eb8",
    "mode": "0644",
    "msg": "OK (1039530 bytes)",
    "owner": "root",
    "size": 1039530,
    "src": "/root/.ansible/tmp/ansible-tmp-1702382507.688001-66164892975516/tmpoteco8l3",
    "state": "file",
    "status_code": 200,
    "uid": 0,
    "url": "http://nginx.org/download/nginx-1.18.0.tar.gz"
}
192.168.188.8 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": true,
    "checksum_dest": null,
    "checksum_src": "47b2c5ccd12e2a7088b03d629ff6b9ab18215180",
    "dest": "/usr/local/src/nginx.tar.gz",
    "elapsed": 2,
    "gid": 0,
    "group": "root",
    "md5sum": "b2d33d24d89b8b1f87ff5d251aa27eb8",
    "mode": "0644",
    "msg": "OK (1039530 bytes)",
    "owner": "root",
    "size": 1039530,
    "src": "/root/.ansible/tmp/ansible-tmp-1702382507.711074-254152862891726/tmp0qw6dokb",
    "state": "file",
    "status_code": 200,
    "uid": 0,
    "url": "http://nginx.org/download/nginx-1.18.0.tar.gz"
}
192.168.188.7 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "checksum_dest": null,
    "checksum_src": "47b2c5ccd12e2a7088b03d629ff6b9ab18215180",
    "dest": "/usr/local/src/nginx.tar.gz",
    "elapsed": 59,
    "gid": 0,
    "group": "root",
    "md5sum": "b2d33d24d89b8b1f87ff5d251aa27eb8",
    "mode": "0644",
    "msg": "OK (1039530 bytes)",
    "owner": "root",
    "size": 1039530,
    "src": "/root/.ansible/tmp/ansible-tmp-1702382507.7241063-238681384333872/tmps_HI0T",
    "state": "file",
    "status_code": 200,
    "uid": 0,
    "url": "http://nginx.org/download/nginx-1.18.0.tar.gz"
}

#验证
[root@dns ~]# ls /usr/local/src/
nginx.tar.gz
[root@Rocky8 ~]#ls /usr/local/src/
nginx.tar.gz
[root@Rocky8 ~]# ls /usr/local/src/
nginx.tar.gz
[root@ubuntu1804 ~]# ls /usr/local/src/
nginx.tar.gz
[root@Ubuntu1804 ~]# ls /usr/local/src/
nginx.tar.gz
```

### 2.4.6、Fetch(幂等性)

功能：从远程主机提取文件至ansible的主控端，copy相反，目前不支持目录

常见选项

```
src      #被控制端的源文件路径,只支持文件
dest     #ansible控制端的目录路径
```

范例：

```
ansible websrvs -m fetch -a 'src=/root/test.sh dest=/data/scripts'
```

范例：

```
[root@Ansible /data/ansible]# ansible webservers -m fetch -a 'src=/etc/os-release dest=/data/ansible/'
192.168.188.7 | CHANGED => {
    "changed": true,
    "checksum": "c093c25814ca254ccb3ce594c37715acfd2abd71",
    "dest": "/data/ansible/192.168.188.7/etc/os-release",
    "md5sum": "6038e70c459d5f53686e47be9c6c8781",
    "remote_checksum": "c093c25814ca254ccb3ce594c37715acfd2abd71",
    "remote_md5sum": null
}
192.168.188.8 | CHANGED => {
    "changed": true,
    "checksum": "bbefc9a30d138e8e98b365914fa45f8c7a0ec2ff",
    "dest": "/data/ansible/192.168.188.8/etc/os-release",
    "md5sum": "7bbe99830540f0fff619b428a1ef91c0",
    "remote_checksum": "bbefc9a30d138e8e98b365914fa45f8c7a0ec2ff",
    "remote_md5sum": null
}


[root@Ansible /data/ansible]# tree .
.
├── 192.168.188.7
│   └── etc
│       └── os-release
├── 192.168.188.8
│   └── etc
│       └── os-release
├── ansible.cfg
├── hosts
└── ip.sh

4 directories, 5 files

```

### 2.4.7、File

功能：设置文件属性,创建文件,目录和软链接等

常见选项

```
path #在被控端创建的路径
owner #属主
group #属组
mode #权限
state #状态
  =touch    #创建文件
  =directory #创建目录
  =link #软链接
  =hard     #硬链接
  =absent   #删除
recurse     #yes表示递归授权
```

范例：

```
#创建文件
[root@Ansible /data/ansible]# ansible all -m file -a 'path=/data/file.txt state=touch'

[root@Rocky8 ~]#ls /data/
file.txt
[root@Rocky8 ~]# ls /data/
file.txt

#创建目录
[root@Ansible /data/ansible]# ansible all -m file -a 'path=/data/file.txt state=directory'

#创建软链接
[root@Ansible ~]# ansible all -m file -a 'src=/data/test.txt dest=/opt/test.txt.link state=link'


[root@dns ~]# ls /opt/ -l   		#验证
total 12
-rw-r--r-- 1 root root 15 Dec 12 19:23 ip.log
-rw-r--r-- 1 root root 15 Dec 12 19:23 ip.txt
-rw------- 1 test bin  23 Nov 23  2020 issue.bak
lrwxrwxrwx 1 root root 14 Dec 13 00:11 test.txt.link -> /data/test.txt
[root@Rocky8 ~]#ls /opt/ -l
total 12
-rw-r--r--  1 root root 15 Dec 12 19:23 ip.log
-rw-r--r--  1 root root 15 Dec 12 19:23 ip.txt
-rw-------. 1 test bin  23 Oct 19  2022 issue.bak
-rw-r--r--  1 root root  0 Dec 12 15:39 reset.sh
lrwxrwxrwx  1 root root 14 Dec 13 00:11 test.txt.link -> /data/test.txt

#删除软链接
[root@Ansible ~]# ansible all -m file -a 'path=/opt/test.txt.link state=absent'

```

### 2.4.8、stat

功能：检查文件或文件系统的状态

注意：对于Windows目标，请改用win_stat模块

```
path #文件/对象的完整路径（必须）
```

常用的返回值判断：

```
exists： 判断是否存在
isuid： 调用用户的ID与所有者ID是否匹配
```

范例：

```
[root@Ansible ~]# ansible 127.0.0.1 -m stat -a 'path=/etc/passwd'
127.0.0.1 | SUCCESS => {
    "changed": false,
    "stat": {
        "atime": 1702366957.3817685,
        "attr_flags": "e",
        "attributes": [
            "extents"
        ],
        "block_size": 4096,
        "blocks": 8,
        "charset": "us-ascii",
        "checksum": "61371b6eb814d18d001370a689331ced1b1eb4f8",
        "ctime": 1702366957.3737686,
        "dev": 2050,
        "device_type": 0,
        "executable": false,
        "exists": true,
        "gid": 0,
        "gr_name": "root",
        "inode": 132628,
        "isblk": false,
        "ischr": false,
        "isdir": false,
        "isfifo": false,
        "isgid": false,
        "islnk": false,
        "isreg": true,
        "issock": false,
        "isuid": false,
        "mimetype": "text/plain",
        "mode": "0644",
        "mtime": 1702366957.3737686,
        "nlink": 1,
        "path": "/etc/passwd",
        "pw_name": "root",
        "readable": true,
        "rgrp": true,
        "roth": true,
        "rusr": true,
        "size": 1858,
        "uid": 0,
        "version": "856781905",
        "wgrp": false,
        "woth": false,
        "writeable": true,
        "wusr": true,
        "xgrp": false,
        "xoth": false,
        "xusr": false
    }
}
```

案例：playbook

```
- name: install | Check if file is already configured.
 stat: path={{ nginx_file_path }}
 connection: local
 register: nginx_file_result
- name: install | Download nginx file
 get_url: url={{ nginx_file_url }} dest={{ software_files_path }} 
validate_certs=no
 connection: local
 when:，not. nginx_file_result.stat.exists
```

案例

```
[root@ansible ansible]#cat stat.yml
---
- hosts: websrvs
  
 tasks:
    - name: check file
     stat: path=/data/mysql 
     register: st
    - name: debug
     debug:
       msg: "/data/mysql is not exist"
     when: not st.stat.exists
 [root@ansible ansible]#ansible-playbook   stat.yml
PLAY [websrvs] 
********************************************************************************
***************************************
TASK [Gathering Facts] 
********************************************************************************
*******************************
ok: [10.0.0.7]
ok: [10.0.0.8]
TASK [check file] 
********************************************************************************
************************************
ok: [10.0.0.7]
ok: [10.0.0.8]
TASK [debug] 
********************************************************************************
*****************************************
ok: [10.0.0.7] => {
    "msg": "/data/mysql is not exist"
}
ok: [10.0.0.8] => {
    "msg": "/data/mysql is not exist"
}
PLAY RECAP 
********************************************************************************
*******************************************
10.0.0.7                   : ok=3    changed=0    unreachable=0    failed=0   
skipped=0    rescued=0    ignored=0   
10.0.0.8                   : ok=3    changed=0    unreachable=0    failed=0   
skipped=0    rescued=0    ignored=0
```

### **2.4.9 unarchive** **模块**

功能：解包解压缩

实现有两种用法：

```
将ansible主机上的压缩包传到远程主机后解压缩至特定目录，设置remote_src=no,此为默认值,可省略
将远程本主机上或非ansible的其它主机的某个压缩包解压缩到远程主机本机的指定路径下，需要设置remote_src=yes 
```

常见参数：

```
remote_src #和copy功能一样且选项互斥，yes表示源文件在远程被控主机或其它非ansible的其它主机上，no表示文件在ansible主机上,默认值为no, 此选项代替copy选项
copy #默认为yes，当copy=yes，拷贝的文件是从ansible主机复制到远程主机上，如果设置为copy=no，会在远程主机上寻找src源文件,此选项已废弃
src #源路径，可以是ansible主机上的路径，也可以是远程主机(被管理端或者第三方主机)上的路径，如果是远程主机上的路径，则需要设置remote_src=yes
dest #远程主机上的目标路径
mode #设置解压缩后的文件权限
creates=/path/file #当绝对路径/path/file不存在时才会执行
```

范例：

```
#远程下载包并解压缩
[root@Ansible /data/ansible]# ansible all -m unarchive -a 'src=https://nginx.org/download/nginx-1.20.2.tar.gz dest=/data/ remote_src=yes'
```

### 2.4.10、archive

功能：打包压缩保存在被管理节点

常见选项

```
path   #压缩的文件或目录
dest   #压缩后的文件
format #压缩格式,支持gz,bz2,xz,tar,zip
```

范例：

```
[root@Ansible /data/ansible]# ansible all -m archive -a 'path=/var/log/ dest=/data/log.tar.bz2 format=bz2 owner=test mode=0600'
```

### 2.4.11、hostname

功能：管理主机名

常见选项

```
name  #修改后的主机名称
```

范例

```
[root@Ansible /data/ansible]# ansible 192.168.188.7 -m hostname -a 'name=DNS-server.dingbh.top'

[root@dns ~]# cat /etc/hostname 
DNS-server.dingbh.top

```

### 2.4.12、**Cron** **模块**

功能：计划任务

支持时间：minute，hour，day，month，weekday

常见选项

```
name 		#描述脚本的作用
minute    	#分钟
hour 		#小时
weekday     #周
user 		#任务由哪个用户运行；默认root
job 		#任务
```

范例：

```
#备份数据库脚本
[root@ubuntu1804 ~]# vim mysql_backup.sh

#!/bin/bash
mysqldump -A -F --single-transaction --master-data=2 -q -uroot |gzip > /data/mysql_`date +%F_%T`.sql.gz

#创建任务
[root@Ansible /data/ansible]# ansible 192.168.188.200 -m cron -a 'name=msyql_backup hour=2 minute=30 

[root@ubuntu1804 ~]# crontab -l
#Ansible: msyql_backup
30 2 * * 1-5 /root/mysql_backup.sh


#禁用计划任务
[root@Ansible /data/ansible]# ansible 192.168.188.200 -m cron -a 'name=msyql_backup hour=2 minute=30 weekday=1-5 job=/root/mysql_backup.sh disabled=yes'

[root@ubuntu1804 ~]# crontab -l
#Ansible: msyql_backup
#30 2 * * 1-5 /root/mysql_backup.sh

#开启计划任务
[root@Ansible /data/ansible]# ansible 192.168.188.200 -m cron -a 'name=msyql_backup hour=2 minute=30 weekday=1-5 job=/root/mysql_backup.sh disabled=no'

[root@ubuntu1804 ~]# crontab -l
#Ansible: msyql_backup
30 2 * * 1-5 /root/mysql_backup.sh

#删除任务
[root@Ansible /data/ansible]# ansible 192.168.188.200 -m cron -a 'name=msyql_backup hour=2 minute=30 weekday=1-5 job=/root/mysql_backup.sh state=absent'

[root@ubuntu1804 ~]# crontab -l

```

### 2.4.13、yum和apt

功能：管理软件包

yum 管理软件包，只支持RHEL，CentOS，fedora，不支持Ubuntu其它版本

apt 模块管理 Debian 相关版本的软件包

yum常见选项

```
name #软件包名称
state #状态
  =present #安装,此为默认值
  =absent #删除
  =latest #最新版
list            #列出指定包
enablerepo #启用哪个仓库安装
disablerepo #不使用哪些仓库的包
exclude #排除指定的包
validate        #是否检验,默认为yes
```

范例：

```
#安装最新nginx
[root@Ansible /data/ansible]# ansible 192.168.188.88 -m yum -a 'name=nginx state=latest'

#验证
[root@Rocky8 ~]# rpm -qi nginx
Name        : nginx
Epoch       : 1
Version     : 1.14.1
Release     : 9.module_el8.0.0+184+e34fea82
Architecture: x86_64
Install Date: Wed 13 Dec 2023 05:03:36 AM CST
Group       : System Environment/Daemons
Size        : 1734583
License     : BSD
Signature   : RSA/SHA256, Thu 10 Oct 2019 05:44:27 AM CST, Key ID 05b555b38483c65d
Source RPM  : nginx-1.14.1-9.module_el8.0.0+184+e34fea82.src.rpm
Build Date  : Tue 08 Oct 2019 05:17:55 AM CST
Build Host  : x86-02.mbox.centos.org
Relocations : (not relocatable)
Packager    : CentOS Buildsys <bugs@centos.org>
Vendor      : CentOS
URL         : http://nginx.org/
Summary     : A high performance web server and reverse proxy server
Description :
Nginx is a web server and a reverse proxy server for HTTP, SMTP, POP3 and
IMAP protocols, with a strong focus on high concurrency, performance and low
memory usage.


#安装zabbix-agent
[root@Ansible /data/ansible]# ansible 192.168.188.8 -m yum -a 'name=zabbix-agent state=latest'

[root@Rocky8 ~]#rpm -qi zabbix-agent
package zabbix-agent is not installed
[root@Rocky8 ~]#rpm -qi zabbix-agent
Name        : zabbix-agent
Version     : 6.0.22
Release     : release1.el8
Architecture: x86_64
Install Date: Wed 13 Dec 2023 05:05:33 AM CST
Group       : Applications/Internet
Size        : 2210107
License     : GPLv2+
Signature   : RSA/SHA512, Mon 25 Sep 2023 06:42:15 PM CST, Key ID 082ab56ba14fe591
Source RPM  : zabbix-6.0.22-release1.el8.src.rpm
Build Date  : Mon 25 Sep 2023 06:41:38 PM CST
Build Host  : builds.zabbix.lan
Relocations : (not relocatable)
URL         : http://www.zabbix.com/
Summary     : Zabbix agent
Description :
Old implementation of zabbix agent.
To be installed on monitored systems.

#卸载nginx
[root@Ansible /data/ansible]# ansible 192.168.188.88 -m yum -a 'name=nginx state=absent'

```

### 2.4.14、**yum_repository** **模块**

功能: 此模块实现yum的仓库配置管理

常见选项

```
name				#仓库id
description 		#仓库描述名称,对应配置文件中的name=
baseurl 			#仓库的地址
gpgcheck 			#验证开启
gpgkey              #仓库公钥路径
```

范例：

```
[root@Ansible /data/ansible]# ansible 192.168.188.8 -m yum_repository -a  'name="zabbix" description="zabbix repo" baseurl="https://mirrors.huaweicloud.com/zabbix/zabbix/6.0/rhel/8/x86_64/" gpgcheck=no'

[root@Rocky8 ~]#cat /etc/yum.repos.d/zabbix.repo 
[zabbix]
baseurl = https://mirrors.huaweicloud.com/zabbix/zabbix/6.0/rhel/8/x86_64/
gpgcheck = 0
name = zabbix repo

#安装zabbix-agent
[root@Ansible /data/ansible]# ansible 192.168.188.8 -m yum -a 'name=zabbix-agent state=latest'

[root@Rocky8 ~]#rpm -qi zabbix-agent
package zabbix-agent is not installed
[root@Rocky8 ~]#rpm -qi zabbix-agent
Name        : zabbix-agent
Version     : 6.0.22
Release     : release1.el8
Architecture: x86_64
Install Date: Wed 13 Dec 2023 05:05:33 AM CST

		.......
		
#卸载zabbix—agent
[root@Ansible /data/ansible]# ansible 192.168.188.8 -m yum -a 'name=zabbix-agent state=absent'
```

### 2.4.15、service

此模块和sytemd功能相似,选项很多相同

功能：管理服务

常见选项

```
name #服务名称
state #服务状态
=started #启动
=stopped #停止
=restarted    #重启
=reloaded    #重载
enabled #开启自启动
daemon_reload   #加载新的配置文件,适用于systemd模块
```

范例：

```
#安装nginx应用
[root@Ansible /data/ansible]# ansible 192.168.188.88 -m yum -a 'name=nginx state=latest'


[root@Rocky8 ~]# systemctl status nginx.service 
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: inactive (dead)

Dec 13 05:03:36 Rocky8.8 systemd[1]: nginx.service: Unit cannot be reloaded because it is inactive.
Dec 13 06:48:58 Rocky8.8 systemd[1]: nginx.service: Unit cannot be reloaded because it is inactive.

#设置开启启动并即时启动应用
[root@Ansible /data/ansible]# ansible 192.168.188.88 -m service -a 'name=nginx state=started enabled=yes'

#查看状态
[root@Rocky8 ~]# systemctl status nginx.service 
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2023-12-13 06:51:12 CST; 4s ago
  Process: 20566 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 20564 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 20563 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 20568 (nginx)
    Tasks: 3 (limit: 23016)
   Memory: 7.2M
   CGroup: /system.slice/nginx.service
           ├─20568 nginx: master process /usr/sbin/nginx
           ├─20569 nginx: worker process
           └─20570 nginx: worker process

Dec 13 06:51:12 Rocky8.8 systemd[1]: Starting The nginx HTTP and reverse proxy server...
Dec 13 06:51:12 Rocky8.8 nginx[20564]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Dec 13 06:51:12 Rocky8.8 nginx[20564]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Dec 13 06:51:12 Rocky8.8 systemd[1]: nginx.service: Failed to parse PID from file /run/nginx.pid: Invalid argument
Dec 13 06:51:12 Rocky8.8 systemd[1]: Started The nginx HTTP and reverse proxy server.


#远程开启coclpit服务
[root@Ansible ~]# ansible 192.168.188.8 -m service -a 'name=cockpit enabled=yes state=started'


```

### 2.4.16、**User** **模块**

功能：管理用户

常见选项

```
name 			#创建的名称
uid 			#指定uid
group 			#指定基本组
shell 			#登录shell类型默认/bin/bash
create_home 	#是否创建家目录
password 		#设定对应的密码，必须是加密后的字符串才行，否则不生效
system 			#yes表示系统用户
groups          #附加组
append 			#追加附加组使用,yes表示增加新的附加组
state    		#absen删除
remove 			#yes表示删除用户时将家目录一起删除
generate_ssh_key #创建私钥
ssh_keyu_bits    #私钥位数
ssh_key_file     #私钥文件路径
```

范例：

```
#创建账号
[root@Ansible /data/ansible]# ansible webservers -m user -a 'name=www uid=88 group=www system=yes  shell=/sbin/nologin password="$6$tM6wF3dVY1LLmtgP$zRRh3ydULjfZmZEbfdqEjpBAAFTn.hOFRDcaKGvGhGeOrHsjcZkDfPJPFlyE1xT5Ho2mkwrzZgNOkiSe1BI/a0"'

#事先要生成sha512密码
[root@Rocky8 ~]#openssl passwd -6 123456
$6$tM6wF3dVY1LLmtgP$zRRh3ydULjfZmZEbfdqEjpBAAFTn.hOFRDcaKGvGhGeOrHsjcZkDfPJPFlyE1xT5Ho2mkwrzZgNOkiSe1BI/a0

#验证
[root@dns-server ~]# id www
uid=88(www) gid=88(www) groups=88(www)
[root@dns-server ~]# cat /etc/shadow
			
			.....
			
www:$6$tM6wF3dVY1LLmtgP$zRRh3ydULjfZmZEbfdqEjpBAAFTn.hOFRDcaKGvGhGeOrHsjcZkDfPJPFlyE1xT5Ho2mkwrzZgNOkiSe1BI/a0:19705::::::

[root@Rocky8 ~]#id www
uid=88(www) gid=88(www) groups=88(www)
[root@Rocky8 ~]#cat /etc/shadow|grep www
www:$6$tM6wF3dVY1LLmtgP$zRRh3ydULjfZmZEbfdqEjpBAAFTn.hOFRDcaKGvGhGeOrHsjcZkDfPJPFlyE1xT5Ho2mkwrzZgNOkiSe1BI/a0:19705::::::

```

#生成私钥对

```
#生成私钥对
[root@Ansible /data/ansible]# ansible webservers -m user -a 'name=www generate_ssh_key=yes ssh_key_bits=4096 ssh_key_file=.ssh/id_rsa'

#验证
[root@Rocky8 ~]# tree /home/www/.ssh/
/home/www/.ssh/
├── id_rsa
└── id_rsa.pub

0 directories, 2 files
```

### 2.4.17、group

功能：管理组

常见选项

```
name        #指定组名称
gid         #指定gid
state 
  =present  #创建,默认
  =absent   #删除
```

范例：

```
#创建group
[root@Ansible /data/ansible]# ansible webservers -m group -a 'name=www gid=88 system=yes'

#验证
[root@dns-server ~]# groups www
www : www

[root@Rocky8 ~]#groups www
www : www

```

### 2.4.18、**Lineinfile** **模块**

ansible在使用sed进行替换时，经常会遇到需要转义的问题，而且ansible在遇到特殊符号进行替换时，会存在问题，无法正常进行替换 。ansible自身提供了两个模块：lineinfile模块和replace模块，可以方便的进行替换一般在ansible当中去修改某个文件的单行进行替换的时候需要使用lineinfile模块

功能：相当于sed，主要用于修改一行的文件内容

常见选项

```
path 			#被控端文件的路径
regexp			#正则匹配语法格式,表示被替换的内容
line 			#替换为的内容
state 			#absent表示删除
insertafter 	#插入到替换内容前面,如和regexp同时存在,只在没找到与regexp匹配时才使用insertafter
insertbefore 	#插入到替换内容后面,如和regexp同时存在,只在没找到与regexp匹配时才使用insertafter
backrefs 		#支持后面引用,yes和no
backup 			#修改前先备份
create 			#如果文件不存在,则创建,默认不存在会出错
mode 			#指定权限
owner 			#指定用户
group 			#指定组

#注意
regexp参数 ：使用正则表达式匹配对应的行，当替换文本时，如果有多行文本都能被匹配，则只有最后面被
匹配到的那行文本才会被替换，当删除文本时，如果有多行文本都能被匹配，这么这些行都会被删除。
```

**注意:** 如果想进行多行匹配进行替换需要使用replace模块

范例

```
#修改httpd服务端口号
[root@Ansible /data/ansible]# ansible webservers -m lineinfile -a "path=/etc/httpd/conf/httpd.conf regexp='^Listen' line='Listen 8080'"

#重启服务
[root@Ansible /data/ansible]# ansible webservers -m service -a 'name=httpd state=restarted'


#验证
[root@Rocky8 ~]#grep ^Listen /etc/httpd/conf/httpd.conf
Listen 8080

[root@Rocky8 ~]#ss -ntl   #8080端口已生效
State              Recv-Q             Send-Q                           Local Address:Port                           Peer Address:Port             Process             
LISTEN             0                  128                                    0.0.0.0:22                                  0.0.0.0:*                                    
LISTEN             0                  128                                          *:8080                                      *:*                                    
LISTEN             0                  128                                       [::]:22                                     [::]:*    

#修改selinux
[root@Ansible /data/ansible]# ansible all -m lineinfile -a "path=/etc/selinux/config regexp='^SELINUX=' line='SELINUX=disabled'"

#修改
```

### 2.4.19、**Replace** **模块**

该模块有点类似于sed命令，主要也是基于正则进行匹配和替换，建议使用

功能: 多行修改替换

常见选项

```
path 			#被控端文件的路径
regexp 			#正则匹配语法格式,表示被替换的内容
replace 		#替换为的内容
after           #插入到替换内容前面,
before          #插入到替换内容后面
backup          #修改前先备份
mode            #指定权限
owner           #指定用户
group           #指定组
```

范例：

```
#注释多行
[root@Ansible ~]# ansible all -m replace -a "path=/etc/fstab regexp='^(UUID.*)' replace='#\1'"

#验证功能
[root@dns-server ~]# cat /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Sun Oct  1 19:54:06 2023
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /                       xfs     defaults        0 0
#UUID=e7282410-a69a-4f1d-99fe-10ab77afb6a4 /boot                   xfs     defaults        0 0
/dev/mapper/centos-data /data                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0

#取消注释
[root@Ansible ~]# ansible all -m replace -a "path=/etc/fstab regexp='^#(UUID.*)' replace='\1'"


#验证结果
[root@dns-server ~]# cat /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Sun Oct  1 19:54:06 2023
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=e7282410-a69a-4f1d-99fe-10ab77afb6a4 /boot                   xfs     defaults        0 0
/dev/mapper/centos-data /data                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0
```

### 2.4.20、**SELinux** **模块**

功能: 该模块管理 SELInux 策略

常见选项

```
policy     #指定SELINUXTYPE=targeted
state      #指定SELINUX=disabled
```

范例: 

```
[root@Rocky8 ~]# getenforce 0
Enforcing

#关闭selinux功能，需重启生效
[root@Ansible ~]# ansible 192.168.188.88 -m selinux -a 'state=disabled'

#查看selinux功能已关闭，需重启生效
[root@Rocky8 ~]# getenforce 0
Permissive

```

### 2.4.21、reboot

功能: 重启

常见选项

```
msg               #重启提示
pre_reboot_delay  #重启前延迟时间的秒数
post_reboot_delay #重启后延迟时间的秒数后,再验证系统正常启动
reboot_timeout    #重启后延迟时间再执行测试成功与否的命令
test_command      #执行测试成功与否的命令
```

范例

```
[root@Ansible ~]# ansible 192.168.188.8 -m reboot -a 'msg="host will be reboot!" pre_reboot_delay=10'
```

### 2.4.22、mounte

功能: 挂载和卸载文件系统

常见选项

```
src    #源设备路径，或网络地址
path   #挂载至本地哪个路径下
fstype #设备类型； nfs
opts   #挂载的选项
state  #挂载还是卸载
  =present #永久挂载，但没有立即生效
  =absent #卸载临时挂载,并删除永久挂载
  =mounted #临时挂载
  =unmounted #临时卸载
```

范例

```
#修改fstab文件永久挂载,但不立即生效
mount websrvs -m mount -a 'src="UUID=b3e48f45-f933-4c8e-a700-22a159ec9077" path=/home fstype=xfs opts=noatime state=present'

#临时取消挂载
mount websrvs -m mount -a 'path=/home fstype=xfs opts=noatime state=unmounted'

#永久挂载,并立即生效
ansible websrvs -m mount -a 'src=10.0.0.8:/data/wordpress path=/var/www/html/wpcontent/uploads opts="_netdev" state=mounted'

[root@Ansible /data/ansible]# ansible 192.168.188.200 -m mount -a "src=/dev/sr0 path=/root/mnt  fstype=iso9660 state=mounted"

#永久卸载,并立即生效
ansible websrvs -m mount -a 'src=10.0.0.8:/data/wordpress path=/var/www/html/wpcontent/uploads state=absent'
```

### 2.4.23、Setup模块

功能： setup 模块来收集主机的系统信息，这些 facts 信息可以直接以变量的形式使用，但是如果主机

较多，会影响执行速度

可以使用 gather_facts: no 来禁止 Ansible 收集 facts 信息

常见选项

```
filter #指定过滤条件
```

范例

```
#显示主机系统信息
[root@Ansible ~]# ansible 192.168.188.7 -m setup > setup.log

ansible all -m setup
ansible all -m setup -a "filter=ansible_nodename"
ansible all -m setup -a "filter=ansible_hostname"
ansible all -m setup -a "filter=ansible_domain"
ansible all -m setup -a "filter=ansible_memtotal_mb"
ansible all -m setup -a "filter=ansible_memory_mb"
ansible all -m setup -a "filter=ansible_memfree_mb"
ansible all -m setup -a "filter=ansible_os_family"
ansible all -m setup -a "filter=ansible_distribution"
ansible all -m setup -a "filter=ansible_distribution_major_version"
ansible all -m setup -a "filter=ansible_distribution_version"
ansible all -m setup -a "filter=ansible_processor_vcpus"
ansible all -m setup -a "filter=ansible_all_ipv4_addresses"
ansible all -m setup -a "filter=ansible_architecture"
ansible all -m setup -a "filter=ansible_uptime_seconds"
ansible all -m setup -a "filter=ansible_processor*"
ansible all -m setup -a 'filter=ansible_env'
```

### 2.4.24、debug

功能: 此模块可以用于输出信息,并且通过 msg 定制输出的信息内容,功能类似于echo命令

注意: msg后面的变量有时需要加 " " 引起来

常见选项

```
msg       #指定命令输出的信息
var       #指定变量名,和msg互斥
verbosity #详细度
```

范例: debug 模块默认输出Hello world

```
[root@Ansible ~]# ansible all -m debug 
192.168.188.8 | SUCCESS => {
    "msg": "Hello world!"
}
192.168.188.88 | SUCCESS => {
    "msg": "Hello world!"
}
192.168.188.200 | SUCCESS => {
    "msg": "Hello world!"
}
192.168.188.7 | SUCCESS => {
    "msg": "Hello world!"
}

```

### 2.4.25、sysctl

功能: 修改内核参数

常见选项

```
name  #内核参数
value #指定值
state #是否保存在sysctl.conf文件中,默认present
sysctl_set #使用sysctl -w 验证值生效
```

范例

```
#开启forward功能
[root@Ansible ~]# ansible 192.168.188.8 -m sysctl -a 'name=net.ipv4.ip_forward value=1 state=present'
192.168.188.8 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": true
}


[root@Rocky8 ~]#sysctl -a |grep net.ipv4.ip_forward
net.ipv4.ip_forward = 1
net.ipv4.ip_forward_update_priority = 1
net.ipv4.ip_forward_use_pmtu = 0

#删除forward功能
[root@Ansible ~]# ansible 192.168.188.8 -m sysctl -a 'name=net.ipv4.ip_forward  state=absent'

[root@Rocky8 ~]#cat /etc/sysctl.conf 	#默认只删除文件内容

[root@Rocky8 ~]#sysctl -a |grep net.ipv4.ip_forward
net.ipv4.ip_forward = 1
net.ipv4.ip_forward_update_priority = 1
net.ipv4.ip_forward_use_pmtu = 0

#要删除需重写参数
[root@Rocky8 ~]#sysctl -w net.ipv4.ip_forward=0
net.ipv4.ip_forward = 0
[root@Rocky8 ~]#sysctl -a |grep net.ipv4.ip_forward
net.ipv4.ip_forward = 0
net.ipv4.ip_forward_update_priority = 1
net.ipv4.ip_forward_use_pmtu = 0
```

### 2.4.26、**pam_limits**

功能: 管理资源限制

范例:

```
- name: Change Limit /etc/security/limit.conf
 pam_limits:
   domain: "*"
   limit_type: "{{ item.limit_type }}"
   limit_item: "{{ item.limit_item }}"
   value: "{{ item.value }}"
 loop:
   - { limit_type: 'soft', limit_item: 'nofile',value: '100000' }
   - { limit_type: 'hard', limit_item: 'nofile',value: '10000' }
```

### 2.4.27、 **apt_repository** **模块**

功能: 此模块实现apt的仓库配置管理

常见选项

```
repo 				#仓库信息
state               #添加或删除
update_cache        #是否apt update,默认yes
filename            #仓库文件,默认放在/etc/apt/sources.list.d/file.list
```

### 2.4.28、**apt_key** **模块**

功能: 添加和删除apt key

常见选项

```
url 			#key路径
state           #添加或删除
```

范例: 生成ceph仓库配置

```
#先导入key,注意先后顺序
ansible ubuntu-servers -m apt_key -a
'url=https://download.ceph.com/keys/release.asc state=present'
#再生成apt配置,如果不导入key此步会出错
ansible ubuntu-servers -m apt_repository -a 'repo="deb 
http://mirror.tuna.tsinghua.edu.cn/ceph/debian-pacific focal main" 
filename=ansible_ceph'
#验证结果
[root@ubuntu2004 ~]#cat /etc/apt/sources.list.d/ansible_ceph.list 
deb http://mirror.tuna.tsinghua.edu.cn/ceph/debian-pacific focal main
```

### 2.4.29、其他模块

ansible 还提供了很多针对各种应用的模块,比如

```
nginx_status_info
nginx_status_facts
mysql_db   #需要安装MySQL-python包
mysql_user #需要安装MySQL-python包
redis
mongodb*
postgresql*
haproxy
git
```

```
{
  "pool": "www",
  "process manager": "dynamic",
  "start time": 1702573063,
  "start since": 1054,
  "accepted conn": 3,
  "listen queue": 0,
  "max listen queue": 0,
  "listen queue len": 2048,
  "idle processes": 1,
  "active processes": 1,
  "total processes": 2,
  "max active processes": 1,
  "max children reached": 0,
  "slow requests": 0
}
```



# 3、Playbook

## 3.1、**playbook**介绍

官方链接

```
https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html
```

### 3.1.1、playbook组成

![image-20231219094720131](C:\Users\m1526\AppData\Roaming\Typora\typora-user-images\image-20231219094720131.png)

```
1、一个 playbook(剧本)文件是一个YAML语言编写的文本文件
2、通常一个playbook只包括一个play 
3、一个 play的主要包括两部分: 主机和tasks. 即实现在指定一组主机上执行一个tasks定义好的任务列表。
4、一个tasks中可以有一个或多个task任务
5、每一个Task本质上就是调用ansible的一个module
6、在复杂场景中,一个playbook中也可以包括多个play，实现对多组不同的主机执行不同的任务
```

### 4.1.2、**Playbook** **与** **Ad-Hoc** **对比**

```
1、Playbook是对多个 AD-Hoc 的一种编排组合的实现方式
2、Playbook能控制任务执行的先后顺序
3、Playbook可以持久保存到文件中从而方便多次调用运行，而Ad-Hoc只能临时运行。
4、Playbook适合复杂的重复性的任务，而Ad-Hohttps://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.htmlc适合做快速简单的一次性任务
```

## 3.2、yaml语言

### 3.2.1、 YAML 语言介绍

```
1、YAML：YAML Ain't Markup Language，即YAML不是标记语言。不过，在开发的这种语言时，YAML的意思其实是："Yet Another Markup Language"（仍是一种标记语言）

2、YAML是一个可读性高的用来表达资料序列的格式。

3、YAML参考了其他多种语言，包括：XML、C语言、Python、Perl以及电子邮件格式RFC2822等。

4、Clark Evans在2001年在首次发表了这种语言，另外Ingy döt Net与Oren Ben-Kiki也是这语言的共同设计者
目前很多最新的软件比较流行采用此格式的文件存放配置信息，如:ubuntu，anisble，docker，kubernetes等

YAML 官方网站：
http://www.yaml.org

ansible 官网:
https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html
```

### 3.2.2、 **YAML** **语言特性**

```
YAML的可读性好
YAML和脚本语言的交互性好
YAML使用实现语言的数据类型
YAML有一个一致的信息模型
YAML易于实现
YAML可以基于流来处理
YAML表达能力强，扩展性好
```

### 3.2.3、YAML语法简介

```
1、在单一文件第一行，用连续三个连字号"-" 开始，还有选择性的连续三个点号( ... )用来表示文件的结尾

2、次行开始正常写Playbook的内容，一般建议写明该Playbook的功能

3、使用#号注释代码

4、缩进的级别也必须是一致的，同样的缩进代表同样的级别，程序判别配置的级别是通过缩进结合换行来实现的

5、缩进不支持tab,必须使用空格进行缩进

6、缩进的空格数不重要，只要相同层级的元素左对齐即可

7、YAML文件内容是区别大小写的，key/value的值均需大小写敏感

8、多个key/value可同行写也可换行写，同行使用，分隔

9、key后面冒号要加一个空格 比如: key: value

10、value可是个字符串，也可是另一个列表

11、YAML文件扩展名通常为yml或yaml
```

### 3.2.4、 **支持的数据类型**

```
YAML 支持以下常用几种数据类型：

​	标量：单个的、不可再分的值

​	对象：键值对的集合，又称为: 字典（dictionary）/ 哈希（hashes） / 映射（mapping）

​	数组：一组按次序排列的值，又称为: 列表（list）/ 序列（sequence）
```

#### 3.2.4.1、 **scalar** **标量**

key对应value

```
name: ding
age: 18
```

使用缩进的方式

```
name: 
 wang
age:
 18
```

标量是最基本的，不可再分的值，包括：

```
	字符串
​	布尔值
​	整数
​	浮点数
​	Null
​	时间
​	日期
```

#### 3.2.4.2、Dictionary字典

一个字典是由一个或多个key与value构成

key和value之间用冒号 ：分隔

冒号 : 后面有一个空格

所有 k/v 可以放在一行，,每个 k/v 之间用逗号分隔

所有每个 k/v 也可以分别放在不同行,一对k/v放在独立的一行

格式

```
account: { name: wang, age: 30 }
```

使用缩进方式

```
account: 
 name: wang
 age: 18
```

范例：

```
#不同行
# An employee record
name: Example Developer
job: Developer
skill: Elite(社会精英)
#同一行,也可以将key:value放置于{}中进行表示，用,分隔多个key:value
# An employee record
{name: "Example Developer", job: "Developer", skill: "Elite"}
```

#### 3.2.4.3、**List** **列表**

列表由多个元素组成

每个元素放在不同行，每个元素一行,且元素前均使用中横线 - 开头，并且中横线 - 和元素之间有一个空格

也可以将所有元素用 [ ] 括起来放在同一行,每个元素之间用逗号分隔



格式

```
course: [ linux , golang , python ]
```

也可以写成以 - 开头的多行

```
course:
 - linux
 - golang
 - python
```

元素里也可以包含字典

```
course:
 - linux: manjaro
 - golang: gin
 - python: django
```

范例：

```
#不同行,行以-开头,后面有一个空格
# A list of tasty fruits
- Apple
- Orange
- Strawberry
- Mango
#同一行
[Apple,Orange,Strawberry,Mango]
```

范例：YAML 表示一个家庭

```
name: John Smith
age: 41
gender: Male
spouse: { name: Jane Smith, age: 37, gender: Female } # 写在一行里
 name: Jane Smith   #也可以写成多行
 age: 37
 gender: Female
children: [ {name: Jimmy Smith,age: 17, gender: Male}, {name: Jenny Smith, age: 
13, gender: Female}, {name: hao Smith, age: 20, gender: Male } ]  #写在一行
 - name: Jimmy Smith    #写在多行,更为推荐的写法
   age: 17
   gender: Male
 - {name: Jenny Smith, age: 13, gender: Female}
 - {name: hao Smith, age: 20, gender: Male }    
```

### 3.2.5、三种常见的数据格式

```
	XML：Extensible Markup Language，可扩展标记语言，可用于数据交换和配置

​	JSON：JavaScript Object Notation, JavaScript 对象表记法，主要用来数据交换或配置，不支持注释

​	YAML：YAML Ain't Markup Language YAML 不是一种标记语言， 主要用来配置，大小写敏感，不支持tab
```

![image-20231220163909461](Ansible自动化/image-20231220163909461.png)

```
可以用工具互相转换，参考网站：
https://www.json2yaml.com/
http://www.bejson.com/json/json2yaml/
```

## 3.3、playbook核心组件

官方文档

```
https://docs.ansible.com/ansible/latest/reference_appendices/playbooks_keywords.html#playbook-keywords
```

一个playbook 中由多个组件组成,其中所用到的常见组件类型如下:

```
1、Hosts 执行的远程主机列表
2、Tasks 任务集,由多个task的元素组成的列表实现,每个task是一个字典,一个完整的代码块功能需最少元素需包括 name 和 task,一个name只能包括一个task
3、Variables 内置变量或自定义变量在playbook中调用
4、Templates 模板，可替换模板文件中的变量并实现一些简单逻辑的文件
5、Handlers 和 notify 结合使用，由特定条件触发的操作，满足条件方才执行，否则不执行
6、tags 标签 指定某条任务执行，用于选择运行playbook中的部分代码。ansible具有幂等性，因此会自动跳过没有变化的部分，即便如此，有些代码为测试其确实没有发生变化的时间依然会非常地长。此时，如果确信其没有变化，就可以通过tags跳过此些代码片断
```

###  3.3.1、hosts组件

Hosts：playbook中的每一个play的目的都是为了让特定主机以某个指定的用户身份执行任务。hosts用于指定要执行指定任务的主机，须事先定义在主机清单中

```
one.example.com
one.example.com:two.example.com
192.168.1.50
192.168.1.*
Websrvs:dbsrvs     #或者，两个组的并集
Websrvs:&dbsrvs   #与，两个组的交集
webservers:!dbsrvs  #在websrvs组，但不在dbsrvs组
```

案例：

```
- hosts: websrvs:appsrvs
```

### 3.3.2、remote_user组件

remote_user: 可用于Host和task中。也可以通过指定其通过sudo的方式在远程主机上执行任务，其可用于play全局或某任务；此外，甚至可以在sudo时使用sudo_user指定sudo时切换的用户

```
- hosts: websrvs
 remote_user: root
  
 tasks:
   - name: test connection
     ping:
     remote_user: root
     sudo: yes #默认sudo为root
     sudo_user:ding    #sudo为ding
```

### 3.3.3、task列表和action组件

```
play的主体部分是task list，task list中有一个或多个task,各个task 按次序逐个在hosts中指定的所有主机上执行，即在所有主机上完成第一个task后，再开始第二个task
task的目的是使用指定的参数执行模块，而在模块参数中可以使用变量。模块执行是幂等的，这意味着多次执行是安全的，因为其结果均一致每个task都应该有其name，用于playbook的执行结果输出，建议其内容能清晰地描述任务执行步骤。
如果未提供name，则action的结果将用于输出
```

**task两种格式：**

```
action: module arguments  #示例: action: shell wall hello 
module: arguments   #建议使用 #示例: shell: wall hello
```

注意：shell和command模块后面跟命令，而非key=value

范例: 包安装nginx

```
[root@Ansible /data/ansible]# cat nginx.yaml 
---

- hosts: webservers
  remote_user: root
  gather_facts: no   #不收集系统信息,提高执行效率
  
  tasks:
  - name: "包安装nginx"
    yum: 
      name: nginx
      state: present
  - name: "data page"
    copy: 
      src: files/index.html
      dest: /usr/share/nginx/html 
  - name: "start service"
    service: 
      name: nginx
      state: started
      enabled: yes 
```

### **3.3.4**、 其它组件说明

某任务的状态在运行后为changed时，可通过"notify"通知给相应的handlers任务还可以通过"tags"给task 打标签，可在ansible-playbook命令上使用-t指定进行调用

### **4.3.5、 ShellScripts VS Playbook** **案例**

```
#SHELL脚本实现
#!/bin/bash
# 安装Apache
yum install --quiet -y httpd 
# 复制配置文件
cp /tmp/httpd.conf /etc/httpd/conf/httpd.conf
cp/tmp/vhosts.conf /etc/httpd/conf.d/
# 启动Apache，并设置开机启动
systemctl enable --now httpd 
#Playbook实现
---
- hosts: websrvs
 remote_user: root
 gather_facts: no
  
 tasks:
   - name: "安装Apache"
     yum: name=httpd
   - name: "复制配置文件"
     copy: src=/tmp/httpd.conf dest=/etc/httpd/conf/
   - name: "复制配置文件"
     copy: src=/tmp/vhosts.conf dest=/etc/httpd/conf.d/
   - name: "启动Apache，并设置开机启动"
     service: name=httpd state=started enabled=yes
```

## 3.4、playbook命令

格式

```
ansible-playbook <filename.yml> ... [options]
```

常见选项

```
--syntax,--syntax-check      #语法检查,功能相当于bash -n 
-C --check #模拟执行dry run ,只检测可能会发生的改变，但不真正执行操作
--list-hosts    #列出运行任务的主机
--list-tags #列出tag
--list-tasks #列出task
--limit 主机列表 #只针对主机列表中的特定主机执行
-i INVENTORY, --inventory INVENTORY  #指定主机清单文件,通常一个项对应一个主机清单文件
--start-at-task START_AT_TASK #从指定task开始执行,而非从头开始,START_AT_TASK为任务的name
-v -vv  -vvv #显示过程
```

**范例: 一个简单的 playbook**

```
[root@ansible ansible]#cat hello.yml
---
- hosts: websrvs
 tasks:
    - name: hello
     command: echo "hello ansible"
[root@ansible ansible]#ansible-playbook hello.yml
[root@ansible ansible]#ansible-playbook -v hello.yml
```

**范例: 检查和限制主机**

```
ansible-playbook file.yml  --check #只检测
ansible-playbook file.yml  
ansible-playbook file.yml  --limit websrvs
```

**范例: 一个playbook 多个play**

```
cat test_plays.yaml
---
- hosts: localhost
 remote_user: root
 gather_facts: no
 tasks:
    - name: play1
         command: echo "play1"
- hosts: webservers
 remote_user: root
 gather_facts: no
 tasks:
    - name: play2
     command: echo "play2"
```

## 3.5、Playbook案例

### 4.5.1、利用playbook创建mysql用户

范例：mysql_user.yml

```
---
- hosts: webservers
 remote_user: root
 gather_facts: no
 tasks:
   - {name: create group, group: name=mysql system=yes gid=306}
   - name: create user
     user: name=mysql shell=/sbin/nologin system=yes group=mysql uid=306 
home=/data/mysql create_home=no  
```

### 4.5.2、利用playbook安装nginx

```
[root@Ansible /data/ansible]# vim nginx.yaml 

---

- hosts: webservers
  remote_user: root
  
  tasks:
  - name: "包安装nginx"
    yum: 
      name: nginx
      state: present
  - name: "data page"
    copy: 
      src: files/index.html
      dest: /usr/share/nginx/html 
  - name: "start service"
    service: 
      name: nginx
      state: started
      enabled: yes  
```

#### 范例：ansible 安装nginx服务并配置文件及页面

#### 4.5.2.1、文件准备

```
[root@Ansible /data/ansible]# tree /data/ansible/
/data/ansible/
├── config_files
│   ├── nginx.conf
│   └── pc.conf
├── hosts
├── nginx2.yaml
├── nginx_html
│   ├── index.html
│   └── test.html
├── nginx.yaml

5 directories, 9 files
```

#### 4.5.2.2、yaml文件准备

```
[root@Ansible /data/ansible]# vim nginx2.yaml 

---
- hosts: webservers
  remote_user: root
  gather_facts: no
  
  tasks:
  - name: 包安装nginx
    yum: 
      name: nginx
      state: present
  - name: config file
    copy:
      src: /data/ansible/config_files/nginx.conf
      dest: /etc/nginx/ 
    notify: restart service 
  - name: config file
    copy:
      src: /data/ansible/config_files/pc.conf
      dest: /etc/nginx/conf.d/
    notify: restart service 
  - name: create data dir 
    file:
      path: /data/website
      state: directory
  - name: data page
    copy: 
      src: ./nginx_html/
      dest: /data/website/
      owner: nginx
      group: nginx
  - name: start service
    service: 
      name: nginx
      state: started
      enabled: yes 

  handlers:						#添加模块
    - name: restart service
      service:
        name: nginx
        state: restarted
 
 
 

```

#### 4.5.2.3、web页面准备

```
[root@Ansible /data/ansible]# tree nginx_html/
nginx_html/
├── index.html
└── test.html

0 directories, 2 files


 #以打包的方式进行界面部署
 [root@Ansible /data/ansible/nginx_html]# tar zcf ../nginx_html.tar.gz *

```

#### 4.5.2.4、检查yml文件语法

```
[root@Ansible /data/ansible]# ansible-playbook -C nginx2.yaml 

PLAY [webservers] ****************************************************************************************************************************************************

TASK [包安装nginx] ******************************************************************************************************************************************************
ok: [192.168.188.7]
ok: [192.168.188.8]
ok: [192.168.188.88]

TASK [config file] ***************************************************************************************************************************************************
ok: [192.168.188.7]
ok: [192.168.188.8]
ok: [192.168.188.88]

TASK [config file] ***************************************************************************************************************************************************
ok: [192.168.188.7]
ok: [192.168.188.8]
ok: [192.168.188.88]

TASK [create data dir] ***********************************************************************************************************************************************
ok: [192.168.188.7]
ok: [192.168.188.8]
ok: [192.168.188.88]

TASK [data page] *****************************************************************************************************************************************************
ok: [192.168.188.7]
ok: [192.168.188.8]
ok: [192.168.188.88]

TASK [start service] *************************************************************************************************************************************************
ok: [192.168.188.88]
ok: [192.168.188.7]
ok: [192.168.188.8]

PLAY RECAP ***********************************************************************************************************************************************************
192.168.188.7              : ok=6    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
192.168.188.8              : ok=6    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
192.168.188.88             : ok=6    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

#### 4.5.2.5、执行yaml文件

```
[root@Ansible /data/ansible]# ansible-playbook nginx2.yaml 
```

#### 4.5.2.6、验证结果

192.168.188.7

![image-20231219111517365](Ansible自动化/image-20231219111517365.png)

192.168.188.8

![image-20231219111532815](Ansible自动化/image-20231219111532815.png)

192.168.188.88

![image-20231219113823973](Ansible自动化/image-20231219113823973.png)

以打包压缩的方式部署web界面

```
[root@Ansible /data/ansible]# cat nginx3.yaml 
---
- hosts: webservers
  remote_user: root
  gather_facts: no
  
  tasks:
  - name: install nginx
    yum:
      name: nginx
      state: present
  - name: config file
    copy:
      src: /data/ansible/nginx_config/nginx.conf
      dest: /etc/nginx/ 
    notify: restart service 
    tags:
      - config
  - name: config file
    copy:
      src: /data/ansible/nginx_config/pc.conf
      dest: /etc/nginx/conf.d/
    notify: restart service 
    tags:
      - config

  - name: create html data
    file:
      path: /data/website
      state: directory
      owner: nginx
      group: nginx
  - name: data pages
    unarchive:
      src: nginx_html.tar.gz
      dest: /data/website/
      remote_src: no
      owner: nginx
      group: nginx
    tags:
      - html
      - data
  - name: start service
    service:
      name: nginx
      state: started
      enabled: yes
#触发器
  handlers:
    - name: restart service
      systemd:
        name: nginx
        state: restarted
```



## 3.6、忽略错误ignore_errors

如果一个task出错,默认将不会继续执行后续的其它task

利用 ignore_errors: yes 可以忽略此task的错误,继续向下执行playbook其它task 

```
[root@ansible ansible]#cat test_ignore.yml
---
- hosts: websrvs
  
 tasks:
   - name: error
     command: /bin/false
     ignore_errors: yes
   - name: continue
     command: wall continue
```

范例：安装nginx，ignore error，添加 ignore_errors: yes 

```
[root@Ansible /data/ansible]# vim nginx.yaml 

---

- hosts: webservers
  remote_user: root
  
  tasks:
  - name: "包安装nginx"
    yum: 
      name: nginx
      state: present
  - name: "data page"
    copy: 
      src: files/index.html
      dest: /usr/share/nginx/html 
    ignore_errors: yes 
  - name: "start service"
    service: 
      name: nginx
      state: started
      enabled: yes 
      
[root@Ansible /data/ansible]# ansible-playbook nginx.yaml 

PLAY [webservers] ****************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************
ok: [192.168.188.7]
ok: [192.168.188.88]
ok: [192.168.188.8]

TASK [包安装nginx] ******************************************************************************************************************************************************
ok: [192.168.188.88]
ok: [192.168.188.8]
ok: [192.168.188.7]

TASK [data page] *****************************************************************************************************************************************************
An exception occurred during task execution. To see the full traceback, use -vvv. The error was: If you are using a module and expect the file to exist on the remote, see the remote_src option
fatal: [192.168.188.7]: FAILED! => {"changed": false, "msg": "Could not find or access 'files/index.html'\nSearched in:\n\t/data/ansible/files/files/index.html\n\t/data/ansible/files/index.html\n\t/data/ansible/files/files/index.html\n\t/data/ansible/files/index.html on the Ansible Controller.\nIf you are using a module and expect the file to exist on the remote, see the remote_src option"}
...ignoring
An exception occurred during task execution. To see the full traceback, use -vvv. The error was: If you are using a module and expect the file to exist on the remote, see the remote_src option
fatal: [192.168.188.88]: FAILED! => {"changed": false, "msg": "Could not find or access 'files/index.html'\nSearched in:\n\t/data/ansible/files/files/index.html\n\t/data/ansible/files/index.html\n\t/data/ansible/files/files/index.html\n\t/data/ansible/files/index.html on the Ansible Controller.\nIf you are using a module and expect the file to exist on the remote, see the remote_src option"}
...ignoring
An exception occurred during task execution. To see the full traceback, use -vvv. The error was: If you are using a module and expect the file to exist on the remote, see the remote_src option
fatal: [192.168.188.8]: FAILED! => {"changed": false, "msg": "Could not find or access 'files/index.html'\nSearched in:\n\t/data/ansible/files/files/index.html\n\t/data/ansible/files/index.html\n\t/data/ansible/files/files/index.html\n\t/data/ansible/files/index.html on the Ansible Controller.\nIf you are using a module and expect the file to exist on the remote, see the remote_src option"}
...ignoring

TASK [start service] *************************************************************************************************************************************************
ok: [192.168.188.7]
ok: [192.168.188.8]
ok: [192.168.188.88]

PLAY RECAP ***********************************************************************************************************************************************************
192.168.188.7              : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1   
192.168.188.8              : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1   
192.168.188.88             : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1 
```



## 3.7、 Playbook中使用handlers和notify

### 3.7.1、handlers和notify

Handlers本质是task list ，类似于MySQL中的触发器触发的行为，其中的task与前述的task并没有本质上的不同，只有在关注的资源发生变化时，才会采取一定的操作。

Notify对应的action 在所有task都执行完才会最后被触发，这样可避免多个task多次改变发生时每次都触发执行指定的操作，Handlers仅在所有的变化发生完成后一次性地执行指定操作。

在notify中列出的操作称为handler，也即notify中调用handler中定义的操作

```
注意:

​	如果多个task通知了相同的handlers， 此handlers仅会在所有task结束后运行一 次。

​	只有notify对应的task发生改变了才会通知handlers， 没有改变则不会触发handlers 

​	handlers 是在所有前面的tasks都成功执行才会执行,如果前面任何一个task失败,会导致handler跳过执行
```

案例：

```
[root@Ansible /data/ansible]# vim nginx2.yaml 

---
- hosts: webservers
  remote_user: root
  gather_facts: no
  
  tasks:
  - name: 包安装nginx
    yum: 
      name: nginx
      state: present
  - name: config file
    copy:
      src: /data/ansible/config_files/nginx.conf
      dest: /etc/nginx/ 
    notify:
    - restart service
    - notify message
  - name: config file
    copy:
      src: /data/ansible/config_files/pc.conf
      dest: /etc/nginx/conf.d/
    notify: 
    - restart service
    - notify message
  - name: create data dir 
    file:
      path: /data/website
      state: directory
  - name: data page
    copy: 
      src: ./nginx_html/
      dest: /data/website/
      owner: nginx
      group: nginx
  - name: start service
    service:
      name: nginx
      state: started
      enabled: yes

  handlers:
    - name: restart service
      service:
        name: nginx
        state: restarted
    - name: notify message
      debug: msg="nginx is restarted"

```

```
---
- hosts: websrvs
 remote_user: root
 gather_facts: no
 tasks:
    - name: Install httpd
     yum: name=httpd state=present
    - name: Install configure file
     copy: src=files/httpd.conf dest=/etc/httpd/conf/
     notify: 
        - restart httpd 
        - wall
    - name: ensure apache is running
      service: name=httpd state=started enabled=yes
   
 handlers:
    - name: restart httpd
      service: name=httpd state=restarted
    - name: wall
     command: wall "The config file is changed"
```

范例: 部署haproxy

```
- hosts: haservers
 tasks:
   - name: Configure Haproxy Server
     copy:
       src: ./haproxy.cfg.j2
       dest: /etc/haproxy/haproxy.cfg
     notify: Restart Haproxy Server
   - name: Started Haproxy Server
     systemd:
       name: haproxy
       state: started
 handlers:
   - name: Restart Haproxy Server
     systemd:
       name: haproxy
       state: restarted
```

### 3.7.2、**force_handlers**

如果不论前面的task成功与否,都希望handlers能执行, 可以使用force_handlers: yes 强制执行handler

范例: 强制调用handlers

```
- hosts: websrvs
 force_handlers: yes #无论task中的任何一个task失败,仍强制调用handlers
 tasks:
    - name: config file
     copy: src=nginx.conf  dest=/etc/nginx/nginx.conf
     notify: restart nginx
    - name: install package
     yum: name=no_exist_package
 handlers:
    - name: restart nginx
      service: name=nginx state=restarted
```



## 3.8、Playbook中使用tags组件

官方文档:

```
https://docs.ansible.com/ansible/latest/user_guide/playbooks_tags.html
```

默认情况下， Ansible 在执行一个 playbook 时，会执行 playbook 中所有的任务

在playbook文件中，可以利用tags组件，为特定 task 指定标签，当在执行playbook时，可以只执行特定tags的task,而非整个playbook文件，可以一个task对应多个tag,也可以多个task对应同一个tag，还有另外3个特殊关键字用于标签, tagged, untagged 和 all,它们分别是仅运行已标记，只有未标记和所有任务。

tags 主要用于调试环境



范例： tag 标签

```
[root@Ansible /data/ansible]# cat nginx3.yaml 
---
- hosts: webservers
  remote_user: root
  gather_facts: no
  
  tasks:
  - name: 包安装nginx
    yum: 
      name: nginx
      state: present
  - name: config file
    copy:
      src: /data/ansible/config_files/nginx.conf
      dest: /etc/nginx/ 
    notify: restart service 
    tags:
      - config
  - name: config file
    copy:
      src: /data/ansible/config_files/pc.conf
      dest: /etc/nginx/conf.d/
    notify: restart service
    tags:
      - config

  - name: create data dir
    file:
      path: /data/website
      state: directory
  - name: data page
    copy: 
      src: ./nginx_html/
      dest: /data/website/
      owner: nginx
      group: nginx
    tags: 
      - html
      - data
  - name: start service
    service: 
      name: nginx
      state: started
      enabled: yes

  handlers:
    - name: restart service
      service: 
        name: nginx
        state: restarted
        
#列出标签
[root@Ansible /data/ansible]# ansible-playbook --list-tags nginx3.yaml 

playbook: nginx3.yaml

  play #1 (webservers): webservers	TAGS: []
      TASK TAGS: [config, data, html]
      
      
#执行有指定标签的任务
[root@Ansible /data/ansible]# ansible-playbook -t data,config nginx3.yaml 

#忽略执行指定标签的task
[root@ansible ~]#ansible-playbook --skip-tags conf httpd.yml

#忽略执行没有标签的任务,即只执行有标签的任务
[root@ansible ~]#ansible-playbook   httpd.yml --skip-tags untagged
```

## 3.9 Playbook中使用变量

Playbook中同样也支持变量

变量名：仅能由字母、数字和下划线组成，且只能以字母开头

变量定义：

```
variable=value
variable: value
```

范例：

```
http_port=80
http_port: 80
```

**变量调用方式：**

通过 {{ variable_name }} 调用变量，且变量名前后建议加空格，有时用"{{ variable_name }}"才生效

**变量来源：**

1. ansible 的 setup facts 远程主机的所有变量都可直接调用

2. 通过命令行指定变量，优先级最高

   ```
   ansible-playbook -e varname=value test.yml
   ```

3. 在playbook文件中定义

   ```
   vars:
   
    var1: value1
   
    var2: value2
   ```

4. 在独立的变量YAML文件中定义

   ```
   - hosts: all
    vars_files:
      - vars.yml
   ```

   

5. 在主机清单文件中定义

   ```
   主机（普通）变量：	主机组中主机单独定义，优先级高于公共变量
   
   组（公共）变量：	 针对主机组中所有主机定义统一变量
   ```

   

6. 在项目中针对主机和主机组定义

   在项目目录中创建 host_vars和group_vars目录

7. 在role中定义

   **变量的优先级从高到低如下**

   ```
   -e 选项定义变量 -->playbook中vars_files --> playbook中vars变量定义 -->host_vars/主机名文件 -->主机清单中主机变量--> group_vars/主机组名文件-->group_vars/all文件--> 主机清单组变量
   ```


### 3.9.1、使用setup 模块中变量

#### 3.9.1.1、使用facts变量

本模块自动在playbook调用，生成的系统状态信息, 并将之存放在facts变量中facts 包括的信息很多,如: 主机名,IP,CPU,内存,网卡等

facts 变量的实际使用场景案例

```
通过facts变量获取被控端CPU的个数信息,从而生成不同的Nginx配置文件
通过facts变量获取被控端内存大小信息,从而生成不同的memcached的配置文件
通过facts变量获取被控端主机名称信息,从而生成不同的Zabbix配置文件
通过facts变量获取被控端网卡信息,从而生成不同的主机名
```

案例：使用setup变量

范例：修改随机数为主机名

```
[root@Ansible /data/ansible]# cat hostname.yaml 
---
- hosts: webservers


  tasks:
  - name: genater random
    shell:
      cmd: openssl rand -base64 12 |tr -dc '[:alnum:]'
    register:
      num
  - name: display num
    debug:
      msg: "{{ num.stdout }}"
  - name: set hostname
    hostname:
      name: "web-{{ num.stdout }}"


[root@Ansible /data/ansible]# ansible-playbook -C hostname.yaml 
```



```
默认ubuntu2004不支持rocky系统hostname模块
解决办法：
[root@Ansible /data/ansible]# vim /usr/lib/python3/dist-packages/ansible/modules/system/hostname.py

#文件721行处添加rocky判断

class ALTLinuxHostname(Hostname):
    platform = 'Linux'
    distribution = 'Rocky'
    strategy_class = RedHatStrategy

```

#### **3.9.1.2** **性能优化**

每次执行playbook,默认会收集每个主机的所有facts变量,将会导致速度很慢,可以采用下面方法加速

##### 方法一

##### 关闭facts采集加速执行,此方法将导致无法使用facts变量

```
- hosts: all
 gather_facts: no
```

##### 方法二

##### 当使用 gather_facts: no 关闭 facts，确实能加速 Ansible 执行，但是有时候又需要使用 facts 中的内容，还希望执行的速度快，这时候可以设置facts 的缓存,将facts变量信息存在redis服务器中

```
[root@ansible ~]# cat /etc/ansible/ansible.cfg
[defaults]
# smart 表示默认收集 facts，但 facts 已有的情况下不会收集，即使用缓存 facts
# implicit 表示默认收集 facts，要禁止收集，必须使用 gather_facts: False
# explicit 则表示默认不收集，要显式收集，必须使用gather_facts: True
gathering = smart              #在使用 facts 缓存时设置为smart
fact_caching_timeout = 86400   #缓存时长
fact_caching = redis           #缓存存在redis中
fact_caching_connection = 10.0.0.100:6379:0  #0表示redis的0号数据库
#若redis设置了密码
fact_caching_connection = 10.0.0.100:6379:0:password
```

### **3.9.2**、 在playbook命令行中定义变量

范例：

```
[root@Ansible /data/ansible]# cat var.yaml 
---
- hosts: webservers


  tasks:
  - name: install git
    yum: 
      name: "{{ pkname }}"
      state: present
[root@Ansible /data/ansible]# ansible-playbook -e pkname=git var.yaml

```

在文件中定义多个变量

```
[root@Ansible /data/ansible]# cat vars 
pkname1: git
pkname2: vsftpd
[root@Ansible /data/ansible]# cat var.yaml 
---
- hosts: webservers


  tasks:
  - name: install git
    yum: 
      name: "{{ pkname1 }}"
      state: present
  - name: install git
    yum: 
      name: "{{ pkname2 }}"
      state: present
#测试
[root@Ansible /data/ansible]# ansible-playbook -e pkname1=git -e pkname2=vsftpd var.yaml -C
或
[root@Ansible /data/ansible]# ansible-playbook -e '@vars' var.yaml -C
```

### 3.9.3、在playbook文件中定义变量

此方式定义的是私有变量,即只能在当前playbook中使用,不能被其它Playbook共用

范例：

```
[root@Ansible /data/ansible]# cat var.yaml 
---
- hosts: webservers
  vars:
    username: nginx
    groupname: nginx

  tasks:
  - name: mkdir diractory
    file: 
      path: /data/nginx_file.log
      state: touch
      owner: "{{ username }}"
      group: "{{ groupname}}"

[root@Ansible /data/ansible]# ansible-playbook var.yaml 

#验证
[root@web-sC3eTNr3O9TRLR94 ~]# ls /data/nginx_file.log  -l
-rw-r--r-- 1 nginx nginx 0 Dec 22 05:45 /data/nginx_file.log

[root@web-hzuoQcWtDBiopZH ~]#ls /data/nginx_file.log  -l
-rw-r--r-- 1 nginx nginx 0 Dec 22 05:45 /data/nginx_file.log

[root@web-KwGCVbfNE6ToZZ3 ~]# ls /data/nginx_file.log  -l
-rw-r--r-- 1 nginx nginx 0 Dec 22 05:45 /data/nginx_file.log
```

范例：变量的相互调用

```
[root@ansible ~]#cat var4.yaml
---
  - hosts: websrvs
   remote_user: root
   vars:
     collect_info: "/data/test/{{ansible_default_ipv4['address']}}/"
   tasks:
      - name: create IP directory
       file: name="{{collect_info}}" state=directory 
#执行结果
tree /data/test/
/data/test/
└── 10.0.0.102
1 directory, 0 files
```

范例: 变量的相互调用

```
[root@Ansible /data/ansible]# cat touch.yaml 
---
- hosts: webservers
  remote_user: root
  vars:
    duffix: "txt"
    file: "{{ ansible_nodename }}.{{ duffix }}"

  tasks:
  - name: create files
    file:
      path: "/data/{{ file }}"
      state: touch
[root@Ansible /data/ansible]# ansible-playbook touch.yaml
```

### 3.9.4、**使用专用的公共的变量文件**

可以在一个独立的playbook文件中定义公共变量，在其它的playbook文件中可以引用变量文件中的变量

此方式比playbook中定义的变量优化级高

范例：

```
[root@Ansible /data/ansible]# cat vars.yml 
username: hang 
groupname: nginx
[root@Ansible /data/ansible]# cat var.yaml 
---
- hosts: webservers
  vars_files:			#定位变量文件
    - vars.ym

  tasks:
  - name: mkdir diractory
    file: 
      path: /data/hang.log
      state: touch
      owner: "{{ username }}"
      group: "{{ groupname }}"

[root@Ansible /data/ansible]# ansible-playbook --syntax-check var.yaml 

playbook: var.yaml
[root@Ansible /data/ansible]# ansible-playbook var.yaml

#测试
[root@web-sC3eTNr3O9TRLR94 ~]# ls /data/hang.log -l
-rw-r--r-- 1 hang nginx 0 Dec 22 07:04 /data/hang.log

[root@web-hzuoQcWtDBiopZH ~]#ls /data/hang.log -l
-rw-r--r-- 1 hang nginx 0 Dec 22 07:04 /data/hang.log

[root@web-KwGCVbfNE6ToZZ3 ~]# ls /data/hang.log -l
-rw-r--r-- 1 hang nginx 0 Dec 22 07:04 /data/hang.log
```

### 3.9.5、**在主机清单中定义主机和主机组的变量**

#### **3.9.5.1** **所有项目的主机变量**

在inventory 主机清单文件中为指定的主机定义变量以便于在playbook中使用

范例：

```
[websrvs]
www1.dingbh.top http_port=80 maxRequestsPerChild=808
www2.dingbh.top http_port=8080 maxRequestsPerChild=909
```

#### **3.9.5.2** **所有项目的组（公共）变量**

在inventory 主机清单文件中赋予给指定组内所有主机上的在playbook中可用的变量，如果和主机变是

同名，优先级低于主机变量

#### 3.9.5.3、案例

```
[websrvs:vars]
http_port=80
ntp_server=ntp.wang.org
nfs_server=nfs.wang.org
[all:vars]
# --------- Main Variables ---------------

# Cluster container-runtime supported: docker, containerd
CONTAINER_RUNTIME="docker"

# Network plugins supported: calico, flannel, kube-router, cilium, kube-ovn
CLUSTER_NETWORK="calico"

# Service proxy mode of kube-proxy: 'iptables' or 'ipvs'
PROXY_MODE="ipvs"

# K8S Service CIDR, not overlap with node(host) networking
SERVICE_CIDR="192.168.0.0/16"

# Cluster CIDR (Pod CIDR), not overlap with node(host) networking
CLUSTER_CIDR="172.16.0.0/16"

# NodePort Range
NODE_PORT_RANGE="20000-60000"

# Cluster DNS Domain
CLUSTER_DNS_DOMAIN="magedu.local."
```

案例

```
[root@Ansible /data/ansible]# cat hostname2.yaml 
---
- hosts: webservers
  remote_user: root
  gather_facts: no


  tasks:
  - name: change hostname
    hostname:
      name: "{{ host }}-{{ domain }}"

#测试
[root@web-KwGCVbfNE6ToZZ3 ~]# hostname
www2-dingbh.top

[root@web-hzuoQcWtDBiopZH ~]#hostname
www1-dingbh.com

[root@web-sC3eTNr3O9TRLR94 ~]# hostname
www3-dingbh.top
```

### **3.9.6**、针对当前项目的主机和主机组的变量

上面的方式是针对所有项目都有效,而官方更建议的方式是使用ansible特定项目的主机变量和组变量，生产建议在每个项目对应的目录中创建额外的两个变量目录,分别是host_vars和group_vars

```
1、host_vars下面的文件名和主机清单主机名一致,针对单个主机进行变量定义
格式:host_vars/hostname
2、针对单个组进行变量定义
格式: gorup_vars/groupname
3、group_vars/all文件内定义的变量对所有组都有效
```

范例: 特定项目的主机和组变量

```
[root@Ansible /data/ansible/wordpress]# tree
.
├── group_vars
│   └── webservers
├── hostname2.yaml
└── host_vars
    ├── 192.168.188.7
    ├── 192.168.188.8
    └── 192.168.188.88

2 directories, 5 files

[root@Ansible /data/ansible/wordpress]# cat group_vars/webservers 
domain: dingbh.top

[root@Ansible /data/ansible/wordpress]# cat host_vars/192.168.188.*
id: 7
id: 8
id: 88

[root@Ansible /data/ansible/wordpress]# cat hostname2.yaml 
---
- hosts: webservers
  remote_user: root
  gather_facts: no


  tasks:
  - name: change hostname
    hostname:
      name: "web-{{ id }}{{ domain }}"


#测试
[root@Ansible /data/ansible/wordpress]# ansible webservers -m setup -a 'filter=ansible_hostname'
192.168.188.7 | SUCCESS => {
    "ansible_facts": {
        "ansible_hostname": "web-7",
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false
}
192.168.188.88 | SUCCESS => {
    "ansible_facts": {
        "ansible_hostname": "web-88",
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false
}
192.168.188.8 | SUCCESS => {
    "ansible_facts": {
        "ansible_hostname": "web-8",
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false
}
[root@Ansible /data/ansible/wordpress]# ansible webservers -m setup -a 'filter=ansible_nodename'
192.168.188.7 | SUCCESS => {
    "ansible_facts": {
        "ansible_nodename": "web-7.dingbh.top",
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false
}
192.168.188.88 | SUCCESS => {
    "ansible_facts": {
        "ansible_nodename": "web-88.dingbh.top",
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false
}
192.168.188.8 | SUCCESS => {
    "ansible_facts": {
        "ansible_nodename": "web-8.dingbh.top",
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false
}
```

方法2：直接cp配置文件

### 3.9.7、**register** **注册变量**

在playbook中可以使用register将捕获命令的输出保存在临时变量中，方便后续调用此变量,比如可以使用debug模块进行显示输出

```
[root@centos8 ~]#cat register1.yml 
- hosts: dbsrvs
 tasks:
    - name: get variable
     shell: hostname
     register: name
    - name: "print variable"
     debug:
       msg: "{{ name }}"                  #输出register注册的name变量的全部信息,注意变量要加" "引起来
        #msg: "{{ name.cmd }}"             #显示命令
        #msg: "{{ name.rc }}"             #显示命令成功与否
        #msg: "{{ name.stdout }}"         #显示命令的输出结果为字符串形式,所有结果都放在一行里显示,适合于结果是单行输出
        #msg: "{{ name.stdout_lines }}"   #显示命令的输出结果为列表形式,逐行标准输出,适用于多行显示
        #msg: "{{ name['stdout_lines'] }}" #显示命令的执行结果为列表形式,和效果上面相同
        #msg: "{{ name.stdout_lines[0] }}" #显示命令的输出结果的列表中的第一个元素
        
#说明
第一个 task 中，使用了 register 注册变量名为 name ；当 shell 模块执行完毕后，会将数据放到该变量中。
第二给 task 中，使用了 debug 模块，并从变量name中获取数据。
```

范例: 利用debug 模块输出变量

```
[root@Ansible /data/ansible]# cat register.yaml 
---
- hosts: webservers
  remote_user: root

  tasks:
  - name: genarate random
    shell: "openssl rand -base64 12 |tr -dc '[:alnum:]'"
    register:
       num

  - name: display
    debug:
      msg: "{{ num.stdout }}"
  - name: set hostname
    hostname:
      name: web-{{ num.stdout }}
```

## 3.10、Template模板

### 3.10.1、 template

template功能：可以根据和参考模块文件，动态生成相类似的配置文件

template文件存建议放于templates目录下，且命名为 .j2 结尾

yaml/yml 文件和templates目录平级，此时playbook中指定模版文件时可不用指定路径, 目录结构如下

示例：

```
 ./
 ├── temnginx.yml
 └── templates
         └── nginx.conf.j2
```

范例：利用template 同步nginx配置文件

```
#准备templates/nginx.conf.j2文件
[root@Ansible /data/ansible]# tree templates/
templates/
├── nginx.conf.j2
└── pc.conf.j2

0 directories, 2 files

[root@Ansible /data/ansible]# cat nginx_template.yaml 
---
- hosts: webservers
  remote_user: root
  gather_facts: yes 
  
  tasks:
  - name: install nginx
    yum:
      name: nginx
      state: present

  - name: config file
    template:						#使用template模块并改名
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf	#注意此处改成服务配置文件名称
    notify: restart service 
    tags:
      - config
  - name: config file
    template:
      src: pc.conf.j2
      dest: /etc/nginx/conf.d/pc.conf
    notify: restart service 
    tags:
      - config

  - name: create html data
    file:
      path: /data/website
      state: directory
      owner: nginx
      group: nginx

  - name: data pages
    unarchive:
      src: nginx_html.tar.gz
      dest: /data/website/
      remote_src: no
      owner: nginx
      group: nginx
    tags:
      - html
      - data
  - name: start service
    service:
      name: nginx
      state: started
      enabled: yes

  handlers:
    - name: restart service
      systemd:
        name: nginx
        state: restarted


#测试
[root@web-7 ~]# ps aux|grep nginx
root      39081  0.0  0.0  39308  1072 ?        Ss   09:50   0:00 nginx: master process /usr/sbin/nginx
nginx     39082  0.0  0.0  39800  1840 ?        S    09:50   0:00 nginx: worker process
nginx     39083  0.0  0.0  39800  1840 ?        S    09:50   0:00 nginx: worker process
nginx     39084  0.0  0.0  39800  1840 ?        S    09:50   0:00 nginx: worker process
nginx     39085  0.0  0.0  39800  1840 ?        S    09:50   0:00 nginx: worker process
root      39096  0.0  0.0 112808   964 pts/0    R+   09:51   0:00 grep --color=auto nginx

[root@web-88 ~]# ps aux|grep nginx
root      100795  0.0  0.0 103536  2268 ?        Ss   09:50   0:00 nginx: master process /usr/sbin/nginx
nginx     100796  0.0  0.2 124948  8088 ?        S    09:50   0:00 nginx: worker process
nginx     100797  0.0  0.2 124948  8128 ?        S    09:50   0:00 nginx: worker process
nginx     100798  0.0  0.2 124948  8128 ?        S    09:50   0:00 nginx: worker process
nginx     100799  0.0  0.2 124948  8128 ?        S    09:50   0:00 nginx: worker process
nginx     100800  0.0  0.2 124948  8132 ?        S    09:50   0:00 nginx: worker process
nginx     100801  0.0  0.2 124948  8132 ?        S    09:50   0:00 nginx: worker process
nginx     100802  0.0  0.2 124948  8064 ?        S    09:50   0:00 nginx: worker process
nginx     100803  0.0  0.2 124948  8132 ?        S    09:50   0:00 nginx: worker process
root      100824  0.0  0.0 221940  1196 pts/0    S+   09:51   0:00 grep --color=auto nginx

#查看配置文件已根据cpu数自动调整worker数
[root@web-7 ~]# cat /etc/nginx/nginx.conf|head -n5
user nginx;
worker_processes 4;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;


[root@web-88 ~]# cat /etc/nginx/nginx.conf|head -n5
user nginx;
worker_processes 8;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

```

### 3.10.2、template中使用流程控制 for 和 if

template中也可以使用流程控制 for 循环和 if 条件判断，实现动态生成文件功能

#### 3.10.2.1、for 循环

格式

```
{% for i in EXPR %}
 ...
{% endfor %}
#示例:
{% for i in range(1,10) %}
 server_name web{{i}};
{% endfor %}
```

范例：

```
#for.yaml文件
[root@Ansible /data/ansible]# cat for.yaml 
---
- hosts: webservers
  vars:
    vhost:
    - 8001
    - 8002
    - 8003
    - 8004


  tasks:
  - name: config files
    template:
      src: for.conf.j2
      dest: /etc/nginx/conf.d/for.conf

#template模板文件配置
[root@Ansible /data/ansible]# cat templates/for.conf.j2 
{%for i in vhost %}
server {
    listen {{ i }};
    server_name www.dingbh.top;
    root /data/website/;
    location = /basic_status {
        stub_status;
    }
}
{% endfor %}
```

测试：

```
#测试
[root@Ansible /data/ansible]# ansible-playbook for.yaml 

[root@web-7 ~]# cat /etc/nginx/conf.d/for.conf 
server {
    listen 8001;
    server_name www.dingbh.top;
    root /data/website/;
    location = /basic_status {
        stub_status;
    }
}
server {
    listen 8002;
    server_name www.dingbh.top;
    root /data/website/;
    location = /basic_status {
        stub_status;
    }
}
server {
    listen 8003;
    server_name www.dingbh.top;
    root /data/website/;
    location = /basic_status {
        stub_status;
    }
}
server {
    listen 8004;
    server_name www.dingbh.top;
    root /data/website/;
    location = /basic_status {
        stub_status;
    }
}

```

案例：自定义访问server_name及html文件路径.for实现

```
#定义yaml文件及变量
[root@Ansible /data/ansible]# cat for1.yaml 
---
- hosts: webservers
  vars:
    vhost:
    - listen: 8001
      host: www.a.com
      root: /data/website-a
    - listen: 8002
      host: www.b.com
      root: /data/website-b
    - listen: 8003
      host: www.c.com
      root: /data/website-c
    - listen: 8004
      host: www.d.com
      root: /data/website-d


  tasks:
  - name: config files
    template:
      src: for1.conf.j2
      dest: /etc/nginx/conf.d/for1.conf

#定义模板
[root@Ansible /data/ansible]# cat templates/for1.conf.j2 
{%for i in vhost %}
server {
    listen {{ i.listen }};
    server_name {{ i.host }};
    root {{ i.root }};
    location = /basic_status {
        stub_status;
    }
}
{% endfor %}


#测试
[root@web-7 ~]# cat /etc/nginx/conf.d/for1.conf 
server {
    listen 8001;
    server_name www.a.com;
    root /data/website-a;
    location = /basic_status {
        stub_status;
    }
}
server {
    listen 8002;
    server_name www.b.com;
    root /data/website-b;
    location = /basic_status {
        stub_status;
    }
}
server {
    listen 8003;
    server_name www.c.com;
    root /data/website-c;
    location = /basic_status {
        stub_status;
    }
}
server {
    listen 8004;
    server_name www.d.com;
    root /data/website-d;
    location = /basic_status {
        stub_status;
    }
}

```

#### 3.10.2.2、if 条件判断

在模版文件中还可以使用 if条件判断，决定是否生成相关的配置信息

范例：

```
[root@Ansible /data/ansible]# cat for2.yaml 
---
- hosts: webservers
  vars:
    vhost:
    - listen: 8001
      host: www.a.com
      root: /data/website-a
    - listen: 8002
      host: www.b.com
#      root: /data/website-b
    - listen: 8003
      host: www.c.com
      root: /data/website-c
    - listen: 8004
      host: www.d.com
#      root: /data/website-d


  tasks:
  - name: config files
    template:
      src: for2.conf.j2
      dest: /etc/nginx/conf.d/for2.conf


[root@Ansible /data/ansible]# cat templates/for2.conf.j2 
{%for i in vhost %}
server {
    listen {{ i.listen }};
    server_name {{ i.host }};
    {% if i.root is defined%}
root {{ i.root }};
    {% endif %}
location = /basic_status {
        stub_status;
    }
}
{% endfor %}

#测试
[root@web-7 ~]# cat /etc/nginx/conf.d/for2.conf 
server {
    listen 8001;
    server_name www.a.com;
    root /data/website-a;
    location = /basic_status {
        stub_status;
    }
}
server {
    listen 8002;
    server_name www.b.com;
    location = /basic_status {
        stub_status;
    }
}
server {
    listen 8003;
    server_name www.c.com;
    root /data/website-c;
    location = /basic_status {
        stub_status;
    }
}
server {
    listen 8004;
    server_name www.d.com;
    location = /basic_status {
        stub_status;
    }
}
```

实例：生成主从复制配置文件

```sh
[root@Ansible /data/ansible]# cat hosts

[webservers]
192.168.188.8  role=master
192.168.188.7  role=slave 
192.168.188.88 role=slave

[root@Ansible /data/ansible]# cat mysql.yaml 
---
- hosts: webservers
  vars:
    vhost:
    - server_id: 101
      role: master
    - server_id: 102
      role: slave
  tasks:
  - name: config files
    template:
      src: my.cnf.j2
      dest: /tmp/my.cnf
      
[root@Ansible /data/ansible]# cat templates/my.cnf.j2 
[mysqld]
{% if role == "master" %}
server_id=101
log-bin
{% elif role == "slave" %}
server_id=102;
readonly
{% endif %}


#生成的结果
[root@web-7 ~]# cat /tmp/my.cnf 
[mysqld]
server_id=102;
readonly

[root@web-8 ~]#cat /tmp/my.cnf 
[mysqld]
server_id=101
log-bin

[root@web-88 ~]# cat /tmp/my.cnf 
[mysqld]
server_id=102;
readonly
```

## 3.11、使用循环迭代

迭代：当有需要重复性执行的任务时，可以使用迭代机制

### 3.11.1、**迭代** **loop (with_items)**

对迭代项的引用，固定内置变量名为"item"

要在task中使用with_items给定要迭代的元素列表

注意: ansible2.5版本后,可以用loop代替with_items

**列表元素格式：**

```
- 字符串
- 字典
```

范例：

```
---
- hosts: websrvs
 remote_user: root
  
 tasks:
    - name: add several users
     user: name={{ item }} state=present groups=wheel
     loop:
        - testuser1
        - testuser2
        - testuser3
        
#上面语句的功能等同于下面的语句
    - name: add several users
     user: name=testuser1 state=present groups=wheel
    - name: add several users
     user: name=testuser2 state=present groups=wheel
    - name: add several users
     user: name=testuser3 state=present groups=wheel
```

范例: 安装多个软件包

```
# cat install_packages.yml
- hosts: webservers
 tasks:
    - name: Installed Httpd Php-fpm Package
     yum: name={{ pack }} state=latest
     vars:
       pack:
          - httpd
          - php-fpm
#方法2
# cat install_packages2.yml
- hosts: webservers
 tasks:
    - name: Installed Httpd Php-fpm Package
     yum: 
        name: "{{ item }}"
        state: latest
     loop:
       - httpd
       - php-fpm

# cat install_packages3.yml
---
- hosts：websrvs
 remote_user: root
  
 tasks
    - name: install some packages
     yum: name={{ item }} state=present
     loop:
        - nginx
        - memcached
        - php-fpm
```

范例：卸载 mariadb 

```
---
#remove mariadb server
- hosts: appsrvs:!10.0.0.8
 remote_user: root
 tasks:
    - name: stop service
     shell: /etc/init.d/mysqld stop
    - name: delete files and dir
     file: path={{ item }} state=absent
     with_items:
        - /usr/local/mysql
        - /usr/local/mariadb-10.2.27-linux-x86_64
        - /etc/init.d/mysqld
        - /etc/profile.d/mysql.sh
        - /etc/my.cnf
        - /data/mysql
    - name: delete user
     user: name=mysql state=absent remove=yes
```

范例：

```
---
- hosts: websrvs
 remote_user: root
 tasks:
    - name: copy file
     copy: src={{ item }} dest=/tmp/{{ item }}
     with_items:
        - file1
        - file2
        - file3
    - name: yum install httpd
     yum: name={{ item }}  state=present 
     with_items:
        - apr
        - apr-util
        - httpd
```

范例: 初始化安装软件包

```
- name: Installed Packages All
 yum:
   name: "{{ item }}"
   state: present
 loop:
     - nfs-utils
     - net-tools
     - wget
     - rsync
     - tree
     - lrzsz
     - vim
     - unzip
     - httpd-tools
     - bash-completion
     - iftop
     - iotop
     - gzip
     - bzip2
     - xz
     - psmisc
     - bind-utils
     - MySQL-python
```

### **3.11.2** **迭代嵌套子变量**

在迭代中，还可以嵌套子变量，关联多个变量在一起使用

示例： 

```
---
- hosts: websrvs
 remote_user: root
 tasks:
   - name: add some groups
     group: name={{ item }} state=present
     loop:
       - nginx
       - mysql
       - apache
   - name: add some users
     user: name={{ item.user }} group={{ item.group }} uid={{item.uid}} state=present
     loop:
       - { user: 'nginx', group: 'nginx',uid: "80" }
       - { user: 'mysql', group: 'mysql' ,uid: "3306"}
       - { user: 'apache', group: 'apache',uid: "8080"}
```

范例：创建多个组及用户

```
[root@Ansible /data/ansible]# cat group.yaml 
---
- hosts: webservers
  remote_user: root

  tasks:
  - name: create groups
    group: 
      name: "{{ item }}"
      state: present
    loop:
      - g1
      - g2
      - g3

  - name: create users
    user: 
      name: "{{ item.name }}"
      group: "{{ item.group }}"
      home: "{{ item.home }}"
      create_home: yes
      state: present
    loop:
      - { name: 'user1', group: 'g1', home: '/data/user1' }
      - { name: 'user2', group: 'g2', home: '/data/user2' }
      - { name: 'user3', group: 'g3', home: '/data/user3' } 
```



### 3.11.3、编译安装nginx服务

#### 3.11.3.1、准备yaml文件

```
[root@Ansible /data/ansible]# cat install_nginx.yaml 
---
- hosts: webservers
  remote_user: root
  vars:
    version: "1.24.0"
    url: "https://nginx.org/download/nginx-{{ version }}.tar.gz"
    dir: "/usr/local/src"
    install_dir: "/apps/nginx/"

  tasks:
  - name: install depend on pages
    yum:
      name: "{{ item }}"
      state: latest
    loop:
      - gcc
      - make
      - pcre-devel
      - openssl-devel
      - zlib-devel
      - perl-ExtUtils-Embed
  - name: get nginx source
    unarchive:
      src: "{{ url }}"
      dest: "{{ dir }}"
      remote_src: yes
      owner: nginx
      group: nginx
  - name: configure and install
    shell:
      cmd: "./configure --prefix=/apps/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-stream --with-stream_ssl_module --with-stream_realip_module && make -j {{ ansible_processor_vcpus }} && make install"
      chdir: "{{ dir }}/nginx-{{ version }}"
      creates: "{{ install_dir }}" 

  - name: create user
    user:
      name: nginx
      system: yes
      shell: /sbin/nologin
      create_home: no
  - name:
    template:
      src: nginx.conf.j2
      dest: "{{ install_dir }}conf/nginx.conf"
      owner: nginx
      group: nginx
    notify: restart service

  - name: create nginx_config files
    file:
      path: "{{install_dir}}conf.d"
      state: directory
      owner: nginx
      group: nginx

  - name: copy nginx_config files
    template:
      src: blog.conf.j2
      dest: "{{ install_dir }}conf.d/pc.conf"
      owner: nginx
      group: nginx
    notify: restart service
      
  - name: create html data files
    file:
      path: /data/website
      state: directory
      owner: nginx
      group: nginx

  - name: data pages
    unarchive:
      src: nginx_html.tar.gz
      dest: /data/website/
      remote_src: no
      owner: nginx
      group: nginx
    tags:
      - html
      - data

  - name: nginx service
    template:
      src: nginx.service.j2
      dest: /usr/lib/systemd/system/nginx.service

  - name: start service
    systemd:
      name: nginx
      state: started
      daemon_reload: yes
      enabled: yes 

  handlers:
    - name: restart service
      systemd:
        name: nginx
        state: restarted
```

#### 3.11.3.2、准备nginx配置模板文件

```
[root@Ansible /data/ansible]# tree templates/
templates/
├── blog.conf.j2
├── eth0.yaml.j2
├── for1.conf.j2
├── for2.conf.j2
├── for.conf.j2
├── my.cnf.j2
├── nginx.conf
├── nginx.conf.j2
├── nginx.service.j2
└── pc.conf.j2

0 directories, 10 files


[root@Ansible /data/ansible]# cat templates/nginx.conf.j2 
user nginx;
worker_processes  {{ ansible_processor_vcpus*2 }};

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;
    include /etc/nginx/conf.d/*.conf;
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```

#### 3.11.3.3、准备nginx实例pc.nginx.j2文件

```
[root@Ansible /data/ansible]# cat templates/pc.conf.j2 

server {
    listen 80;
    server_name www.dingbh.top;
    root /data/website/;
    location = /basic_status {
        stub_status;
    }
}

```

#### 3.11.3.4、准备web界面文件

```
[root@Ansible /data/ansible]# tree nginx_html
nginx_html
├── index1.html
├── index.html
├── test2.html
└── test.html

0 directories, 4 files
[root@Ansible /data/ansible]# ls -l nginx_html.tar.gz 
-rw-r--r-- 1 root root 262 Dec 23 13:25 nginx_html.tar.gz

[root@Ansible /data/ansible]# cat nginx_html/index.html 
<h1> welcome to ansible website V1.0 </h1>

```

#### 3.11.3..5、ansible管理端执行yaml脚本

```
[root@Ansible /data/ansible]# ansible-playbook install_nginx.yaml 
[root@Ansible /data/ansible]# ansible-playbook install_nginx.yaml 

PLAY [webservers] ****************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************
ok: [192.168.188.88]
ok: [192.168.188.8]

TASK [install depend on pages] ***************************************************************************************************************************************
ok: [192.168.188.88] => (item=gcc)
ok: [192.168.188.8] => (item=gcc)
ok: [192.168.188.88] => (item=make)
ok: [192.168.188.8] => (item=make)
ok: [192.168.188.88] => (item=pcre-devel)
ok: [192.168.188.8] => (item=pcre-devel)
ok: [192.168.188.88] => (item=openssl-devel)
ok: [192.168.188.8] => (item=openssl-devel)
ok: [192.168.188.88] => (item=zlib-devel)
ok: [192.168.188.8] => (item=zlib-devel)
ok: [192.168.188.88] => (item=perl-ExtUtils-Embed)
ok: [192.168.188.8] => (item=perl-ExtUtils-Embed)

TASK [get nginx source] **********************************************************************************************************************************************
ok: [192.168.188.8]
ok: [192.168.188.88]

TASK [configure and install] *****************************************************************************************************************************************
ok: [192.168.188.8]
ok: [192.168.188.88]

TASK [create user] ***************************************************************************************************************************************************
ok: [192.168.188.8]
ok: [192.168.188.88]

TASK [template] ******************************************************************************************************************************************************
changed: [192.168.188.88]
changed: [192.168.188.8]

TASK [create nginx_config files] *************************************************************************************************************************************
ok: [192.168.188.8]
ok: [192.168.188.88]

TASK [copy nginx_config files] ***************************************************************************************************************************************
ok: [192.168.188.8]
ok: [192.168.188.88]

TASK [create html data files] ****************************************************************************************************************************************
ok: [192.168.188.8]
ok: [192.168.188.88]

TASK [data pages] ****************************************************************************************************************************************************
ok: [192.168.188.8]
ok: [192.168.188.88]

TASK [nginx service] *************************************************************************************************************************************************
ok: [192.168.188.8]
ok: [192.168.188.88]

TASK [start service] *************************************************************************************************************************************************
ok: [192.168.188.8]
ok: [192.168.188.88]

RUNNING HANDLER [restart service] ************************************************************************************************************************************
changed: [192.168.188.88]
changed: [192.168.188.8]

PLAY RECAP ***********************************************************************************************************************************************************
192.168.188.8              : ok=13   changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
192.168.188.88             : ok=13   changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   


```

#### 3.11.3.6、访问测试

```
[root@Ansible /data/ansible]# curl www.dingbh.top
<h1> welcome to ansible website V1.0 </h1>

```

192.168.188.8

![image-20231225215415379](Ansible自动化/image-20231225215415379.png)

192.168.188.88

![image-20231225215439338](Ansible自动化/image-20231225215439338.png)

## 3.12、条件判断when



实例：判断操作系统OS版本

```
[root@Ansible /data/ansible]# cat when.yaml 
---
- hosts: all 
  remote_user: root


  tasks:
  - name: install http apache
    yum:
      name: httpd
    when:
      - ansible_distribution_file_variety == "RedHat"
  - name: install apache2
    apt:
      name: apache2
    when:
      - ansible_distribution_file_variety == "Debian"

```

实例：关闭Ubuntu主机

```
[root@Ansible /data/ansible]# cat when2.yaml 
---
- hosts: all
  remote_user: root


  tasks:
  - name: shutdown ubuntu hosts
    reboot:
    when: 
      - ansible_facts['distribution'] == "Ubuntu"
      - ansible_distribution_major_version ==  "18"

```



## 3.13、分组block

当想在满足同样条件下，执行多个任务时，就需要分组。而不再针对每个任务都是用 when

```
[root@ansible ansible]#cat block.yml
---
- hosts: localhost
  
 tasks:
    - block:
        - debug: msg="first"
        - debug: msg="second"
     when:
        - ansible_facts['distribution'] == "CentOS"
        - ansible_facts['distribution_major_version'] == "8"
        
#相当于下面写法
---
- hosts: localhost
  
 tasks:
     - debug: msg="first"
       when:
         - ansible_facts['distribution'] == "CentOS"
         - ansible_facts['distribution_major_version'] == "8"
     
     - debug: msg="second"
       when:
         - ansible_facts['distribution'] == "CentOS"
         - ansible_facts['distribution_major_version'] == "8"
```

案例：实现多任务的执行

```
[root@Ansible /data/ansible]# cat block.yaml 
---
- hosts: all
  remote_user: root


  tasks:
  - name: run some tasks on ubuntu systems
    block:

      - shell: wall task1
        register: work1
      - debug:
          msg: "{{ work1 }}"
          
      - shell: wall task2
        register: work
      - debug:
          msg: "{{ work }}"

    when: 
      - ansible_facts['distribution'] == "Ubuntu"
      - ansible_distribution_major_version ==  "18"
```

## 3.14、**changed_when**

### 3.14.1、**关闭** **changed** **状态**

当确定某个task不会对被控制端做修改时但执行结果却显示是黄色的changed状态,可以通过changed_when: false 关闭changed状态

```
[root@Ansible /data/ansible]# cat test_change.yaml

---
- hosts: webservers


  tasks:
    - name: check nginx service
      shell:
        cmd: ps aux|grep nginx
      changed_when: false #stop changed status

```

### 3.14.2、利用changed_when检查task返回结果

​	changed_when 检查task返回结果,决定是否继续向下执行

```
[root@ansible ansible]#cat test_changed_when.yml

---
- hosts: websrvs
 tasks:
   - name: install nginx
     yum: name=nginx
   - name: config file
     template: src="nginx.conf.j2" dest="/etc/nginx/nginx.conf"
     notify: restart nginx
   - name: check config
     shell: /usr/sbin/nginx -t
     register: check_nginx_config 
     changed_when:
       - check_nginx_config.stdout.find('successful') #如果执行结果中有successful字符串,则继续执行,如果没有则停止向下执行
       - false        #nginx -t 每次成功执行是changed状态,关闭此changed状态
   - name: start service
     service: name=nginx state=started enabled=yes
 handlers:
   - name: restart nginx
     service: name=nginx state=restarted
     
```

实例：changed_when实现nginx服务语法的健康检查

```
[root@ansible ansible]#cat test_changed_when.yml
 
---
- hosts: webservers
  remote_user: root
  vars:
    version: "1.24.0"
    url: "https://nginx.org/download/nginx-{{ version }}.tar.gz"
    dir: "/usr/local/src"
    install_dir: "/apps/nginx/"

  tasks:
  - name: install depend on pages
    yum:
      name: "{{ item }}"
      state: latest
    loop:
      - gcc
      - make
      - pcre-devel
      - openssl-devel
      - zlib-devel
      - perl-ExtUtils-Embed
  - name: get nginx source
    unarchive:
      src: "{{ url }}"
      dest: "{{ dir }}"
      remote_src: yes
      owner: nginx
      group: nginx
  - name: configure and install
    shell:
      cmd: "./configure --prefix=/apps/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-stream --with-stream_ssl_module --with-stream_realip_module && make -j {{ ansible_processor_vcpus }} && make install"
      chdir: "{{ dir }}/nginx-{{ version }}"
      creates: "{{ install_dir }}" 

  - name: create user
    user:
      name: nginx
      system: yes
      shell: /sbin/nologin
      create_home: no
  - name:
    template:
      src: nginx.conf.j2
      dest: "{{ install_dir }}conf/nginx.conf"
      owner: nginx
      group: nginx
    notify: restart service

  - name: create nginx_config files
    file:
      path: "{{install_dir}}conf.d"
      state: directory
      owner: nginx
      group: nginx

  - name: copy nginx_config files
    template:
      src: blog.conf.j2
      dest: "{{ install_dir }}conf.d/pc.conf"
      owner: nginx
      group: nginx
    notify: restart service

  - name: check config      
    shell:
      cmd: "{{ install_dir }}sbin/nginx -t"
    register: check_nginx_config
    changed_when:
      - check_nginx_config.stdout.find('successful') #如果执行结果中有successful字符串,则继续执行,如果没有则停止向下执行
      - false   #nginx -t 每次成功执行是changed状态,关闭此changed状态

  - name: create html data files
    file:
      path: /data/website
      state: directory
      owner: nginx
      group: nginx

  - name: data pages
    unarchive:
      src: nginx_html.tar.gz
      dest: /data/website/
      remote_src: no
      owner: nginx
      group: nginx
    tags:
      - html
      - data

  - name: nginx service
    template:
      src: nginx.service.j2
      dest: /usr/lib/systemd/system/nginx.service

  - name: start service
    systemd:
      name: nginx
      state: started
      daemon_reload: yes
      enabled: yes 

  handlers:
    - name: restart service
      systemd:
        name: nginx
        state: restarted

```

## 3.15、滚动执行

管理节点过多导致的超时问题解决方法

默认情况下，Ansible将尝试并行管理playbook中所有的机器。对于滚动更新用例，可以使用serial关键字定义Ansible一次应管理多少主机，还可以将serial关键字指定为百分比，表示每次并行执行的主机数占总数的比例

范例：

```
#vim test_serial.yml
---
- hosts: all
 serial: 2  #每次只同时处理2个主机,将所有task执行完成后,再选下2个主机再执行所有task,直至所
有主机
 gather_facts: False
 tasks:
   - name: task one
 comand: hostname
   - name: task two
     command: hostname
```

范例：

```
- name: test serail
 hosts: all
 serial: "20%"   #每次只同时处理20%的主机
```

范例：

```
[root@Ansible /data/ansible]# cat test_serial.yaml 
---
- hosts: all
  serial: "20%"   #每次执行主机数量的20%
  gather_facts: false


  tasks:
  - name: task1
    debug:
      msg: "task1"

  - name: task2
    debug:
      msg: "task2"

```

案例

```
[root@ansible ansible]#cat test_serial.yml
---
- hosts: websrvs
 serial: 1
  
 tasks:
    - name: task1
     shell: wall "{{ansible_nodename}} is running task1"
    - name: task2
     shell: wall "{{ansible_nodename}} is running task2"
    - name: task3
     shell: wall "{{ansible_nodename}} is running task3"
```

小知识：

```
vim语法搜索高亮两种设置方法

1、Linux系统永久设置语法搜索高亮
[root@Ansible /data/ansible]# vim /root/.vimrc 

  1 set ts=2
  2 set nu
  3 set et
  4 set hlsearch   #开启此项
  
2、Linux系统临时设置语法搜索高亮

vim打开文档→命令行中输入:set hlsearch


关闭高亮显示
vim打开文档→命令行中输入:nohl
```

## 3.16、委派至其它主机执行

利用委托技术,可以在非当前被控主机的其它主机上执行指定操作

范例: 将任务委派给指定的主机执行

```
[root@Ansible /data/ansible]# cat delegate.yaml 
---
- hosts: webservers 
  


  tasks: 
  - name: show ip address
    command: hostname -I
    delegate_to: "192.168.188.7"
    delegate_facts: true

```

范例: 创建普通用户基于ssh key验证

```
[root@Ansible /data/ansible]# cat ssh_key.yaml 
---
- hosts: webservers
  vars:
  - user_name: cios_key

  tasks:
  - name: create manager {{ user_name }}
    user:
      name: "{{ user_name }}"
      generate_ssh_key: yes
      ssh_key_bits: 2048
      ssh_key_file: "/home/{{ user_name }}/.ssh/id_rsa"
    register: user_message
    delegate_to: localhost
    run_once: true


#node 节点打印
  - name: 打印管理用户的key结果
    debug:
      msg: "{{ user_message.ssh_public_key }}"

# 在被控制端创建用户

  - name: 被控制端创建用户
    user:
      name: "{{ user_name }}"

  - name: 被控制端创建用户.ssh目录
    file:
      path: "/home/{{ user_name }}/.ssh"
      state: directory
      owner: "{{ user_name }}"
      group: "{{ user_name }}"
      mode: "0700"

  - name: 将管理端{{ user_name }}用户的key存储到被控端
    copy:
      content: "{{ user_message.ssh_public_key }}"
      dest: "/home/{{ user_name }}/.ssh/authorized_keys"
      owner: "{{ user_name }}"
      group: "{{ user_name }}"
      mode: "0600"

  - name: 配置被控制端sudo提权，最后追加一行
    lineinfile:
      path: /etc/sudoers
      line: "{{ user_name }} ALL=(ALL) NOPASSWD:ALL"

#测试
[root@Ansible /data/ansible]# su - cios_key
$ ssh 192.168.188.7
Last login: Mon Oct  2 18:40:32 2023 from 192.168.188.100
[cios_key@centos7 ~]$ exit
logout
Connection to 192.168.188.7 closed.
$ ssh 192.168.188.8
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Fri May 12 01:07:12 2023
[cios_key@Rocky8 ~]$exit
logout
Connection to 192.168.188.8 closed.
$ ssh 192.168.188.88
Warning: Permanently added '192.168.188.88' (ECDSA) to the list of known hosts.
Activate the web console with: systemctl enable --now cockpit.socket

[cios_key@Rocky8 ~]$ exit
logout
Connection to 192.168.188.88 closed.
$ exit
```

## 3.17、**只执行一次**

利用 run_once 指令可以只执行一次,而非在所有被控主机都执行

```
[root@Ansible /data/ansible]# cat run_once.yaml 
---
- hosts: webservers

  tasks:
  - name:
    command: hostname
    run_once: true


[root@Ansible /data/ansible]# ansible-playbook run_once.yaml 

PLAY [webservers] ****************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************
ok: [192.168.188.7]
ok: [192.168.188.88]
ok: [192.168.188.8]

TASK [command] *******************************************************************************************************************************************************
changed: [192.168.188.8]

PLAY RECAP ***********************************************************************************************************************************************************
192.168.188.7              : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
192.168.188.8              : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
192.168.188.88             : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

## 3.18、环境变量

临时修改环境变量

```
[root@Ansible /data/ansible]# cat environment.yaml 
---
- hosts: webservers

  tasks:
  - name: print messager
    shell: echo $PATH
    environment:
      PATH: "/apps/nginx/bin:{{ ansible_env.PATH}}"
  - name: display messager
    shell:
      cmd: "echo $PATH"
```

## 3.19、Yaml文件的相互调用

### **3.19.1 include**

利用include 或 include_tasks 可以在某个task中调用其它的只有task内容的yaml文件

```
[root@Ansible /data/ansible]# cat a.yaml 
---
- hosts: webservers

  tasks:
  - name: run a job
    command: wall run a job
  - name: excute b.yaml
    include: b.yaml			#调用另一个yaml文件
    #include_tasks: b.yml 	#另一种写法

[root@Ansible /data/ansible]# cat b.yaml 
- name: run b job
  command: wall run b job

```

### 3.19.2、import_playbook

还可以将多个包含完整内容的yml文件由一个yml统一调用

```
[root@Ansible /data/ansible]# cat a1.yaml 
---
- hosts: webservers

  tasks:
  - name: run a1 job
    command: wall run a1 job
  - name: excute b.yaml
    include: b.yaml
[root@Ansible /data/ansible]# cat a2.yaml 
---
- hosts: webservers

  tasks:
  - name: run a2 job
    command: wall run a2 job
  - name: excute b.yaml
    include: b.yaml
    
[root@Ansible /data/ansible]# cat b.yaml 
- name: run b job
  command: wall run b job
  
[root@Ansible /data/ansible]# cat main.yaml 
- import_playbook: a1.yaml
- import_playbook: a2.yaml
     

[root@Ansible /data/ansible]# ansible-playbook main.yaml 

PLAY [webservers] ****************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************
ok: [192.168.188.7]
ok: [192.168.188.88]
ok: [192.168.188.8]

TASK [run a1 job] ****************************************************************************************************************************************************
changed: [192.168.188.7]
changed: [192.168.188.88]
changed: [192.168.188.8]

TASK [run b job] *****************************************************************************************************************************************************
changed: [192.168.188.7]
changed: [192.168.188.8]
changed: [192.168.188.88]

PLAY [webservers] ****************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************
ok: [192.168.188.7]
ok: [192.168.188.88]
ok: [192.168.188.8]

TASK [run a2 job] ****************************************************************************************************************************************************
changed: [192.168.188.7]
changed: [192.168.188.8]
changed: [192.168.188.88]

TASK [run b job] *****************************************************************************************************************************************************
changed: [192.168.188.7]
changed: [192.168.188.8]
changed: [192.168.188.88]

PLAY RECAP ***********************************************************************************************************************************************************
192.168.188.7              : ok=6    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
192.168.188.8              : ok=6    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
192.168.188.88             : ok=6    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

# 4、Roles角色

角色是ansible自1.2版本引入的新特性，用于层次性、结构化地组织playbook。roles能够根据层次型结构自动装载变量文件、tasks以及handlers等。要使用roles只需要在playbook中使用include指令即可。简单来讲，roles就是通过分别将变量、文件、任务、模板及处理器放置于单独的目录中，并可以便捷地include它们的一种机制。角色一般用于基于主机构建服务的场景中，但也可以是用于构建守护进程等场景中

运维复杂的场景：建议使用 roles，代码复用度高

roles：多个角色的集合目录， 可以将多个的role，分别放至roles目录下的独立子目录中,如下示例

```
roles/
 mysql/
 nginx/
 tomcat/
 redis/
```

默认roles存放路径

```
/root/.ansible/roles
/usr/share/ansible/roles
/etc/ansible/roles
```

官方文档: 

```
https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html
```

## 4.1、Ansible Roles目录编排

roles目录结构如下所示

![image-20240104095943239](Ansible自动化/image-20240104095943239.png)

```
roles目录结构：

#任务执行

playbook1.yml
playbook2.yml

[root@Ansible /data/ansible]# tree roles/		#任务材料
roles/
├── mysql
│   ├── files
│   ├── handles
│   ├── tasks
│   └── template
├── nginx
│   ├── files
│   ├── handles
│   ├── tasks
│   └── template
├── php-fpm
│   ├── files
│   ├── handles
│   ├── tasks
│   └── template
├── redis
│   ├── files
│   ├── handles
│   ├── tasks
│   └── template
└── tomcat
    ├── files
    ├── handles
    ├── tasks
    └── template

25 directories, 0 files

```

**Roles****各目录作用**

```
roles/project/ :项目名称,有以下子目录
	files/ ：存放由copy或script模块等调用的文件
	templates/：template模块查找所需要模板文件的目录
	tasks/：定义task,role的基本元素，至少应该包含一个名为main.yml的文件；其它的文件需要在此文件中通过include进行包含
	handlers/：至少应该包含一个名为main.yml的文件；此目录下的其它的文件需要在此文件中通过include进行包含
vars/：定义变量，至少应该包含一个名为main.yml的文件；此目录下的其它的变量文件需要在此文件中通过include进行包含,也可以通过项目目录中的group_vars/all定义变量,从而实现角色通用代码和项目数据的分离
meta/：定义当前角色的特殊设定及其依赖关系,至少应该包含一个名为main.yml的文件，其它文件需在此文件中通过include进行包含
default/：设定默认变量时使用此目录中的main.yml文件，比vars的优先级低
```

实例：LNMP架构实现

```
[root@Ansible /data/ansible]# tree *role*
mysql_role.yaml [error opening dir]
nginx_role.yaml [error opening dir]
php-fpm_role.yaml [error opening dir]
redis_role.yaml [error opening dir]
roles
├── mysql
│   ├── files
│   ├── handles
│   ├── tasks
│   └── template
├── nginx
│   ├── files
│   ├── handles
│   ├── tasks
│   └── template
├── php-fpm
│   ├── files
│   ├── handles
│   ├── tasks
│   └── template
├── redis
│   ├── files
│   ├── handles
│   ├── tasks
│   └── template
└── tomcat
    ├── files
    ├── handles
    ├── tasks
    └── template

25 directories, 0 files
```



## **4.2** **创建** **role**

创建role的步骤

```
1 创建role的目录结构.在以roles命名的目录下分别创建以各角色名称命名的目录，如mysql等,在每个角色命名的目录中分别创建相关的目录和文件,比如tasks、files、handlers、templates和vars等目录；用不到的目录可以创建为空目录，也可以不创建
2 编写和准备指定role的功能文件,包括: tasks,templates,vars等相关文件
3 编写playbook文件调用上面定义的role,应用到指定的主机
```

针对大型项目使用Roles进行编排

范例: 利用 ansible-galaxy 创建角色目录的结构

```
#创建初始化目录结构
[root@Ansible /opt]# ansible-galaxy role init test_role #生成初始话role目录
[root@Ansible /opt]# tree test_role/
test_role/
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── README.md
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml

8 directories, 8 files
```

范例：roles的目录结构

```
nginx-role.yml 
roles/
└── nginx 
     ├── files
     │   └── nginx.conf
     ├── tasks
     │   ├── groupadd.yml 
     │   ├── install.yml 
     │   ├── main.yml 
     │   ├── restart.yml 
     │   └── useradd.yml 
     └── vars 
         └── main.yml
```

## **4.3 Playbook** **调用角色**

调用角色方法1：

```
---
- hosts: websrvs
 remote_user: root
 roles:
 - mysql
 - memcached
 - nginx  
```

调用角色方法2：

键role用于指定角色名称，后续的k/v用于传递变量给角色

```
---
- hosts: all
 remote_user: root
 roles:
   - role: mysql
     username: mysql
   - { role: nginx, username: nginx }
```

调用角色方法3：

还可基于条件测试实现角色调用

```
---
- hosts: all
 remote_user: root
 roles:
   - { role: nginx, username: nginx, when: ansible_distribution_major_version == '7' }
```

范例：

```
---
- hosts: webservers
 roles:
   - common
   - role: foo_app_instance
     vars:
       dir: '/opt/a'
       app_port: 5000
     tags: typeA
   - role: foo_app_instance
     vars:
       dir: '/opt/b'
       app_port: 5001
     tags: typeB
```

范例：

```
---
- hosts: webservers
 roles:
   - { role: foo, vars: { message: "first" } }
   - { role: foo, vars: { message: "second" } }
```

## **4.4 Roles** **中** **Tags** **使用**

```
[root@ansible ~]#vi app-role.yml
---
#可以有多个play
- hosts: lbserver
 roles:
    - role: haproxy
    - role: keepalived
    
- hosts: appsrvs
 remote_user: root
 roles:
    - { role: nginx ,tags: [ 'nginx', 'web' ] ,when: 
ansible_distribution_major_version == "6" }
    - { role: httpd ,tags: [ 'httpd', 'web' ] }
    - { role: mysql ,tags: [ 'mysql', 'db' ] }
    - role: mariadb 
     tags: 
        - mariadb
        - db
 tags: app  #play的tag
[root@ansible ~]#ansible-playbook --tags="nginx,mysql" app-role.yml
```

## **4.5** **实战案例**

### 4.5.1、实现httpd角色

```
#创建角色相关的目录
[root@ansible ~]#mkdir -pv /data/ansible/roles/httpd/{tasks,handlers,files}
  
#创建角色相关的文件
[root@ansible ~]#cd /data/ansible/roles/httpd/
#main.yml 是task的入口文件
[root@ansible ~]#vim tasks/main.yml
- include: group.yml
- include: user.yml
- include: install.yml
- include: config.yml
- include: index.yml
- include: service.yml
[root@ansible ~]#vim tasks/group.yml
- name: create apache group
 group: name=apache system=yes gid=80
  
[root@ansible ~]#vim tasks/user.yml
- name: create apache user
 user: name=apache system=yes shell=/sbin/nologin home=/var/www/ uid=80
group=apache
[root@ansible ~]#vim tasks/install.yml
- name: install httpd package
 yum: name=httpd
#注意: 文件是放在files目录下,但src的路径无需写files目录名
[root@ansible ~]#vim tasks/config.yml
- name: config file
 copy: src=httpd.conf dest=/etc/httpd/conf/ backup=yes
 notify: restart
[root@ansible ~]# tasks/index.yml
- name: index.html
 copy: src=index.html dest=/var/www/html/
[root@ansible ~]#vim tasks/service.yml
- name: start service
  service: name=httpd state=started enabled=yes
[root@ansible ~]#vim handlers/main.yml
- name: restart
  service: name=httpd state=restarted
  
#在files目录下准备两个文件
[root@ansible ~]#ls files/
httpd.conf index.html
[root@ansible ~]#tree /data/ansible/roles/httpd/
/data/ansible/roles/httpd/
├── files
│   ├── httpd.conf
│   └── index.html
├── handlers
│   └── main.yml
└── tasks
   ├── config.yml
   ├── group.yml
   ├── index.yml
   ├── install.yml
   ├── main.yml
   ├── service.yml
   └── user.yml
3 directories, 10 files
#在playbook中调用角色
[root@ansible ~]#vim /data/ansible/role_httpd.yml
---
# httpd role
- hosts: websrvs
 remote_user: root
 roles:
    - httpd
    
#运行playbook
[root@ansible ~]#ansible-playbook /data/ansible/role_httpd.yml
```

### 4.5.2、实现nginx角色

```
[root@ansible ~]#mkdir -pv 
/data/ansible/roles/nginx/{tasks,handlers,templates,vars}

#创建task文件
[root@ansible ~]#cd /opt/roles/nginx/
[root@Ansible /opt/roles/nginx]# cat tasks/main.yml 
- name: install depend on pages
  yum:
    name: "{{ item }}"
    state: latest
  loop:
    - gcc
    - make
    - pcre-devel
    - openssl-devel
    - zlib-devel
    - perl-ExtUtils-Embed

- name: create {{ group }}
  group:
    name: "{{ group }}"
    gid: "{{ gid }}"

- name: create {{ user }}
  user:
    name: "{{ user }}"
    system: yes
    uid: "{{ uid }}"
    shell: /sbin/nologin
    create_home: no
    group: "{{ group }}"

- name: get nginx source
  include: get_nginx.yml

- name: configure and install
  shell:
    cmd: "./configure --prefix=/apps/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-stream --with-stream_ssl_module --with-stream_realip_module && make -j {{ ansible_processor_vcpus }} && make install"
    chdir: "{{ dir }}/nginx-{{ version }}"
    creates: "{{ install_dir }}" 

- name:
  template:
    src: nginx.conf.j2
    dest: "{{ install_dir }}/conf/nginx.conf"
    owner: "{{ user }}"
    group: "{{ group }}"
  notify: restart service

- name: create nginx_config files
  file:
    path: "{{install_dir}}/conf.d"
    state: directory
    owner: "{{ user }}"
    group: "{{ group }}"

- name: copy nginx_config files
  template:
    src: blog.conf.j2
    dest: "{{ install_dir }}/conf.d/pc.conf"
    owner: "{{user}}"
    group: "{{ group }}"

- name: file 
  file:
    path: "{{ install_dir }}"
    owner: "{{ user }}"
    group: "{{ group }}"
    recurse: yes
  notify: restart service
    
- name: create html data files
  file:
    path: /data/website
    state: directory
    owner: "{{ user }}"
    group: "{{ group }}"

- name: data pages
  unarchive:
    src: nginx_html.tar.gz
    dest: /data/website/
    remote_src: no
    owner: "{{ user }}"
    group: "{{ group }}"
  tags:
    - html
    - data

- name: nginx service
  template:
    src: nginx.service.j2
    dest: /usr/lib/systemd/system/nginx.service

- name: start service
  systemd:
    name: nginx
    state: started
    daemon_reload: yes
    enabled: yes 

#handlers:
#  - name: restart service
#    systemd:
#      name: nginx
#      state: restarted

#相互调用
[root@Ansible /opt/roles/nginx]# cat tasks/get_nginx.yml 
- name:
  unarchive:
    src: "https://nginx.org/download/nginx-1.24.0.tar.gz"
    dest: "/usr/local/src"
    remote_src: yes 
    owner: "{{ user }}"
    group: "{{ group }}"

#创建两个template文件
[root@Ansible /opt/roles/nginx]# ls -l templates/
total 12
-rw-r--r-- 1 root root 331 Jan  4 06:23 blog.conf.j2
-rw-r--r-- 1 root root 676 Jan  4 04:03 nginx.conf.j2
-rw-r--r-- 1 root root 439 Jan  4 03:58 nginx.service.j2

#设置变量
[root@Ansible /opt]# grep -Ev '#|^$' hosts 


[webservers]
192.168.188.8
192.168.188.81

[webservers:vars]
version="1.24.0"
url="https://nginx.org/download/nginx-1.24.0.tar.gz"
dir="/usr/local/src"
install_dir="/apps/nginx"
user="nginx"
group="nginx"
gid=88
uid=88
root_path="/data/php"
fqdn=www.dingbh.top

[dbservers]
192.168.188.88

[all:vars]
domain=dingbh.top



#nginx角色目录结构
[root@Ansible ~]# tree /opt/roles/nginx/
/opt/roles/nginx/
├── files
│   └── nginx_html.tar.gz
├── handlers
│   └── main.yml
├── tasks
│   ├── get_nginx.yml
│   └── main.yml
└── templates
    ├── blog.conf.j2
    ├── nginx.conf.j2
    └── nginx.service.j2

4 directories, 7 files

#在playbook中调用角色
[root@Ansible /opt]# cat nginx_role.yml 
---
- hosts: webservers
  remote_user: root

  roles:
    - nginx
    
[root@Ansible /opt]# ansible-playbook nginx_role.yml 
```

### 4.5.3、实现php-fpm角色

```
#目录结构
[root@Ansible /opt]# tree roles/php-fpm/
roles/php-fpm/
├── files
│   ├── test.php
│   └── www.conf
├── handlers
│   └── main.yaml
├── tasks
│   └── main.yml
└── templates
    ├── php-fpm.conf.j2
    ├── php.ini.j2
    └── www.conf.j2

4 directories, 7 files

#playbook文件
[root@Ansible /opt]# cat php-fpm_role.yml 

---
- hosts: webservers
  remote_user: root

  roles:
    - php-fpm


#创建tasks
[root@Ansible /opt/roles/php-fpm]# cat tasks/main.yml 
- name: Disable SELinux
  selinux:
    state: disabled

- name: install pages
  yum: 
    name: "{{ item }}"
  loop:
    - php-fpm
    - php-mysqlnd
    - php-json
    - php-xml
    - php-gd
    - php-pecl-zip

- name: php path
  file:
    path: /var/lib/php
    owner: "{{ user }}"
    group: "{{ group }}"
    recurse: yes

- name: config php.ini
  template:
    src: php.ini.j2
    dest: /etc/php.ini
  notify: restart php-fpm

#- name: config www
#  template:
#    src: www.conf.j2
#    dest: /etc/php-fpm.d/www.conf
#  notify: restart php-fpm
- name: copy www.conf
  copy: 
    src: www.conf
    dest: /etc/php-fpm.d/    
  notify: restart php-fpm

- name: config nginx
  template:
    src: php-fpm.conf.j2
    dest: "{{ install_dir}}/conf.d/php-fpm.conf"
  notify: restart php-fpm

- name: create php datadir
  file:
    name: "{{ root_path }}"
    state: directory
    owner: "{{ user }}"
    group: "{{ group }}"
    

- name: index.php
  copy:
    src: test.php
    dest: "{{ root_path }}/"

- name: start php-fpm
  service:
    name: php-fpm
    state: started
    daemon_reload: yes
    enabled: yes

```

```
#创建templates文件
[root@Ansible /opt/roles/php-fpm]# cat templates/php-fpm.conf.j2 
server {
    listen 80;
    server_name www.dingbh.top;
    root  /data/website;
    index index.php index.html;
    location ~ \.php$|pm_status|ping {
         root           /data/php;
         fastcgi_pass   127.0.0.1:9000;
         fastcgi_index index.php;
         fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
         include       fastcgi_params;
     }   
}


[root@Ansible /opt/roles/php-fpm]# grep -Ev ';|^$' templates/www.conf.j2 
[www]
user = {{ user }}
group = {{ group }}
listen =  127.0.0.1:9000
listen.acl_users = apache,nginx
pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35
 
pm.status_path = /pm_status
 
ping.path = /ping
ping.response = big pong pong
slowlog = /var/log/php-fpm/www-slow.log
php_admin_value[error_log] = /var/log/php-fpm/www-error.log
php_admin_flag[log_errors] = on
php_value[session.save_handler] = files
php_value[session.save_path]    = /var/lib/php/session
php_value[soap.wsdl_cache_dir]  = /var/lib/php/wsdlcache

[root@Ansible /opt/roles/php-fpm]# grep -Ev ';|^$' templates/php.ini.j2 
[PHP]
engine = On
short_open_tag = Off
precision = 14
output_buffering = 4096
zlib.output_compression = Off
implicit_flush = Off
unserialize_callback_func =
serialize_precision = -1
disable_functions =
disable_classes =
zend.enable_gc = On
expose_php = On
max_execution_time = 360
max_input_time = 600
memory_limit = 128M
error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT
display_errors = Off
display_startup_errors = Off
log_errors = On
log_errors_max_len = 1024
ignore_repeated_errors = Off
ignore_repeated_source = Off
report_memleaks = On
html_errors = On
variables_order = "GPCS"
request_order = "GP"
register_argc_argv = Off
auto_globals_jit = On
post_max_size = 80M
auto_prepend_file =
auto_append_file =
default_mimetype = "text/html"
default_charset = "UTF-8"
doc_root =
user_dir =
enable_dl = Off
file_uploads = On
upload_max_filesize = 80M
max_file_uploads = 20
allow_url_fopen = On
allow_url_include = Off
default_socket_timeout = 60
[CLI Server]
cli_server.color = On
[Date]
date.timezone = Asia/Shanghai
[filter]
[iconv]
[intl]
[sqlite3]
[Pcre]
pcre.jit=0
[Pdo]
[Pdo_mysql]
pdo_mysql.cache_size = 2000
pdo_mysql.default_socket=
[Phar]
[mail function]
sendmail_path = /usr/sbin/sendmail -t -i
mail.add_x_header = On
[ODBC]
odbc.allow_persistent = On
odbc.check_persistent = On
odbc.max_persistent = -1
odbc.max_links = -1
odbc.defaultlrl = 4096
odbc.defaultbinmode = 1
[Interbase]
ibase.allow_persistent = 1
ibase.max_persistent = -1
ibase.max_links = -1
ibase.timestampformat = "%Y-%m-%d %H:%M:%S"
ibase.dateformat = "%Y-%m-%d"
ibase.timeformat = "%H:%M:%S"
[MySQLi]
mysqli.max_persistent = -1
mysqli.allow_persistent = On
mysqli.max_links = -1
mysqli.cache_size = 2000
mysqli.default_port = 3306
mysqli.default_socket =
mysqli.default_host =
mysqli.default_user =
mysqli.default_pw =
mysqli.reconnect = Off
[mysqlnd]
mysqlnd.collect_statistics = On
mysqlnd.collect_memory_statistics = Off
[PostgreSQL]
pgsql.allow_persistent = On
pgsql.auto_reset_persistent = Off
pgsql.max_persistent = -1
pgsql.max_links = -1
pgsql.ignore_notice = 0
pgsql.log_notice = 0
[bcmath]
bcmath.scale = 0
[browscap]
[Session]
session.save_handler = files
session.use_strict_mode = 0
session.use_cookies = 1
session.use_only_cookies = 1
session.name = PHPSESSID
session.auto_start = 0
session.cookie_lifetime = 0
session.cookie_path = /
session.cookie_domain =
session.cookie_httponly =
session.serialize_handler = php
session.gc_probability = 1
session.gc_divisor = 1000
session.gc_maxlifetime = 1440
session.referer_check =
session.cache_limiter = nocache
session.cache_expire = 180
session.use_trans_sid = 0
session.sid_length = 26
session.trans_sid_tags = "a=href,area=href,frame=src,form="
session.sid_bits_per_character = 5
[Assertion]
zend.assertions = -1
[mbstring]
[gd]
[exif]
[Tidy]
tidy.clean_output = Off
[soap]
soap.wsdl_cache_enabled=1
soap.wsdl_cache_dir="/tmp"
soap.wsdl_cache_ttl=86400
soap.wsdl_cache_limit = 5
[sysvshm]
[ldap]
ldap.max_links = -1
[dba]
[curl]
[openssl]

```

#准备files文件

```
[root@Ansible /opt/roles/php-fpm]# grep -Ev ';|^$'  files/*
files/test.php:<?php
files/test.php:?>
files/www.conf:[www]
files/www.conf:user = nginx
files/www.conf:group = nginx
files/www.conf:listen =  0.0.0.0:9000
files/www.conf:listen.acl_users = apache,nginx
files/www.conf:pm = dynamic
files/www.conf:pm.max_children = 50
files/www.conf:pm.start_servers = 5
files/www.conf:pm.min_spare_servers = 5
files/www.conf:pm.max_spare_servers = 35
files/www.conf: 
files/www.conf:pm.status_path = /pm_status
files/www.conf: 
files/www.conf:ping.path = /ping
files/www.conf:ping.response = big pong pong
files/www.conf: 
files/www.conf:slowlog = /var/log/php-fpm/www-slow.log
files/www.conf: 
files/www.conf: 
files/www.conf: 
files/www.conf: 
files/www.conf: 
files/www.conf: 
files/www.conf:php_admin_value[error_log] = /var/log/php-fpm/www-error.log
files/www.conf:php_admin_flag[log_errors] = on
files/www.conf:php_value[session.save_handler] = files
files/www.conf:php_value[session.save_path]    = /var/lib/php/session
files/www.conf:php_value[soap.wsdl_cache_dir]  = /var/lib/php/wsdlcache
```

准备handles文件

```
[root@Ansible /opt/roles/php-fpm]# cat handlers/main.yaml 
- name: restart php-fpm
  service:
    name: php-fpm
    state: restarted

- name: restart nginx
  service:
    name: nginx
    state: restarted

```

### **4.5.4** **调用变量**

```
[root@ansible ~]#vim /data/ansible/roles/test_role/tasks/main.yml
- name: Include OS-specific variables
 include_vars: {{ ansible_os_family}}.yml
....
[root@ansible ~]#ls /data/ansible/roles/test_role/vars/
Archlinux.yml Debian.yml FreeBSD.yml OpenBSD.ymL RedHat.yml
```

### 4.5.5、实现多角色的选择

```
[root@ansible ~]#vim /data/ansible/role_httpd_nginx.yml 
---
- hosts: websrvs
  
 roles:
    - {role: httpd,tags: [httpd,web], when: ansible_distribution_major_version=="7" }
    - {role: nginx,tags: [nginx,web], when: ansible_distribution_major_version=="8" }

[root@ansible ~]#ansible-playbook -t nginx /data/ansible/role_httpd_nginx.yml
```

### **4.5.6** **依赖其它角色**

```
[root@ansible roles]#vim wordpress/meta/main.yml
dependencies:
  - role: nginx
  - role: php-fpm
  - role: mysql
```

### 4.5.7、依赖安装wordpress

实例：安装wordpress

```
[root@Ansible /opt]# cat wordpress_role.yml 
---
- hosts: webservers
  remote_user: root

  roles:
    - wordpress
```

meta依赖文件

```
[root@Ansible /opt]# cat roles/wordpress/meta/main.yml 

dependencies:
  # List your role dependencies here, one per line. Be sure to remove the '[]' above,
  # if you add dependencies to this list.
  - role: nginx
  - role: php-fpm
```

![image-20240106185236854](Ansible自动化/image-20240106185236854.png)



![image-20240106185310240](Ansible自动化/image-20240106185310240.png)

# 5、Ansible Tower 介绍

```
公司中实现运维自动化的架构中主要用到ansible，但是ansible脚本在部署服务器指令行中显得不太直观Ansible Tower (最早以前叫’AWX’)是能够帮助IT团队更容易使用Ansible的解决方案。

Ansible Tower是一个图形化基于WEB的任务调度，复杂服务部署，IT自动化的一个管理平台，属于发布配置管理系统，支持Api及界面操作，基于Django编写Tower允许对用户进行权限控制，即使某用户不能传送某SSH凭证，你也可以通过Tower来对该用户共享该凭证。我们可以通过图形化界面来管理 Inventory，也可以对各种各样的云资源做同步。Tower可以记录所有job的日志，也可以与LDAP集成，并且拥有强大的可浏览的REST API。Tower也提供了命令行工具，可以与Jenkins轻松集成。Provisioning回调对自动伸缩拓扑图提供了强大的支持。

可以在 Ansible Tower 页面 <http://ansible.com/tower> _了解Tower更多的功能，并且了解如何下载使用。Tower的免费版本最多支持10个节点，并且Ansible公司会提供强大的支持。可以支持用Ansible playbook来安装Tower
```

![image-20240106185450242](Ansible自动化/image-20240106185450242.png)

安装完成后的登录初始界面,需要订阅获取license文件才能使用

![image-20240106185514004](Ansible自动化/image-20240106185514004.png)

![image-20240106185520458](Ansible自动化/image-20240106185520458.png)

# 6、Ansible推荐学习资料

```
http://galaxy.ansible.com
https://galaxy.ansible.com/explore#/
http://github.com/
http://ansible.com.cn/
https://github.com/ansible/ansible
https://github.com/ansible/ansible-examples
```

```
curl -fsSL https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.28/deb/Release.key |
 gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

