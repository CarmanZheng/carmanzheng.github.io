---
title: Kubernetes
layout: post
tags: DevOps
categories: '边缘计算'
---

### 1.部署方式

**传统部署方式**： 应用直接在物理机上部署，机器的资源分配不好，出现bug时，可能机器的大部分资源被某个应用占用，导致其他应用无法正常运行，无法做到应用隔离

**虚拟机部署**：在单个物理机上运行多个虚拟机， 每个虚拟机都是完整独立的系统，性能损耗大

**容器部署**：所有容器共享主机的系统，相当于轻量级的虚拟机，性能损耗小，资源隔离，CPU内存实现按需分配

### 2.应用场景

**单台机器**：直接使用docker + docker-compose就够了，方便轻松

**多台机器（3-4台）**：单台机器单独配置运行环境+负载均衡器，亦可以轻松化解

**n多台机器（成百上千）**：借助k8s，可实现集中式的管理集群机器和应用，加机器、版本升级、版本回滚、不停机的灰度更新，确保高可用、高性能、高扩展

### 3.Kubernetes集群架构

#### 	master节点

​	主节点：控制平台，不需要很高的性能、不跑任务，通常一个就行了，也可以开多个主节点来提高集群的可用度

#### 	work节点

​	工作节点：可以是物理机或者虚拟机，任务都在这里跑，机器的性能需要好点；通常有很多个，可以不断的加机器扩大集群；每个工作节点都由主节点管理。

#### 	Pod

​	K8S管理和调度的最小单位；每个pod都有自己的虚拟IP。**一个工作节点可以有多个pod**，**一个pod可以包含一个或多个容器**；主节点会考量负载自动调度pod的  哪个节点运行

### 4.集群安装

![](D:\07_GitHub\GitHub_Blogs\carmanzheng.github.io\assets\images\image-20220214091811996.png)