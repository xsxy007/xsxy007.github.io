---
layout: post
title: Linux之systemd服务配置及自动重启
date: 2019-09-09
tags: linux
---

# Linux之systemd服务配置及自动重启

## 0 背景

在linux上开发时，往往需要将自己的程序做成服务，并且实现服务开机自动重启，以及服务崩溃后自动重启功能，本文就对该功能的实现做简单介绍，实现方法很简单，使用linux系统的systemd即可实现

## 1 systemd介绍

历史上，linux的启动一直采用init进程，比如

```shell
$ sudo /etc/init.d/apache2 start
```

或者

```shell
$ service apache2 start
```

这种方法有两个缺点。

一是启动时间长。init进程是串行启动，只有前一个进程启动完，才会启动下一个进程。  
二是启动脚本复杂。init进程只是执行启动脚本，不管其他事情。脚本需要自己处理各种情况，这往往使得脚本变得很长。  

Systemd 就是为了解决这些问题而诞生的。它的设计目标是，为系统的启动和管理提供一套完整的解决方案。

根据 Linux 惯例，字母d是守护进程（daemon）的缩写。 Systemd 这个名字的含义，就是它要守护整个系统。使用了 Systemd，就不需要再用init了。Systemd 取代了initd，成为系统的第一个进程（PID 等于 1），其他进程都是它的子进程。

systemctl是 Systemd 的主命令，用于管理系统。对于用户来说，最常用的是下面这些命令，用于启动和停止 Unit（主要是 service）。

### -立即启动一个服务

```shell
$ systemctl start apache.service
```
 
### -立即停止一个服务

```shell
$ systemctl stop apache.service
```
 
### -重启一个服务

```shell
$ systemctl restart apache.service
```
 
### -杀死一个服务的所有子进程

```shell
$ systemctl kill apache.service
```
 
### -重新加载一个服务的配置文件

```shell
$ systemctl reload apache.service
```
 
### -重载所有修改过的配置文件

```shell
$ systemctl daemon-reload
```
 
### -显示某个 Unit 的所有底层参数

```shell
$ systemctl show httpd.service
```
 
### -显示某个 Unit 的指定属性的值

```shell
$ systemctl show -p CPUShares httpd.service
```
 
### -设置某个 Unit 的指定属性

```shell
$ systemctl set-property httpd.service CPUShares=500
```
本文主要是对systemd的使用进行介绍，如果想进一步了解systemd的基本知识，可查阅相关资料

## 2 服务端脚本
这里我们写一个php的服务脚本server.php，用来实现一个服务

```php
<?php
$sock = socket_create(AF_INET, SOCK_DGRAM, SOL_UDP);
socket_bind($sock, '0.0.0.0', 10000);
for (;;) {
    socket_recvfrom($sock, $message, 1024, 0, $ip, $port);
    $reply = str_rot13($message);
    socket_sendto($sock, $reply, strlen($reply), 0, $ip, $port);
}
```
运行后可使用lsof指令来查看端口占用情况

```shell
lthpc@lthpc:~$ lsof -i:10000
COMMAND   PID  USER   FD   TYPE   DEVICE SIZE/OFF NODE NAME
php     40446 lthpc    3u  IPv4 37381218      0t0  UDP *:10000 
```
使用nc指令模拟客户端测试

```shell
$ nc -u 127.0.0.1 10000
Hello, world!
Uryyb, jbeyq!
```

3 创建服务
接下来使用systemd创建一个服务，写一个服务配置文件`/etc/systemd/system/rot13.service`

```
[Unit]
Description=ROT13 demo service
After=network.target
StartLimitIntervalSec=0
 
[Service]
Type=simple
Restart=always
RestartSec=1
User=ltpc
ExecStart=/usr/bin/env php /path/to/server.php
 
[Install]
WantedBy=multi-user.target
```

有几点需要注意，为了使服务能够自动无限次重启，需要增加以下几个配置

```
StartLimitIntervalSec=0

Restart=always

RestartSec=1
```

关于配置文件的具体参数含义，可参考该[文档](https://www.freedesktop.org/software/systemd/man/systemd.service.html)

设置好后，可以运行如下语句启动服务

```shell
$ systemctl start rot13
```

运行后，便启动了名为rot13的服务，可使用status查看服务状态

```shell
lthpc@lthpc:~/workspace_zong/tcptest$ systemctl status rot13
● rot13.service - ROT13 demo service
   Loaded: loaded (/etc/systemd/system/rot13.service; disabled; vendor preset: enabled)
   Active: active (running) since 一 2019-10-28 11:25:43 CST; 1min 28s ago
 Main PID: 44532 (php)
    Tasks: 1
   Memory: 5.2M
      CPU: 24ms
   CGroup: /system.slice/rot13.service
           └─44532 php /home/lthpc/workspace_zong/tcptest/server.php
 
10月 28 11:25:43 lthpc systemd[1]: Started ROT13 demo service.
```

为了开机自动启动，执行下以下语句

```shell
$ systemctl enable rot13
```

同样的，可以使用nc指令模拟客户端测试，可以看到服务已经正常启动运行了！

4 自动重启
为了测试是否可以正常自动重启，我们手动杀掉启动的服务进程，再查看进程号发现已经更换PID号了，说明重启过进程，并且使用nc -u 127.0.0.1 10000指令测试依然可以调用服务

```shell
$ systemctl status rot13 | grep PID
 Main PID: 44532 (php)
$ sudo kill 44532
$ systemctl status rot13 | grep PID
 Main PID: 44255 (php)
```

注意输入`systemctl stop rot13`时服务是不会重启的，所以如果有参数需要修改，直接运行stop指令改完再start就可以了


----

编写服务配置

```shell
vim /lib/systemd/system/website.service

[Unit]
Description=website
After=network.target
 
[Service]
Type=forking
ExecStart=/home/monitor/website/start.sh
ExecReload=/home/monitor/website/restart.sh
ExecStop=/home/monitor/website/shutdown.sh
 
[Install]
WantedBy=multi-user.target
```

编写对应启动停止等脚本
开启：

```shell
vim /home/monitor/website/start.sh
```

```shell
#!/bin/sh
export JAVA_HOME=/usr/java/jdk1.8.0_144
export PATH=$JAVA_HOME/bin:$PATH
 
nohup java -jar /home/monitor/website/demo-0.0.1-SNAPSHOT.jar &
```

关闭：

```shell
vim /home/monitor/website/shutdown.sh
```

```shell
#!/bin/sh
ps -ef | grep demo-0.0.1-SNAPSHOT.jar | grep -v grep | awk '{print $2}' | xargs kill -9
```

重启：

```shell
vim /home/monitor/website/restart.sh
```

```shell
#!/bin/sh
export JAVA_HOME=/usr/java/jdk1.8.0_144
export PATH=$JAVA_HOME/bin:$PATH
 
ps -ef | grep demo-0.0.1-SNAPSHOT.jar | grep -v grep | awk '{print $2}' | xargs kill -9
nohup java -jar /home/monitor/website/demo-0.0.1-SNAPSHOT.jar &
```

授权运行：

```shell
chmod +x start.sh shutdown.sh restart.sh
```

配置自启动
开机启动：`systemctl enable website`

启动：`systemctl start website`

关闭：`systemctl stop website`

重启：`systemctl restart website`

查看状态：`systemctl status website`

修改服务配置重新生效：`systemctl daemon-reload`

常见systemctl错误码
| code  | desc                                                     |
|:------|:---------------------------------------------------------|
| 0     | 命令成功结束                                             |
| 1     | 通用未知错误                                             |
| 2     | 误用shell命令                                            |
| 126   | 命令不可执行                                             |
| 127   | 没找到命令                                               |
| 128   | 无效退出参数                                             |
| 128+x | Linux 信号x的严重错误                                    |
| 130   | Linux 信号2 的严重错误，即命令通过SIGINT（Ctrl＋Ｃ）终止 |
| 203   | 缺失脚本执行器标识                                       |
| 255   | 退出状态码越界                                           |

> 备注  
如运行sh脚本报出如下错误：  
/bin/sh^M: 坏的解释器: 没有那个文件或目录  
是由于在windows下编辑的脚本文件拷贝至linux导致的，windows下编辑文本每一行结尾是\n\r，而Linux下则是\n,  
解决方法：  
在终端输入sed -i 's/\r$//' daemon.sh  
sed -i 's/\r$//' daemon.sh 会把make-all-linux-project.sh中的行尾的\r替换为空白,其中daemon.sh为报错的脚本。  
为避免该错误，可以直接在linux新建脚本或者再linux环境下拷贝。  


