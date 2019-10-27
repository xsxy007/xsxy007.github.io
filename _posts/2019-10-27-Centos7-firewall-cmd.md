---
layout: post
title: Centos7-firewall-cmd
date: 2019-10-27
tags: linux
---

### firewall-cmd

> CentOS 7 默认使用的防火墙是firewalld，不是CentOS 6的iptables

查看防火墙状态

```shell
systemctl status firewalld
```

也可以

```shell
firewall-cmd --state
```

启动防火墙

```shell
systemctl start firewall
# 或者
systemctl start firewalld.service
```

关闭防火墙

```shell
systemctl stop firewall
# 或者
systemctl stop firewalld.service
```

重启防火墙

```shell
systemctl restart firewall
# 或者
systemctl restart firewalld.service
```

设置开机启用防火墙

```shell
systemctl enable firewalld.service
```

设置开机不启用防火墙

```shell
systemctl disable firewalld.service
```

添加防火墙端口

```shell
firewall-cmd --zone=public --add-port=3306/tcp --permanent
```

添加做个端口

```shell
firewall-cmd --zone=public --add-port=80-90/tcp --permanent
```

> 命令解析：  
– zone #作用域  
–add-port=3690/tcp # 添加端口 格式为 端口/协议  
–permanent 永久性添加，没有这个重启就会失效  

添加端口之后重新载入

```shell
firewall-cmd --reload
```

列出支持的zone

```shell
firewall-cmd --get-zones
```

列出支持的服务(在列表中的服务是放行的)

```shell
firewall-cmd --get-services
```

查看ftp服务是否支持

```shell
firewall-cmd --query-service ftp
```

临时开放ftp服务

```shell
firewall-cmd add-service=ftp
```

永久开放ftp服务

```shell
firewall-cmd --add-service=ftp --permanent
```

永久移除ftp服务

```shell
firewall-cmd --remove-service=ftp --permanent
```

永久添加8080端口

```shell
firewall-cmd --add-port=8080/tcp --permanent
```

查看规则，这个命令和iptables是相同的

```shell
iptables -L -n
```

查看已经开放的端口

```shell
firewall-cmd --list-ports
```

查看防火墙所有的信息

```shell
firewall-cmd --list-all
```

查看帮助

```shell
man firewall-cmd
```

查看本机已经启用监听的端口

```shell
ss -ant
# centos7以下使用： netstat -ant
```