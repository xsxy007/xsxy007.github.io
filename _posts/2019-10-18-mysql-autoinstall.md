---
layout: post
title: mysql8.0-autoinstall
date: 2019-10-18
tags: sql
---



### 提供一个自动安装mysql8.0的脚本

> 使用时，直接将以下内容复制到一个*.sh文本中，更改可执行权限后执行，按照提示操作

```shell
#!/bin/bash
############   一键安装 MySQL 8.0 脚本  ############
# Updates：可以在安装的时候更改root初始密码        #
#######################  END  ######################

function install(){
	#先把多余的卸载了
	unload
	cd ~
	U_mysqlUrl=https://dev.mysql.com/get/mysql80-community-release-el7-1.noarch.rpm
	#下载MySQL8.0的rpm包
	if [ -e "./mysql80-community-release-el7-1.noarch.rpm" ]
	then
		echo -e "\033[1;32m正在解压~~~\033[0m"
	else
		wget ${U_mysqlUrl}
		if [ $? -ne 0 ]
			then
			echo -e "\033[1;31m下载 MySQL 安装包失败，请检查您的网络！\033[0m"
			exit
		fi
	fi

	#解压rpm包 ##*/  表示删除最后一个/ 及其左边的字符
	rpm -Uvh ${U_mysqlUrl##*/}

	#安装MySQL8.0
	yum install mysql-server -y

	#启动MySQL
	echo -e "                  ************  \033[1;33m正在启动 MySQL\033[0m  *****************"
	systemctl start mysqld.service

	#查看状态
	a=$(systemctl status mysqld.service | grep Active)
	b=${a##*active}
	c=${b%% since*}
	d="(running)"
	if [ "$c" = "$d" ]
	then
		echo -e "             **************  \033[1;32mMySQL 启动成功\033[0m ******************"
	fi

	#找出初始密码，建立登录脚本
	findpass=$(grep 'temporary password' /var/log/mysqld.log)
	initpass=${findpass##* }
	rm -rf ~/.my.cnf
	touch ~/.my.cnf
	echo "[client]" > ~/.my.cnf
	echo "user=root" >> ~/.my.cnf
	echo "password=\"${initpass}\"" >> ~/.my.cnf
	#设置此登录文件只有所有者可以读写
	chmod 770 ~/.my.cnf

	#修改密码
	chose_pass
}

#修改密码
function chose_pass() {
	declare -i settime=0
	while [ ${settime} -lt 3 ]
	do
		read -p "是否现在修改 MySQL 的 root 密码？[y/n] " chose
		if [[ "$chose" = "n" || "$chose" = "N" ]]
		then
			notchk_pass
			exit
		elif [[ "$chose" = "y" || "$chose" = "Y" ]]
		then
			modify_pass
			((settime+=3))
			exit
		else
			echo -e "\033[1;33m请输入 y 或 n\033[0m"
		fi
		settime=$((++settime))			
	done
}
#不修改密码提示函数
function notchk_pass() {
	echo -e "\033[1;34;40mMySQL安装成功！！"
	echo "###############  请在输入命令 mysql 后进入数据库  ######################"
	echo "## 输入语句：ALTER USER 'root'@'localhost' IDENTIFIED BY '新密码';     #"
	echo "## 并在 ~/.my.cnf 文件中更新你的密码，以方便日后免密登录               #"
	echo "########################################################################"
	echo -e "\033[0m"
}

#修改密码操作函数
function modify_pass() {
	#如果密码格式不符合3次则退出
	declare -i chktime=0
	while [ ${chktime} -lt 3 ]
	do
		echo -e "\033[1;32m请输入您的新密码"
		read -p "至少一个大写字母、一个小写字符、数字和符号：" NEWPASS
		echo -e "\033[0m"	
				
		SQL="ALTER USER 'root'@'localhost' IDENTIFIED BY \"${NEWPASS}\";flush privileges;"
		mysql --connect-expired-password -e "${SQL}"
		if [ $? -eq 0 ]
		then
			#修改登录脚本
			rm -rf ~/.my.cnf
			touch ~/.my.cnf
			echo "[client]" > ~/.my.cnf
			echo "user=root" >> ~/.my.cnf
			echo "password=\"${NEWPASS}\"" >> ~/.my.cnf
			chmod 0600 ~/.my.cnf
			
			echo -e "\033[1;34;40mMySQL安装成功！！"
			echo "###############  请在输入命令 mysql 后进入数据库  ######################"
			echo "## 自动登录脚本：~/.my.cnf                                             #"
			echo "########################################################################"
			echo -e "\033[0m"
			rm -rf transfer.txt
			#设置对了直接退出
			((chktime+=4))
			exit
		fi
		chktime=$((++chktime))
		#第三次的时候提示并退出
		if [ ${chktime} -eq 3 ]
		then
			echo -e "\033[1;31m密码修改失败，请自行修改密码\033[0m"
			exit
		fi
	done
}

#卸载
function unload() {
	#停掉 MySQL 数据库
	systemctl stop mysqld.service
	#卸载各项服务
	yum remove mysql mysql-server mysql-libs compat-mysql51 -y
	#删除各个目录
	rm -rf /var/lib/mysql
	rm -rf /etc/my.cnf
	rm -rf /usr/share/mysql-8.0/
	touch list.txt
	#查一下继续卸载
	rpm -qa | grep mysql > list.txt
	for a in `cat list.txt`
	do
		yum remove $a -y
	done
	rm -rf list.txt
	#删除自动登录配置文件
	rm -rf ~/.my.cnf
	#删除日志文件
	rm -rf /var/log/mysqld.log
	#删除安装包
	#rm -rf ~/mysql80-community-release-el7-1.noarch.rpm
}

clear

echo -e "\033[1;36m############  欢迎使用一键安装 MySQL 8.0 脚本  ############\033[0m"
echo -e "\033[1;36m#                                                         #\033[0m"
echo -e "\033[1;36m#                    请保持网络畅通···                    #\033[0m"
echo -e "\033[1;36m#                                                         #\033[0m"
echo -e "\033[1;36m#                    1、安装 MySQL                        #\033[0m"
echo -e "\033[1;36m#                    2、卸载 MySQL                        #\033[0m"
echo -e "\033[1;36m#                    3、退出                              #\033[0m"
echo -e "\033[1;36m#                                                         #\033[0m"
echo -e "\033[1;36m###########################################################\033[0m"

declare -i style
read -p "请输入选项 [ 1 | 2 | 3 ]：" style

case "$style" in
1)
	#检查目录是否存在
	if [ -e "/var/lib/mysql80" ]
	then
		echo -e "\033[1;33m您的系统已安装 MySQL 数据库，请检查目录\033[0m"
	else
	install
	fi
	;;
2)
	unload
	;;
3)
	exit
	;;
*)
	echo -e "\033[5;32;40m参数错误！！！\033[0m"
	;;
esac

```

