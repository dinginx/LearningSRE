# keystone安装和部署

```sh
[root@Rocky8 ~]# mysql -u root -p

CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY '123456';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY '123456';
  
[root@Rocky8 ~]# openssl rand -hex 10
```

```sh
[root@Rocky8 ~]# yum install openstack-keystone httpd mod_wsgi
[root@Rocky8 ~]# vim  /etc/keystone/keystone.conf 

[DEFAULT]
...
admin_token = 7bbcf723ccbc14f03e68   #初始令牌

[database]
...
connection = mysql+pymysql://keystone:123456@controller/keystone

[token]
...
provider = fernet
```

```sh
[root@Rocky8 ~]# su -s /bin/sh -c "keystone-manage db_sync" keystone
[root@Rocky8 ~]# keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
```

