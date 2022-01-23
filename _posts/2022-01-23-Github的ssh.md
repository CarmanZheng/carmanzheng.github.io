---
title: Github中的SSH
layout: post
tags: Github
categories: 'IT'
---

本文主要介绍配置多个ssh的案例

同时配置github和gitee远程仓库，命令执行窗口为git；

```sh
# windows用户使用右键文件夹，git bash here
```

step 1 设置全局用户名和用户邮箱，这个就是本地的用户信息

命令如下

```sh
git config --global  user.name "这里换上你的用户名"
git config --global user.email "这里换上你的邮箱"
```

step 2 生成本地私钥和公钥

```sh
# 单个文件生成，没必要配置文件名，使用这个
ssh-keygen -t rsa -C "这里换上你的邮箱"
# 本案例配置多个，所以要使用文件名
ssh-keygen -t rsa -C "这里换上你gitee注册的邮箱" -f ~/.ssh/gitee_id
ssh-keygen -t rsa -C "这里换上你github注册的邮箱" -f ~/.ssh/github_id
```

这个过程中会出现文件报错地址的选择和其他确定的选项，可以直接回车（输入yes的时候，尽量输入一下）

这个过程结束，windows系统可以在`C:\Users\Administrator\.ssh`下，Linux用户可以在`~/.ssh/`目录下看到总共4个文件，分别是

```sh
gitee_id
gitee_id.pub
github_id
github_id.pub
```

step 3 配置文件编写

在这个文件下，添加名为 `config`的文件，文件内容如下

```sh
# gitee
Host gitee.com
HostName gitee.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/gitee_id
# github
Host github.com
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/github_id
```

此时文件夹中就有5个文件了，分别是

```sh
config
gitee_id
gitee_id.pub
github_id
github_id.pub
```

step 4 复制公钥到github/gitee

在github右上角头像下的三角形图标，找到`settings`,打开，找到`SSH and GPG keys`

点击 `new key`，

```sh
# 第一行为名称填写，自己随便取名；
# 第二行为公钥填写，将 github_id.pub 中的内容粘贴在此处，并保存
```

在gitee上，同样右上角头像下的三角形图标，找到设置，进入，点击`安全设置\SSH设置`

```sh
# 将 gitee_id.pub 中的内容粘贴在此处，并保存
# 粘贴后，名字会自己生成，且为自己的邮箱
```

step 5 验证

验证github，在git中输入

```sh
$ ssh -T git@github.com
Hi CarmanZheng! You've successfully authenticated, but GitHub does not provhell access.
```

验证gitee，在git中输入

```sh
# 特别注意，其中有个yes的输入，不要直接敲回车；需要手动输入yes后回车 （坑的很）
$ ssh -T git@gitee.com
The authenticity of host 'gitee.com (180.97.125.228)' can't be established.
ECDSA key fingerprint is SHA256:FQGC9Kn/eye1W8icdBgrQp+KkGYoFgbVr17bmjey0Wc
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'gitee.com,180.97.125.228' (ECDSA) to the list own hosts.
Hi CarmanZheng! You've successfully authenticated, but GITEE.COM does not pe shell access.
```

所有工作完成后，会在`C:\Users\Administrator\.ssh` 或者 `~/.ssh/`目录下生成一个`known_host`的文件，里面记录了主机地址和秘钥

文件夹中文件为6个，分别为

```sh
config
gitee_id
gitee_id.pub
github_id
github_id.pub
known_hosts
```

打完收工!