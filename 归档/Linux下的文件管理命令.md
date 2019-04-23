# 0.  引入

​	Linux下的文件管理命令是入门Linux的基础，在此博客会讲解常见的文件管理命令的使用方法。

# 1.  常用文件管理命令

### 1.1  ls	

  ls是用来列出当前文件夹内容的命令。

**使用方法**:

```bash
ls [options] DIRPATH|FILE
# 参数指定DIRPATH时，显示的是文件夹下的内容
# 参数指定是FILE时，显示的是文件本身的内容。以下除了-l和-i之外，剩余选项对FILE参数几乎没有什么作用

常用选项：
	-d：列出目录本身
	-l：详细显示文件的信息，包括属组、属主、权限等等
	--color：一般是各大发行版默认的选项，会根据文件的特征以不同的颜色显示文件名
	-a：显示DIRPATH指定目录下的所有文件，包括隐藏文件，也包括.和..
	-A：显示DIRPATH指定目录下除.和..的所有文件
	-i：显示文件的inode号
```

> 1、`.`代表的是当前目录，`..`代表的是当前目录的上级目录
>
> 2、注意：目录本身保存的是其下文件的信息，而ls -l显示的目录大小是目录本身的大小，若要查看文件夹其下所有内容的大小之和，请使用`du -m -d 0 DIRPATH`命令。
>
> 3、常见的Linux发行版都会给`ls -l`添加命令别名`ll`，即使用`ll`命令等同于使用`ls -l`

**示例**：

1、显示/etc目录本身的详细信息

```bash
ll /etc -d 
```

2、显示/etc/fstab文件的inode号

```bash
ls -i /etc/fstab
```

3、显示/var目录下所有文件的详细信息(包括.和..）

```bash
ll -a /var
```



### 1.2  mkdir

  mkdir是用来创建目录用的。

**使用方法**：

```bash
mkdir [options] DIR

常用选项：
	-m,--mode：指定创建出的目录的权限
	-p,--parent：当创建的文件夹没有父目录时使用，使得在创建该目录时，会将其父目录一起创建。
```

**示例**：

1、如果当前的系统上没有/haha目录，请用一条指令创建/haha/hahaha/hahahaha目录

```bash
mkdir -p /haha/hahaha/hahahaha
```

2、创建一个目录/hehe，该目录的权限为rwxrwxrwx。

```bash
mkdir -m 777 /hehe
```



> 一般情况下，第一次入门Linux的学习者还不懂权限是什么，因此上述示例的第2个可以先忽略。



### 1.3  cp命令

cp命令是拷贝命令，可以理解为copy的缩写

使用方法：cp命令有多种使用方法。

```bash
1、cp [OPTIONS] SOURCEFILE1 [SOURCEFILE2] ... DSTDIR
	# 将一个或多个文件(可以是目录文件)拷贝至一个文件夹下
2、cp [OPTIONS] SOURCEFILE NEWFILE	
	# 将一个文件(可以是目录文件)拷贝，粘贴并重命名为一个新的文件
3、cp [OPTIONS] -t DSTDIR SOURCEFILE1 [SOURCEFILE2] ...
	# 加一个-t选项可以使命令中的源和目的位置交换
	
常用选项：
	-i：交互式的复制文件，即每拷贝一个文件都会询问用户
	-r,-R：递归复制，可以将目录下的所有文件拷贝
	-f：若目标文件或目录已有相同的文件，是否覆盖
	-v：详细的显示过程
	-P,--no-dereference：不追踪链接文件的目标文件
	--preserve：拷贝过程需要保留的特征，包括如下
		ownership：属主属组
		timestamp：时间戳
		mode：权限
		links：链接。若保留该特征，则拷贝命令会直接对链接的目标文件做拷贝
		xattr：扩展属性
		context：上下文
		all：保留所有特征
	-d：表示对于链接文件，在拷贝时不拷贝链接的目标文件。相当于--no-dereference 					--preserve=links
	-a：用来做备份，相当于-dR --preserve=all
```



**示例**：

1、备份`/etc/yum.repos.d/CentOS-Base.repo`。

```bash
cp -a /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
```

2、拷贝`/tmp`目录下的所有内容至`/tmp/trash`目录下，并显示详细信息

```bash
cp -R /tmp /tmp/bash
```

该操作并不可行，系统会报错`cp: cannot copy a directory, ‘/tmp’, into itself, ‘/tmp/trash’`。原因可以自己想想。



### 1.4  mv

  mv命令是用来做文件移动和重命名用的。

**使用格式**：

```bash
1、mv [OPTIONS] SOURCEFILE1 [SOURCEFILE2] ... DSTDIR
	# 将一个或多个文件(可以是目录文件)移动至一个文件夹下
2、mv [OPTIONS] SOURCEFILE [DIRPATH/]NEWFILE	
	# 移动并重命名源文件
3、mv [OPTIONS] -t DSTDIR SOURCEFILE1 [SOURCEFILE2] ...
	# 加一个-t选项可以使命令中的源和目的位置交换
	
常用选项：
	-i：交互式的移动
	-f：强制覆盖
```

**示例**：

将`/etc/yum.repos.d/CentOS-Base.repo`重命名为`/etc/yum.repos.d/CentOS-Base.repo.bak`

```bash
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
```



### 1.5  rm和rmdir

rm命令是用来删除文件用的；rmdir是用来移除空目录用的，不过rm可以代替rmdir来删除空目录，因此该指令较为鸡肋。

**rm的使用格式**：

```bash
rm [OPTIONS] FILENAME

常用选项：
	-i：交互式删除，这是默认选项
	-r：递归删除目录下的所有文件
	-f：强制删除文件，即不交互式删除
```

**rmdir的使用格式**：

```bash
rmdir [OPTIONS] EMPTYFILE

常用选项：
	-p：递归删除空目录
```

**示例**：

删除`/tmp`目录下的所有文件。

```bash
rm -rf /tmp
```

### 1.6  touch

touch命令使用来创建或更新文件时间用的。要学会touch指令的使用，要先懂得时间戳是什么

#### 1.6.1  时间戳

每个文件都有三个时间，分别是：

​	access time：表示文件的上次读取时间;

​	modify time：指的是文件的上次内容修改时间;

​	change time：指的是文件的上次更改时间，这包括了文件的内容、权限和属组、属主等信息

**使用格式**：

```bash
touch [OPTIONS] FILE1 [FILE2]...

常用选项：
	-a：仅修改文件的读取时间为当前的时间
	-m：仅修改文件的修改时间为当前的时间
	-t：衔接timestamp，前面加上-a或-m参数一起使用来指定修改的是访问时间或修改时间，衔接的		  参数是格式为[[CC]YY]MMDDhhmm[.ss]的时间。CC指的是年份的前两位，可以省略，Linux会自         动指定为当前世纪。从左到右需要写的内容为年，月，日，小时，分，秒。

# 一般不带选项是用来创建文件的，但若想创建的文件已存在则更新该文件的所有时间戳。
```

> 很多人都会在想有没有-c选项来更新changetime，其实并没有这个选项，原因是每次更新access time和mtime时，都会自动更新changetime为当前的时间。

**示例**：

创建一个名为testfile的文件，并将其的atime修改为2000年1月1日0小时0分0秒

```bash
touch -at 200010100000 testfile
```



# 2.  文件的元数据信息

使用stat查看文件的元数据信息。元数据信息包括我们之前提到的时间戳。

**使用格式**：

```bash
stat [OPTIONS] FILE

# stat的使用一般不需要使用选项，因此在此不做介绍
```



接下来以一个例子来讲解stat显示出的详细信息，显示上面创建的testile文件的元数据。

```bash
stat testfile
```

显示的内容如下：

![1554017683003](images/1554017683003.png)

**常用的信息：**

- Inode：Inode号

- Links：文件的硬链接数量

- Access：文件隐藏权限和特殊00权限

- Uid：文件属主及其Uid

- Gid：文件的属组及其Gid

- Context：文件的安全上下文

- Access：文件的acess time

- Modifty：文件的modifty time

- Change：文件的change time
