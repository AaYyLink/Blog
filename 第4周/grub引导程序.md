# 0.  引入

本篇博客会介绍grub的相关概念和grub的使用。









# 1.  grub的相关概念





## 1.1  grub的版本

grub有不同的版本，下面介绍的是不同的grub版本和使用其的操作系统。

grub 0.x(grub legacy)：CentOS 6所使用的grub版本

grub 1.x(grub 2)：CentOS 7所使用的grub版本





> grub legacy由于其相比新版本相比更轻量化，而用做安卓手机的引导程序。





## 1.2  grub legacy的各个阶段

grub的启动分为下面三个阶段：

**stage 1**：存储于MBR上。启动mbr中得boot loader，而boot loader仅仅只是主程序段。

**stage 1_5**：存储于MBR之后的扇区上。让stage 1中的boot loader能识别stage 2所在分区的文件系统。

**stage 2**：存储于`/boot/grub`上。加载/boot/grub下的配置信息





## 1.3  stage2的功能

(1)提供菜单，并且提供交互式的接口(该菜单可以被隐藏)

​	e：编辑模式，用于编辑菜单

​	c：命令模式(grub的内置命令行)，这就是所谓的交互式接口

(2)加载用户选择的内核

​	这个允许用户传递参数给内核

(3)菜单提供保护机制(若没有这个的话，系统的启动密码可以被随意修改)，有两种认证方式：

1. 编辑时需要用户做认证
2. 加载特定的内核前需要做认证





## 1.4  grub如何访问boot目录

分为两种情况：

1.**单独分区**/boot目录的情况：

此时，原先以根文件系统为根的/boot目录会作为grub的根目录，因此访问grub所在的分区是绕过原先的根文件系统，直接将/boot目录所在的分区挂载到grub所访问的根。所以访问根文件系统下的/boot/vmlinuz在grub中只用访问/vmlinuz即可。

2.**不单独分区/**boot目录的情况：

即/boot目录是属于原先根文件系统，此时grub会挂载根文件系统所在的分区作为grub的根目录。此时访问/boot/vmlinuz还是访问/boot/vmlinuz。





**如何选择单独分区还是不单独分区？**

根文件系统不使用逻辑卷时，可以不单独分区/boot;

根文件系统所在的分区使用了逻辑卷技术时，单独分区/boot。

原因是grub启动时加载的驱动程序仅包含驱动基本磁盘设备的设备驱动，这导致了其无法识别逻辑卷。针对于软RAID而言grub只能识别RAID1。









# 2  grub命令接口的使用

grub下可以使用的命令有：

help：获取帮助列表

help KEYWORD：获取详细帮助信息

find (hd#,#)/PATH/TO/SOMEFILE：确定某个文件是否存在于对应的分区上，若已设定了根分区，则可以省略前面的(hd#,#)

root (hd#,#)：设定哪个分区是根分区，第一个#为设备号，第二个#为分区号

kernel /PATH/TO/SOMEFILE：设定本次启动所使用到的kernel文件，后面还可以跟许多内核支持的cmdline参数

​	例如：init=/PATH/TO/INIT，selinux=0

initrd /PATH/TO/INITRAMFS_FILE：指定ramdisk文件为内核的加载提供额外文件(例如磁盘驱动)

boot：启动选定的内核



我们可以尝试使用命令行启动系统：

![1555231098014](images/1555231098014.png)

开机时进入上面的界面后，按下c键(如果开机时没有出现该画面，先将/boot/grub/grub.conf中的hiddenmenu行删除)

之后会出现下面的命令行

![1555231276821](images/1555231276821.png)

然后输入下面的命令

```bash
root (hd#,#)
kernel /vmlinuz-VERSION-RELEASE ro root=/dev/DEVICE [quiet]
initrd /initramfs-VERSION-RELEASE.img
boot
```

第二行的命令ro代表以只读方式挂载根文件系统,root后指定根文件系统所在的分区，quiet表示不输出提示信息，









# 3  配置grub菜单





## 3.1  grub菜单配置基础讲解

grub菜单的配置文件为：`/boot/grub/grub.conf`，也可以使用其链接文件`/etc/grub.conf`。

其内容如下：

```
  1 # grub.conf generated by anaconda
  2 #
  3 # Note that you do not have to rerun grub after making changes to this file
  4 # NOTICE:  You have a /boot partition.  This means that
  5 #          all kernel and initrd paths are relative to /boot/, eg.
  6 #          root (hd0,0)
  7 #          kernel /vmlinuz-version ro root=/dev/mapper/VolGroup-lv_root
  8 #          initrd /initrd-[generic-]version.img
  9 #boot=/dev/sda
 10 default=0
 11 timeout=5
 12 splashimage=(hd0,0)/grub/splash.xpm.gz
 13 title CentOS 6 (2.6.32-696.el6.x86_64)
 14         root (hd0,0)
 15         kernel /vmlinuz-2.6.32-696.el6.x86_64 ro root=/dev/mapper/VolGroup-lv_    root nomodeset rd_NO_LUKS LANG=en_US.UTF-8 rd_NO_MD rd_LVM_LV=VolGroup/lv_swap     SYSFONT=latarcyrheb-sun16 crashkernel=auto rd_LVM_LV=VolGroup/lv_root  KEYBOA    RDTYPE=pc KEYTABLE=us rd_NO_DM rhgb quiet
 16         initrd /initramfs-2.6.32-696.el6.x86_64.img
```

该文件由全局配置和各个菜单项的配置组成。

**各个字段的意义**：

上图中9-12行为全局配置。

default=#: 设定默认启动的菜单项；落单项(title)编号从0开始,按title从上到下数；

timeout=#：指定菜单界面等待用户操作的时长，若用户在这段时间没有操作，则会选定defualt指定的菜单项启动。

splashimage=(hd#,#)/PATH/TO/XPM_PIC_FILE：指明菜单背景图片文件路径（只支持14位色，640*480，可以用gimp制作这种图片）；

hiddenmenu：开机时是否自动跳过grub的菜单，上面的内容没有该字段说明开机时不会跳过grub的菜单。

password [--md5] STRING: 菜单编辑认证，若无--md5则使用明文密码；



**上图中的13行后是菜单项配置**：

title TITLE：定义菜单项“标题”, 可出现多次；

root (hd#,#)：grub查找stage2及kernel文件所在设备分区；为grub的“根”; 

kernel /PATH/TO/VMLINUZ_FILE [PARAMETERS]：启动的内核及其参数

initrd /PATH/TO/INITRAMFS_FILE:内核会使用的ramdisk文件

password [--md5] STRING: 启动选定的内核或操作系统时进行认证；

 



> 其实每个title下的菜单项配置就是grub命令行中的命令。
>
> 使用grub-md5-crypt命令：可以生成密码对应的MD5





## 3.2  配置grub菜单示例

要做有密码的配置，所以先输入下面的命令得到MD5值

```
[root@localhost ~]# grub-md5-crypt
Password: 
Retype password: 
$1$cam0L0$Gowlal4HN9xkXDg7pF4TR0
```

这里我所作的配置是加入菜单编辑认证和加入一个新的菜单项，编辑完后，配置文件的内容如下：

```
  1 # grub.conf generated by anaconda
  2 #
  3 # Note that you do not have to rerun grub after making changes to this file
  4 # NOTICE:  You have a /boot partition.  This means that
  5 #          all kernel and initrd paths are relative to /boot/, eg.
  6 #          root (hd0,0)
  7 #          kernel /vmlinuz-version ro root=/dev/mapper/VolGroup-lv_root
  8 #          initrd /initrd-[generic-]version.img
  9 #boot=/dev/sda
 10 default=0
 11 timeout=5
 12 password --md5 $1$cam0L0$Gowlal4HN9xkXDg7pF4TR0
 13 splashimage=(hd0,0)/grub/splash.xpm.gz
 14 title CentOS 6 (2.6.32-696.el6.x86_64)
 15         root (hd0,0)
 16         kernel /vmlinuz-2.6.32-696.el6.x86_64 ro root=/dev/mapper/VolGroup-lv_root nomodeset rd_NO_LUKS LANG=en_US.UTF-8 rd_NO_MD rd_LVM_LV=VolGroup/lv_swap SYSFONT=latarcyrheb-sun16     crashkernel=auto rd_LVM_LV=VolGroup/lv_root  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM rhgb quiet
 17         initrd /initramfs-2.6.32-696.el6.x86_64.img
 18 title mylinux
 19         root (hd0,0)
 20         kernel /vmlinuz-2.6.32-696.el6.x86_64 ro root=/dev/mapper/VolGroup-lv_root
 21         initrd /initramfs-2.6.32-696.el6.x86_64.img
 22         password --md5 $1$cam0L0$Gowlal4HN9xkXDg7pF4TR0
```

编辑完后尝试重启系统，显示的grub菜单如下

![1555234646653](images/1555234646653.png)

此时按e是无法编辑的，我们需要先输入密码(不输入密码也不会影响没有设置认证的菜单项的运行)

按下p输入密码。之后就可以编辑了。

这里我们选择mylinux按回车。会显示下图，提示我们输入密码：

![1555234942267](images/1555234942267.png)

输入密码后，就可以启动操作系统了。





## 3.3  单用户模式

grub还有一个很关键的功能就是进入单用户模式，进入单用户模式需要在grub菜单中编辑对应的菜单项。

启用单用户模式的步骤。

在下面的界面敲入e。

![1555235239438](images/1555235239438.png)



后再弹出的界面中选定kernel的配置项，再按e。

![1555235298332](images/1555235298332.png)

之后再命令行后面多加一个1(或者为s,S,single)

![1555235410418](images/1555235410418.png)

之后按回车会回到刚刚的菜单栏，然后按b就可以启动单用户模式了。

> 单用户下可以在不知道原始的root密码的情况下修改root密码。







# 4  grub的安装

grub的安装可以使用`grub-install`命令。其使用格式如下：

```bash
grub-install --root-directory=ROOT /dev/DISK
# 该--root-directory后跟的根是安装grub的/boot目录中的根。假如安装grub的目录为/mnt/boot，则该选项后所跟的就是/mnt
# /dev/DISK指定的是要将Boot Loader安装到MBR的设备
```

也可以使用`grub`命令,该命令会直接打开grub命令行。在grub命令行下安装grub至MBR上可以使用以下命令：

```
root (hd#,#)
setup (hd#)
```



以上两个命令都会将grub的boot loader安装到MBR中。







# 5  grub练习





## 5.1  新增一个硬盘，其可以提供直接单独运行bash的功能

首先对新增的硬盘做分区和格式化

```bash
[root@localhost ~]# fdisk /dev/sdb
[root@localhost ~]# fdisk -l /dev/sdb

Disk /dev/sdb: 21.5 GB, 21474836480 bytes
255 heads, 63 sectors/track, 2610 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x82ea2ad5

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1               1        1306    10490413+  83  Linux
/dev/sdb2            1307        1568     2104515   82  Linux swap / Solaris
/dev/sdb3            1569        1582      112455   83  Linux

## sdb1作为根文件系统所在的分区，sdb2作为交换分区，sdb3作为boot目录所在的分区
[root@localhost ~]# partx -a /dev/sdb
[root@localhost ~]# cat /proc/partitions 
major minor  #blocks  name

   8        0   20971520 sda
   8        1     512000 sda1
   8        2   20458496 sda2
   8       16   20971520 sdb
   8       17   10490413 sdb1
   8       18    2104515 sdb2
   8       19     112455 sdb3
 253        0   18423808 dm-0
 253        1    2031616 dm-1
[root@localhost ~]# mke2fs -t ext4 /dev/sdb1
[root@localhost ~]# mke2fs -t ext4 /dev/sdb3
[root@localhost ~]# mkswap /dev/sdb2
```

之后将根目录所在的分区和boot目录所在的分区挂载

```bash
[root@localhost ~]# mkdir /mnt/sysroot
[root@localhost /]# mkdir /mnt/boot
```

在根文件系统下创建对应的目录

```bash
[root@localhost /]# cd /mnt/sysroot/
[root@localhost sysroot]# mkdir -pv root usr bin sbin lib lib64 home dev tmp mnt media var etc boot proc sys 
mkdir: created directory `root'
mkdir: created directory `usr'
mkdir: created directory `bin'
mkdir: created directory `sbin'
mkdir: created directory `lib'
mkdir: created directory `lib64'
mkdir: created directory `home'
mkdir: created directory `dev'
mkdir: created directory `tmp'
mkdir: created directory `mnt'
mkdir: created directory `media'
mkdir: created directory `var'
mkdir: created directory `etc'
mkdir: created directory `boot'
mkdir: created directory `proc'
mkdir: created directory `sys'
```

将bash文件拷贝至其/bin目录下,并且拷贝其依赖项到新的艮目

```bash
[root@localhost sysroot]# cp /bin/bash ./bin/
[root@localhost sysroot]# ls ./bin/
bash

```

使用grub-install命令安装grub至新的boot目录

```bash
[root@localhost ~]# grub-install --root-directory=/mnt /dev/sdb
Probing devices to guess BIOS drives. This may take a long time.
/dev/mapper/VolGroup-lv_root does not have any corresponding BIOS drive.
[root@localhost ~]# ls /mnt/boot/
grub
[root@localhost ~]# ls /mnt/boot/grub/
device.map     ffs_stage1_5      minix_stage1_5     stage2           xfs_stage1_5
e2fs_stage1_5  iso9660_stage1_5  reiserfs_stage1_5  ufs2_stage1_5
fat_stage1_5   jfs_stage1_5      stage1             vstafs_stage1_5
```

之后在boot分区上，copy当前系统使用的内核、ramdisk文件。

```bash
[root@localhost sysroot]# cd /mnt/boot/
[root@localhost boot]# cp /boot/vmlinuz-2.6.32-696.el6.x86_64 ./
[root@localhost boot]# cp /boot/initramfs-2.6.32-696.el6.x86_64.img ./
[root@localhost ~]# ldd /bin/bash
	linux-vdso.so.1 =>  (0x00007ffff596c000)
	libtinfo.so.5 => /lib64/libtinfo.so.5 (0x0000003924200000)
	libdl.so.2 => /lib64/libdl.so.2 (0x0000003919e00000)
	libc.so.6 => /lib64/libc.so.6 (0x000000391a200000)
	/lib64/ld-linux-x86-64.so.2 (0x0000003919a00000)
[root@localhost ~]# cp /lib64/libtinfo.so.5 /mnt/sysroot/lib64/
[root@localhost ~]# cp /lib64/libdl.so.2 /mnt/sysroot/lib64/
[root@localhost ~]# cp /lib64/libc.so.6 /mnt/sysroot/lib64/
[root@localhost ~]# cp /lib64/ld-linux-x86-64.so.2 /mnt/sysroot/lib64/

```

然后配置新boot分区上的grub.conf文件(该文件原先并不存在，得自己创建)，并在其中新增一个菜单项：

```bash
[root@localhost boot]# vim /mnt/boot/grub/grub.conf
  1 default=0
  2 timeout=5
  3 title onlybash
  4         root (hd0,2)
  5         kernel /vmlinuz-2.6.32-696.el6.x86_64 selinux=0 ro root=/dev/sda1 init    =/bin/bash
  6         initrd /initramfs-2.6.32-696.el6.x86_64.img
  7

# 对于我们的新主机，该硬盘会是其识别的第一个硬盘，而我们刚刚创建的boot目录所在的分区是第3个，所以上面第4，5行写成这样。  
# 而kernel启动可以指定init程序为/bin/bash
```

做完后记得先使用`sync`命令同步磁盘。

新建一个虚拟机，其使用的磁盘是我们刚刚在CentOS 6上创建的磁盘。

![1555239700354](images/1555239700354.png)



开启这个虚拟机，会有下面的grub菜单。

![1555239786671](images/1555239786671.png)

按回车键就可以启动最普通的bash了。启动bash后如下，其上只能运行bash内置的命令

![1555243733546](images/1555243733546.png)





## 5.2  破坏本机MBR上的grub stage1，然后再救援模式下修复之

使用dd命令破坏MBR上的boot loader(grub stage1)。

```bash
[root@localhost boot]# dd if=/dev/zero of=/dev/sda count=1 bs=200
1+0 records in
1+0 records out
200 bytes (200 B) copied, 0.000143087 s, 1.4 MB/s
```

然后重启，画面会如下所示：

![1555244949971](images/1555244949971.png)

此时我们需要插入安装此系统的光盘。开机后会是下面这样，选择Rescue installed system(救援模式)，按回车。

![1555245092327](images/1555245092327.png)



之后默默等待至出现下面的界面，然后选择你想要显示的语言和你的键盘类型，选择English和us-acentos。

![1555245187331](images/1555245187331.png)

![1555245900508](images/1555245900508.png)

选择是否要开启网络，这里选择不开启。

![1555245323051](images/1555245323051.png)

接着选continue

![1555245363564](images/1555245363564.png)

接着会显示提示信息，说的是你的系统被挂载在`/mnt/sysinmage`下，若要选择其为根文件系统可以使用`chroot /mnt/sysimage`命令，若你退出命令行则会直接重启。

![1555245419931](images/1555245419931.png)

接着再按一次回车，会出现这个画面，我们选择第一个选项，然后按方向右键，选定OK后回车。

![1555245577292](images/1555245577292.png)

然后就会出现救援模式的命令行。我们输入下面的命令

```bash
chroot /mnt/sysimage/
grub-install --root-directory=/ /dev/sda
```

之后重启就可以正常启动系统了。(开机时可能要修复某些文件，要花上点时间)





## 5.3  为grub设置保护功能

这点在上面的`3.2  配置grub菜单示例`中已经讲过了，所以不再此再次演示。