# 0.  shell script介绍

shell script 是shell内置的脚本语言，通过编写shell script可以完成自动化的命令行运行，并且其中还有分支结构和循环结构，这大大提高了shell script自动化的灵活性。由于Linux系统中内置了很多的shell，而每个shell对应的shell script是不同的，所以接下来我们介绍的便是最典型的bash的shell script。



# 1.  bash脚本编写入门

bash脚本的保存格式是`名称.sh`,并且文件首行必须写上`#!/bin/bash`，该行并不是注释，而是标注这个shell script必须使用`/bin/bash`这个shell脚本来读取运行。

正确的命名文件并且在首行加入`#!/bin/bash`后，就可以编写自己想要的脚本了。若不加入分支、循环等结构bash脚本的每一行都是一个可以在命令行执行的命令(除注释外)。

> 养成编写脚本良好习惯：需要在脚本前加入几行注释，这些注释包括author,email,version,descripion等。

**运行脚本的方法**：

1. 使用chmod修改文件的运行权限，让当前bash进程的拥有者可以运行。
2. 使用bash文件运行之，例如`bash script.sh`



**示例**：

编写一个脚本输出"hello,world"。

```bash
vim helloworld.sh
```

helloworld.sh的内容是以下这样

```bash
#!/bin/bash
echo "hello,world"
```

接着采用修改权限的方法，使其运行

```bash
chmod 777 helloworld.sh
./helloworld.sh
```

也可以使用用bash文件运行之

```bash
bash helloworld.sh
```



以上是最基础的bash脚本，可以多添加几行命令，这样在运行时可以一次性自动完成多条命令。接下来要介绍下bash的变量，一般每个语言会设有变量这种机制。变量可以理解为一段命名的存储空间，通过命名达到用户容易存储数据的目的。

# 2.  bash变量



## 2.1  变量命名的规则

1. 首个字符只能是`_`、字母。
2. 变量名称只能包含`_`、字母、数字
3. 不能使用bash的关键字命名(例如分支语句的if,fi等)
4. 最好做到见名知意



变量依据作用范围、参与的运算、以及表示的数据范围来分类。接下来就介绍下bash中的变量种类。



## 2.2  bash变量种类的划分

bash中的变量种类一般依据生效范围来划分：

1. **本地变量**：该类变量只在当前的bash进程生效。
2. **环境变量**：该类变量在当前的bash进程及其子进程生效
3. **局部变量**：一般只在bash脚本的某一部分生效(某一部分通常指函数)
4. **位置变量**：在脚本代码被执行时，命令行传递给其的参数，用\${1},\${2}...，\${10}等表示
5. **特殊变量**：特殊变量都是固定的，包含着特殊的值，下面列出所有的特殊变量：
   1. **$?**：上个命令执行后的返回值，若为0，则表示执行成功，若为其他数则为执行失败
   2. **$0**：这个bash文件的文件名
   3. **$#**：执行该脚本时传入的参数个数
   4. **$***：传递给该脚本的所有参数
   5. **$@**：引用传递给该脚本的所有参数



## 2.3  bash各类变量的使用方式

### 2.3.1  本地变量：

**变量赋值**：name='value'
value是以下三种：
	1.直接赋值字符串：name='content'

​	2.引用其他以有变量：name="\${oldvariable}"或name="​\$oldvariable"

​	3.引用其他命令执行后的回显字符：name=\`COMMAND\`,name=\$(COMMAND)

**变量的引用方法**：\$name或​\${name}，使用前者在编写时可能会出现错误。例如：想引用name变量后再加一个字符a的话用前者`​$namea`会被bash解释器错认为是引用namea变量，而使用`​${name}a`来表示则不会被解释错误。



赋值时，使用双引号和单引号是由分别的。

**双引号**：强引用，不会引用`$`符号后的变量值

**单引号**：弱引用，会引用`$`符号后的值



若要显示所有已定义的变量，则可以使用`set`命令。

若要销毁一个变量则可以使用`unset`命令



**使用示例**：

```bash
#直接引用字符串
[root@localhost ~]# test_name=120

#使用set查看已定义的变量
[root@localhost ~]# set | grep test_name
test_name=120

# 弱引用举例
[root@localhost ~]# echo "$test_name"
120

# 强引用举例
[root@localhost ~]# echo '$test_name'
$test_name

# 变量引用
[root@localhost ~]# test_name1="$test_name"
[root@localhost ~]# echo "$test_name1"
120

# 命令引用,用变量保存当前系统的用户数量：
[root@localhost ~]# users_count=`wc -l /etc/passwd | cut -d " " -f1`
[root@localhost ~]# echo ${users_count}
56

# 尝试销毁上面创建的users_count变量
[root@localhost ~]# unset users_count
[root@localhost ~]# echo ${users_count}
（这里会输出空白）

# 尝试给PATH添加搜索的路径：
[root@localhost ~]# PATH="${PATH}:/tmp"
[root@localhost ~]# echo $PATH
/usr/lib64/qt-3.3/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin:/tmp
```



### 2.3.2  环境变量

环境变量可以由以有的本地变量得到：

```bash
export name
delcare -x name
```

也可以直接新建一个环境变量并赋值

```bash
export name=VALUE
declare -x name=VALUE
```



**变量的引用方法**：\$name, ${name}

可以使用`export`,`env`,`printenv`三个指令来显示所有的环境变量

可以使用`unset name`来销毁变量



bash有很多内置的环境变量，bash进程运行过程中会使用到这些环境变量，所以不能随便乱修改。

表较常见的bash内建的环境变量名称及其值如下表格所示：

| 变量     | 值及其意义                                                   |
| -------- | ------------------------------------------------------------ |
| PATH     | 变量内容为用冒号隔开的路径，从那些路径中查找到用户键入的命令字符串所对应的命令文件 |
| SHELL    | 变量内容为当前终端使用的shell                                |
| UID      | 当前用户的UID                                                |
| HISTSIZE | history可以保存的最大命令数量                                |
| HOME     | 当前用户的家目录，同‘~’                                      |
| PWD      | 当前所在目录的路径，使用pwd指令时会调用PWD环境变量           |
| OLDPWD   | 之前所在的目录                                               |
| HISTFILE | 保存history的文件                                            |
| PS1      | 命令提示符格式,默认为[\u@\h \W]$                             |

> 环境变量的详细解释和使用可以使用man bash来查看



**使用示例**：

```bash
# 尝试先命名本地变量，再将其修改为环境变量
[root@localhost /]# baidu='www.baidu.com'
[root@localhost /]# export baidu

# 切换到子进程，查看子进程是否还有环境变量的值（查看完成记得exit返回到父进程）
[root@localhost /]# bash
[root@localhost /]# echo $baidu
www.baidu.com

# 用三种显示所有环境变量的方式，查看命名的变量是否还在：
[root@localhost /]# env | grep baidu
baidu=www.baidu.com
[root@localhost /]# export | grep baidu
declare -x baidu="www.baidu.com"
[root@localhost /]# printenv | grep baidu
baidu=www.baidu.com

# 删除声明的环境变量
[root@localhost /]# unset baidu
[root@localhost /]# echo "$baidu"
（这里会输出空白）

# 尝试修改环境变量PS1，使其可以显示当前的时间(注意必须要使用单引号，否则'\$'部分将无法正常显示)：
[root@localhost /]# PS1='[\u@\h \W\t]\$'
[root@localhost /11:01:12]#
```



### 2.3.3  位置变量

位置变量从\$1开始到\$∞，若引用的位置变量没被传入值，引用内容的则为空。



**示例**：

```bash
[root@localhost ~]# vim test.sh
[root@localhost ~]# cat test.sh
#!/bin/bash
echo "$1"
echo "$2"
echo "$3"
[root@localhost ~]# bash test.sh 1 2
1
2

[root@localhost ~]# bash test.sh 1 2 3
1
2
3
```



### 2.3.4  特殊变量

特殊变量的作用在上面已经介绍过了，这里不在介绍。

**示例**：

```bash
[root@localhost ~]# vim test.sh 
[root@localhost ~]# cat test.sh 
#!/bin/bash
echo "$0"
echo "$#"
echo "$*"
echo "$@" 
[root@localhost ~]# bash test.sh 1 2 3 4
test.sh
4
1 2 3 4
1 2 3 4
[root@localhost ~]# man bash
[root@localhost ~]# echo $?
0
[root@localhost ~]# man haha
No manual entry for haha
[root@localhost ~]# echo $?
16
```



# 3  bash的算术运算

我们在前面的对于变量类型的分类是通过作用范围来划分的，其实变量类型还可以分为字符串型、字符型、数值型等等。在bash中的变量都是以字符串型保存的。而用字符串型的保存的内容都是无法做到数字的加减乘除的。想要做到字符串的加就会像一下这样：

```
[root@localhost ~]# x=1
[root@localhost ~]# y=2
[root@localhost ~]# echo "$x+$y"
1+2
```

不过bash设置了相关的指令和符号帮助字符串可以以数值的方式加减乘除。



首先介绍下bash的运算操作符有：+,-,\*,/,%(取余),\*\*(幂)



**bash的算术运算的实现方式：**

1. let VARIABLE=EXPRESSION
2. $[EXPRESSION]
3. $((EXPRESSION))
4. 命令：expr ARG1 OP ARG2，例如A=\$(expr \$B \\* \$C)

我个人较为推荐使用第一种方法，写起来很顺手，读起来也很方便。



bash支持增强型赋值

`+=`,`-=`,`*=`,`/=`,`%=`

以及自增`++`，自减`--`



**示例**：

编写一个脚本，这个脚本可以实现自动添加三个用户(用户名自己取),然后计算这三个用户的uid之和

```bash
#/bin/bash
useradd aaa
useradd bbb
useradd ccc
aaaid=$(id -u aaa)
bbbid=$(id -u bbb)
cccid=$(id -u ccc)
let sum=$aaaid+$bbbid+$cccid
echo "$sum"
```

