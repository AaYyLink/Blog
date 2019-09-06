# 0. Nginx介绍

nginx是俄罗斯人开发的，它可以用来做http服务器、反代服务器以及邮件服务器，并且其可以反代传输层的协议。其并发响应能力要优于httpd。但一般情况下其流行用于反代服务。



本节主要介绍Nginx的http功能配置，并且给出的都为最常用最重要的配置。
## 0.1 Nginx程序结构

```
master/worker
	一个master进程：
		负载加载和分析配置文件、管理或启动worker进程、平滑升级（不要停止Nginx的服务也可以升级其版本）
	一个或多个worker进程
		处理并响应用户请求
	缓存相关的进程：
		cache loader：载入缓存对象
		cache manager：管理缓存对象
```

# 1. Nginx安装

Nginx的官方有源码包和yum源，也可以寻找其他EPEL源来进行yum安装。

以下介绍源码包的编译安装并附上官方yum源（CentOS7版本）。

## 1.1 编译安装

源码包的编译安装需要预先安装依赖项：

```nginx
yum groupinstall "Development Tools" "Server Platform Development" -y
yum install pcre-devel pcre openssl openssl-devel zlib zlib-devel -y
```

然后下载官方的压缩包：

```
http://nginx.org/en/download.html
去上面的网页，下载自己想要的版本
例如我的下载链接是：http://nginx.org/download/nginx-1.16.1.tar.gz

cd /usr/local/src
wget http://nginx.org/download/nginx-1.16.1.tar.gz
```

然后进行解压缩

```
tar -xvf nginx-1.16.1.tar.gz
```

解压缩后，需要编译：

```nginx
./configure --prefix=/usr/share/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --http-proxy-temp-path=/var/lib/nginx/tmp/proxy --http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi --http-uwsgi-temp-path=/var/lib/nginx/tmp/uwsgi --http-scgi-temp-path=/var/lib/nginx/tmp/scgi --pid-path=/run/nginx.pid --lock-path=/run/lock/subsys/nginx --user=nginx --group=nginx --with-file-aio --with-ipv6 --with-http_auth_request_module --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_addition_module --with-http_xslt_module=dynamic --with-http_image_filter_module=dynamic --with-http_geoip_module=dynamic --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_degradation_module --with-http_slice_module --with-http_stub_status_module --with-http_perl_module=dynamic --with-mail=dynamic --with-mail_ssl_module --with-pcre --with-pcre-jit --with-stream=dynamic --with-stream_ssl_module --with-google_perftools_module --with-debug --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -m64 -mtune=generic' --with-ld-opt='-Wl,-z,relro -specs=/usr/lib/rpm/redhat/redhat-hardened-ld -Wl,-E'

make&make install
```

加入启动项后，需要书写Unit文件，使其可以通过systemtl操控。Unit文件的内容如下

```nginx
 vim /usr/lib/systemd/system/nginx.service
[Unit]
Description=The nginx HTTP and reverse proxy server
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
# Nginx will fail to start if /run/nginx.pid already exists but has the wrong
# SELinux context. This might happen when running `nginx -t` from the cmdline.
# https://bugzilla.redhat.com/show_bug.cgi?id=1268621
ExecStartPre=/usr/bin/rm -f /run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
KillSignal=SIGQUIT
TimeoutStopSec=5
KillMode=process
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

> configure的选项可以参考用yum源安装nginx后，使用nginx -V命令查看到的选项。
>
> Unit文件的编写也可以参考yum源安装nginx后产生的/usr/lib/systemd/system/nginx.service文件。



## 1.2 yum源安装

yum仓库配置：

```nginx
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
```



# 2. Nginx基础配置

**Nginx主配置文件**：/etc/nginx/nginx.conf

**Nginx区域配置文件**：/etc/nginx/conf.d/*.conf

**Nginx模块配置文件**：/usr/share/nginx/modules/*.conf

以上区域配置文件和模块配置文件都是由include命令定义的。



## 2.0  官方文档查阅方法

配置文件的组成内容由命令和参数组成，其中参数可以用变量来代替，而变量分为：内置变量及自定义变量。自定义变量可以使用set命令来定义。所以学习nginx的配置，其实就是学习命令的用法和变量的意义。

以下会讲解常见的命令和变量。但在这之前，先讲解nginx官方文档如何查阅命令和内置变量。

```
nginx命令参考：http://nginx.org/en/docs/dirindex.html
nginx内置变量参考：http://nginx.org/en/docs/varindex.html
两者具体参考方法如下：

命令参考方法：
官方文档中命令会给出其具体参数配置（Syntax），默认参数配置（Default），上下文（Context），其中上下文就是其被包含在什么{}中，例如listen的上下文是server，那么listen的定义肯定是下面这样的。
server {
	listen PORT;
}
而有时候，上下文会是main，他会直接写在配置文档中，而不被任何花括号圈中，例如：http。（可以查看/etc/nginx/nginx.conf验证），但这类上下文为main的都是一大个功能模块，就如我们上面nginx介绍的反代服务器，http服务器，邮箱服务器。
用专业术语来说，一个个花括号所圈起来的是一个个配置段，最外层的就是主配置段(main)。

内置变量参考方法：
内置变量的参考较为简单，官方文档会给出其意义并举例，参考其即可。
```

## 2.1 Nginx基础命令

Nginx的常见的命令如下

| 命令及参数                             | 实际意义                                                     |
| -------------------------------------- | ------------------------------------------------------------ |
| listen PORT [default_server]           | 监听的端口，并设置该网页是否为默认网页                       |
| server_name DOMAIN_NAME                | 域名                                                         |
| root DIRECTORY                         | 网页所在的目录                                               |
| alias PATH                             | 其上下文为location，其PATH参数是用来替代location给出的路径的。即用户访问的实际目录为alias后给出的路径 |
| location                               | location可以通过名称或正则表达式，区分各个资源的访问方法，具体的详解将在后面给出 |
| user USER [GROUP]                      | worker进程的用户                                             |
| worker_processes [NUMBER\|auto]        | worker进程数量，auto启动和主机核心数一样的进程数量           |
| error_log PATH                         | 定义错误日志的存放路径                                       |
| pid PATH                               | 定义pid文件的存放路径                                        |
| include FILE                           | 包含的配置文件，可以使用通配符                               |
| events {…}                             | 事件驱动模型配置                                             |
| worker_connection NUM                  | 在events   {…}上下文中，用来定义每个worker能响应的最大连接数量 |
| access_log PATH FORMAT                 | 访问日志路径及其格式                                         |
| sendfile [on/off]                      | 一般使用on，可以提升性能                                     |
| tcp_nopush                             | 一般使用on，可以提升性能                                     |
| tcp_nodelay                            | 一般使用on，可以提升性能                                     |
| keepalive_timeout                      | 保持连接，超时时长                                           |
| error_page CODE ... [=[RESPONSE]] URI; | 错误代码及用户对于这些错误代码所访问到的页面                 |

> 配置文件中每一个命令一般都以分号(;)结尾，但后跟花括号的命令则不用。

## 2.2 Nginx基础内置变量

| 内置变量             | 意义                                                         |
| -------------------- | ------------------------------------------------------------ |
| remote_addr          | 客户端地址                                                   |
| remote_port          | 客户端端口号                                                 |
| remote_user          | basic认证时提供的用户名                                      |
| time_local           | 本地时间                                                     |
| request              | 请求报文的起始行,<method><URL><VERSION>                      |
| status               | 响应码                                                       |
| body_bytes_sent      | 响应报文的字节数                                             |
| http_referer         | 引用                                                         |
| http_user_agent      | 用户浏览器                                                   |
| http_x_forwarded_for | 该服务器代理的客户端IP地址   （官方文档中对应   proxy_add_x_forwarded_for） |

## 2.3  Nginx静态网页配置

nginx不像LAMP一样有主主机和虚拟主机的区分，其每一个server命令对应一个主机。

配置一个静态网页的过程如下：

```nginx
vim /etc/nginx/conf.d/vhost.conf
server {
        listen 80;
        server_name www.vhost.com;
        root /data/nginx/vhost;
}

mkdir /data/nginx/vhost -pv
echo "<h1>Welcome to Nginx</h1><p>this is vhost</p>" >> /data/nginx/vhost/index.html

echo "192.168.100.30 www.vhost.com" >> /etc/hosts
```

> 需要注意的是，若要通过主机名区分server，则需要修改hosts文件。如果想通过IP地址访问新建立的静态网页，则需要在listen 80修改为listen 80 default，并将nginx.conf中原来有listen ... default的命令中的default 删去。



# 3. Nginx进阶配置

以上的基础配置对于配置一个静态网页以绰绰有余，但是在企业中肯定是不可能仅仅只用那么几个功能的。下面就讲讲Nginx的进阶配置。

首先将main配置段常见的配置命令分为以下四种：

- 正常运行必备的配置（包括了上面很多的基础配置命令）
- 优化性能相关的配置
- 用于调试及定位问题相关的配置
- 事件驱动相关的配置

以下将从上面4个分类角度来介绍Nginx常用的进阶配置。

## 3.1 正常运行必备的配置

> Syntax是语法，Default是默认配置，Context是上下文。

### 3.1.1 user

```nginx
Syntax:		user user [group];
Default:	user nobody nobody;
Context:	main
```

指定运行worker进程的用户和组。

### 3.1.2 pid

```nginx
Syntax:		pid file;
Default:	pid logs/nginx.pid;
Context:	main
```

指定存储nginx主进程进程号码的文件路径

### 3.1.3 include

```nginx
Syntax:		include file | mask;
Default:	—
Context:	any
```

指定包含进来的其他配置文件，mask指的是可以使用通配符匹配

### 3.1.4 load_module

```nginx
Syntax:		load_module file;
Default:	—
Context:	main
This directive appeared in version 1.9.11.
```

指明要装载的动态模块

## 3.2 性能优化相关的配置

### 3.2.1 worker_processes

```nginx
Syntax:		worker_processes number | auto;
Default:	worker_processes 1;
Context:	main
The auto parameter is supported starting from versions 1.3.8 and 1.2.5.
```

定义启动的worker进程数量，若为auto则启动的数量与核心数相同。

### 3.2.2 worker_cpu_affinity

```nginx
Syntax:		worker_cpu_affinity cpumask ...;
			worker_cpu_affinity auto [cpumask];
Default:	—
Context:	main
```

讲一个worker进程绑定在某块或某几块CPU上。

例如：一台电脑上有4个CPU，想将第1个worker进程绑定在第0个CPU（CPU从0开始计数），第2个绑定在第1个CPU，以此类推。则配置如下：

```
worker_processes    4;
worker_cpu_affinity 0001 0010 0100 1000;
```

若想把第一个worker进程绑定在第0个和第2个CPU上，第二个绑定在第1个和第3个上。

```nginx
worker_processes    2;
worker_cpu_affinity 0101 1010;
```



auto的用法比较特殊，使用其可以让所有worker进程随机绑定在指定的CPU上，例如：

```nginx
worker_cpu_affinity auto 01010101
```

表示worker进程会自动绑定在第0、2、4、6块CPU上。

### 3.2.3 worker_priority

```nginx
Syntax:		worker_priority number;
Default:	worker_priority 0;
Context:	main
```

指定worker进程的nice值，设定worker进程优先级；[-20,20]

### 3.2.4 worker_rlimit_nofile

```nginx
Syntax:		worker_rlimit_nofile number;
Default:	—
Context:	main
```

worker进程所能够打开的文件数量上限，nginx总共可以打开的文件数量上限=进程数*worker_rlimit_nofile定义的数量。

### 3.2.5 关于性能优化的总结

上面的这些都是性能优化是所需要使用到的命令，但如何性能优化，我在这里给出一些小小提示。例如，worker_processes设定的值，请不要超过主机最大的核心数量，因为这样会大大产生进程之间的切换，反而使得性能不升反降，而有时候一台服务器上可能并不运行nginx一个程序，因此要考虑到其他程序占用核心的情况。总之，在启用各个服务时，务必让服务器有空闲的进程以备不时之需。CPU的绑定可以提高缓存命中率。worker_rlimit_nofile的设定尽量配合ab命令压力测试取出一个最适合本机的值。

## 3.3  调试、定位问题

### 3.3.1 daemon

```nginx
Syntax:		daemon on | off;
Default:	daemon on;
Context:	main
```

是否以守护进程方式运行Nignx；CentOS6上若要以守护进程方式运行，则必须启用。CenOS7上则不必要，因为已有systemd。

### 3.3.2 master_process

```nginx
Syntax:		master_process on | off;
Default:	master_process on;
Context:	main
```

是否启用worker进程，若使用off，则可以进行调试。

### 3.3.3 error_log

```nginx
Syntax:		error_log file [level];
Default:	error_log logs/error.log error;
Context:	main, http, mail, stream, server, location
```

设定错误日志的保存路径及日志级别。

日志级别有：debug，info，notice，warn，error，crit，alert或emerg

设置的级别及其以上的级别会被记录在错误日志中。

## 3.4 事件驱动相关的配置

事件驱动相关的配置上下文都为events {...}

### 3.4.1 worker_connections

```nginx
Syntax:		worker_connections number;
Default:	worker_connections 512;
Context:	events
```

每个worker进程所能够打开的最大并发连接数数量。
nginx总共可以打开的进程数量=worker_processes * worker_connections

### 3.4.2 use

```nginx
Syntax:		use method;
Default:	—
Context:	events
```

指明并发连接请求的处理方法；通常不需要指明，因为nginx会默认使用最有效的方法。尽量使用epoll而不是select，因为select最大并发连接为1024。所以use epoll。

### 3.4.3 accept_mutex

```nginx
Syntax:		accept_mutex on | off;
Default:	accept_mutex off;
Context:	events
```

处理新的连接请求的方法；on意味着由各worker轮流处理新请求，Off意味着每个新请求的到达都会通知所有的worker进程；

# 4. Nginx关于http的配置

```nginx
Syntax:		http { ... }
Default:	—
Context:	main
```

http也是一个指令，其参数是花括号中的配置内容，配置以main段为上下文。

## 4.1 与套接字相关的配置

### 4.1.1 server

```nginx
Syntax:		server { ... }
Default:	—
Context:	http
```

用来设定一个主机，在其中使用listen、root、server_name等命令定义该主机。

### 4.1.2 listen

```nginx
Syntax:		listen address[:port] [default_server] [ssl] [http2 | spdy] [proxy_protocol] [setfib=number] [fastopen=number] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [deferred] [bind] [ipv6only=on|off] [reuseport] [so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]];
listen port [default_server] [ssl] [http2 | spdy] [proxy_protocol] [setfib=number] [fastopen=number] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [deferred] [bind] [ipv6only=on|off] [reuseport] [so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]];
listen unix:path [default_server] [ssl] [http2 | spdy] [proxy_protocol] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [deferred] [bind] [so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]];
Default:	listen *:80 | *:8000;
Context:	server
```

listen的参数太多，以下列出常用的参数及其具体作用或用法：

default_server：设定为默认虚拟主机；
ssl：限制仅能够通过ssl连接提供服务；
backlog=number：后援队列长度；
rcvbuf=size：接收缓冲区大小；
sndbuf=size：发送缓冲区大小；

### 4.1.3 server_name

```nginx
Syntax:		server_name name ...;
Default:	server_name "";
Context:	server
```

server_name后面可以跟多个主机名称，这些主机名称也可以加入通配符或用正则表达式替换。例如：

通配符匹配：server_name \*.magedu.com  www.magedu.\*

正则表达式匹配（需要以\~开头）：\~\^www\\d\+\.magedu\.com\$

如果匹配到多个，则按以下顺序匹配：

```nginx
1.确定的名称，不含通配符和正则表达式
2.以*号开头的最长的通配符段
3.以*号结尾的最长的通配符段
4.正则表达式
```

### 4.1.4 tcp_nodelay

```nginx
Syntax:		tcp_nodelay on | off;
Default:	tcp_nodelay on;
Context:	http, server, location
```

在keepalived模式下的连接是否启用TCP_NODELAY选项。可以提高性能，一般选择on。

### 4.1.5 sendfile

```nginx
Syntax:		sendfile on | off;
Default:	sendfile off;
Context:	http, server, location, if in location
```

是否启用sendfile功能。可以提高性能，一般选择on。

## 4.2 定义路径相关的配置

### 4.2.1 root

```nginx
Syntax:		root path;
Default:	root html;
Context:	http, server, location, if in location
```

设置web资源路径映射。其中相对路径是相对于/usr/share/nginx的，也就是相对于configure的--prefix选项内容的路径。（可以使用nginx -V查看configure路径）

### 4.2.2 location

```nginx
Syntax:		location [ = | ~ | ~* | ^~ ] uri { ... }
			location @name { ... }
Default:	—
Context:	server, location
```

location是用来匹配用户访问的URL，然后在花括号中定义这类URL的特殊配置。例如：

```
location /link/ {
	root /data/nginx/link;
}
```

意思为当用户访问的是link目录以下的资源时，则以/data/nginx/link作为web资源路径映射的目录。

location后所跟的四种符号意义如下：

1. =：对URI做精确匹配；
2. ~：对URI做正则表达式模式匹配，区分字符大小写；
3. ~*：对URI做正则表达式模式匹配，不区分字符大小写；
4. ^~：对URI的左半部分做匹配检查，不区分字符大小写；
5. 不带符号：匹配起始于此uri的所有的url

相同的符号匹配最长的。

**匹配优先级**：=, ^~, ～/～*，不带符号

以官方网站给出的例子理解上面的优先级：

```nginx
location = / {
    [ configuration A ]
}

location / {
    [ configuration B ]
}

location /documents/ {
    [ configuration C ]
}

location ^~ /images/ {
    [ configuration D ]
}

location ~* \.(gif|jpg|jpeg)$ {
    [ configuration E ]
}
```

访问`/`时，会匹配A。如果访问的是`/index.html`时，会匹配B。如果访问的是`/documents/document.html`则会匹配C。`/images/1.gif`，则会匹配D。`/documents/1.jpg`，则会匹配E。

### 4.2.3 alias

```nginx
Syntax:		alias path;
Default:	—
Context:	location
```

定义路径别名。例如：

```nginx
location /i/ {
    alias /data/w3/images/;
}
```

客户端访问`/i/top.gif`则会发送`/data/w3/images/top.gif`。

有时候alias可以使用root替代，则尽量使用root替代，例如：

```
location /images/ {
    alias /data/w3/images/;
}
```

替代为root

```
location /images/ {
    root /data/w3;
}
```

### 4.2.4 index

```nginx
Syntax:		index file ...;
Default:	index index.html;
Context:	http, server, location
```

用来定义一个资源路径默认访问的页面。例如配置为`index index.html`则访问`/`时，显示的是index.html文件的内容。可以定义多个文件，匹配时从左到右匹配，并且最后一个元素可以是具有绝对路径的文件。

### 4.2.5 error_page

```nginx
Syntax:		error_page code ... [=[response]] uri;
Default:	—
Context:	http, server, location, if in location
```

在反馈给用户的状态码是错误的时候，显示给用户对应的错误网页。

## 4.3 定义客户端请求的相关配置

### 4.3.1 keepalive_timeout

```nginx
Syntax:		keepalive_timeout timeout [header_timeout];
Default:	keepalive_timeout 75s;
Context:	http, server, location
```

设定保持连接的超时时长，若为0则表示禁止保持连接。

### 4.3.2 keepalive_requests

```nginx
Syntax:		keepalive_requests number;
Default:	keepalive_requests 100;
Context:	http, server, location
This directive appeared in version 0.8.0.
```

在一次长连接上所允许请求的资源的最大数量。

### 4.3.4 keepalive_disable

```nginx
Syntax:		keepalive_disable none | browser ...;
Default:	keepalive_disable msie6;
Context:	http, server, location
```

指定对哪种浏览器禁用保持连接。msie6禁用老版本的MSIE。safari禁用mac系统或类mac系统上的safari以及类safari浏览器的长连接。

### 4.3.5 send_timeout

```nginx
Syntax:		send_timeout time;
Default:	send_timeout 60s;
Context:	http, server, location
```

向客户端发送响应报文的超时时长，此处的超时时长，是指两次写操作之间的间隔时长。如果客户端在此期间未收到任何内容，则会关闭连接。

### 4.3.6 client_body_buffer_size

```nginx
Syntax:		client_body_buffer_size size;
Default:	client_body_buffer_size 8k|16k;
Context:	http, server, location
```

用于接收客户端请求报文的body部分的缓冲区大小；默认为16k；超出此大小时，其将被暂存到磁盘上的由client_body_temp_path指令所定义的位置；

### 4.3.7 client_body_temp_path

```nginx
Syntax:		client_body_temp_path path [level1 [level2 [level3]]];
Default:	client_body_temp_path client_body_temp;
Context:	http, server, location
```

设定用于存储客户端请求报文的body部分的临时存储路径及子目录结构和数量；

其中子目录结构是使用level1-3来定义的，分级存放的好处就是降低查询的速度。例如：

```
client_body_temp_path /spool/nginx/client_temp 1 2;
```

临时文件的存放可以是

```
/spool/nginx/client_temp/7/45/00000123457
```

其中1表示用1位16进制数字表示一级子目录：0-f，2表示用2位16进制数字表示2及子目录：00-ff。

## 4.4 对客户端进行限制的相关配置

### 4.4.1 limit_rate（一般不限速）

```nginx
Syntax:	limit_rate rate;
Default:	
limit_rate 0;
Context:	http, server, location, if in location
```

限制响应给客户端的传输速率，单位是bytes/second，0表示无限制；

### 4.4.2 limit_except（安全）

```nginx
Syntax:		limit_except method ... { ... }
Default:	—
Context:	location
```

限制对指定的请求方法之外的其它方法的使用客户端（看举例更好理解一些）。

例如：

```
limit_except GET {
	allow 192.168.1.0/24;
	deny  all;
}
```

指定GET方法以外的方法，只允许192.168.1.0/24网段的主机使用。

### 4.4.3 aio（建议为on）

```nginx
Syntax:		aio on | off | threads[=pool];
Default:	aio off;
Context:	http, server, location
This directive appeared in version 0.8.11.
```

是否启用异步I/O功能。

### 4.4.4 directio（不关键）

```nginx
Syntax:		directio size | off;
Default:	directio off;
Context:	http, server, location
This directive appeared in version 0.7.7.
```

直接IO，在Linux主机启用O_DIRECT标记，此处意味文件大于等于给定的大小时使用，例如directio 4m;不关键，使用默认值即可。

### 4.4.5 open_file_cache（重要）

```nginx
Syntax:		open_file_cache off;
			open_file_cache max=N [inactive=time];
Default:	open_file_cache off;
Context:	http, server, location
```

nginx可以缓存的信息有以下三种：

(1) 文件的描述符、文件大小和最近一次的修改时间；
(2) 打开的目录结构；
(3) 没有找到的或者没有权限访问的文件的相关信息；

max=N：可缓存的缓存项上限；达到上限后会使用LRU算法实现缓存管理；

inactive=time：缓存项的非活动时长，在此处指定的时长内未被命中的或命中的次数少于open_file_cache_min_uses指令所指定的次数的缓存项即为非活动项；

### 4.4.6 open_file_cache_valid

```nginx
Syntax:		open_file_cache_valid time;
Default:	open_file_cache_valid 60s;
Context:	http, server, location
```

缓存项有效性的检查频率

### 4.4.7 open_file_cache_min_uses

```nginx
Syntax:		open_file_cache_min_uses number;
Default:	open_file_cache_min_uses 1;
Context:	http, server, location
```

在open_file_cache指令的inactive参数指定的时长内，至少应该被命中多少次方可被归类为活动项

### 4.4.8 open_file_cache_errors

```nginx
Syntax:		open_file_cache_errors on | off;
Default:	open_file_cache_errors off;
Context:	http, server, location
```

是否缓存查找时发生错误的文件一类的信息

## 4.5 ngx_http_access_module模块

该模块的功能实现的是基于ip的访问控制功能。

### 4.5.1 allow

```nginx
Syntax:		allow address | CIDR | unix: | all;
Default:	—
Context:	http, server, location, limit_except
```

允许访问的指定的网络或地址。如果还指定了特殊值unix，则表示允许访问所有unix域套接字。

### 4.5.2 deny

```nginx
Syntax:		deny address | CIDR | unix: | all;
Default:	—
Context:	http, server, location, limit_except
```

拒绝访问的指定的网络或地址。如果还指定了特殊值unix，则表示拒绝访问unix域套接字。

## 4.6 ngx_http_auth_basic_module模块

该模块主要实现基于用户的访问控制，使用basic机制进行用户认证；

### 4.6.1 auth_basic

```nginx
Syntax:		auth_basic string | off;
Default:	auth_basic off;
Context:	http, server, location, limit_except
```

是否开启basic认证功能，如果为off表示关闭，如果使用string，则会在用户认证时，输出string的提示。

### 4.6.2 auth_basic_user_file

```nginx
Syntax:		auth_basic_user_file file;
Default:	—
Context:	http, server, location, limit_except
```

basic认证时，用户名和密码所保存的文件。得使用htpasswd实现：

```
yum install httpd-tools -y 
htpasswd -c -m /etc/nginx/.ngxpasswd tom #第一次创建密码文件需要添加-c参数
htpasswd -m /etc/nginx/.ngxpasswd jerry
```

然后用此参数指向该文件即可。

## 4.7 ngx_http_stub_status_module

该模块主要用于输出nginx的基本状态信息。

### 4.7.1 stub_status

```nginx
Syntax:		stub_status;
Default:	—
Context:	server, location
```

可以基于location来访问nginx的基本状态信息。配置示例如下：

```
location  /ngxstatus {
	stub_status;
}
```

我的访问结果如下：

```
Active connections: 1 
server accepts handled requests
 5 5 5 
Reading: 0 Writing: 1 Waiting: 0
```

各个字段的意义如下：

```
Active connections: 活动状态的连接数；

accepts：已经接受的客户端请求的总数；

handled：已经处理完成的客户端请求的总数；

requests：客户端发来的总的请求数；

Reading：处于读取客户端请求报文首部的连接的连接数；

Writing：处于向客户端发送响应报文过程中的连接数；

Waiting：处于等待客户端发出请求的空闲连接数；
```

## 4.8 ngx_http_log_module

该模块主要用来指定nginx日志的格式

### 4.8.1 **log_format** 

```nginx
Syntax:		log_format name [escape=default|json|none] string ...;
Default:	log_format combined "...";
Context:	http
```

用来定义一种日志格式，然后让在设定日志具体格式时调用。name是用来定义日志格式的名称，使其可以被调用；string中则是具体的日志格式定义，可以使用nginx核心模块及其它模块内嵌的变量。

### 4.8.2 access_log

```nginx
Syntax:		access_log path [format [buffer=size] [gzip[=level]] [flush=time] [if=condition]];
			access_log off;
Default:	access_log logs/access.log combined;
Context:	http, server, location, if in location, limit_except
```

设定访问日志路径，并且设置其格式（可以是自己在log_format命令中定义的）。还可以缓冲相关（buffer和flush）的设置以及数据压缩(gzip)方面的设置。其中压缩的等级为1-9级，等级越低，解压缩越快，压缩比率越低。

### 4.8.3 open_log_file_cache

```nginx
Syntax:		open_log_file_cache max=N [inactive=time] [min_uses=N] [valid=time];
			open_log_file_cache off;
Default:	open_log_file_cache off;
Context:	http, server, location
```

缓存各日志文件相关的元数据信息，各个选项意义如下：

```nginx
max：缓存的最大文件描述符数量；
min_uses：在inactive指定的时长内访问大于等于此值方可被当作活动项；
inactive：非活动时长；
valid：验正缓存中各缓存项是否为活动项的时间间隔；
```

## 4.9 ngx_http_gzip_module

ngx_http_gzip_module模块是一个使用“gzip”方法压缩响应的过滤器。 这通常有助于将传输数据的大小减少一半甚至更多。

### 4.9.0 模块配置示例

```nginx
gzip  on;
gzip_comp_level 6;
gzip_min_length 64;
gzip_proxied any;
gzip_types text/xml text/css  application/javascript;	
```

### 4.9.1 gzip

```nginx
Syntax:		gzip on | off;
Default:	gzip off;
Context:	http, server, location, if in location
```

是否开启gzip压缩。

### 4.9.2 gzip_comp_level

```nginx
Syntax:		gzip_comp_level level;
Default:	gzip_comp_level 1;
Context:	http, server, location
```

设定压缩的等级。压缩的等级为1-9级，等级越低，解压缩越快，压缩比率越低。

### 4.9.3 gzip_disable

```nginx
Syntax:		gzip_disable regex ...;
Default:	—
Context:	http, server, location
This directive appeared in version 0.6.23.
```

对正则表达式匹配到的浏览器取消压缩功能。

### 4.9.4 gzip_min_length

```nginx
Syntax:		gzip_min_length length;
Default:	gzip_min_length 20;
Context:	http, server, location
```

启用压缩功能的响应报文长度阈值，也就是响应报文长度大于多少了才会进行压缩，以KB为单位。

### 4.9.5 gzip_buffers

```nginx
Syntax:		gzip_buffers number size;
Default:	gzip_buffers 32 4k|16 8k;
Context:	http, server, location
```

支持实现压缩功能时为其配置的缓冲区数量及每个缓存区的大小；默认情况下，缓冲区大小等于一个内存页面。

### 4.9.6 gzip_proxied

```nginx
Syntax:		gzip_proxied off | expired | no-cache | no-store | private | no_last_modified | no_etag | auth | any ...;
Default:	gzip_proxied off;
Context:	http, server, location
```

nginx作为代理服务器接收到从被代理服务器发送的响应报文后，在何种条件下启用压缩功能的；

各参数意义如下：

```
off:禁用所有代理请求的压缩，忽略其他参数;

expired:如果响应头包含“Expires”字段，其值为禁用缓存，则启用压缩;

no-cache:如果响应头包含带有“no-cache”参数的“Cache-Control”字段，则启用压缩;

no-store:如果响应头包含带有“no-store”参数的“Cache-Control”字段，则启用压缩;

private:如果响应头包含带有“private”参数的“Cache-Control”字段，则启用压缩;

no_last_modified:如果响应头不包含“Last-Modified”字段，则启用压缩;

no_etag:如果响应头不包含“ETag”字段，则启用压缩;

auth:如果请求标头包含“Authorization”字段，则启用压缩;

any:为所有代理请求启用压缩。
```

### 4.9.7 gzip_types

```nginx
Syntax:		gzip_types mime-type ...;
Default:	gzip_types text/html;
Context:	http, server, location
```

压缩过滤器，仅对此处设定的MIME类型的内容启用压缩功能。MIME类型可以是text/xml、text/css 、application/javascript。

## 4.10 ngx_http_ssl_module

基于SSL认证的模块。该模块并不是默认模块，需要在编译时加入--with-http_ssl_module参数，并且其需要OpenSSL库。

### 4.10.0 模块配置示例

	server {
		listen 443 ssl;
		erver_name www.magedu.com;
		root /vhosts/ssl/htdocs;
		ssl on;
		ssl_certificate /etc/nginx/ssl/nginx.crt;
		ssl_certificate_key /etc/nginx/ssl/nginx.key;
		ssl_session_cache shared:sslcache:20m;
	}	
### 4.10.1 ssl 

```nginx
Syntax:		ssl on | off;
Default:	ssl off;
Context:	http, server
```

是否启用ssl，该指令在1.15.0版本中已过时， 应该使用listen指令的ssl参数。

### 4.10.2 ssl_certificate

```nginx
Syntax:		ssl_certificate file;
Default:	—
Context:	http, server
```

当前虚拟主机使用PEM格式的证书文件

### 4.10.3 ssl_certificate_key

```nginx
Syntax:		ssl_certificate_key file;
Default:	—
Context:	http, server
```

当前虚拟主机上与其证书匹配的私钥文件

### 4.10.4 ssl_protocols

```nginx
Syntax:		ssl_protocols [SSLv2] [SSLv3] [TLSv1] [TLSv1.1] [TLSv1.2] [TLSv1.3];
Default:	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
Context:	http, server
```

nginx支持的ssl协议版本。

### 4.10.5 ssl_session_cache

```nginx
Syntax:		ssl_session_cache off | none | [builtin[:size]] [shared:name:size];
Default:	ssl_session_cache none;
Context:	http, server
```

设置SSL缓存相关的参数，也可以直接关闭SSL缓存（off）

各参数意义如下：

```
off:禁止会话被缓存，并且告诉客户端会话不会被缓存

none:禁止会话被缓存，但告诉客户端会话会被缓存。

builtin:一个用OpenSSL构建的缓存; 仅由一个工作进程使用。 缓存大小在会话中指定。 如果未给出大小，则等于20480个会话。 使用内置缓存可能会导致内存碎片。

shared:所有工作进程之间共享的缓存。 高速缓存大小以字节为单位指定; 一兆字节可以存储大约4000个会话。 每个共享缓存都应具有任意名称。 可以在多个虚拟服务器中使用具有相同名称的缓存。
```

### 4.10.6 ssl_session_timeout

```nginx
Syntax:		ssl_session_timeout time;
Default:	ssl_session_timeout 5m;
Context:	http, server
```

客户端一侧的连接可以复用ssl_session_cache中缓存的ssl参数的有效时长。

## 4.11 ngx_http_rewrite_module

ngx_http_rewrite_module模块用于使用PCRE正则表达式更改请求URI，返回重定向和有条件地选择配置。通俗易懂的来说就是修改客户访问的URL。

### 4.11.1 rewrite

```nginx
Syntax:		rewrite regex replacement [flag];
Default:	—
Context:	server, location, if
```

将用户请求的URI基于regex所描述的模式进行检查，匹配到时将其替换为replacement指定的新的URI。

这里的正则表达式可以在replacement中后向引用，例如:

修改用户访问jpg格式的图片为访问png格式的图片。

```nginx
server {
	... ...
	rewrite /(.*)\.jpg$ /$1.png;
	... ...
}
```

将用户访问的80端口修改为443端口：

```nginx
server {
	... ...
	rewrite /(.*)$ https://www.aayylink.com/$1;	
	... ...
}
```

然而需要明白的是rewrite功能并没有那么简单，一般情况下一个location中可能会有多个rewrite，一个rewrite命令重写完成后，需要查看是否还有匹配的location以及location中是否还有rewrite，经过不断的迭代，到没有匹配的rewrite时，才会反馈结果给用户。

其中flag参数就可以对上面的重写方式进行改良。

flag有以下四种：

```
last:停止处理当前的ngx_http_rewrite_module指令集，并开始搜索与更改的URI匹配的新location;可以理解成C语言循环中的continue，这里的循环体对应location块。

break:停止处理当前的ngx_http_rewrite_module指令集;可以理解为C语言循环中的break。

redirect:返回给客户端302代码，指明这是重定向的信息，并会在响应报文中给出新的location。只有当替换字符串不以“http：//”，“https：//”或“$ scheme”开头时使用。

permanent:返回给客户端301状态码的永久重定向。
```

需要注意的是：redirect和permanent是需要浏览器重新发送请求的，因此可以让其请求另外一个服务器的资源。

### 4.11.2 return

```nginx
Syntax:		return code [text];
			return code URL;
			return URL;
Default:	—
Context:	server, location, if
```

停止处理命令，并将指定的状态码返回给客户端。若为非标准代码444，则会在不发送响应头的情况下关闭连接。

从版本0.8.42开始，可以指定重定向URL（对于代码301,302,303,307和308）或响应正文文本（对于其他代码）。 响应正文文本和重定向URL可以包含变量。 作为特殊情况，可以将重定向URL指定为此服务器的本地URI，在这种情况下，根据请求方案（$ scheme）以及server_name_in_redirect和port_in_redirect指令形成完整重定向URL。

另外，可以将用于具有代码302的临时重定向的URL指定为唯一参数。 这样的参数应该以“http：//”，“https：//”或“$ scheme”字符串开头。 URL可以包含变量。

### 4.11.3 rewrite_log

```nginx
Syntax:		rewrite_log on | off;
Default:	rewrite_log off;
Context:	http, server, location, if
```

是否开启重写日志。

### 4.11.4 if

```nginx
Syntax:		if (condition) { ... }
Default:	—
Context:	server, location
```

该命令先判断condition，若为真则会执行花括号中的内容。

condition的具体使用如下：

```
变量名;如果变量的值为空字符串或“0”，则为false;

使用“=”和“！=”运算符比较变量和字符串;

使用“〜”（对于区分大小写的匹配）和“〜*”（对于不区分大小写的匹配）运算符，将变量与正则表达式进行匹配。正则表达式可以包含可供以后在$ 1 .. $ 9变量中重用的捕获。负操作符“！〜”和“！〜*”也可用。如果正则表达式包含“}”或“;”字符，则整个表达式应包含在单引号或双引号中。

使用“-f”和“！-f”运算符检查文件是否存在;

使用“-d”和“！-d”运算符检查目录是否存在;

使用“-e”和“！ -  e”运算符检查文件，目录或符号链接是否存在;

使用“-x”和“！-x”运算符检查可执行文件。

#在1.0.1版之前，任何以“0”开头的字符串都被视为false值。
```



官方配置案例：

```nginx
if ($http_user_agent ~ MSIE) {
    rewrite ^(.*)$ /msie/$1 break;
}

if ($http_cookie ~* "id=([^;]+)(?:;|$)") {
    set $id $1;
}

if ($request_method = POST) {
    return 405;
}

if ($slow) {
    limit_rate 10k;
}

if ($invalid_referer) {
    return 403;
}
```

第1个案例表示重定向所有浏览器匹配到MSIE的客户端请求，第3个案例表示拒绝POST方法并返回405状态码。

### 4.11.5 set

```nginx
Syntax:		set $variable value;
Default:	—
Context:	server, location, if
```

用户自定义变量；例如：`set $im images`

## 4.12 ngx_http_referer_module

ngx_http_referer_module模块用于阻止对“Referer”头字段中具有无效值的请求访问站点。也就是防盗链用的，防盗链的意义可以用一个例子来理解：当我们转载他人的文章到自己的博客时，可能会发现有一些图片显示不了。而这里的"显示"不了，其实就是"引用(referer)"不了。

该模块有一个内置变量：

```nginx
$invalid_referer:如果“Referer”请求标头字段值被认为有效，则为空字符串，否则为“1”。
```

一般而言，这个变量会配合if使用，例如4.11.4中官方案例的最后一个。

### 4.12.1 valid_referers

```nginx
Syntax:		valid_referers none | blocked | server_names | string ...;
Default:	—
Context:	server, location
```

这个命令就是用来设定\$invalid\_referer的值。若匹配到后面的元素则会将\$invalid\_referer变量设定为空串，否则，设定为1。

各参数的意义如下。

```
none:请求报头中并无"referer"字段。

blocked:“Referer”字段出现在请求报头中，但其值已被防火墙或代理服务器删除; 这些值是不以“http：//”或“https：//”开头的字符串。

server_names:指定合法的主机名，可以不止一个。而server_names的参数可以是以下两种：
	arbitrary string：完整的主机名或是URI前缀，主机名称的开头或结尾可以包含"*"。在检查期间，忽略“Referer”字段中的服务器端口。
	regular expression：正则表达式，必须以~符号开头。
```

官方案例：

```nginx
valid_referers none blocked server_names
               *.example.com example.* www.example.org/galleries/
               ~\.google\.;
```

一般情况下最后一项`~\.google\.`都需要添加，因为这样子才可以允许搜索引擎收入自己的网站。也就是说别人可以用谷歌搜索到你的网站。

