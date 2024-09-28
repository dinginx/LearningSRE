# ansible初始化

需求

修改主机名

关闭防火墙

关闭selinux

## 1.1、目录结构

```
[root@Jumpserver /data/ansible]#tree
.
├── ansible.cfg
├── hosts
├── init0_role.yml
└── roles
    └── init0
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

10 directories, 11 files

```

