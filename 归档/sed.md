# 1.  sed介绍

sed是用于过滤和转换文本的流编辑器，过滤和转换文本的操作以行为单位，每一次都处理一行的内容。



# 2.  sed使用方法

首先讲解一下sed的**使用格式**：

```
sed [OPTIONS]... {script} [input-file]...
```

其中常用的**OPTION**有：

| OPTION | 意义                                                      |
| ------ | --------------------------------------------------------- |
| -r     | 让ACTION支持扩展正则表达式                                |
| -n     | 使用安静模式，显示只经过特殊处理的行（默认显示所有航）    |
| -e     | 可以使用-e script1 -e script2 -e script3 来指定多脚本运行 |
| -f     | 从指定的文件中读取脚本并运行                              |
| -i     | 直接修改源文件                                            |



script内容由地址定界和编辑命令组成：

**地址定界**的方式有以下几种（一下中的#表示数字）：

| 格式              | 意义                                                         |
| ----------------- | ------------------------------------------------------------ |
| #                 | 指定行                                                       |
| $                 | 最后一行,并没有指定第一行的^                                 |
| /regexp/          | 任何能够被regexp所匹配到的行                                 |
| \%regexp%         | 跟上面的效果一样，只不过是换了成%做边界符                    |
| startline,endline | 有多种表示方式：<br />#，/regexp/：从#行开始到第一次被/regexp/所匹配到的行结束，中间的所有行<br />#,#：从#行到#行<br />/regexp1/,/regexp2/：从第一次被/regex1/匹配到的行开始，到第一次被/regexp2/匹配到的行结束，中间的所有行<br />#，+n：表示从#行开始，一直到向下的n行 |
| first~step        | 指定行的开始，以及其步长                                     |
| 不进行地址定界    | 保留所有的行                                                 |



常见的**编辑命令**有以下几种：

| 编辑命令             | 意义                                                         |
| -------------------- | ------------------------------------------------------------ |
| d                    | 删除地址定界匹配到的行                                       |
| =                    | 为地址定界匹配到的行加上行号                                 |
| a \text              | 在匹配到的行后追加文本，支持使用\n实现多行追加，也支持使用命令行末尾加入'\'来进行多行添加。 |
| i \text              | 在匹配到的行前加入文本，也支持像上面的多行追加               |
| c \text              | 将匹配到的行替换为text                                       |
| p                    | 打印匹配到的行                                               |
| s/regexp/replacement | 替换有regexp正则表达式匹配到的内容，其操作和vim下的替换一样。其后可跟两个选项：1、g：代表全局替换。2、i：不区分大小写。也支持像vim一样---替换分隔符为@或#。<br />并且支持后向引用 |
| w /path/to/somefile  | 将指定的内容另存至/path/to/somefile路径所指定的文件中        |
| r /path/to/somefile  | 在文件的指定位置插入另一个文件的所有内容，完成文件合并       |



## 2.1  sed的替换示例

其实sed最常用的还是他的替换功能，接下来我会用一个例子讲解一下如何使用替换功能筛选出自己想要的内容（下面的内容借鉴与鸟哥Linux私房菜）



**示例：我们想要取得ifconfig命令中的ipv4地址。**

首先用grep将有inet的行筛选出来

```shell
[root@localhost Packages]# ifconfig | grep "inet\>"
        inet 192.168.3.64  netmask 255.255.255.0  broadcast 192.168.3.255
        inet 192.168.10.30  netmask 255.255.255.0  broadcast 192.168.10.255
        inet 127.0.0.1  netmask 255.0.0.0
        inet 192.168.122.1  netmask 255.255.255.0  broadcast 192.168.122.255
```

然后我们可以观察，ip地址前的内容可以用正则表达式`s/.*inet[[:space:]]\+`来匹配，因此用以下操作将其剔除``

```shell
[root@localhost Packages]# ifconfig | grep "inet\>" | sed "s/^.*inet[[:space:]]\+//g"
192.168.3.64  netmask 255.255.255.0  broadcast 192.168.3.255
192.168.10.30  netmask 255.255.255.0  broadcast 192.168.10.255
127.0.0.1  netmask 255.0.0.0
192.168.122.1  netmask 255.255.255.0  broadcast 192.168.122.255
```

然后我们可以注意ip地址后的内容可以用正则表达式`[[:space:]]\+.*$`匹配，因此用以下操作将其剔除

```shell
[root@localhost Packages]# ifconfig | grep "inet\>" | sed "s/^.*inet[[:space:]]\+//g" | sed "s/[[:space:]]\+.*$//g"
192.168.3.64
192.168.10.30
127.0.0.1
192.168.122.1
```

上面的内容就是一次一次通过管道将多余的内容剔除的过程。这就是用sed筛选文本的思路所在。



## 2.2  sed的其他示例

**示例1**：显示`/etc/fstab`文件中的非注释行。

```shell
[root@localhost Packages]# sed "/^#/d" /etc/fstab 

/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=3cc31987-a4dc-4ecb-8cbd-9a2db1f8a2ec /boot                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0
/dev/cdrom /mnt/cdrom iso9660 defaults 0 0
```

也可以使用grep来筛选。

```shell
[root@localhost Packages]# grep -v "^#" /etc/fstab 

/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=3cc31987-a4dc-4ecb-8cbd-9a2db1f8a2ec /boot                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0
/dev/cdrom /mnt/cdrom iso9660 defaults 0 0
```



**示例2**：将`/etc/fstab`文件中的奇数行保存至`/tmp/fstab`文件中

```shell
sed "1~2 w/tmp/fstab" /etc/fstab 
```



**示例3**：echo一个文件路径给sed命令然后取出其路径名

取出路径名

```shell
[root@localhost Packages]# echo "/mnt/cdrom/Package/ftp" | sed 's@[^/]\+/\?$@@g'
/mnt/cdrom/Package/
```

若要取出其基名，则可以使用grep：

```shell
[root@localhost Packages]# echo "/mnt/cdrom/Package/ftp/" | grep -o "[^/]\+/\?$"
ftp/
```

