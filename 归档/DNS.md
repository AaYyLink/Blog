[TOC]



# 1.  DNS介绍

DNS：Domain Name System，域名系统。是一项基于C/S模型的互联网协议负责将域名解析为IP地址，也可以将IP地址解析为域名。使用TCP或UDP的53端口。

**由来**：在互联网早期，主机都是使用本地的hosts文件来解析域名的，但是互联网规模不断增大，一个主机不可能将所有的域名和IP地址的映射关系都记录下来。因此出现了DNS，DNS部署在各个服务器上，主机通过网络访问DNS服务器就可以将自己想要访问的域名解析成IP地址。但是，一台服务器是不可能保存所有的映射记录的，因此会由很多台服务器来保存这些记录，而所有DNS服务器的组织结构呈现一个树形结构。如下：

![img](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1557500791237&di=2b5bbc711172de70cbf216a4c793e63f&imgtype=0&src=http%3A%2F%2Fwww.178linux.com%2Fueditor%2Fphp%2Fupload%2Fimage%2F20150407%2F1428417126641471.jpg)

**DNS服务器的组织结构**

如上图所示，最上面一层的就是根节点，根节点由分布在世界各地的13台服务器维护，其中10台在美国，另外3台在日本、英国和瑞典，其中1台是主根服务器在美国，其他的都是辅根服务器；然后下面是一级域到N级域。我们购买域名时通常是购买二级域的域名。一级域中的各个域名都是有特别意义的，其数量有260多个。常见的如下：

| 名称 | 意义             |
| :--: | ---------------- |
| com  | 公司、行号、企业 |
| org  | 组织、机构       |
| edu  | 教育单位         |
| gov  | 政府单位         |
| net  | 网络、通讯       |
| mil  | 军事单位         |
|  cn  | 中国             |

再下面是二级域，这一级的域名一般供用户购买注册，只要别人没注册过你都可以注册那个域名（当然是需要收费的，在中国购买还需要备案）。

然后是三级域到n级域，这些等级的域名都是让购买二级域名的用户自己随便注册的。

</br>

</br>

# 2.  DNS运行原理

DNS的目的就是提供域名解析服务，因此这里讲解一下一个主机的域名解析过程。

若一个主机发起一个域名解析请求：

1、先查找自己本机上的host文件上是否有对应的映射条目，若有，则直接使用之。

2、查看本机的DNS缓存是否有对应的映射条目，若有则使用之。

3、向自己配置的DNS服务端发起解析请求，DNS服务器若发现其是自己负责的域，则会查询数据库然后返回答案，若不是自己的负责的域并且配置了转发，则会通过递归的方式查询结果，并将查询结果。

4、若上面的过程都失败了，则会请求根服务器，通过不断迭代来查询。

</br>

DNS的**查询类型**：

**递归查询**：指的是客户机和服务器之间的查询，即只发送一起请求，其他的工作有上层服务器解决。最后一层一层的将反馈结果到客户端。

**迭代查询**：指的是DNS与DNS服务器之间的方式，即最初的DNS服务器负责发起请求，而其他涉及到的DNS服务器只负责响应即可，然后一直查找到目标DNS服务器，然后将结果返回给客户端。

</br>

DNS**服务器的类型**：

**主DNS服务器**：维护所负责解析的域内解析库服务器；解析库由管理员维护。

**从DNS服务器**：从主DNS服务器或其他的从DNS服务器那里复制（即区域传递）解析库。

**缓存DNS服务器**：提高DNS的访问速度，实现快速解析，在安装完成DNS软件后自动开启这个功能，通过/etc/named.conf添加forward only来设定。

</br>

</br>

# 3.  常用的DNS客户端程序

DNS客户端程序可以通过安装bind-utils程序包来获得。

常用的DNS客户端程序：dig、host、nslookup。

</br>

## 3.1  dig

```shell
dig  [-t RR_TYPE]  name  [@SERVER]  [query options]
	用于测试dns系统，因此其不会查询hosts文件；
	查询选项：
		+[no]trace：跟踪解析过程；
		+[no]recurse：进行递归解析；
	注意：反向解析测试
		dig  -x  IP
	模拟完全区域传送：
		dig  -t  axfr  DOMAIN  [@server]
```

</br>

## 3.2  host

```shell
host  [-t  RR_TYPE]  name  SERVER_IP
```

</br>

## 3.3  nslookup

	nslookup  [-options]  [name]  [server]	
	交互式模式：
	nslookup>
		server  IP：以指定的IP为DNS服务器进行查询；
		set  q=RR_TYPE：要查询的资源记录类型；
		name：要查询的名称；
</br>

## 3.4  rndc

全称为：remote name domain controller

监听于953/tcp，默认监听于127.0.0.1地址，因此仅允许本地使用。

```shell
rndc COMMAND

COMMAND:
	status：显示当前服务器关于dns的状态
	reload：重载主配置文件和区域解析库文件
	reload ZONE：重载某个特定的区域解析库文件
	retransfer zone：手动启动区域传送过程，而不管序列号是否增加
	notify zone：重新对区域传送发通知
	reconfig：重载主配置文件
	querylog：开启或关闭查询日志
	trace：递增debug级别
	trace LEVEL：指定使用的级别
```

一般情况下，服务器的服务是不能随便重启的，因此dns做过修改后，会通过`rndc reload`来将更新服务。

</br>

</br>

# 4.  DNS服务配置

</br>

## 4.1  配置基础

DNS是一项服务，在Linux中通过named进程来实现这项服务，named属于bind程序包或bind-chroot程序包，后者比前者多了一个沙箱机制，让named进程运行在沙箱内，限制其权限。

因此若要在Linux上启动DNS服务，需要首先安装bind程序包，这个程序包在光盘源上是有的，可以直接使用yum来安装。

DNS的**配置文件**：

/etc/named.conf，/etc/named.rfc1912.zones：DNS的主配置文件

/var/named/：用来存放每个**域文件**

> /var/named目录下，预先就存在named.localhost和named.loopback文件。named.localhost是用来保存当前主机的正向解析库的。named.loopback是用来保存当前主机的反向解析库的。
>
> 若忘掉域文件的配置，则可以参考其来做配置。

</br>

### 4.1.1  资源记录基本介绍

域文件中的内容由各种rr（Resource Record，资源记录）组成。资源记录可以被分为以下几类：

| 类型  | 用途                                                         |
| ----- | ------------------------------------------------------------ |
| SOA   | Start Of Authority，起始授权记录； 一个区域解析库有且只能有一个SOA记录，而且必须放在第一条； |
| NS    | Name Service，域名服务记录；一个区域解析库可以有多个NS记录；其中一个为主的； |
| A     | Address, IPv4地址记录，将域名解析成IPv4地址的条目；FQDN --> IPv4； |
| AAAA  | IPv6地址记录，将域名解析成IPv6地址的条目；FQDN --> IPv6；    |
| CNAME | Canonical Name，别名记录；                                   |
| PTR   | 将IP地址解析成域名的条目：IP--> FQDN；                       |
| MX    | Mail eXchanger，邮件交换器；一般指向DNS管理员的邮箱          |

> FQDN：全限定域名，同时带有主机名和域名的名称。例如：主机名是bigserver，域名是mycompany.com，那么FQDN就是bigserver.mycompany.com。

rr的书写格式如下：

```shell
name  	[TTL] 	IN	RR_TYPE 		value
```

不同类型的rr对应的name和

资源记录定义时，需要注意以下几点：

TTL可以从全局继承。表示该记录被其他dns服务器查询到后保留到对方服务器上的缓存当中并保持多少秒，可以从全局继承。

可以用@引用当前区域的名字

同一个名字可以通过多条记录定义多个不同的值；此时DNS服务器会以轮询方式响应。

</br>

### 

### 4.1.2 各类资源记录的格式

参照rr的书写格式

```
name  	[TTL] 	IN	RR_TYPE 		value
```

</br>

**SOA**

```shell
name: 当前区域的名字；例如”aayylink.com.”，或者“2.3.4.in-addr.arpa.”
value：有多部分组成
		(1) 当前区域的区域名称（也可以使用主DNS服务器名称）；
		(2) 当前区域管理员的邮箱地址；但地址中不能使用@符号，一般使用点号来替代；
		(3) (主从服务协调属性的定义以及否定答案的TTL)
	
例如：
aayylink.com. 	86400 	IN 		SOA 	aayylink.com. 	admin.aayylink.com.  (
		2017010801	; serial,需要手动修改
		2H 			; refresh
		10M 		; retry
		1W			; expire
		1D			; negative answer ttl 
		)
```

</br>

**NS**

```shell
name: 当前区域的区域名称
value：当前区域的某DNS服务器的名字，例如ns.aayylink.com.
	注意：一个区域可以有多个ns记录； 
					
例如：
	aayylink.com. 	86400 	IN 	NS  	ns1.aayylink.com.
	aayylink.com. 	86400 	IN 	NS  	ns2.aayylink.com.
```

</br>

**MX**

```shell
name: 当前区域的区域名称
value：当前区域某邮件交换器的主机名
	注意：MX记录可以有多个；但每个记录的value之前应该有一个数字表示其优先级；

例如：
	aayylink.com. 		IN 	MX 	10  	mx1.aayylink.com.
	aayylink.com. 		IN 	MX 	20  	mx2.aayylink.com.
```

</br>

**A**

```shell
name：某FQDN，例如www.aayylink.com.
value：某IPv4地址；
				
例如：
	www.aayylink.com.		IN 	A	1.1.1.1
	www.aayylink.com.		IN 	A	1.1.1.2
	bbs.aayylink.com.		IN 	A	1.1.1.1
```

**AAAA**

```shell
name：FQDN
value: IPv6
```

</br>

**PTR**

```shell
name：IP地址，有特定格式，IP反过来写，而且加特定后缀；例如1.2.3.4的记录应该写为4.3.2.1.in-addr.arpa.
value：FQND
例如：
	4.3.2.1.in-addr.arpa.  	IN 		www.aayylink.com.
```

</br>

**CNAME**

```shell
name：FQDN格式的别名；
value：FQDN格式的正式名字；
例如：
	web.aayylink.com.  	IN  	CNAME  www.aayylink.com.
```

</br>

**配置需要注意的点**：

```shell
(1) TTL可以从全局继承；
(2) @表示当前区域的名称；
(3) 相邻的两条记录其name相同时，后面的可省略；
(4) 对于正向区域来说，各MX，NS等类型的记录的value为FQDN，此FQDN应该有一个A记录；
```



## 4.2  DNS主服务器配置

首先总结一下主服务器配置的过程：

1. 若是第一次配置DNS，则先修改`/etc/named.conf`配置文件
2. 修改区域配置文件`/etc/named.rfc1912.zone`配置文件，增加正向解析和反向解析的配置
3. 创建正向解析和反向解析的域文件，用来保存映射信息
4. 若是第一次使用DNS服务，则需开启bind守护进程。
5. 让DNS服务重新读取配置文件
6. 若需要用本机测试DNS服务，且是第一次测试，则在测试之前修改本机的DNS网络配置。然后使用bind-utils工具组来测试DNS服务是否运行正常。



### 4.2.1  正向解析区域配置

>在搭建前，记得使用yum来安装bind程序包：yum install bind|bind-chroot，并提前安装bind-utils程序包：yum install bind-utils

首先更改主配置文件`/etc/named.conf`文件，使其能够监听在特定的端口上。并关闭仅允许本地查询。

```shell
~]# vim /etc/named.conf 
```

![1557536193565](images/1557536193565.png)

我在这里就设置了开启监听于192.168.10.30的53端口上，然后加两个斜杠可以注释allow-query行。

> 若出于实验目的，则可以将其中的dnssec-enable yes和dnssec-validation yes改成dnssec-enable no和dnssec-validation no。

</br>

然后开始修改`/etc/named.rfc1912.zones`文件，这个文件是用来修改关于某个区域的配置的。只需在文件的最后面加入新的解析配置即可。由于我们配置的是DNS主服务器，因此配置的新的项目如下。

```shell
~]# vim /etc/named.rfc1912.zones 
```

![1557536850903](images/1557536850903.png)

图中最后一块的配置是我添加的区域配置。

第一行**zone**用来指定域。

**type**字段后所跟的是解析类型，其中解析的类型可以分为master、slave、hint、forward。分别是主、从、根、转发服务器。	

**file**字段后所跟的是域文件，用来保存域的具体数据。自动保存于/var/named/目录下，不需要指定完整的路径。

</br>

然后配置域文件。

```shell
~]# vim /var/named/aayylink.com.zone
$TTL 86400
$ORIGIN aayylink.com.
@       IN  SOA @   admin.aayylink.com. (
            20190511	；serial, 也即是数据库的版本号；主服务器数据库内容发生变化时，其版本号递增,需要手动修改
            1H			；refresh, 从服务器每多久到主服务器检查序列号更新状况
            5M			；retry4, 从服务器从主服务器请求同步解析库失败时，再次发起尝试请求的时间间隔
            2H			；expire，从服务器始终联系不到主服务器时，多久之后放弃从主服务器同步数据；然后停止提供服务
            1D			；否定答案的缓存时长，即某主机针对某个FQDN查询失败，则不能再次发起请求
)
        IN  NS      ns1
        IN  MX  10  mx1
        IN  MX  20  mx2
ns1     IN  A       192.168.10.30
mx1     IN  A       192.168.10.10
mx2     IN  A       192.168.10.30
www     IN  A       192.168.10.30
web     IN  CNAME   www
bbs     IN  A       192.168.10.10
bbs     IN  A       192.168.10.30
```

上述的配置若有问题，请参照4.1.2中**需要注意的点**。

还需要注意的是对应于根域的`.`不要忘记加上了；其中时间的单位默认为秒，H对应小时，M对应分钟，D对应天，W对应周。

</br>

配置完成后，可以先对配置文件的内容做检验，查看是否有错误。

```shell
~]# named-checkconf
# 若不显示任何信息，则代表配置没有错误
~]# named-checkzone aayylink.com /var/named/aayylink.com.zone
# 若显示OK，则代表配置没有错误
```

</br>

然后开启named守护进程。

```shell
~]# systemctl start named
```

</br>

修改本机的DNS网络配置，使其指向当前主机。

```shell
~]# vim /etc/resolv.conf 
# Generated by NetworkManager
nameserver 192.168.10.30
```

> /etc/resolve.conf的具体配置可以参考这篇博客：https://www.cnblogs.com/Alight/p/4351155.html

</br>

然后重载DNS。

```shell
rndc reload
```

</br>

然后就可以使用bind-utils工具进行查看了。

```shell
~]# dig www.aayylink.com

; <<>> DiG 9.9.4-RedHat-9.9.4-51.el7 <<>> www.aayylink.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32676
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.aayylink.com.		IN	A

;; ANSWER SECTION:
www.aayylink.com.	86400	IN	A	192.168.10.30

;; AUTHORITY SECTION:
aayylink.com.		86400	IN	NS	ns1.aayylink.com.
aayylink.com.		86400	IN	NS	ns2.aayylink.com.

;; ADDITIONAL SECTION:
ns1.aayylink.com.	86400	IN	A	192.168.10.30

;; Query time: 0 msec
;; SERVER: 192.168.10.30#53(192.168.10.30)
;; WHEN: Sat May 11 09:53:02 CST 2019
;; MSG SIZE  rcvd: 95
```

</br>

### 4.2.2  反向解析区域配置

配置区域配置文件

```shell
~]# vim /etc/named.rfc1912.zones 
```

![1557539487283](images/1557539487283.png)

</br>

配置域文件

```shell
~]# vim /var/named/192.168.10.zone
$TTL 86400
$ORIGIN 10.168.192.in-addr.arpa.
@       IN  SOA     @   admin.aayylink.com. (
        20190511
        1H
        5M
        2H
        1D
)
        IN  NS      ns1.aayylink.com.
30      IN  PTR     ns1.aayylink.com.
10      IN  PTR     mx1.aayylink.com.
30      IN  PTR     mx2.aayylink.com.
30      IN  PTR     www.aayylink.com.
10      IN  PTR     bbs.aayylink.com.
30      IN  PTR     bbs.aayylink.com.
```

</br>

检测语法是否错误：

```shell
~]# named-checkconf
~]# named-checkzone 10.168.192.in-addr.arpa /var/named/192.168.10.zone
```

</br>

重载配置文件：

```shell
~]# rndc reload
```

</br>

使用bind-utils工具来查询测试：

```shell
~]# dig -x 192.168.10.10

; <<>> DiG 9.9.4-RedHat-9.9.4-51.el7 <<>> -x 192.168.10.10
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 21218
;; flags: qr aa rd; QUERY: 1, ANSWER: 2, AUTHORITY: 1, ADDITIONAL: 2
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;10.10.168.192.in-addr.arpa.	IN	PTR

;; ANSWER SECTION:
10.10.168.192.in-addr.arpa. 86400 IN	PTR	ns2.aayylink.com.
10.10.168.192.in-addr.arpa. 86400 IN	PTR	bbs.aayylink.com.
10.10.168.192.in-addr.arpa. 86400 IN	PTR	mx1.aayylink.com.

;; AUTHORITY SECTION:
10.168.192.in-addr.arpa. 86400	IN	NS	ns1.aayylink.com.

;; ADDITIONAL SECTION:
ns1.aayylink.com.	86400	IN	A	192.168.10.30

;; Query time: 0 msec
;; SERVER: 192.168.10.30#53(192.168.10.30)
;; WHEN: Sat May 11 10:08:43 CST 2019
;; MSG SIZE  rcvd: 137
```

```shell
~]# dig -x 192.168.10.30

; <<>> DiG 9.9.4-RedHat-9.9.4-51.el7 <<>> -x 192.168.10.30
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 61607
;; flags: qr aa rd; QUERY: 1, ANSWER: 3, AUTHORITY: 1, ADDITIONAL: 2
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;30.10.168.192.in-addr.arpa.	IN	PTR

;; ANSWER SECTION:
30.10.168.192.in-addr.arpa. 86400 IN	PTR	bbs.aayylink.com.
30.10.168.192.in-addr.arpa. 86400 IN	PTR	mx2.aayylink.com.
30.10.168.192.in-addr.arpa. 86400 IN	PTR	www.aayylink.com.
30.10.168.192.in-addr.arpa. 86400 IN	PTR	ns1.aayylink.com.

;; AUTHORITY SECTION:
10.168.192.in-addr.arpa. 86400	IN	NS	ns1.aayylink.com.

;; ADDITIONAL SECTION:
ns1.aayylink.com.	86400	IN	A	192.168.10.30

;; Query time: 0 msec
;; SERVER: 192.168.10.30#53(192.168.10.30)
;; WHEN: Sat May 11 10:09:37 CST 2019
;; MSG SIZE  rcvd: 155
```

</br>

</br>

## 4.3  DNS从服务器的配置

DNS从服务器，不需要自己创建域文件，它会自动从其他DNS主服务器或DNS从服务器上同步域文件。

从服务器的配置过程总结如下：

1. 若是第一次配置DNS服务，先修改`/etc/named.conf`配置文件。
2. 先配置DNS主服务器（从服务器）允许向哪些主机做区域传送；默认为向所有主机；应该配置仅允许从服务器；然后重新读取配置文件。
3. DNS主服务器的域文件中需要配置一条指向从服务器的NS记录，并为其配置A记录。
4. 配置从服务器的区域配置文件
5. 从服务器重新读取配置文件
6. 查看`/var/named/slaves`目录下是否有对应的文件
7. 若需要用本机测试DNS服务，且是第一次测试，则在测试之前修改本机的DNS网络配置。然后使用bind-utils工具组来测试DNS服务是否运行正常。

</br>

**本次配置服务器IP地址**

| 服务器类型 | IP地址        |
| :--------: | ------------- |
|  主服务器  | 192.168.10.30 |
|  从服务器  | 192.168.10.10 |

</br>

从服务器的`/etc/named.conf`配置文件同**4.2.1 正向解析区域配置**中的一样，因此不在赘述。

</br>

配置主服务器的`/etc/name.conf`配置文件

```shell
~]# vim /etc/named.conf
```

![1557542763414](images/1557542763414.png)

</br>

配置DNS主服务器的域文件

```shell
~]# vim /var/named/aayylink.com.zone 
```

![1557542891662](images/1557542891662.png)

```shell
~]# vim /var/named/192.168.10.zone
```

![1557542717501](images/1557542717501.png)

</br>

配置从服务器的区域配置文件

```shell
~]# vim /etc/named.rfc1912.zones 
```

![1557541768439](images/1557541768439.png)

</br>

修改本机的`/etc/resolve.conf`文件。

```shell
~]# vim /etc/resolv.conf
nameserver 192.168.10.10
```

</br>

查看`/var/named/slaves`目录下是否有对应的文件

```shell
~]# ll /var/named/slaves/
total 8
-rw-r--r--. 1 named named 457 May 11 10:52 192.168.10.zone
-rw-r--r--. 1 named named 594 May 11 10:52 aayylink.zone
```

</br>

使用bind-utils工具测试。

若配置正确结果与主服务器的解析结果相同。

</br>

## 4.4  DNS子域配置

子域是在DNS迭代查询的过程中所涉及到的。

子域配置的步骤总结如下：

1. 先在父服务器上对子域授权。（记得重新读取配置）
2. 在子服务器配置子域，同主服务器的配置
3. 测试：在父服务器上访问子域的解析。

</br>

这次配置的服务器：

| 类型       | IP地址        |
| ---------- | ------------- |
| 父服务器   | 192.168.10.30 |
| 子域服务器 | 192.168.10.10 |

</br>

子域授权只需修改父服务器上的正向解析域文件即可。添加一个关于子域的NS记录，以及该NS的FQDN对应的A记录。

```shell
~]# vim /var/named/aayylink.com.zone
```

![1557544487212](images/1557544487212.png)

父服务器的DNS服务重新读取配置文件

```shell
~]# rndc reload
```

</br>

然后在子域上配置`ops.aayylink.com`区域的配置即可。步骤同主配置文件。

先配置区域配置文件

```shell
~]# vim /etc/named.rfc1912.zones 
... ...
... ...
上面省略多行,在文件末尾加入
zone "ops.aayylink.com" IN {
    type master;
    file "ops.aayylink.com.zone";
};
```

然后配置域文件

```shell
$TTL 86400
$ORIGIN ops.aayylink.com.
@   IN  SOA     @   admin.ops.aayylink.com. (
    20190511
    1H
    5M
    2H
    1D
    )

    IN  NS      ns1
    IN  MX  10  mx1.aayylink.com.
    IN  MX  20  mx2.aayylink.com.
ns1 IN  A       192.168.10.10
www IN  A       192.168.20.10
mx1.aayylink.com.   IN  A       192.168.10.30
mx2.aayylink.com.   IN  A       192.168.10.10
web IN  CNAME   www
stu IN  A       192.168.20.10
```

子域服务器重新读取配置文件

```shell
rndc reload
```

</br>

## 4.5  基本安全控制

acl：访问控制列表；把一个或多个地址归并一个命名的集合，随后通过此名称即可对此集全内的所有主机实现统一调用；
	acl定义格式:
	acl  acl_name  {
		ip;
		net/prelen;
	};
				
	示例：
	acl  mynet {
		172.16.0.0/16;
		27.0.0.0/8;
	};
				
	bind中四个内置的acl
		none：没有一个主机；
		any：任意主机；
		local：本机；
		localnet：本机所在的IP所属的网络；
				
	访问控制指令：
	allow-query  {};  允许查询的主机；白名单；
	allow-transfer {};  允许向哪些主机做区域传送；默认为向所有主机；应该配置仅允许从服务器；
	allow-recursion {}; 允许哪此主机向当前DNS服务器发起递归查询请求； 记得要删除全局开启recursion那一行,修改的是主配置文件
	allow-update {}; DDNS，允许动态更新区域数据库文件中内容；一般要关闭。