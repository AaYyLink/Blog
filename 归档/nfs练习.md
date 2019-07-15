>wordpress的压缩包下载：
>
>链接：https://pan.baidu.com/s/1uor8hvqLNtGxCK4h9bZg0w 
>提取码：uvue 



```
(1) nfs server导出/data/application/web，在目录中提供wordpress; 
(2) nfs client挂载nfs server导出的文件系统至/var/www/html；
(3) 客户端（lamp）部署wordpress，并让其正常访问；要确保能正常发文章，上传图片；
(4) 客户端2(lamp)，挂载nfs server导出的文件系统至/var/www/html；验正其wordpress是否可被访问； 要确保能正常发文章，上传图片；
对于数据库服务需要实现：
(1) nfs server导出/data/application/mysql目录；
(2) 两个nfs client挂载/data/application/mysql至本地的/mydata目录；本地的mysqld或mariadb服务的数据目录设置为/mydata, 要求服务能正常启动，且可正常存储数据；
```

**本次配置**：

| 主机          | 身份                              |
| ------------- | --------------------------------- |
| 192.168.10.10 | nfs服务端                         |
| 192.168.10.20 | nfs客户端，可以写和读，LMAP服务端 |
| 192.168.10.30 | nfs客户端，可以写和读，LAMP服务端 |

# 192.168.10.10的配置

**创建相关目录**

```shell
[root@10 ~]# mkdir /data/application/web -pv
mkdir: 已创建目录 "/data"
mkdir: 已创建目录 "/data/application"
mkdir: 已创建目录 "/data/application/web"
[root@10 ~]# mkdir /data/application/mysql
```

**安装对应的软件包**：

```shell
[root@10 ~]# yum install nfs-utils -y
```

**添加关联用户**：

```shell
[root@10 ~]# useradd -u 5000 nfsuser
```

**修改对应目录的权限**：

```shell
[root@10 ~]# chown -Rf 5000.5000 /data/application/web/ 
[root@10 ~]# chown -Rf 27.27 /data/application/mysql
```

修改/etc/exports文件共享文件系统：

```shell
[root@10 ~]# vim /etc/exports
/data/application/web 192.168.10.0/24(rw,anonuid=5000,anongid=5000)
/data/application/mysql 192.168.10.0/24(rw,anonuid=27,anongid=27)
```

**开启nfs服务**：

```shell
[root@10 ~]# systemctl start nfs
```



# 192.168.10.20的配置

**安装LAMP的运行环境**

```shell
[root@20 ~]# yum install httpd mariadb-server php-mysql php -y
```

**创建相关文件并修改其权限**：

```shell
[root@20 ~]# mkdir /mydata
[root@20 ~]# chown mysql.mysql /mydata/
```

**修改MariaDB数据文件的目**录：

```shell
  1 [mysqld]
  2 datadir=/mydata
  3 socket=/var/lib/mysql/mysql.sock
  4 # Disabling symbolic-links is recommended to prevent assorted security risks
  5 symbolic-links=0
  6 # Settings user and group are ignored when systemd is used.
  7 # If you need to run mysqld under a different user or group,
  8 # customize your systemd unit file for mariadb according to the
  9 # instructions in http://fedoraproject.org/wiki/Systemd
 10 skip-name-resolve=on
 11 
 12 [mysqld_safe]
 13 log-error=/var/log/mariadb/mariadb.log
 14 pid-file=/var/run/mariadb/mariadb.pid
 15       
 16 #    
 17 # include all files from the config directory
 18 #    
 19 !includedir /etc/my.cnf.d
```

其中第2行是需要修改的，第10行添加跳过反解析过程。



**启动数据库服务**：

```shell
[root@20 ~]# systemctl start mariadb
```



**尝试挂载目标文件系统**：

```shell
[root@20 ~]# showmount -e 192.168.10.10
Export list for 192.168.10.10:
/data/application/mysql 192.168.10.0/24
/data/application/web   192.168.10.0/24
[root@20 ~]# mount 192.168.10.10:/data/application/mysql /mydata
[root@20 ~]# mount 192.168.10.10:/data/application/web /var/www/html/
```

创建数据库，并给予wpadmin授权：

```shell
[root@20 ~]# mysql -uroot -predhat
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 11
Server version: 5.5.56-MariaDB MariaDB Server

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create database wordpress;
Query OK, 1 row affected (0.01 sec)

MariaDB [(none)]> grant all on wordpress.* to wpadmin@'127.0.0.1' identified by 'redhat';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.01 sec)

MariaDB [(none)]> quit
Bye
```

在/var/www/html目录将wordpress压缩包解压开（下载链接在最上方）：

```shell
[root@20 html]# pwd
/var/www/html
[root@20 html]# unzip wordpress-4.3.1-zh_CN.zip 
```

修改wordpress的配置：

```shell
[root@20 wordpress]# pwd
/var/www/html/wordpress
[root@20 wordpress]# cp wp-config-sample.php wp-config.php
[root@20 wordpress]# vim wp-config.php 
[root@20 wordpress]# cat wp-config.php -n	
	... ...
    23	define('DB_NAME', 'wordpress');
    24	
    25	/** MySQL数据库用户名 */
    26	define('DB_USER', 'wpadmin');
    27	
    28	/** MySQL数据库密码 */
    29	define('DB_PASSWORD', 'redhat');
    30	
    31	/** MySQL主机 */
    32	define('DB_HOST', '127.0.0.1');
    ... ...
```

修改上面的几行即可。

