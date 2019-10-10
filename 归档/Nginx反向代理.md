0. 反向代理介绍

![1569398533164](images/1569398533164.png)

**反向代理工作原理**：当客户端发送请求给**反向代理服务器**时，该反向代理服务器并没有真正的请求资源，也不会直接响应客户端的请求，而是在反向代理服务器本地再次生成一个请求发送给真正拥有资源的**后端服务器**。然后后端服务器会将资源响应给反向代理服务器，反向代理服务器再将资源返回给客户端。

**反向代理的优点**：反向代理可以隐藏内网主机的真正IP地址，起到安全的作用；反向代理服务器可以缓存请求到的资源，减轻后端服务器的访问压力。

**反向代理的缺点**：反向代理的缺点就在于其并发能力相对NAT模型弱，因为它对于后端服务器的请求是需要使用套接字的，在不开启保持连接的模式下，反代服务器（反向代理服务器）每接收到一个请求都会使用一个套接字跟后端做连接，而套接字的数量是有上限的。

# 1. Nginx反向代理介绍

Nginx有多种反向代理功能，并且设立了多种反向代理的模块。

| 反向代理模块             | 功能                                                         |
| ------------------------ | ------------------------------------------------------------ |
| ngx_http_proxy_module    | 用来反向代理http的请求                                       |
| ngx_http_headers_module  | 将代理服务器响应给客户端的报文中添加自定义首部，也可以修改原来首部的值 |
| ngx_http_fastcgi_module  | 用来反向代理fastcgi的请求，可以用来构建LAMP模型              |
| ngx_http_upstream_module | 定义server组，这些组可以在反代时引用，以实现负载均衡         |
| ngx_stream_core_module   | 可以实现传输层的反向代理                                     |

接下来将会对这些反向代理模块进行一个个的详解。

# 2. 反向代理模块详解

## 2.1 ngx_http_proxy_module

该模块负责http和https协议的反向代理配置

### 2.1.1 proxy_pass

```nginx
Syntax:	proxy_pass URL;
Default:	—
Context:	location, if in location, limit_except
```

**功能**：该命令用来在反代时，定义反代服务器访问的资源。

该命令的使用有一些注意点（这个注意点在以后的配置当中尤其重要）：

```
1.当proxy_pass后面所跟的URL不包含uri时，则会将location中的uri传递给后端主机
	例如：下面的响应资源为http://192.1.168.10.20:80/index.html
		location /index.html{
			proxy_pass http://192.1.168.10.20:80;
		}
2.1.若proxy_pass后面所跟的URL包含uri时，则使用proxy_pass后的uri传递给后端主机
	例如：下面的响应资源为http://192.1.168.10.20:80/a.jpg
		location /index.html{
			proxy_pass http://192.1.168.10.20:80/a.jpg;
		}
3.如果location定义其uri时使用了正则表达式的模式，或在if语句或limt_execept中使用proxy_pass指令，则proxy_pass之后必须不能使用uri; 用户请求时传递的uri将直接附加代理到的服务的之后；
	例如：下面的响应资源为http://192.1.168.10.20:80/index.html
		location ~* /uri/ {
			proxy_pass http://192.1.168.10.20:80;
		}
```

根据上面的注意点来看下面的问题：

虽然下面两个proxy_pass的指向只是在最后面差了一个`/`，但却是上面注意点中第一种情况和第二种情况的区别，所以其访问结果是不同的。

```nginx
location /test/index.html{
    proxy_pass http://192.1.168.10.20:80;
}
```

```nginx
location /test/index.html {
  	proxy_pass http://192.1.168.10.20:80/;
}
```

### 2.1.2 proxy_set_header

```nginx
Syntax:		proxy_set_header field value;
Default:	proxy_set_header Host $proxy_host;
			proxy_set_header Connection close;
Context:	http, server, location
```

**功能**：在向后端转发的请求中，新增或修改头部内容。

这个命令的常见用法有以下两种：

```nginx
proxy_set_header X-Real-IP  $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

其中\$remote\_addr，上一个客户端的地址（当然，上一个客户端可能也是一个代理服务器）。

\$proxy_add_x_forwarded_for的信息有好多个，并用逗号分割，其中第一个为真正的客户端，后面的一个个是中间经过的代理服务器。

### 2.1.3 proxy_cache_path

```nginx
Syntax:		proxy_cache_path path [levels=levels] [use_temp_path=on|off] keys_zone=name:size [inactive=time] [max_size=size] [manager_files=number] [manager_sleep=time] [manager_threshold=time] [loader_files=number] [loader_sleep=time] [loader_threshold=time] [purger=on|off] [purger_files=number] [purger_sleep=time] [purger_threshold=time];
Default:	—
Context:	http
```

顾名思义，用来定义代理缓存的路径。其中重要参数的使用方法如下：

```
path：定义缓存的根目录。
levels：定义缓存的层次结构，可有三层，每层可有1~2位16进制数表示。例如：levels=1:2；表示缓存有两层结构，第一层的目录为0-F，第二层为00-FF。
keys_zone：用来定义缓存的名称以及缓存的hash表大小。例如：keys_zone=pcache:10m
max_size：用来定义缓存本身的大小，max_size=2g
```

**使用举例**：

```nginx
proxy_cache_path /data/nginx/proxy_cache levels=1:2 keys_zone=pcache:20m max_size=4g;
```

### 2.1.4 proxy_cache

```nginx
Syntax:		proxy_cache zone | off;
Default:	proxy_cache off;
Context:	http, server, location
```

调用或禁用缓存。调用的缓存是通过proxy_cahce_path命令定义的，通过调用keys_zone中的名称来调用缓存。

例如调用上面举例中的缓存：

```nginx
proxy_cache pcache;
```

### 2.1.5 proxy_cache_key

```nginx
Syntax:	 	proxy_cache_key string;
Default:	proxy_cache_key $scheme$proxy_host$request_uri;
Context:	http, server, location
```

定义缓存内容的键。用下面的例子来说明：

```nginx
proxy_cache_key $scheme$proxy_host$request_uri

proxy_cache_key $proxy_host$request_uri
```

其中\$scheme表示客户端访问的协议（http或https），\$proxy_host表示后端服务器的域名（地址）和端口，\$request_uri表示请求的网页资源。

分析上面两个个命令区别：第2个key并没有加上协议，所以当多个客户端访问同一个主机的同一个资源时，不管是http还是https，命中的都是同一个缓存项。

下面用三个命令做例子，用大写字母表示缓存项，相同的大写字母表示相同的缓存项：

```nginx
proxy_cache_key $scheme$proxy_host$request_uri：
	客户端访问的资源					缓存命中项
	https://192.1.168.10.10/index.html --->	A
	http://192.1.168.10.10/index.html  ---> 	B
	https://192.1.168.10.20/index.html --->	C
	http://192.1.168.10.20/index.html  --->	D

proxy_cache_key $proxy_host$request_uri
	客户端访问的资源					缓存命中项
	https://192.1.168.10.10/index.html --->	A
	http://192.1.168.10.10/index.html  ---> 	A	
	https://192.1.168.10.20/index.html --->	B
	http://192.1.168.10.20/index.html  --->	B

proxy_cache_key $request_uri
	客户端访问的资源					缓存命中项
	https://192.1.168.10.10/index.html --->	A
	http://192.1.168.10.10/index.html  ---> 	A
	https://192.1.168.10.20/index.html --->	A
	http://192.1.168.10.20/index.html  --->	A
```

### 2.1.6 proxy_cache_valid

```nginx
Syntax:		proxy_cache_valid [code ...] time;
Default:	—
Context:	http, server, location
```

设置对于特定响应码的缓存时长。由于没有默认值，该命令必须设置，否则不会启用缓存功能。若命令中未设定code，则设置的是200, 301, 302响应码的缓存时长。

**使用举例**：

```nginx
proxy_cache_valid 200 302 301 1h;
proxy_cache_valid any 1m;
#对于响应码为200,302,301的响应资源缓存1小时，其它响应码的响应资源为1m。
```

### 2.1.7 proxy_cache_use_stale

```nginx
Syntax:		proxy_cache_use_stale error | timeout | invalid_header | updating | http_500 | http_502 | http_503 | http_504 | http_403 | http_404 | http_429 | off ...;
Default:	proxy_cache_use_stale off;
Context:	http, server, location
```

过期缓存响应设置。当后端服务器不在提供某一个资源，而反代服务器本地有这个资源的缓存时，什么情况下反代服务器可以使用该资源做响应。

其中重点参数的意义：

```
error:表示任何情况下都可以使用过期缓存响应。
http_500：响应码为500的可以响应
http_503：响应码为503的可以响应。
http_...：响应码为...的可以响应。
```

### 2.1.8 proxy_cache_methods

```nginx
Syntax:		proxy_cache_methods GET | HEAD | POST ...;
Default:	proxy_cache_methods GET HEAD;
Context:	http, server, location
This directive appeared in version 0.7.59.
```

该命令在nginx0.7.59后的版本才出现。

表示请求头部中的方法是设定的值时，才能被缓存。

其中GET，HEAD为默认的方法，也是永远被选中缓存的方法。简单的来说就是：你使用这个命令时，即使没有声明GET、HEAD这两个方法，也会被自动添加进去。

该方法一般情况下使用默认值即可。

### 2.1.9 proxy_hide_header

```nginx
Syntax:		proxy_hide_header field;
Default:	—
Context:	http, server, location
```

默认情况下，反代服务器的响应报文头部中的，“Date”, “Server”, “X-Pad”, and “X-Accel-...” 的field:value是不会被传送给客户端的。使用该命令则可以通过声明其它的field来隐藏对应的field:value。

### 2.1.10 proxy_connect_timeout

```nginx
Syntax:		proxy_connect_timeout time;
Default:	proxy_connect_timeout 60s;
Context:	http, server, location
```

反代服务器和后端服务器的建立连接时的超时时长，一般不超过75s。

### 2.1.11 proxy_read_timeout

```nginx
Syntax:		proxy_read_timeout time;
Default:	proxy_read_timeout 60s;
Context:	http, server, location
```

反代服务器向后端服务器读取报文的超时时间。这里的超时时间是指建立连接后，两次读取之间的时间。而非整个响应资源的获取时间。（如果一个资源过大，可能会分多次读取和请求），如果代理服务器没有在这段时间内读取到新的报文，则关闭连接。

### 2.1.12 proxy_send_timeout

```nginx
Syntax:		proxy_send_timeout time;
Default:	proxy_send_timeout 60s;
Context:	http, server, location
```

反代服务器向后端服务器发送报文请求的超时时间。这里的超时时间是建立连接后的，是两次发送报文请求之间的时间。而非请求整个资源的时间，如果后端服务器在这段时间内没有收到新的报文请求，则关闭连接。

## 2.2 ngx_http_headers_module

该模块可以在响应给客户端的报文中添加 “Expires” 和“Cache-Control” 两个特定的field以及其它自定义的field，也可以修改指定field的value。

### 2.2.1  add_header

```nginx
Syntax:		add_header name value [always];
Default:	—
Context:	http, server, location, if in location
```

添加新的field(name)及其value。只对特定响应码的响应资源生效。

不同的nginx版本，生效的响应码不同。如下：

```
1.3.10: 200, 201
1.1.16: 200, 201, 204, 206, 301, 302, 303, 304, 307 
1.13.0: 200, 201, 204, 206, 301, 302, 303, 304, 307, 308
```

**使用举例**：

```nginx
add_header X-Via  $server_addr;
add_header X-Accel $server_name;
# $server_addr和$server_name是代理服务器的地址和主机名。
```

## 2.3 ngx_http_fastcgi_module

该模块是用来做fastcgi的反代服务器的。可以基于该模块做出LNMP的架构。

### 2.3.1 fastcgi_pass

```nginx
Syntax:		fastcgi_pass address;
Default:	—
Context:	location, if in location
```

设置后端的fastcgi的地址及端口。address包括地址（主机名）和端口，也可以使用UNIX-domain套接字替代。

**使用举例**：

```nginx
fastcgi_pass localhost:9000;

fastcgi_pass unix:/tmp/fastcgi.socket;
```

### 2.3.2 fastcgi_index

```nginx
Syntax:		fastcgi_index name;
Default:	—
Context:	http, server, location
```

访问fastcgi时的默认主页资源，这个命令会设置\$fastcgi_script_name的值为该命令的name参数。

**使用举例**：

```nginx
fastcgi_index index.php;
fastcgi_param SCRIPT_FILENAME /data/nginx/php-content$fastcgi_script_name;
```

### 2.3.3 fastcgi_param

```nginx
Syntax:		fastcgi_param parameter value [if_not_empty];
Default:	—
Context:	http, server, location
```

设置传送给fastcgi服务端的参数。该设置可被继承。

**使用举例**：

```nginx
fastcgi_param SCRIPT_FILENAME /data/nginx/php-content$fastcgi_script_name;
```

### 2.3.4 fastcgi_cache_path

```nginx
Syntax:		fastcgi_cache_path path [levels=levels] [use_temp_path=on|off] keys_zone=name:size [inactive=time] [max_size=size] [manager_files=number] [manager_sleep=time] [manager_threshold=time] [loader_files=number] [loader_sleep=time] [loader_threshold=time] [purger=on|off] [purger_files=number] [purger_sleep=time] [purger_threshold=time];
Default:	—
Context:	http
```

反代fastcgi的缓存创建设置，配置参数的使用参考上面的proxy_cache_path

### 2.3.5 fastcgi_cache

```nginx
Syntax:		fastcgi_cache zone | off;
Default:	fastcgi_cache off;
Context:	http, server, location
```

反代fastcgi的缓存调用，配置参数的使用参考上面的proxy_cache.

### 2.3.6 fastcgi_cache_key

```nginx
Syntax:	fastcgi_cache_key string;
Default:	—
Context:	http, server, location
```

缓存时的键，配置参数的使用参考上面的proxy_cache_key

### 2.3.7 fastcgi_cache_methods

```nginx
Syntax:		fastcgi_cache_methods GET | HEAD | POST ...;
Default:	fastcgi_cache_methods GET HEAD;
Context:	http, server, location
This directive appeared in version 0.7.59.
```

0.7.59版本才出现。

配置哪些方法可以被缓存。使用方法参考上面的proxy_cache_methods。

### 2.3.8 fastcgi_cache_min_uses

```nginx
Syntax:		fastcgi_cache_min_uses number;
Default:	fastcgi_cache_min_uses 1;
Context:	http, server, location
```

缓存空间中的缓存项在inactive定义的非活动时间内至少要被访问到此处所指定的次数方可被认作活动项；

### 2.3.9 fastcgi_cache_valid

```nginx
Syntax:		fastcgi_cache_valid [code ...] time;
Default:	—
Context:	http, server, location
```

哪些响应码的内容可以被缓存，缓存时间是多少。使用方法参考上面的proxy_cache_valid。

### 2.3.10 fastcgi_keep_conn

```nginx
Syntax:		fastcgi_keep_conn on | off;
Default:	fastcgi_keep_conn off;
Context:	http, server, location
This directive appeared in version 1.1.4.
```

设置fastcgi的保持连接，可以提高性能。

## 2.4 ngx_http_upstream_module

该模块用来定义server组，提供给proxy_pass,fastcgi_pass等命令调用。若订阅nginx商业版，则可以使用健康监测实现高可用代理。

### 2.4.1 upstream

 ```nginx
Syntax:		upstream name { ... }
Default:	—
Context:	http
 ```

用来定义server组，server的表示可以混用TCP和UNIX-domain套接字；会通过权重调度后端服务器，当被调度的后端服务器发生错误时，会将请求调度给其他服务器，若所有服务器都响应错误，则使用最后一台被调度中的后端服务器的响应结果做响应。接下来用示例讲解使用方法：

**使用示例**：

```nginx
upstream backend {
    server backend1.example.com weight=5;
    server 127.0.0.1:8080       max_fails=3 fail_timeout=30s;
    server unix:/tmp/backend3;

    server backup1.example.com  backup;
}
```

server命令跟上具体的后端服务器IP或者UNIX-domain套接字，以及一些参数，这些参数的功能如下：

| 参数         | 功能                                                         |
| ------------ | ------------------------------------------------------------ |
| weight       | 该服务器的权重                                               |
| max_fails    | 后端服务器响应失败的次数上限                                 |
| fail_timeout | 后端服务器失败超时时间                                       |
| backup       | 指定该服务器为备份服务器，在其他所有后端服务器失效时，在使用其做后端服务器 |



### 2.4.2 server

```nginx
Syntax:		server address [parameters];
Default:	—
Context:	upstream
```

查看上面的2.1。

### 2.4.3 least_conn

```nginx
Syntax:		least_conn;
Default:	—
Context:	upstream
This directive appeared in versions 1.3.1 and 1.2.2.
```

采用最少连接算法调度后端服务器。

### 2.4.4 ip_hash

```nginx
Syntax:		ip_hash;
Default:	—
Context:	upstream
```

采用源地址hash算法调度后端服务器。

### 2.4.5 hash

```nginx
Syntax:		hash key [consistent];
Default:	—
Context:	upstream
This directive appeared in version 1.7.2.
```

指定关键字进行hash，这里的关键字可以包含普通文本，变量或它们的组合。

consitent：一致性hash，这种hash方法更能适用于高可用的场景。

缓存场景的使用示例：

```nginx
hash $request_uri consistent;
```

### 2.4.6 keepalive

```nginx
Syntax:		keepalive connections;
Default:	—
Context:	upstream
This directive appeared in version 1.1.4.
```

设定每个worker进程保留的空闲的长连接数量。

## 2.5 ngx_stream_core_module

该模块可以实现TCP的四层代理，即：可以让nginx代理其他类型的协议，例如：ssh、ftp。

注意：该模块不是默认模块，需要在编译时加入`--with-stream`选项。

### 2.5.1 stream

```nginx
Syntax: 	stream { ... }
Default: 	—
Context: 	main
```

该命令在main中使用，为该模块的配置提供上下文。

### 2.5.2 server

```nginx
Syntax: 	server { ... }
Default: 	—
Context: 	stream
```

配置虚拟主机，类似于httpd模块中的server命令。

### 2.5.3 模块使用示例

```nginx
stream {
    server{
        listen 192.168.3.111:22;
        proxy_pass 192.168.10.20:22;
    }
}
```



# 3. nginx反代示例

## 3.1 nginx web反代

| 服务器角色 | IP地址                             |
| ---------- | ---------------------------------- |
| 反代服务器 | 192.168.3.121/24，192.168.10.10/24 |
| 后端服务器 | 192.168.10.20/24                   |
| Client     | 192.168.3.3/ 24                    |

1.在反代服务器上安装nginx：

```bash
[root@node1 ~]# yum install nginx -y
```

2.创建相关文件：

```shell
mkdir /data/nginx/proxy_cache -pv 
```

3.修改主配置文件`/etc/nginx/nginx.conf`实现反代缓存，在http段中插入以下内容：

```nginx
proxy_cache_path /data/nginx/proxy_cache levels=1:2 keys_zone=pcache:20m max_size=4g;
```

4.添加nginx反代配置文件

```bash
[root@node1 ~]# vim /etc/nginx/conf.d/proxy.conf
server {
        listen 80;
        server_name www.aayylink.com;
        index index.html;
        location ~* \.* {
                proxy_pass http://192.168.10.20:80;
                proxy_cache pcache;
                proxy_cache_key $proxy_host$request_uri;
                proxy_cache_valid 200 302 301 1h;
                proxy_cache_valid any 1m;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
}
```

5.检查并启动服务（或重载配置文件）

```shell
第一次启动nginx：
[root@node1 ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@node1 ~]# systemctl start nginx

重新载入配置文件：
[root@node1 ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@node1 ~]# nginx -s reload
```

6.后端服务器安装httpd程序

```shell
[root@node2 ~]# yum install httpd -y
```

7.创建后端服务器的网页资源，并启动httpd服务

```shell
[root@node2 ~]# echo "<h1>upstream server</h1>" > /var/www/html/index.html
[root@node2 ~]# systemctl start httpd
```

8.修改本地的hosts文件内容，添加如下

```
192.168.3.121 www.aayylink.com
```

9.通过网页进入`www.aayylink.com`，若出现upstream server，则代表配置成功。

10.可以查看缓存文件夹，来验证缓存是否生效

```shell
[root@node1 ~]# cat /data/nginx/proxy_cache/3/27/9b879d030baa265777df07e5afc07273 
ei"19-594643894c213"
KEY: 192.168.10.20/
HTTP/1.1 200 OK
Date: Tue, 08 Oct 2019 11:14:09 GMT
Server: Apache/2.4.6 (CentOS)
Last-Modified: Tue, 08 Oct 2019 11:10:50 GMT
ETag: "19-594643894c213"
Accept-Ranges: bytes
Content-Length: 25
Connection: close
Content-Type: text/html; charset=UTF-8

<h1>upstream server</h1>
```

## 3.2 nginx反代fastcgi实现LNMP模型

| 服务器角色                | IP地址                            |
| ------------------------- | --------------------------------- |
| 反代服务器                | 192.168.3.121/24,192.168.10.10/24 |
| 后端服务器、MariaDB服务器 | 192.168.10.20/24                  |
| Client                    | 192.168.3.3/24                    |

反代服务器：不仅仅进行反代，并可以在本地提供静态资源。

后端服务器：提供动态资源及MariaDB服务。

> 本次实验将用到wordpress的压缩包，可以通过下面下载：
>
> 链接：https://pan.baidu.com/s/1uor8hvqLNtGxCK4h9bZg0w 
> 提取码：uvue 

1.反代服务器安装相应程序包

```shell
[root@node1 ~]# yum install nginx -y
```

2.创建相关文件：

```shell
[root@node1 ~]# mkdir /data/nginx/web -pv
[root@node1 ~]# mkdir /data/nginx/fastcgi_cache -pv
```

3.主配置文件中的httpd段加入下面一行，配置反代fastcgi的缓存：

```nginx
fastcgi_cache_path /data/nginx/fastcgi_cache levels=1:2 keys_zone=fcache:20m max_size=4g;
```

4.创建fastcgi反代配置文件

```shell
[root@node1 ~]# vim /etc/nginx/conf.d/fastcgi.conf
server {
        listen 80;
        root /data/nginx/web;
        server_name www.aayylink.com;
        index index.html index.php;
        location ~* \.php$ {
                fastcgi_pass 192.168.10.20:9000;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME /data/nginx/dynamic$fastcgi_script_name;
                include fastcgi_params;
                fastcgi_cache fcache;
                fastcgi_cache_key $proxy_host$request_uri;
               	fastcgi_cache_valid 200 302 301 1h;
                fastcgi_cache_valid any 1m;
        }
        location ~* ^/(status|ping)$ {
                include fastcgi_params;
                fastcgi_pass 192.168.10.20:9000;
                fastcgi_param SCRIPT_FILENAME $fastcgi_script_name;
        }
}
```

5.反代服务器配置验证并开启nginx（或重启nginx）：

```bash
第一次启动nginx：
[root@node1 ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@node1 ~]# systemctl start nginx

重新载入配置文件：
[root@node1 ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@node1 ~]# nginx -s reload
```

6.后端服务器安装相应程序：

```bash
[root@node2 ~]# yum install php php-mysql php-fpm php-mbstring php-mcrypt mariadb-server -y
```

7.后端服务器进行fastcgi的相关配置，通过修改`/etc/php-fpm.d/www.conf`文件：

```
listen = 0.0.0.0:9000
表示监听在所有网卡的9000端口上

#listen.allowed_clients = 0.0.0.0
用#号注释该行，表示允许所有的客户端访问本机的fastcgi

pm.max_children = 150
能创建的最大子进程数量为150，可以不改

ping.path = /ping
设置ping的路径，该URI会在用户访问时返回pong，前提是fastcgi在正确运行

pm.status_path = /status
设置status的路径，该URI会在用户访问时返回fastcgi的状态，前提是fastcgi在正确运行
```

8.修改mariadb的配置文件，在`mysqld`段中加入如下：

```
skip_name_resolve=ON
跳过域名反解

innodb_file_per_table=ON
一个表对应一个文件
```

9.在后端服务器创建相关文件夹：

```shell
[root@node2 ~]# mkdir /var/lib/php/session/ -pv
[root@node2 ~]# mkdir /data/nginx/dynamic -pv
```

10.后端服务器启动php-fpm及mariadb的守护进程：

```shell
[root@node2 ~]# systemctl start php-fpm
[root@node2 ~]# systemctl start mariadb
```

11.将wordpress的压缩包在对应目录解压：

192.168.10.10：

```bash
[root@node1 web]# cd /data/nginx/web/
[root@node1 web]# unzip wordpress-4.3.1-zh_CN.zip 
[root@node1 web]# ln -sv wordpress wp
'wp' -> 'wordpress'
```

后端服务器：

```shell
[root@node2 dynamic]# cd /data/nginx/dynamic/
[root@node2 dynamic]# unzip wordpress-4.3.1-zh_CN.zip
[root@node2 dynamic]# ln -sv wordpress wp
‘wp’ -> ‘wordpress’
```

12.后端服务器mariadb初始化：

```shell
[root@node2 dynamic]# mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] Y
New password: [此处是root用户密码]
Re-enter new password: [此处是root用户密码]
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] n
 ... skipping.

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

13.创建对应数据库并授权：

```shell
[root@node2 dynamic]# mysql -uroot -predhat
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 9
Server version: 5.5.56-MariaDB MariaDB Server

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create database wp;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> grant all on wp.* to 'wpadmin'@'%' identified by 'redhat';
Query OK, 0 rows affected (0.00 sec)
```

14.绑定wordpress的数据库：

```shell
[root@node2 dynamic]# cd /data/nginx/dynamic/wp
[root@node2 wp]# ls
index.php        wp-admin              wp-content         wp-load.php      wp-signup.php
license.txt      wp-blog-header.php    wp-cron.php        wp-login.php     wp-trackback.php
readme.html      wp-comments-post.php  wp-includes        wp-mail.php      xmlrpc.php
wp-activate.php  wp-config-sample.php  wp-links-opml.php  wp-settings.php
[root@node2 wp]# cp wp-config-sample.php wp-config.php
--- 修改以下三行内容，如下 ---
define('DB_NAME', 'wp');
define('DB_USER', 'wpadmin');
define('DB_PASSWORD', 'redhat');
```

15.访问`www.aayylink.com/wp`网站查看是否搭建成功：

若搭建正确，则访问结果如下：

![1570610519574](images/1570610519574.png)

16.若想要让该网页可以上传图片，则需要修改wordpress文件夹的权限，使得该文件夹可以被fastcgi的子进程访问：

```shell
[root@node2 wp]# setfacl -Rm u:apache:rwx /data/nginx/dynamic/
```

