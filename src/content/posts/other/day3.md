---
title: day3
published: 2025-10-18
description: "linux 学习笔记"
tags: ["linux"]
category: ctf
draft: false
---

# Liunx 学习笔记

#### touch 的使用
touch 命令主要用于创建空文件和修改文件的时间戳
```bash
# 创建单个文件
touch filename.txt

# 创建多个文件
touch file1.txt file2.txt file3.txt

# 只修改访问时间
touch -a filename.txt

# 修改访问时间为指定时间
touch -a -t 202312251430.00 filename.txt
# 格式：YYYYMMDDhhmm.ss
```
注意除了可以用touch 创建命令还可以使用文件重定向符号来创建文件如
```bash
# echo 输出的hellohello 将会被重定向到file1.txt 中
echo "Hello_world" > file1.txt
```

#### 如何修改用户的密码
```bash
# 使用passwd 命令
passwd
```
#### 如何切换账户
```bash
# 使用su命令
su        # 切换为 root 账号但是保留当前环境

su -      # 切换为 root 账号

su - [目标用户名] # 切换为目标用户账号 同时回到该用户的 home 路径

su -c command    # 使用
```

#### 如何压缩解压文件
##### 使用 tar
```bash
tar [选项] [文件名] [文件或目录...]

# .tar.gz gzip 压缩
tar -czvf 压缩包名.tar.gz 要压缩的文件或目录

# bzip 压缩
tar -cjvf 压缩包名.tar.bz2 要压缩的文件或目录

# xz 压缩
tar -cJvf 压缩包名.tar.xz 要压缩的文件或目录

# 速度优先用 gzip，压缩率优先用 xz，平衡选择用 bzip2。

# 解压
tar -xzvf 压缩包名.tar.gz
tar -xjvf 压缩包名.tar.bz2
tar -xJvf 压缩包名.tar.xz

tar -xvf filename.tar.*    # 适用于所有格式
```

#### 如何查看基本日志
```txt
# 系统主日志文件
/var/log/syslog          # Ubuntu/Debian 系统日志
/var/log/messages        # CentOS/RHEL 系统日志

# 认证相关日志
/var/log/auth.log        # 认证日志（Ubuntu/Debian）
/var/log/secure          # 认证日志（CentOS/RHEL）

# 内核日志
/var/log/kern.log        # 内核消息

# 启动日志
/var/log/boot.log        # 系统启动日志

# 包管理器日志
/var/log/dpkg.log        # Debian/Ubuntu 包安装日志

# 系统服务日志
/var/log/cron            # 计划任务日志
/var/log/mail.log        # 邮件服务日志
/var/log/faillog         # 失败登录尝试
```


##### 使用 journalctl 查看完整的系统日志
```
# 查看完整的系统日志
journalctl

# 查看实时日志（类似 tail -f）
journalctl -f

# 查看最近日志
journalctl -n 20         # 最后20行
journalctl --since "1 hour ago"

# 按服务查看日志
journalctl -u nginx.service
journalctl -u ssh.service

# 按优先级过滤
journalctl -p err        # 只看错误
journalctl -p warning    # 只看警告
journalctl -p info       # 只看信息

# 优先级级别：emerg(0), alert(1), crit(2), err(3), warning(4), notice(5), info(6), debug(7)

# 按时间查看
journalctl --since "2024-01-15 09:00:00"
journalctl --since yesterday
journalctl --since "1 hour ago"
journalctl --until "30 minutes ago"

# 查看特定时间段的日志
journalctl --since "09:00" --until "10:00"

# 使用 less 查看日志（推荐）
less /var/log/syslog

# 在 less 中的常用操作：
# - 空格键：向下翻页
# - b：向上翻页
# - /关键词：搜索
# - n：下一个匹配项
# - N：上一个匹配项
# - g：跳到文件开头
# - G：跳到文件结尾
# - q：退出
```

#### Liunx 常用二进制文件目录:
 + /bin 系统基本命令二进制文件目录 -> ls cp mv rm ...
 + /sbin 系统管理二进制文件    -> fdisk ...
 + /usr/bin 用户命令二进制文件 -> python vim gcc tar ...
 + /usr/local/bin 第三方软件二进制库 ...

#### 文件的赋权
在Liunx 中 给文件赋权主要通过chmod 命令实现。以下是详细的使用方法

```bash
chmod [选项] 权重 文件名
```
权限表示方法:
 - 1. 数字表示法:
     + **r = 4**     # 读 100
     + **w = 2**     # 写 010
     + **x = 1**     # 执行 001
 - 2. 符号表示法:
     + 用户类别:
       * **u** 文件的所有者
       * **g** 所属组的用户
       * **o** 其他用户
       * **a** 所有用户
     + 操作符
       * **\+** 添加权限
       * **\-** 移除权限
       * **=**  覆盖权限
     + 权限类型
       * **r** 读权限
       * **w** 写权限
       * **x** 执行权限

常用选项:
 + **-R** 递归处理目录以及其子目录
 + **-v** 显示权限变更信息
 + **-c** 只有变更时显示信息

实操:
```bash
# 数字表示法
chmod 755 filename  # 所有者：rwx，组用户：r-x，其他用户：r-x
chmod 644 filename  # 所有者：rw-，组用户：r--，其他用户：r--
chmod 777 filename  # 所有用户都有rwx权限（不推荐）

chmod u=rw file       # 设置所有者只有读写权限
chmod g=r file        # 设置组用户只有读权限
chmod o= file         # 设置其他用户无任何权限
chmod u=rwx,g=rx,o= file  # 组合设置

# 递归给目录下所有文件添加执行权限
chmod -R a+X directory/

# 递归设置目录权限
chmod -R u=rwx,g=rx,o= directory/
```
#### 如何运行可执行文件
```
# 直接运行
./script.sh
./script_bin

# 指定运行方式
bash ./script.sh
python3 ./script_py.py
```

#### 递归删除
```
# 递归删除目录及其所有内容
rm -r dir_name

rm -rf dir_name  # 强制递归删除
rm -ri dir_name  # 递归删除，但是逐个提示
```

#### find 和 grep 命令
```bash
find [路径] [选项] [操作]

grep [选项] "模式" [文件]

# 在当前目录查找文件
find . -name "filename.txt"
find . -name "*.txt"           # 通配符查找
find . -name "*.log" -type f   # 只找文件
find . -name "src" -type d     # 只找目录

# 不区分大小写
find . -iname "readme*"

find . -type f                 # 普通文件
find . -type d                 # 目录
find . -type l                 # 符号链接
find . -type s                 # 套接字文件

# 按修改时间
find . -mtime -7              # 7天内修改的文件
find . -mtime +30             # 30天前修改的文件
find . -mmin -60              # 60分钟内修改的文件

# 按访问时间
find . -atime -1              # 1天内访问的文件
find . -ctime +90             # 90天前状态改变的文件

# 按照大小
find . -size +100M            # 大于100MB的文件
find . -size -1G              # 小于1GB的文件

# 按照权限
find . -perm 644              # 权限精确为644的文件
find . -perm -u=x             # 用户有执行权限的文件
find . -perm /g=w             # 组用户有写权限的文件

# 在文件中搜索
grep "error" logfile.txt
grep "TODO" *.py              # 在多个文件中搜索

# 递归搜索
grep -r "function" src/       # 递归搜索目录
grep -r "config" . --include="*.js"  # 只搜索js文件

# 显示行号
grep -n "pattern" file.txt

# 基本正则
grep "^start" file.txt        # 以start开头的行
grep "end$" file.txt          # 以end结尾的行
grep "a.b" file.txt           # a任意字符b

grep -i "pattern" file.txt    # 忽略大小写
grep -v "pattern" file.txt    # 反向匹配（不包含的行）
grep -c "pattern" file.txt    # 统计匹配行数
grep -l "pattern" *.txt       # 只显示包含匹配的文件名
grep -L "pattern" *.txt       # 显示不包含匹配的文件名
grep -A 3 "pattern" file.txt  # 显示匹配行后3行
grep -B 2 "pattern" file.txt  # 显示匹配行前2行
grep -C 2 "pattern" file.txt  # 显示匹配行前后各2行
```

#### 创建软连接
可以将Liunx 近似为Windows 的快捷方式
```bash
ln -s 源文件或目录 链接名称
```
如何识别软链接:

使用 **ls -l** 查看文件权限，软连接文件总是lrwxrwxrwx
同理通过第一位 如果为 **s**rwxrwxrwx 则为套接字文件

#### 进程相关

ps 


