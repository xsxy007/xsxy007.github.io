---
layout: post
title: git-ssh-keygen
date: 2019-10-26
tags: git
---

## ssh-keygen

先看本地是否已经有了密钥
```shell
cd ~/.ssh
```

该文件夹下会包含两个文件
id_rsa --私钥
id_rsa.pub  --公钥
如果没有这两个文件的话就需要重新生成（有的话使用一下命令回覆盖原密钥）

1、首先设置用户名和邮箱

```shell
git config --global user.name "******"
git config --global user.email "******"
```

生成ssh密钥

```shell
ssh-keygen -t rsa -C "*****@qq.com"
```

连续回车即可（如果是重新生成的话，中间需要输入`y`）

----

一台电脑生成多个sshkey
现在大多数公司都有 GIT 来管理代码版本控制了，既然用到 GIT，相信大家都接触过 Github、Gitlab、Gitee 这些远程仓库，或者是公司内部自行搭建的 GIT 仓库。  

当用到 SSH 方式来连接 GIT 仓库的时候，难免会同时用到多个仓库，一般生成公私钥的默认配置文件为：  

私钥：C:\Users\xxx.ssh\id_rsa

公钥：C:\Users\xxx.ssh\id_rsa.pub

那么问题来了，我先生成 Github 的，再生成 GitLab 的，那么后面配置的 Gitlab 的公私钥文件会覆盖前面配置 Github 的，从而导致 Github 仓库无法连接。。

这样的配置只能同时连接一种类型的仓库，如何在同一台电脑做到同时连接多个不同的仓库呢？

### 生成多个仓库公钥

1、生成Github的

```shell
ssh-keygen -t rsa -b 4096 -C "****@qq.com" -f ~/.ssh/github_id_rsa
```

2、生成Gitlab的

```shell
ssh-keygen -t rsa -b 4096 -C "***@qq.com" -f ~/.ssh/gitlab_id_rsa
```

如果还有其他仓库类似，用`-f` 来指定不同文件的名称：`xxx_id_rsa`,从而区分不同的仓库

### 将生成的公钥添加到仓库里边

复制`xxx_id_rsa`公钥文件的内容放到对应的仓库里边，以下是Gibhub示例：

![github](/images/posts/2019/git-ssh-keygen.webp.png)

### 添加config配置

在`~/.ssh`目录下建立config文件，添加一下内容：

```
# github
Host github.com
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/github_id_rsa

# gitlab
Host gitlab.com
HostName gitlab.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/gitlab_id_rsa

# 多个依此类推
# ...
```

### 测试连通性
分别测试多个仓库的连通性，验证配置是否生效。

1、测试Github：

$ ssh -T git@github.com

2、测试Gitlab：

$ ssh -T git@gitlab.com

以下是 Github 连通示例：

$ ssh -T git@github.com
Enter passphrase for key '/c/Users/xxx/.ssh/github_id_rsa':
Hi javastacks! You've successfully authenticated, but GitHub does not provide shell access.
这样配置完，我们就能愉快的使用各种不同的仓库了~