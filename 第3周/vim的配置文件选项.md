# 1.  vim的配置文件类型

vim的配置文件和bash一样都分为全局的配置文件和用户的配置文件。

全局配置文件：/etc/vimrc

用户配置文件：~/.vimrc

> 用户配置文件默认不存在，需要用户自己创建



# 2.  vim的配置文件选项

set nu ：显示行号

set nonu ：取消显示行号

set ai ：启动自动缩进的功能

set noai ：取消自动缩进的功能

set ic ：忽略字符的大小写

set noic ：不忽略字符的大小写

set sm ：括号匹配（老版本会有这个选项，新版本是一直打开的，无法关闭）

set nosm ：取消括号匹配

syntax on|off ：开启或关闭语法高亮

set hlsearch ：开启搜索高亮

set nohlsearch ：取消搜索高亮

set tabstop=# ：设备tab键的缩进为#个空格符



# 3.  示例

vim定制自动缩进四个字符

```bash
echo "set tabstop=4" >> /etc/vimrc
```

