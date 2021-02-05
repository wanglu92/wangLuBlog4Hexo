---
title: Linux使用
date: 2021-02-03 21:31:19
tags:
---

kali Linux 安全渗透测试使用Linux

补天？？？一个白帽子网站？？？

LAMP`Linux apache mysql php` 

LNMP`Linux nginx mysql php`

## 环境准备

+ 使用VMware
+ 使用云服务器

## 开关机和基本目录介绍

Linux没有输出就没有问题

```shell
sync #关机器同步数据
shutdown #关机指令
shutdown -h 10 #10分钟后关机
shutdown -h now #现在关机
shutdown -h 10:00 #指定时间关机
shutdown -h +10 #十分钟后关机
shutdown -r now #现在重启
shutdown -r +10 #十分钟后重启
reboot #立即重启
halt #关机
poweroff #关机
```

### 系统目录结构

```
/bin 常用的命令
/boot 存放启动Linux时的一些核心文件
/dev Device设备的缩写，存放外部设备
/etc 存放所有系统管理需要的配置文件和子目录
/home 用户的主目录，在Linux中，每个用户有一个自己的目录
/lib 这个目录里存放系统最基本的动态链接共享库
/lost+found 一般情况下为空，当系统非法关机时，存放一些文件
/media Linux会自动识别一些设备，例如U盘光驱等，识别后会挂在到该目录下
/mnt 系统提供的让用户临时挂在别的文件系统的
/opt 这是给主机额外安装软件所摆放的目录
/proc 虚拟目录，它是系统内存的映射，可以通过直接访问这个目录来获取系统信息
/root 系统管理员目录
/sbin super user 存放系统管理员使用的系统管理程序
/srv 存放一些服务启动之后需要提取的数据
/tmp 存放临时文件
/usr 非常重要，用户的很多应用程序和文件都放在这个目录下
/usr/bin 系统用户使用的应用程序
/usr/sbin 超级用户使用的比较高级的管理程序和系统守护程序
/sys 文件系统
/var 存放不断扩充的东西，习惯将经常被修改的文件放在该目录下，如何工日志文件
/run 是一个临时文件系统，存储系统启动以来的信息。当系统启动时，这个目录下的文件应该被删除。
/www 服务器中一些网页相关的东西
```

## 常用的基本命令

```
mkdir 创建文件夹
mkdir -p 递归创建文件夹

rmdir dir_name 删除空的文件夹
rmdir -p dir_name 删除不为空的文件夹

cp old_path new_path 赋值文件或者文件夹

rm 
-f 不会出现警告，强制删除
-r 递归删除
-i 互动，弹出询问

mv  移动文件或者文件/重命名文件
-f 强制移动
-u 只替换已经更新过的文件
```

## 文件属性查看和修改

```
ls -la 
```

更改文件属组

```
chgrp [-R] 属组
```

更改文件属主

```
chown [-R] 属主名：属组名 文件名
```

更改文件的9个属性

```
chmod [-R] xyz 文件或目录
```

## 文件内容查看

```
cat 查看第一行开始文件内容
tac 查看最后一行开始文件内容
nl 显示行号
more 可以向后翻页
less 可以向前向后翻页 可以使用/内容来查找（向下） ?查找内容(向上) q退出 n继续找下一个 N向上寻找 
head -n 查看头几行
tail -n 查看尾几行
```

Ifconfig 查看网络配置

## 硬链接和软连接

+ 硬链接：两个指向同一个文件，允许一个文件拥有多个路径。用户可以通过硬链接到一些重要文件上防止误删。

+ 软连接：类似windows下的快捷方式。删除原文件后快捷方式就访问不了。

+ 使用连接

  ```
  ln file_name1 file_name2 硬链接
  ln -s file_name1 file_name3 软连接
  ```

## Vim编辑器

vim通过一些插件可以实现和IDE一样的功能

查看内容、编辑内容、保存内容

> 三种使用模式
>
> 命令模式、输入模式、底线命令模式

```
:set nu 显示行号
:set nonu 不显示行号
```

## Linux账号管理

+ 添加删除和修改账户
+ 用户口令管理
+ 用户组管理

```
useradd 用户名 添加用户
-m:自动创建主目录
-G:分配组

userdel 用户名 删除用户
-r:删除用户的时候同时删除目录

usermod 修改内容 用户名 修改用户
```

/etc/passwd文件

```
su 用户名 切换用户

passwd 用户名 添加密码
passwd -l 用户名 锁定账户
passwd -u 用户名 解锁账户
passwd -d 用户名 清空密码
```

hostname 修改主机名 重连之后会生效

## 用户组管理

增加、删除和修改组

在/etc/group文件下

```
groupadd 组名 添加用户组
-g 指定组id

groupdel 组名 删除组

groupmod 内容 组名 修改组信息
-g id
-n 名

newgrp 用户自己切换组
```

/etc/shadow文件中存放密码

## 磁盘管理

> + df 列出磁盘的整体使用量
>   + -h 修改单位
> + du 检查当前磁盘空间使用量
>   + du -sm /* 检查根目录下每个目录所占用的容量
> + 挂载命令 mount
>   + mount /dev/wang 外部设备 /mnt/wang 挂载目录
>   + 卸载 umount -f 名字 强制卸载

## 进程管理

进程有两种运行方式！前台！后台！

一般服务都是后台运行的，基本的程序都是前台运行的

```
ps 命令 查看当前系统中正在执行的各种进程信息
-a 显示当前终端所有运行的进程信息
-u 用户运行的进程信息
-x 显示后台运行进程的参数

| 管道符
grep 查找符合的字符串

ps -ef:可以查看到父进程的信息

pstree -pu 进程树
-p 显示父id
-u 显示用户组

结束进程
kill pid
-9 强制结束

nohup后台执行程序
```

## 环境安装

安装软件的三种方式

+ rpm安装：红帽包管理工具

+ 解压安装：解压运行即可

+ yum安装

  + ```
    yum -y install 包名
    ```

### JDK安装

+ rpm安装 

```
rpm -ivh rpm包
rpm -qa|grep jdk 检测jdk版本
rpm -e --nodeps jdk版本 卸载
```

配置环境变量

## 防火墙

```
firewall-cmd --list-ports 查看防火墙开启端口
firewall-cmd --list-all 查看全部信息
systemctl status firewalld 查看防火墙状态
service firewalld start
service firewalld restart
service firewalld stop
firewall-cmd --zone=public --add-prot=80/tcp --permanent 开启防火墙端口
systemctl restart firewalld.service 重启防火墙
--zone 作用域
--add-point=80/tcp 添加端口
--permanent 永久生效
```

