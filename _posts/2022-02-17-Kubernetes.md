---
title: Kubernetes
layout: post
tags: DevOps
categories: '边缘计算'
---

### 1.部署方式

|     部署方式     | 说明                                                         |
| :--------------: | :----------------------------------------------------------- |
| **传统部署方式** | 应用直接在物理机上部署，机器的资源分配不好，出现bug时，可能机器的大部分资源被某个应用占用，导致其他应用无法正常运行，无法做到应用隔离 |
|  **虚拟机部署**  | 在单个物理机上运行多个虚拟机， 每个虚拟机都是完整独立的系统，性能损耗大 |
|   **容器部署**   | 所有容器共享主机的系统，相当于轻量级的虚拟机，性能损耗小，资源隔离，CPU内存实现按需分配 |

### 2.应用场景

| 集群规模                  | 部署方式                                                     |
| ------------------------- | ------------------------------------------------------------ |
| **单台机器**              | 直接使用docker + docker-compose就够了，方便轻松              |
| **多台机器（3-4台）**     | 单台机器单独配置运行环境+负载均衡器、，亦可以轻松化解        |
| **n多台机器（成百上千）** | 借助k8s，可实现集中式的管理集群机器和应用，加机器、版本升级、版本回滚、不停机的灰度更新，确保高可用、高性能、高扩展 |

### 3.Kubernetes集群架构说明

| 概念              | 说明                                                         |
| :---------------- | :----------------------------------------------------------- |
| master节点        | 主节点：控制平台，不需要很高的性能、不跑任务，通常一个就行了，也可以开多个主节点来提高集群的可用度 |
| work节点          | 工作节点：可以是物理机或者虚拟机，任务都在这里跑，机器的性能需要好点；通常有很多个，可以不断的加机器扩大集群；每个工作节点都由主节点管理 |
| Pod               | K8S管理和调度的最小单位；每个pod都有自己的虚拟IP。一个工作节点可以有多个pod，一个pod可以包含一个或多个容器；主节点会考量负载自动调度pod的  哪个节点运行 |
| kube-apiserver    | API 服务器，公开了 Kubernetes API                            |
| `etcd` 键值数据库 | 可以作为保存 Kubernetes 所有集群数据的后台数据库             |
| kube-scheduler    | 调度 Pod 到哪个节点运行                                      |
| kube-controller   | 集群控制器                                                   |
| cloud-controller  | 与云服务商交互                                               |

### 4.Kubernetes集群部署方式

https://github.com/lework/kainstall

| K8S部署方式 | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| minikube    | Minicube是一个工具，可以在本地快速运行一个单点的Kubernetes，仅用于尝试kubernetes或日常开发的用户使用 参考 ：`https://minikube.sigs.k8s.io/docs/tutorials/` |
| kubeadm     | Kubeadm也是一个工具，提供`kubeadm init `和`kubeadm join`，用于快速部署`Kubernetes`集群，参考：`https://kubernetes.io/docs/tasks/tools/` |
| 二进制      | 推荐，从官方下载发行版的二进制包，手动部署每个组件，组成k8s集群 |

### 5. Kubeadm搭建

#### 1.系统初始化







### 6.二进制源码搭建





#### 1.系统初始化

```sh
# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld
# 关闭selinux, 所有节点关闭 SELinux
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
# 关闭swap
# 同步服务器时间 (ntpdate time.windows.com),不联网可以自己设定时间，总之保持时间一致
# 更改HOST文件(基于主机名通信，所以要进行IP和主机名的绑定，这个在Master机器上更改就行)
vim /etc/hosts
172.16.32.2 node1
172.16.32.6 node2
172.16.0.4 master
# 每个节点分别设置对应主机名
homenamectl set-homename k8s-master
homenamectl set-homename k8s-node1
homenamectl set-homename k8s-node2
```

#### 2.签发证书

```sh
# 准备cfssl工具
# 自签etcd ssl证书

```

#### 3.下发证书

二进制包下载地址：`https://github.com/etcd-io/etcd/releases`

```sh
# 部署etcd
```
