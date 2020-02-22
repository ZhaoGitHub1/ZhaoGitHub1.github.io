---
title: Linux常用命令
categories: 集成&运维
tags:
  - linux
  - 命令
description: 记录下Linux常用命令
date: 2018-09-15 12:09:07
---

** {{ title }}：** <Excerpt in index | 首页摘要>

<!-- more -->
<The rest of contents | 余下全文>

![linux](linux-commands/linux.jpg)

　　记录下linux常用操作文件命令，以及安装常用软件方法，备忘以及方便查找

## 常用命令

### 文件和目录

- `cd ..`：返回上一级目录
- `cd /`：进入根目录
- `cd ~`：进入用户主目录
- `pwd`：打印当前目录juedui路径
- `ls`：列出当前目录中的文件
- `ll`：列出当前目录中的文件详细信息
- `ls -a`：显示隐藏文件 
- `tree`：显示文件和目录由根目录开始的树形结构
- `lstree`：显示文件和目录由根目录开始的树形结构
- `mkdir` dir1：创建一个叫做 'dir1' 的目录'
- `mkdir` dir1 dir2：同时创建两个目录 
- `mkdir -p` /tmp/dir1/dir2：创建/tmp/dir1/dir2目录树
- `rm -f` file1：删除一个叫做 'file1' 的文件'
- `rmdir` dir1：删除一个叫做 'dir1' 的目录'
- `rm -rf` dir1：删除一个叫做 'dir1' 的目录并同时删除其内容
- `rm -rf` dir1 dir2：同时删除两个目录及它们的内容
- `mv` dir1 dir2：重命名/移动 一个目录

### 文件搜索

- `find . -name` "*.txt"：列出当前目录及子目录下所有后缀为 txt 的文件
- `find . -type f`：列出当前目录及子目录下所有一般文件
- `find . -ctime` -20：列出当前目录及子目录下所有最近 20 天内更新过的文件

### 打包和压缩文件

- `bunzip2` file1.bz2：解压一个叫做 'file1.bz2'的文件 
- `bzip2` file1：压缩一个叫做 'file1' 的文件 
- `gunzip` file1.gz：解压一个叫做 'file1.gz'的文件
- `gzip` file1：压缩一个叫做 'file1'的文件 
- `gzip -9` file1：最大程度压缩  
- `rar a` file1.rar test_file：创建一个叫做 'file1.rar' 的包  
- `rar a` file1.rar file1 file2 dir1：同时压缩 'file1', 'file2' 以及目录 'dir1'  
- `rar x` file1.rar：解压rar包  
- `unrar x` file1.rar：解压rar包  
- `tar -cvf` archive.tar file1：创建一个非压缩的 tarball  
- `tar -cvf` archive.tar file1 file2 dir1：创建一个包含了 'file1', 'file2' 以及 'dir1'的档案文件  
- `tar -tf` archive.tar：显示一个包中的内容 
- `tar -xvf` archive.tar：释放一个包  
- `tar -xvf` archive.tar `-C` /tmp：将压缩包释放到 /tmp目录下  
- `tar -cvfj` archive.tar.bz2 dir1：创建一个bzip2格式的压缩包  
- `tar -jxvf` archive.tar.bz2：解压一个bzip2格式的压缩包  
- `tar -cvfz` archive.tar.gz dir1：创建一个gzip格式的压缩包  
- `tar -zxvf` archive.tar.gz：解压一个gzip格式的压缩包  
- `zip` file1.zip file1：创建一个zip格式的压缩包 
- `zip -r` file1.zip file1 file2 dir1：将几个文件和目录同时压缩成一个zip格式的压缩包  
- `unzip` file1.zip：解压一个zip格式压缩包 

### yum相关

- `yum install` package_name：下载并安装一个软件包  
- `yum localinstall` package_name.rpm：将安装一个软件包，使用你自己的软件仓库为你解决所有依赖关系
- `yum update`：更新当前系统中所有安装的软件包  
- `yum update` package_name：更新一个软件包
- `yum remove` package_name：删除一个软件包  
- `yum list` ：列出当前系统中安装的所有包  
- `yum search` package_name：在仓库中搜寻软件包  
- `yum clean packages`：清理缓存目录下软件包  
- `yum clean headers`：删除所有头文件  
- `yum clean all`： 删除所有缓存的包和头文件 

### 查看文件内容

- `cat` file1：从第一个字节开始正向查看文件的内容
- `more` file1：分页查看一个长文件的内容
- `less` file1：less 与 more 类似，但使用 less 可以随意浏览文件，而 more 仅能向前移动，却不能向后移动，而且 less 在查看之前不会加载整个文件。
- `head` -2 file1：查看一个文件的前两行
- `tail` -2 file1：查看一个文件的最后两行
- `tail -f` file1：实时查看一个文件中的内容

### 文本处理

- `grep` test *file：当前目录中，查找后缀有 file 字样的文件中包含 test 字符串的文件，并打印出该字符串的行
- `grep -r` update /etc/acpi：查找指定目录/etc/acpi 及其子目录（如果存在子目录的话）下所有文件中包含字符串"update"的文件，并打印出该字符串所在行的内容
- `grep -v` test `*`test`*`：查找文件名中包含 test 的文件中不包含test 的行

### 系统设置

- `top`：实时显示 process 的动态
- `free -m`：查看内存使用量和交换区使用量
- `date`：显示当前时间
- `clear`：清屏
- `alias` lx=ls：指定lx别名为ls
- `bind -l`：列出所有按键组合
- `eval`：重新运算求出参数的内容
- `ps -ef|grep` mysql：查看mysql服务进程信息