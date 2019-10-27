---
layout: post
title: shell-about
date: 2019-06-06
tags: shell
----

记录CentOS下，常用的命令。有时候很难记得清楚，同时方便新来的同学查阅。（将不停的追加和完善）

1）查看CPU情况

```shell
cat /proc/cpuinfo |grep "model name" && cat /proc/cpuinfo |grep "physical id"
```

2）查看内存大小

```shell
cat /proc/meminfo |grep MemTotal
```

3）查看硬盘状况

```shell
fdisk -l |grep Disk
df -TH
```

4) 拷贝文件夹（全部内容）

```shell
cp -f file1 file2 （复制文件，强制）
cp -R dir1 dir2 （复制文件夹
```

5）删除文件和文件夹

```shell
rm -f demo.txt (强制删除文件)
rm -rf demo (强制删除文件夹)
```

6)从服务器间拷贝数据（示例路径）

```shell
scp -r root@139.129.99.98:/alidata/software /alidata/software
```

7）解压缩tar.gz文件

```shell
tar -xzvf file.tar.gz
```

8)  显示filename最后100行

```shell
tail -n 100 filename
```

9)  rpm安装

```shell
rpm -ivh MySQL-shared-5.5.47-1.linux2.6.x86_64.rpm
rpm -ivh MySQL-server-5.5.47-1.linux2.6.x86_64.rpm --force --nodeps   (强制安装)
```

10) 6.8版本 服务启动的方法

```shell
service start mysql
```

11) 7.4版本 服务启动的方法

```shell
systemctl start mysqld
```

12) 给文件夹开放权限

```shell
chmod -R 777 文件夹
```

参数-R是递归的意思
777表示开放所有权限