---
title: Ubuntu装机
layout: post
tags: Linux基础
categories: 'Linux'
---

本文主要介绍ubuntu版本下的系统常用软件的装机，所有代码均写为sh命令，方便一键解决

### 1. 镜像源修改

`alisource.list`

```sh
# ubuntu 20.04(focal) 
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
```

`mod_source.sh`

```sh
#!bin/bash
sourceFileBak = "/etc/apt/sources.list.bak"
if [ ! -f "$sourceFileBak"]; then
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
fi
sudo cp ./alisource.list /etc/apt/sources.list
sudo apt-get -y update
sudo apt-get -y upgrade
```

将上述两个文件放在同一文件夹下，执行`mod_source.sh`即可将ubuntu 20.04(focal) 系统源更换为阿里源

### 2. docker 安装

https://developer.aliyun.com/mirror/?spm=a2c6h.13651102.0.0.77cd1b11h9CKdC&serviceType=mirror

`daemon.json`

```sh
{
    "registry-mirrors" : [
    "https://registry.docker-cn.com",
    "https://docker.mirrors.ustc.edu.cn",
    "http://hub-mirror.c.163.com",
    "https://cr.console.aliyun.com/"
  ]
}
```

`docker_install.sh`

```sh
#!bin/bash
# uninstall older versions
sudo apt-get remove docker docker-engine docker.io containerd runc
sudo apt-get update
# step 1: 安装必要的一些系统工具
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
# step 2: 安装GPG证书
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
# Step 3: 写入软件源信息
sudo add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
# Step 4: 更新并安装Docker-CE
sudo apt-get -y update
sudo apt-get -y install docker-ce
# Step 5: 更改镜像源为阿里源
sudo cp ./daemon.json  /etc/docker/daemon.json

# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# apt-cache madison docker-ce
#   docker-ce | 17.03.1~ce-0~ubuntu-xenial | https://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
#   docker-ce | 17.03.0~ce-0~ubuntu-xenial | https://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
# Step 2: 安装指定版本的Docker-CE: (VERSION例如上面的17.03.1~ce-0~ubuntu-xenial)
# sudo apt-get -y install docker-ce=[VERSION]
```

#### 3.docker-compose

```sh
#!bin/bash
# 需要切换到root状态下
curl -L https://get.daocloud.io/docker/compose/releases/download/1.24.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
docker-compose --verison 
```

### 4. kubernetes安装

`k8s_install.sh`

```sh
#!bin/bash
# 需要切换到root状态下
apt-get update && apt-get install -y apt-transport-https
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
```

未完待补充...