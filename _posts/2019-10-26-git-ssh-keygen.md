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
