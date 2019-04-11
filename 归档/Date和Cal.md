date是用来在Linux下查看时间的命令，cal是用来在Linux下查看日历的命令。

**date的常见使用方法**：

```bash
date [OPTION]... [+FORMAT]
```

**cal的常见使用方法**：

```bash
cal [options] [[[day] month] year]

常用选项：
	-y：指定显示哪一年的日历，不衔接参数则显示今年的日历
	
# 不加选项仅显示这个月的日历
```



**示例**：

1、创建一个文件后跟现在的日期。

```bash
touch file_$(date +%Y-%m-%d)
```

2、显示今年的日历

```bash
cal -y
```

