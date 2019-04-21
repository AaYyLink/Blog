# 0.  引入

从静态文件的角度看内核，其由三个部分组成：

1. **内核核心文件**：一般保存为bzimage，一般放置于`/boot/`目录下，名称为`vmlinux-VERSION-RELEASE`。
2. **内核模块(内核对象)**：一般于`/lib/modules/VERSION-RELEASE`目录下分类保存，其保存形式是以`.ko.xz`结尾其中ko代表kernel object。
3. **ramdisk文件**：这是内核加载时需要使用到的辅助性文件，但其并不是必须的，是否必须取决于内核是否能直接驱动rootfs所在的设备；其可加载的驱动如下：
   1. 目标设备驱动，例如SCSI设备的驱动
   2. 逻辑设备的驱动，例如LVM设备的驱动
   3. 文件系统，例如xfs文件系统

# 1.  内核模块概念

Linux是一个单内核多模块的系统，在需要使用到某个模块的时候才将模块加载，不需要使用时就将其卸载。所以Linux虽然是一个单内核的操作系统，但其也充分借鉴了多内核的设计优点。

Linux下的内核模块保存位置一般为`/lib/modules/VERSION-RELEASE`目录下，进入该目录后会还会有各种各样的目录将模块分类，其中的模块文件都以`.ko.xz`为后缀名保存，而`/lib/modules`下用目录名将不同内核版本的模块分类。因此想要进入当前当前系统使用的内核版本对应的内核目录可以使用以下命令

```bash
cd /lib/modules/$(uname -r)
```



在内核编译时，我们可以选择是否安装某个模块，一般情况下有三种关于模块的安装方式：

1. [ ]：N，表示不安装这个模块
2. [M]：Module，表示将其编译为内核模块，用到时在加载这个模块，不用时卸载这个模块。不过不是所有的模块都支持这个安装方式
3. [*]：Y，表示将模块编译进内核核心，随着内核的加载而加载，随着内核的关闭而关闭



> 我说的卸载模块不是将其删除，而是让其不在系统中运行

# 2.  相关命令



## 2.1  uname

**作用**：打印关于系统的信息

**使用方法**:

```bash
uname [OPTIONS]

OPTIONS:
	-r：输出内核版本
	-a：输出所有的信息
	-n：输出本机的主机名
	
## 我们写脚本的时候，若需要输入当前内核版本时，就可以使用该命令做命令引用，从而避免要记下很长的内核版本，更重要的一点是，这样做也可以提高脚本的兼容性，使其无论在什么内核都可以运行。例如：我们要引用当前使用内核的内核文件"ls /boot/vmlinuz-$(uname -r)"
```



## 2.2  模块管理命令



### 2.2.1  lsmod

**作用**：显示当前启用的模块及其信息

**使用方法**：

```bash
lsmod
## 该命令没有选项
lsmod 输出的每一行由"模块名称"、"模块大小"、"被引用次数及被什么模块引用"这些信息组成

一般情况下，我们使用下面的命令行来检验当前系统是否装载某个模块
lsmod | grep MODNAME
```

**其他信息**：其实该命令显示的信息是来自于内核的`/proc/modules`文件中的。



### 2.2.2  modinfo

**作用**：用来查看某个模块的信息

**使用方法**

```bash
modinfo [-F field] [-k kernel] [modulename|filename...]
	-F：指定显示的信息
	-k：查看指定内核的模块信息
	-n：显示文件路径
	
## 该命令可以直接跟模块名称，也可以直接跟模块的文件名。
```

**其他信息**：该命令显示的信息是来自于`/lib/modules`目录中的。



### 2.2.3  modprobe

**作用**：装载或卸载模块，且装载时会自动装载模块的依赖项

**使用方法**：

```bash
modprobe [-r] MODULE
	-r：卸载该模块
## 若不带-r选项，则代表装载该模块及其依赖项
```



### 2.2.4  depmod

**作用**：生成内核模块的依赖关系文件

**使用方法**：

```bash
depmod
## 直接执行命令本身即可，该命令并不常用
```



### 2.2.5  insmod

**作用**：装载模块，但不会提前装载其依赖项，但依赖项没有提前被加载则会导致模块装载不起来

**使用方法**：

```bash
insmod [FILENAME] [MODULE_OPTIONS...]
	FILENAME：模块文件的具体路径；并且需要提前装载其依赖项
	
## 我对于这个模块的理解就是，既然有modprobe，那谁会用这个？
```



### 2.2.6  rmmod

**作用**：卸载某个模块

**使用方法**：

```bash
rmmod MODULE

## 该命令和 modprobe -r MODULE 是一样的，不需要指明模块的具体保存路径
```



# 3.  ramdisk文件制作

以下介绍的两个命令都是用来为当前使用中的内核重新制作ramdisk文件的。



## 3.1  mkinitrd

**使用方法**：

```bash
mkinitrd [OPTION...] [<initrd-image>] <kernel-version>
	--with=<module>：除了默认的模块之外需要装载至initramfs中的模块；
	--preload=<module>：initramfs所提供的模块需要预先装载的模块；不常用
					
示例：mkinitrd /boot/initramfs-$(uname -r).img $(uname -r)
```



## 3.2  dracut

**使用方法**：

```bash
dracut [OPTION...] [<image> [<kernel version>]]

示例：dracut /boot/initramfs-$(uname -r).img  $(uname -r)
```

**其他信息**：dracut是一种较为底层的ramdisk生成工具



> 其实我上面所说的ramdisk并不严谨，现在的CentOS 7早已抛弃了ramdisk，而采用了ramfs的方式来做内核加载时使用到的辅助性工具。与ramdisk的不同之处就在于：ramfs不会出现双缓存-----同样的数据需要保存两份的情况



# 4.  伪文件系统

Linux中有两个伪文件系统：即位于根目录下的`/proc`和`/sys`目录。接下来，我会详解两个伪文件系统的功能。



## 4.1  proc目录

**作用**：内核状态和统计信息的输出接口；同时，提供了一个配置接口`/proc/sys`供用户配置可更改的内核参数。除此之外该伪文件系统还有各个进程对应PID的目录。

在`/proc`下，除了`/proc/sys`目录下的文件可以修改，其余的文件都是不可以修改的。需要注意的是：`/proc/sys`目录下的文件是不能用vim这些文本编辑器做修改的，下面会介绍如何对这些文件做修改。



### 4.1.1  临时修改内核参数

可以使用`sysctl`命令来对文件的内容做**临时**修改，也可以使用将echo内容重定向的方式实现内容的临时修改。

首先介绍`sysctl`命令的使用：

**使用方法**：

```bash
sysctl [OPTIONS]  [VARIABLE[=VALUE]]

OPTIONS：
	-a：显示所有的参数，可以配合grep筛选对应的参数
	-w：修改某个参数的值，后面需要跟上VARIABLE=VALUE
	-p：重新读取/etc/sysctl.conf，也可以跟上配置文件路径，指定读取某个文件的配置
	
VARIABLE：这个变量映射/proc/sys下的文件，举个例子：
kernel.hostname这个变量就是"/proc/sys/kernel/hostaname"这个文件,需要注意的是，是内核参数虚拟成了文件，而不是文件虚拟成了内核参数供用户更改，即："先有kernel.hostname"这个参数，后由内核虚拟成"/proc/sys/kernel/hostaname"这个文件，而这个文件就是基于"Linux下一切皆文件"的原则提供给用户修改内核参数途径的接口
```



我们也可以使用echo将数值重定向置文件中。

**使用方法**：

```bash
echo "PARAMETER" > /PATH/TO/VARIABLE
```



下面介绍几个常见的、可供用户修改的内核参数：

**net.ipv4.ip_forward**：是否开启核心转发功能（开启后可以转发路由包）

**vm.drop_caches**：置1时，回收内存，是用来手动回收内存的内核参数（实际上无法将所有内存回收，因为马上会有新的数据占据buff/cache）

**kernel.hostname**：设置主机名

**net.ipv4.icmp_echo_ignore_all**：是否忽略所有其他主机的ping操作，开启后，自己的主机还是能ping其他人的。



**示例**：

接下来我用上面讲的两种方法开启核心转发功能

```bash
echo "1" > /proc/sys/net/ipv4/ip_forward
```

```bash
sysctl -w net.ipv4.ip_forward=1
```

然后可以使用下面的命令查看核心转发功能是否开启

```bash
sysctl -a | grep "net.ipv4.ip_forward"
```



### 4.1.2 永久修改内核参数

永久修改内核参数则可以使用`/etc/sysctl.conf`,  `/etc/sysctl.d/*.conf `这两个配置文件，一般情况下，使用前者即可。

上面的配置文件可以用vim做修改：

```shell
[root@www 1]# vim /etc/sysctl.conf 
# sysctl settings are defined through files in
# /usr/lib/sysctl.d/, /run/sysctl.d/, and /etc/sysctl.d/.
#
# Vendors settings live in /usr/lib/sysctl.d/.
# To override a whole file, create a new file with the same in
# /etc/sysctl.d/ and put new settings there. To override
# only specific settings, add a file with a lexically later
# name in /etc/sysctl.d/ and put new settings there.
#
# For more information, see sysctl.conf(5) and sysctl.d(5).
net.ipv4.ip_forward = 1
kernel.hostname=www.link.org
net.ipv4.icmp_echo_ignore_all=0
```

在最后面加入自己想要设置的内核参数即可。

**注意**：在该文件中编辑的参数需要在重启后才能生效，若想立即生效可以使用`sysctl -p`。



## 4.2  sys目录

**sysfs**：输出内核识别出的各硬件设备的相关属性信息，也有内核对硬件特性的可设置参数；对此些参数的修改，即可定制硬件设备工作特性；



**udev**：通过读取/sys目录下的硬件设备信息按需为各硬件设备创建设备文件；udev是用户空间程序；专用工具：devadmin, hotplug；



udev为设备创建设备文件时，会读取其事先定义好的规则文件，一般在/etc/udev/rules.d/目录下，以及/usr/lib/udev/rules.d/目录下；

vim /etc/udev/rules.d/70-persisten-net.rules

modprobe -r 卸载网卡驱动

modprobe 装载网卡驱动

若修改了网卡驱动的配置信息，上面两部可以重新读取rules文件



# 5.  获取硬件设备信息的命令

以下列出各种获取硬件设备信息的命令及其具体作用。

| 命令  | 作用                                              |
| ----- | ------------------------------------------------- |
| lscpu | 获取CPU的信息                                     |
| lspci | 获取通过PCI总线接入的IO设备的信息                 |
| lsusb | 获取当前接入的usb的信息                           |
| lsblk | 获取块设备的分区信息，也可列出块设备关于lvm的信息 |



# 6.  screen

接下来会讲解内核的编译，而内核的编译过程是及其的长的，如果出现编译内核所在的终端突然挂掉的情况则会导致编译失败，所以这里介绍一下screen的使用。

**作用**：可以在一个终端上开启多个屏幕，若终端断开，屏幕还会依旧运行，很适合在下载文件、编译内核这些耗费时间很长的过程中使用。

**使用方法**：

```shell
打开screen：screen
拆除screen：Ctrl+a,d（组合键，先按Ctrl+a，后按d）			#拆除后，screen还会在后台运行
列出screen：screen -ls
连接至现有的screen：screen -r SCREEN_ID
关闭screen：exit
```



# 7.  内核编译

内核编译需要经过配置内核选项和编译两个步骤。需要注意的是进行以上两个操作都需要工作目录在编译内核的目录中。

**编译内核的目录**：/usr/src/linux-RELEASE-VERSION



## 7.1  配置内核选项

配置内核选项既支持在已有配置的基础上进行修改配置也支持全新配置的模式进行配置。以上两种方式都是通过直接或间接修改.config文件进行的。



在**已有配置的基础上**进行修改配置的工具有以下4种：

- **make config**：这是通过命令行遍历的方式去配置内核中所有的模块（这是最不好用的）
- **make muconfig**：是基于curses的文本配置窗口
- **make gconfig**：是基于GTK的可视化配置窗口，需要用yum安装“桌面开发包组”
- **make xconfig**：是基于QT开发环境的可视化配置窗口



支持“**全新配置模式**“的命令：

- **make defconfig**：基于内核为目标平台提供的“默认”配置为模板进行配置
- **make allnoconfig**：所有模块的选项为“no”



## 7.2  编译

内核的编译也分为很多种：

```bash
多线程编译：
	`make [-j #]`，用 -j 选项来指定编译内核时使用的线程。

仅编译内核的一部分代码：
	只编译某个子目录中的相关代码：
  		# cd /usr/src/linux
  		# make path/to/dir/
	只编译某个特定的模块：
		# cd /usr/src/linux
		# make path/to/dir/file.ko
		## 其实上面的源文件是以.c结尾的，编译好后保存在/lib/modules/VERSION-release.arch/kernel/下

交叉编译：
	# make  ARCH=arch_name
```

若要获取特定目标的使用帮助，则可以使用以下命令：

```shell
make  ARCH=arch_name help
## 例如：make arch=arm help
```



## 7.3  重新编译

若想要在经过编译操作的内核源码树上重新编译，则需要进行事先清理操作，清理操作有三种：

| 指令           | 行为                                                         |
| :------------------------------- | :----------------------------------------------------------- |
| make clean     | 清理编译生成的绝大部分文件，不过会保留.config文件和编译外部模块所需要的文件 |
| make mrproper  | 清理编译生成的所有文件，包括配置生成的config文件及某些备份文件（在执行此命令前，确保.config文件是备份过的） |
| make distclean | 相当于make mrproper，会额外清理各种patches以及编辑器备份文件（在执行此命令前，确保.config文件是备份过的） |



## 7.4  内核编译过程演示

