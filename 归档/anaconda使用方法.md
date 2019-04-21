# 0.  引入

kickstart是用来在系统安装时读取其配置信息，以实现自动安装的配置文件。

本篇博客会介绍kickstart的配置方式及如何让客户端读取kickstart的配置实现自动安装。



# 1.  kickstart的配置

kickstart文件的内容可以笼统的分为三部分，以下按各部分在kickstart文件中出现的顺序一一列出：

1. **命令段**：用来指定安装前的配置选项，如键盘类型、语言、分区布局等等
2. **程序包段**：指明要安装的程序包和程序包组
3. **脚本段**：用来指定安装系统前执行的脚本和安装系统后执行的脚本。

接下来，我会给出各个段的具体组成。



## 1.1  命令段的构成

**命令段必须有的命令**：

```bash
authconfig：认证方式配置
例如：authconfig --enableshadow --passalgo=sha512

bootloader：定义BootLoader的安装位置及其相关配置
例如：bootloader --location=mbr --driveorder=sda --append="crashkernel=auto rhgb quiet"
## 其中--driveorder指的是安装次序优先级，--append指传递给内核（引导时）的参数

keyboard：设置键盘类型
例如：keyboard us

lang：语言类型
例如：lang zh_CN.UTF8(中文)或者lang en_US.UTF8(英文)

part：分区布局
例如：	part /boot --fstype=ext4 --size=500
	autopart --type=lvm
## 默认单位为MB，第二个指的是采用安装时的默认分区方式

rootpw：管理员密码，这个密码可以使用openssl生成（后面有讲如何生成）
例如：rootpw --iscrypted $6$UzI433imqeZ9rVkF$SUcGPRKVkNZP4W.tW3u4fyVP.d9uaZaIpBXlv1JLX.XS2yiDf3tidIP1bSGArQc86ApV0fnVo49lhPNZHz8B2/

timezone：设置时区
例如：timezone Asia/Shanghai --isUtc

clearpart：关于清除分区
例如：clearpart --none --drives=sda
```

管理员密码的生成可以使用openssl生成，输入以下命令后，输入密码，就可以获得一个可以再kickstart文件中使用的加密密码：

```shell
openssl  passwd  -1  -salt `openssl rand -hex 4`
```

**命令段中还可以附加的命令有**：

```shell
part：创建PV
part pv.008002 --size=51200 

volgroup：创建lvm卷组
例如：volgroup myvg --pesize=4096 pv.008002

logvol:创建逻辑卷
例如：logvol /home --fstype=ext4 --name=lv_home --vgname=myvg --size=5120

install/upgrade：选择是安装还是升级，配置时单独成行即可

text：安装界面的类型，若为text则使用tui，若不设置则使用gui

firewall：防火墙
例如：firewall --disabled

selinux：设置SELINUX
例如：selinux --disabled

halt/poweroff/reboot：安装之后的行为，单独成行即可

repo：指明安装时使用的repository
例如：repo --name='CentOS' --baseurl=cdrom:sr0 --cost=100

url：指明安装时使用的repository，与上面的repo相同，只不过是使用url格式
例如：url --url=https://mirrors.aliyun.com/centos/7/os/x86_64/
```



## 1.2  程序包段的构成

程序包段用来指明要安装的程序包，程序包组，以及不安装的程序包。其组成格式如下

```bash
%packages：程序包段的开始
@group_name：安装一个包组
package：若无添加@则表示安装一个程序包
-package：明确指明不安装的程序包（若存在依赖关系则会被忽略）
%end：程序包段的结束
```

例如：

```bash
%packages
@^graphical-server-environment
@base
@core
@desktop-debugging
@dial-up
@dns-server
@fonts
@ftp-server
@gnome-desktop
@guest-agents
@guest-desktop-agents
@hardware-monitoring
@input-methods
@internet-browser
@kde-desktop
@mail-server
@multimedia
@print-client
@remote-system-management
@x11
chrony
kexec-tools

%end
```



## 1.3  脚本段的构成

脚本段用来指定安装前和安装后执行的脚本。需要注意的是：安装前的脚本运行环境和安装后的运行环境不同。

**安装前的脚本运行环境**：运行在安装介质上的微型Linux系统环境

**安装后的脚本运行环境**：安装完后的系统



**脚本段的格式如下**：

```bash
%pre
<这里输入安装前运行的脚本内容>
%post
<这里输入安装后运行的脚本内容>
```



## 1.4  anaconda-ks.cfg文件

一般情况下，系统安装完成后都会在root的家目录下生成一个anconda-ks.cfg文件，这就是一个kickstart文件。我们配置时就可以参照其进行配置。

anaconda-ks.cfg文件更重要的**功能**是：其记录了我们安装系统时的配置。当我们在多个主机上安装同一种系统而且所有系统的安装配置一样时就可以利用这个特性。即：在第一个主机上安装时，先手动安装，并详细的配置一个个选项。而后，在其他主机安装系统时就不用这么麻烦了，指定安装的第一个主机上的anaconda-ks.cfg文件来进行自动化安装。



## 1.5  system-config-kickstart工具

**作用**：这个工具的作用就是实现可视化的配置kickstart。

**使用方法**：

```shell
system-config-kickstart [KS_FILE]

## 注意：该命令必须要在GUI界面执行
```

GUI界面的使用非常简单，只需要注意修改完要保存就好，这里不再详细介绍。

**注意**：system-config-kickstart工具没有提供关于lvm的分区配置，因此若要配置lvm，则需要用文本手动配置。

> 工作中，若要在企业中部署自动化安装系统的环境，我们可以先在家里用虚拟机使用system-config-kickstart工具做配置，然后将配置导出的文件保存至U盘，然后将U盘中的文件拷贝至服务器上，在服务器上安装出现问题时，再使用kickstart的文本配置做修改。



## 1.6  ksvalidator工具

**作用**：用来检验kickstart文件是否配置错误

**使用方法**：

```shell
ksvalidator KS_FILE
```



## 1.7  kickstart配置示例及如何使用其自动安装

我这里给出一个配置文件示例（若想安装速度快一些，可以将自己不用的程序包删掉）：

```bash
#platform=x86, AMD64, or Intel EM64T
#version=DEVEL
# Install OS instead of upgrade
install
# X Window System configuration information
xconfig  --startxonboot
# Keyboard layouts
# old format: keyboard us
# new format:
keyboard --vckeymap=cn --xlayouts='cn'
# Root password
rootpw --iscrypted $1$EZkQqvdc$LqbefK6IsPKiiCsTlBJla.
# System language
lang zh_CN
# Firewall configuration
firewall --disabled
# System authorization information
auth  --useshadow  --passalgo=sha512
# Use CDROM installation media
cdrom
# Use text mode install
text
# Run the Setup Agent on first boot
firstboot --enable
# SELinux configuration
selinux --disabled
# Do not configure the X Window System
skipx

# System services
services --enabled="chronyd"
ignoredisk --only-use=sda
# Network information
network  --bootproto=dhcp --device=ens33
network  --bootproto=dhcp --device=None
# Halt after installation
halt
# System timezone
timezone Asia/Shanghai
# System bootloader configuration
bootloader --append="crashkernel=auto" --location=mbr --boot-drive=sda
# Clear the Master Boot Record
zerombr
# Partition clearing information
clearpart --all --initlabel
# Disk partitioning information
part /boot --fstype="ext4" --size=500
part pv.008002 --size=10240
volgroup myvg --pesize=4096 pv.008002
logvol / --fstype=xfs --name=lv_root --vgname=myvg --size=5120
logvol swap --fstype=swap --name=lv_swap --vgname=myvg --size=2048
logvol /home --fstype=xfs --name=lv_home --vgname=myvg --size=2048

%packages
@^graphical-server-environment
@base
@core
@desktop-debugging
@dial-up
@dns-server
@fonts
@ftp-server
@gnome-desktop
@guest-agents
@guest-desktop-agents
@hardware-monitoring
@input-methods
@internet-browser
@kde-desktop
@mail-server
@multimedia
@print-client
@remote-system-management
@x11
chrony
kexec-tools

%end
```



我们需要开启http服务，将这个kickstart文件移到`/var/www/html`目录下，并修改其权限，让安装系统的主机可以读取到。

先安装httpd服务

```shell
yum install httpd
```

然后开启之，并且配置防火墙使其放行访问http的流量，然后关闭selinux（以下是临时放行流量和临时关闭selinux）

```shell
systemctl start httpd
firewall-cmd --add-service=http
setenforce 0
```

想要永久开启httpd服务，使防火墙放行访问http的流量和关闭selinux可以使用下面的命令：

```bash
systemctl enable httpd
systemctl start httpd

firewall-cmd --permanent --add-service=http
firewall-cmd --reload

使用下面的命令修改selinux配置文件
vim /etc/selinux/config
将其中的
SELINUX=disabled
```



之后将kickstart文件，放在网页的目录下。并修改其的权限：

```bash
cp ks.cfg /var/www/html/
chmod 777 /var/www/html/ks.cfg
```

要确保这个文件能被下载，使用wget测试一下

```bash
wget http://127.0.0.1/ks.cfg
```

若出现下面的内容，则代表文件可以被下载

```bash
100%[======================================================================>] 1,796       --.-K/s   in 0s 
```

然后就可以让装系统的主机访问这个文件了，需要注意的是另一个主机的光盘镜像文件一定要和ks.cfg的来源主机一样。

将光盘镜像插入到主机中，启动主机，出现以下界面的时候按下`ESC`键。

![1555664415287](images/1555664415287.png)

按ESC后，会出现下面的界面:

![1555664466026](images/1555664466026.png)

然后输入以下内容后按回车就可以启动自动安装了：

```shell
linux ip=<IP_ADDR> netmask=<NETMASK> ks=http://<HTTP-SERVER-ADDRESS>/ks.cfg

其中<IP_ADDR>和<NETMASK>需要配置成能和<HTTP-SERVER-ADDRESS>通信，
<HTTP-SERVER-ADDRESS>的内容是刚刚搭建http服务的主机
```



> 其实上述的内容一点都不自动化，想要更加的自动化安装需要学习PXE，FTP，DHCP服务的搭建，让主机可以在不插入光盘的情况下实现自动化的安装，这样的配置方法会在以后介绍。





# 2.  制作CentOS安装程序并装入kickstart文件实现自动安装



## 2.1  CentOS安装程序启动流程

CentOS安装程序启动流程其实和CentOS的启动流程差不多。他们都包括以下几个过程：

POST--->BootSequence--->MBR(BootLoader)--->grub Stage2--->kernel(借助initrd驱动rootfs)

但是系统安装的过程可以比启动流程多一个步骤，即：读取anaconda配置文件，通过读取配置文件完成安装过程的自动化。不仅如此，两者的BootLoader和grub stage2的保存形式也是不一样的。

其中各个文件的保存形式如下（以下的内容都是保存于镜像中）：

**BootLoader**：isolinux/boot.cat

**Stage2**：isolinux/isolinux.bin

**Stage2的配置文件**：isolinux/isolinux.cfg



下面了解一下Stage2的配置文件：

这个配置文件中以label开始的段，是用来指定一个菜单项的，这里我取出一个关于linux的label来分析一下其书写格式：

```bash
label linux
  menu label ^Install CentOS 7
  kernel vmlinuz
  append initrd=initrd.img inst.stage2=hd:LABEL=CentOS\x207\x20x86_64 quiet
```

上面这段的配置其实就是对应于下图中的第一个菜单项，上面这段中的第二行编辑后可以达到修改菜单项名称的效果。而上面这段中的第一行是用来在boot提示符中使用的，也就是在菜单界面时按下ESC键时可以使用的命令，而这个命令的具体行为就是后面两行，你可以理解为：为`kernel vmlinuz append initrd=initrd.img inst.stage2=hd:LABEL=CentOS\x207\x20x86_64 quiet`取了一个命令别名为`linux`。

![1555667997950](images/1555667997950.png)



## 2.2  自己做一个iso镜像

这里我将讲解如何手动做一个iso镜像文件，并且其可以实现系统自动安装的功能。

首先需要知道的是，一个简单的CentOS可以安装镜像只包含isolinux目录，再此之上加入一个kickstart文件就可以制作成一个简单的可以自动化安装的光盘镜像。

下面我会给出制作步骤（步骤中的文件都是来自于官方镜像文件，而不是自己徒手做的）：

首先挂载官方给出的光盘：

```shell
mount /dev/cdrom /mnt/cdrom
```

创建一个目录用来保存光盘镜像中的文件，然后将官方光盘中的isolinux拷贝至这个目录：

```shell
mkdir /tmp/isodir
cp -rp /mnt/cdrom/isolinux /tmp/isodir
```

然后手写一个kickstart文件，需要注意的是：我们现在仅仅拷贝了isolinux，因此程序包段的内容需要全部清空。

这里我们将刚刚放在http服务器上的文件拷贝至光盘目录中：

```shell
cp /var/www/html/ks.cfg /mnt/isodir/
```

并对其做修改，修改完的内容如下：

```shell
#platform=x86, AMD64, or Intel EM64T
#version=DEVEL
# Install OS instead of upgrade
install
# X Window System configuration information
xconfig  --startxonboot
# Keyboard layouts
# old format: keyboard us
# new format:
keyboard --vckeymap=cn --xlayouts='cn'
# Root password
rootpw --iscrypted $1$EZkQqvdc$LqbefK6IsPKiiCsTlBJla.
# System language
lang zh_CN
# Firewall configuration
firewall --disabled
# System authorization information
auth  --useshadow  --passalgo=sha512
# Use CDROM installation media
cdrom
# Use text mode install
text
# Run the Setup Agent on first boot
firstboot --enable
# SELinux configuration
selinux --disabled
# Do not configure the X Window System
skipx

# System services
services --enabled="chronyd"
ignoredisk --only-use=sda
# Network information
network  --bootproto=dhcp --device=ens33
network  --bootproto=dhcp --device=None
# Halt after installation
halt
# System timezone
timezone Asia/Shanghai
# System bootloader configuration
bootloader --append="crashkernel=auto" --location=mbr --boot-drive=sda
# Clear the Master Boot Record
zerombr
# Partition clearing information
clearpart --all --initlabel
# Disk partitioning information
part /boot --fstype="ext4" --size=500
part pv.008002 --size=10240
volgroup myvg --pesize=4096 pv.008002
logvol / --fstype=xfs --name=lv_root --vgname=myvg --size=5120
logvol swap --fstype=swap --name=lv_swap --vgname=myvg --size=2048
logvol /home --fstype=xfs --name=lv_home --vgname=myvg --size=2048

%packages

%end
```

然后按照修改引导程序的菜单项，将其中的linux改成如下：(即：第四行中插入一个ks的读取路径)

```bash
label linux
  menu label ^Install CentOS 7
  kernel vmlinuz
  append initrd=initrd.img inst.stage2=hd:LABEL=CentOS\x207\x20x86_64 ks=cdrom:/ks.cfg quiet
```





然后我们退出该目录，使用下面的命令将其制作成光盘镜像文件：

```shell
[root@www tmp]# mkisofs -R -J -T -v --no-emul-boot --boot-load-size 4 --boot-info-table -V "CentOS 6 x86_64 boot" -c isolinux/boot.cat -b isolinux/isolinux.bin -o /root/boot.iso   isodir/
```

生成的光盘镜像文件保存于/root/boot.iso。

我这里用的是虚拟机，因此需要将其传输回Windows真机上，才能让另外一台虚拟机加载这个光驱，我们这里用Xshell的`sz`功能进行传输回本地。

```shell
sz /root/boot.iso
```

输入完成后，选择其保存路径。

接着开启一个新的虚拟机，让其读取刚刚创建的光驱，选择`Install CentOS 7`选项就可以开启自动安装了。

------

关于kickstart文件的文本配置可以参考《Installation Guide》