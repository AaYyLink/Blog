# 1.  LVM概念介绍

在Linux传统的磁盘分区方案中，一个文件系统被格式化后就无法改变其大小。而LVM却可以避免这样的缺点，实现文件系统大小的扩大和缩小。

LVM的组织结构如下图所示：

![图片来自于鸟哥Linux私房菜](images/1555932865158.png)

下面先给出几个名词的详细解释：

**PV**（Physical Volume，物理卷）：其实就是一个分区或磁盘，将其打上PV的标志，表示其可以被加入卷组

**VG**（Volume Group，卷组）：将多个分区和磁盘加入到一个组中，每一个卷组都会指定PE大小

**PE**（Physical Extent）：是在创建卷组时指定的，这个值是分配空间给逻辑卷时的最小单位

**LV**（Logical Volume）：逻辑卷，其空间是从VG分割而来的，每一个LV都可以当做一个分区而被格式化、挂载使用

接下来以图来讲解LVM的**运作方式**：

首先，我们将分区（或磁盘）添加物理卷的属性，标记成PV的分区可以被加入到卷组中，卷组将这些物理卷合并，并将其切分成一个个小的物理区域，然后将卷组中的空间以物理区域为单位分配逻辑卷，每个逻辑卷都可以当做一个可以格式化的分区。各个逻辑卷的大小可以在创建后增大或缩小。





# 2.  LVM相关指令

PV相关命令：

```shell
pvcreate：将分区打上PV的标志
pvscan：列出当前系统中所有具有PV属性的磁盘
pvdisplay：显示当前系统中所有PV的状态
pvremove：将分区的PV标志移除
```



VG相关命令：

```shell
vgcreate：用来创建VG
vgscan：列出当前系统中所有的VG
vgdisplay：显示当前系统中所有PV的状态
vgextend：在VG内添加额外的PV
vgreduce：在VG内移除PV
vgchange：配置VG是否启动
vgremove：删除一个VG
```



LV相关命令：

```shell
lvcreate：创建LV
lvscan：查询系统上面的LV
lvdisplay：显示系统上面LV的状态
lvreduce：缩小LV的容量
lvextend：扩充LV的容量
lvremove：删除一个LV
lvresize：对LV进行容量大小的调整
```



# 3.  LVM示例

**示例1**：创建一个VG名称为mygroup，PE大小指定为16M，在其上创建一个逻辑卷名称为haha，大小为4G，格式化为ext4，挂载在/mnt/haha下，而后将其扩展为6G。

```shell
## 创建mygroup，这里我选择使用一整块硬盘
[root@localhost ~]# pvcreate /dev/sdb
[root@localhost ~]# vgcreate mygroup -s 16M /dev/sdb

## 然后创建一个4G大小的逻辑卷
[root@localhost ~]# lvcreate -n haha -L 4G mygroup

## 格式化并挂载
[root@localhost ~]# mke2fs -t ext4 /dev/mygroup/haha 
[root@localhost ~]# mount /dev/mygroup/haha /mnt/haha

## 将其大小修改为6G
[root@localhost ~]# lvextend /dev/mygroup/haha -L 6G
[root@localhost ~]# resize2fs /dev/mygroup/haha
```





