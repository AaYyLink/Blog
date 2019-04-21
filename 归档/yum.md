[TOC]



# 1.  用CD做yum源

下面讲解如何使用CD做yum源。

既然是CD源，那肯定需要在系统中插入CD呀，所以要先往主机中插入CD。

然后挂载CD，我们这里使用永久挂载CD的方式做挂载,并使用`mount -a`挂载`/etc/fstab`下的所有挂载项（由于光盘是只可以读的，所以指定ro选项）。

```shell
mkdir /media/cdrom
echo "/dev/cdrom /media/cdrom iso9660 defaults,ro 0 0" >> /etc/fstab
mount -a
```

挂载光盘完成后，就可以使用光盘源上的内容了，这里我顺便讲解下程序包的保存位置。每个程序包所在的目录的父目录下必定有一个repodata，而我们使用YUM源指定路径时，就是指向这个父目录。下面给出光盘挂载点下的内容：

![1555809071543](images/1555809071543.png)

可以发现其下就有一个repoda，其中Packages就是具体保存程序包的地方。因此，我们设置yum源时，只需使其URL指向光盘的挂载点即可。

接下来直接设置一个关于光盘的yum源。

```shell
[root@www ~]# vim /etc/yum.repos.d/CDROM.repo
[CDROM]
name=CDROM
baseurl=file:///media/cdrom
enabled=1
gpgcheck=0
```

`/etc/yum.repos.d/`下保存了各种各样的yum源配置文件，而且配置文件必须以`.repo`结尾。

然后，我们使用下面的命令验证该仓库是否可用。

```shell
yum repolist all | grep CDROM
```

![1555809715978](images/1555809715978.png)

若后面有enbaled，则代表该仓库可用。

然后我们先清理原来的缓存，然后创建新的缓存：

```shell
yum clean all
yum makecache
```

然后尝试安装一个emacs

```shell
yum install emacs
```

![1555810347016](images/1555810347016.png)

显示的yum源来自于CDROM。说明我们刚刚配置正确



> 其实我们也可以将光盘中的Packages和repodata拷贝至某一个目录下，然后让repo配置文件的baseurl指向新的目录。

# 2.  yum配置文件详解

yum的配置文件有两种：

1：主配置文件：/etc/yum.conf，用来为每个仓库提供公共配置。

2：各仓库的配置，/etc/yum.repos.d/*.repo

下面讲讲两个配置文件的具体内容



## 2.1  主配置文件内容详解

首先讲解一下yum的配置文件，配置文件有自己的内置变量，常见的有：

\$basearch：内容为当前主机的基础平台

\$arch：内容为当前主机的具体平台

\$releasever：当前OS发行版的主版本号

```shell
[main]			# [main]开头表示下面的内容都是公共配置信息
cachedir=/var/cache/yum/$basearch/$releasever			# 用来指定缓存保存的目录
keepcache=0				# 是否保存缓存
debuglevel=2			# 指定调试级别
logfile=/var/log/yum.log			# 日志文件保存位置
exactarch=1			# 是否允许更新不同平台的程序包
obsoletes=1			
gpgcheck=1			# 是否开启验证
plugins=1
installonly_limit=5
bugtracker_url=http://bugs.centos.org/set_project.php?project_id=23&ref=http://bugs.centos.org/bug_report_page.php?category=yum
distroverpkg=centos-release
```



## 2.2  仓库配置文件详解

```shell
[repositoryid]			# 用来唯一标识仓库的id
name=			# 用来描述当前仓库的信息
baseurl=url://path/to/repository/			# 用来标识仓库的位置，url支持http,ftp,file
enabled={1|0}			# 该仓库是否被弃用
gpgcheck={1|0}			# 是否对程序包的来源合法性做检查，若选择是，则需要指定gpgkey
gpgkey=url://path/to/keyfile			# 用来指明gpgkey的路径
cost=			# 指明当前仓库的访问开销
```



# 3  常用yum命令

关于程序包的常用命令：

```shell
yum install <PACKAGENAME> [PACKAGE2NAME]... [--enablerepo=REPOID]	# 后面的选项指定用哪个仓库的程序包
yum update <PACKAGENAME> [PACKAGE2NAME]...
yum update-to <PACKAGENAME> [PACKAGE2NAME]...
yum downgrade <PACKAGENAME> [PACKAGE2NAME]...
yum reinstall <PACKAGENAME> [PACKAGE2NAME]...
yum check-update
yum remove <PACKAGENAME> [PACKAGE2NAME]...
yum list [PACKAGENAME] [PACKAGE2NAME]... 
yum info <PACKAGENAME> [PACKAGE2NAME]...
yum provide <PACKAGENAME> [PACKAGE2NAME]...		# 列出该包提供的特性
yum deplist <PACKAGENAME> [PACKAGE2NAME]...		# 列出该包依赖的特性

## 安装时指定-y参数，可以对交互自动回答为yes
```



关于包组的常用命令有：

```shell
yum groupinstall <GROUPNAME> [GROUP2NAME]...		
yum groupupdate <GROUPNAME> [GROUP2NAME]...
yum grouplist		# 列出所有的包组
yum groupremove <GROUPNAME> [GROUP2NAME]...
yum groupinfo <GROUPNAME> [GROUP2NAME]
```



关于仓库的命令：

```shell
yum repolist [all|enabled|disabled]			# 用来列出已经配置的(所有|可用|不可用)仓库
yum clean [packages|metadata|rpmdb|all]		# 清除(程序包|元数据|rpm数据库|所有)的缓存
yum makecache：创建一个新的缓存，其中包含各个可用仓库的元数据
```



# 4.  使用EPEL源

EPEL源包含了很多CD上没有的程序包，而且EPEL源也是一个安全的源。下载一些CD上没有的程序包的时候，就请认准EPEL源，若EPEL源还没有那就去软件的官网下载，这样可以提高程序包的安全质量。

接下来我们讲解一下如何添加一个EPEL源：

首先选择一个离你最近的源，因为这样会提高下载速度，国内首选中科大和阿里源，这里我选阿里源，打开阿里源网址：

https://opsx.alibaba.com/mirror

找到下图中的选项，点击

![1555815074290](images/1555815074290.png)

选择和你操作系统版本相同的目录，下面我选择的是7Server：

![1555815137888](images/1555815137888.png)

然后选择你主机的平台，这里我选择x86_64

![1555815232212](images/1555815232212.png)

打开后的目录下有repoda和Packages，这无疑是我们配置时需要指定的baseurl：

![1555815246565](images/1555815246565.png)

所以新建一个`.repo`文件：

```shell
[root@www ~]# vim /etc/yum.repos.d/Aliyun-EPEL.repo
[Aliyun-EPEL]
name=Aliyun-EPEL
baseurl=https://mirrors.aliyun.com/epel/7Server/x86_64/		# 该行填写刚刚目录下有repodata的网址
enabled=1
gpgcheck=0
```

然后重建缓存：

```shell
yum clean all
yum makecache
```

EPEL源搭建完成！



# 5.  自己创建一个本地yum仓库

其实我们也可以自己创建一个yum仓库，这里我来演示一下如何创建一个yum仓库。

> 接下来使用到的rpm包，都是从CDROM上拷贝的。其实，我们可以将远程仓库中的程序包下载到本地，然后自己手动创建一个yum源，然后使用ftp服务将其公布到局域网上，让局域网中的人可以快速的下载到原本应该在网上下载的程序包。



首先，创建一个文件夹，用来保存程序包，我们以ftp服务器为例：

```shell
mkdir /var/ftp/pub/Packages
```

将自己想要安装的软件包，从光盘拷贝到该目录下。需要注意的是：需要将软件包的依赖软件包也做拷贝（可以使用yum deplist Package来查看该软件包的依赖项）

这里我完整演示一下：假如我要安装一个ftp程序包，具体该怎么做

```shell
cd /media/cdrom
yum deplist ftp
```

依赖内容如下（其中有些特性的提供者重复了）：

```shell
package: ftp.x86_64 0.17-67.el7
  dependency: libc.so.6(GLIBC_2.15)(64bit)
   provider: glibc.x86_64 2.17-196.el7
  dependency: libncurses.so.5()(64bit)
   provider: ncurses-libs.x86_64 5.9-13.20130511.el7
  dependency: libreadline.so.6()(64bit)
   provider: readline.x86_64 6.2-10.el7
  dependency: libtinfo.so.5()(64bit)
   provider: ncurses-libs.x86_64 5.9-13.20130511.el7
  dependency: rtld(GNU_HASH)
   provider: glibc.x86_64 2.17-196.el7
   provider: glibc.i686 2.17-196.el7
```

接下来将需要的程序包和其所有的依赖拷贝至目标目录下。

```shell
cp glibc-2.17-196.el7.x86_64.rpm /var/ftp/pub/Packages/
cp ncurses-libs-5.9-13.20130511.el7.x86_64.rpm /var/ftp/pub/Packages/
cp readline-6.2-10.el7.x86_64.rpm /var/ftp/pub/Packages/
cp ftp-0.17-67.el7.x86_64.rpm /var/ftp/pub/Packages/
```

在Packages的父目录下创建repodata，这里需要使用`createrepo`命令帮我们完成这个过程：

```shell
cd /var/ftp/pub/
createrepo .		# 指定Packages所在目录的父目录中创建repodata
```



然后配置一个新的配置文件，仓库指向刚刚创建的路径：

```shell
[root@www pub]# vim /etc/yum.repos.d/test.repo
[test]
name=test
baseurl=file:///var/ftp/pub/
enabled=1
gpgcheck=0
```

然后重建缓存

```shell
yum clean all
yum makecache
```

之后使用安装命令检查配置是否成功：

```shell
yum install ftp --enablerepo=test
```

确定Repository ID是否正确。

![1555817979572](images/1555817979572.png)

