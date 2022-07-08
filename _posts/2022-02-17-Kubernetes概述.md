---
title: Kubernetes
layout: post
tags: DevOps
categories: '边缘计算'
---

​		K8s是谷歌在2014年开源的容器化集群管理系统，使用k8s进行容器化应用部署。k8s利于应用的扩展，让容器化应用更加简洁高效。

1.自动装箱

​	基于容器对应用运行环境的资源配置要求，自动部署应用容器

2.自我修复

​	当容器失败时，会对容器进行重启

​	当所部署的Node节点有问题时，会对容器进行重新部署和重新调度

​	当容器未通过监控检查时，会关闭此容器；直到容器正常运行时，才对外提供服务

3.水平扩展

​	通过简单的命令、用户UI界面或基于CPU等资源的使用情况，对应用容器进行规模化扩大或规模剪裁

4.滚动更新

​	可以根据应用的变化，对运行的应用进行一次性或者批量式更新

5.版本回退

​	根据应用部署情况，进行历史版本的即时回退

6.秘钥和配置管理

​	在不需要重新构建镜像的情况下，可以部署和更新秘钥和应用配置，类似热部署

7.存储编排

​	自动实现存储系统挂载和应用，特别对有状态应用实现数据持久化非常重要。存储系统可以来自本地目录、网络存储（NFS、Gluster、Ceph等）、公共云存储服务

8.批处理

​	提供一次性任务、定时任务；满足批量数据处理和分析的场景。

### 1.部署方式

|    部署方 式     | 说明                                                         |
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

<img src="D:\07_GitHub\GitHub_Blogs\assets\images\20220217K8Sprofile\image-20220620205346099.png" alt="image-20220620205346099" style="zoom:67%;" />

|        概念        | 说明                                                         |
| :----------------: | ------------------------------------------------------------ |
|   `master node`    | 主节点：控制平台，不需要很高的性能、不跑任务，通常一个就行了，也可以开多个主节点来提高集群的可用度 |
|  `kube-apiserver`  | API 服务器，公开了 Kubernetes API；集群同一入口，以restful方式，交给etcd存储 |
|       `etcd`       | 存储系统，保存 Kubernetes 所有集群数据的后台数据库           |
|  `kube-scheduler`  | 节点调度，调度 Pod 到哪个节点运行                            |
| `kube-controller`  | 集群控制器，处理集群中常规后台任务，一个资源对应一个控制器   |
| `cloud-controller` | 与云服务商交互                                               |

|     概念      | 说明                                                         |
| :-----------: | ------------------------------------------------------------ |
| `worker node` | 工作节点：可以是物理机或者虚拟机，任务都在这里跑，机器的性能需要好点；通常有很多个，可以不断的加机器扩大集群；每个工作节点都由主节点管理 |
|     `Pod`     | K8S管理和调度的最小单位；`每个pod都有自己的虚拟IP`。Pod内部容器共享同一IP；一个工作节点可以有多个pod，一个pod可以包含一个或多个容器；主节点会考量负载自动调度pod的运行位置； |
|   `kublet`    | master排到node的节点代表，管理本机容器                       |
| `kube-proxy`  | 提供网络代理，负载均衡等操作                                 |

### 4.Kubernetes集群部署方式

<img src="D:\07_GitHub\GitHub_Blogs\assets\images\20220217K8Sprofile\04-第二部分 k8s集群搭建（平台规划和搭建方式介绍）.png" style="zoom: 50%;" />

https://github.com/lework/kainstall

| K8S部署方式 |                             说明                             |                             特点                             |
| :---------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| `minikube`  | Minicube是一个工具，可以在本地快速运行一个单点的Kubernetes，仅用于尝试kubernetes或日常开发的用户使用 参考 ：`https://minikube.sigs.k8s.io/docs/tutorials/` |                                                              |
|  `kubeadm`  | Kubeadm也是一个工具，提供`kubeadm init `和`kubeadm join`，用于快速部署`Kubernetes`集群，参考：`https://kubernetes.io/docs/tasks/tools/` |                  快速，但是产生错误不好排除                  |
|  `二进制`   | 推荐，从官方下载发行版的二进制包，手动部署每个组件，组成k8s集群 | 手动，配置过程较为复杂，但是出现问题能够良好的排除，且能了解K8S的详细流程 |

### 5.Kubeadm搭建

​		kubeadm是官方社区推出的一个用于快速部署k8s集群的工具，这个工具能通过两条指令完成一个k8s集群的部署。

第一步，创建一个Master节点 `kubeadm init`

第二步，将Node节点加入到已创建的Master节点中 `kubeadm join Master的ip和端口`

#### 1.系统初始化

* 一台或者多台机器，操作系统centos7以上版本
* 硬件配置：RAM至少2GB，CPU至少2核，硬盘至少30GB
* 集群中所有机器之间网络互通
* 可以访问外网（需要拉取镜像）
* 进制swap分区

`每台机器执行`

```sh
# 关闭防火墙
systemctl stop firewall
systemctl disable firewall

# 关闭selinux
sed -i 's/enforcing/disable' /etc/selinux/config # 永久关闭
setenforce 0 # 临时关闭

# 关闭swap
sed -ri 's/.*swap.*/%&/' /etc/fstab #永久关闭
swapoff -a # 临时关闭
```

`三台机器分别执行`

```sh
# 根据规划设置主机名
hostnamectl set-hostname k8smaster
hostnamectl set-hostname k8snode1
hostnamectl set-hostname k8snode2

# 将桥接的IPV4流量传递到iptables的链路
cat /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1 
net.bridge.bridge-nf-call-iptables = 1
EOF 

system --system   # 生效

# 时间同步
yum install ntpdate -y
ntpdate time.windows.com
```

`在master主机上添加`

```sh
cat >> /etc/hosts << EOF
192.168.44.146 k8smaster
192.168.44.147 k8snode1
192.168.44.148 k8snode2
EOF
```

#### 2.所有节点安装软件

所有节点都需要安装Docker、kubeadm和kubelet

`三台机器操作`

```sh
# （1）安装 Docker
$ wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O
/etc/yum.repos.d/docker-ce.repo
$ yum -y install docker-ce-18.06.1.ce-3.el7
$ systemctl enable docker && systemctl start docker
$ docker --version

# （2）添加阿里云 YUM 软件源
# 设置仓库地址
$ cat > /etc/docker/daemon.json << EOF
{
"registry-mirrors": ["https://b9pmyelo.mirror.aliyuncs.com"]
}
EOF
# 重启docker
$ systemctl restart docker

# 设置yum源
$ cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernete
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

#（3）安装 kubeadm，kubelet 和 kubect
$ yum install -y kubelet kubeadm kubectl
$ systemctl enable kubelet
```

#### 3.部署k8s Master 和 Node

在192.168.44.146（master节点）上执行

```sh
$ kubeadm init \
--apiserver-advertise-address=192.168.44.146 \   # 与master节点ip对应
--image-repository registry.aliyuncs.com/google_containers \  # 镜像仓库
--kubernetes-version v1.17.0 \   # 填写当前k8s版本
--service-cidr=10.96.0.0/12 \    # 填写与当前ip不同的IP段
--pod-network-cidr=10.244.0.0/16 # 填写与当前ip不同的IP段
```

这个过程使用docker拉取官方镜像，由于默认拉取镜像地址 k8s.gcr.io 国内无法访问，这里指定阿里云镜像仓库地址（即aliyuncs.com/google_containers仓库）。

后面就会出现拉取成功的提示

```sh
Your Kubernetes control-plane has initialized successfully!
....
提示语
....
You should ... 
Then ...(这个后面的提示在node节点执行)
```

`使用kubectl创建,在master节点(You should 后面出现的话)`

```sh
$ mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

`在node节点(出现在提示中then后面的话)`

向集群添加新节点，执行在 kubeadm init 输出的 kubeadm join命令

```sh
$ kubeadm join 192.168.44.146:6443 --token esce21.q6hetwm8si29qxwn \
--discovery-token-ca-cert-hash
sha256:00603a05805807501d7181c3d60b478788408cfe6cedefedb1f97569708be9c5
```

加入完之后，在master中查看节点

```sh
$ kubectl get nodes
```

#### 4.安装Pod网络插件（CNI）

`在master节点`

```sh
$ kubectl apply –f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
$ kubectl get pods -n kube-system
```

`https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml`如下内容

```yaml
---
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: psp.flannel.unprivileged
  annotations:
    seccomp.security.alpha.kubernetes.io/allowedProfileNames: docker/default
    seccomp.security.alpha.kubernetes.io/defaultProfileName: docker/default
    apparmor.security.beta.kubernetes.io/allowedProfileNames: runtime/default
    apparmor.security.beta.kubernetes.io/defaultProfileName: runtime/default
spec:
  privileged: false
  volumes:
  - configMap
  - secret
  - emptyDir
  - hostPath
  allowedHostPaths:
  - pathPrefix: "/etc/cni/net.d"
  - pathPrefix: "/etc/kube-flannel"
  - pathPrefix: "/run/flannel"
  readOnlyRootFilesystem: false
  # Users and groups
  runAsUser:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  # Privilege Escalation
  allowPrivilegeEscalation: false
  defaultAllowPrivilegeEscalation: false
  # Capabilities
  allowedCapabilities: ['NET_ADMIN', 'NET_RAW']
  defaultAddCapabilities: []
  requiredDropCapabilities: []
  # Host namespaces
  hostPID: false
  hostIPC: false
  hostNetwork: true
  hostPorts:
  - min: 0
    max: 65535
  # SELinux
  seLinux:
    # SELinux is unused in CaaSP
    rule: 'RunAsAny'
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: flannel
rules:
- apiGroups: ['extensions']
  resources: ['podsecuritypolicies']
  verbs: ['use']
  resourceNames: ['psp.flannel.unprivileged']
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - nodes/status
  verbs:
  - patch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: flannel
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: flannel
subjects:
- kind: ServiceAccount
  name: flannel
  namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: flannel
  namespace: kube-system
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: kube-flannel-cfg
  namespace: kube-system
  labels:
    tier: node
    app: flannel
data:
  cni-conf.json: |
    {
      "name": "cbr0",
      "cniVersion": "0.3.1",
      "plugins": [
        {
          "type": "flannel",
          "delegate": {
            "hairpinMode": true,
            "isDefaultGateway": true
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        }
      ]
    }
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-flannel-ds
  namespace: kube-system
  labels:
    tier: node
    app: flannel
spec:
  selector:
    matchLabels:
      app: flannel
  template:
    metadata:
      labels:
        tier: node
        app: flannel
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/os
                operator: In
                values:
                - linux
      hostNetwork: true
      priorityClassName: system-node-critical
      tolerations:
      - operator: Exists
        effect: NoSchedule
      serviceAccountName: flannel
      initContainers:
      - name: install-cni-plugin
       #image: flannelcni/flannel-cni-plugin:v1.1.0 for ppc64le and mips64le (dockerhub limitations may apply)
        image: rancher/mirrored-flannelcni-flannel-cni-plugin:v1.1.0
        command:
        - cp
        args:
        - -f
        - /flannel
        - /opt/cni/bin/flannel
        volumeMounts:
        - name: cni-plugin
          mountPath: /opt/cni/bin
      - name: install-cni
       #image: flannelcni/flannel:v0.18.1 for ppc64le and mips64le (dockerhub limitations may apply)
        image: rancher/mirrored-flannelcni-flannel:v0.18.1
        command:
        - cp
        args:
        - -f
        - /etc/kube-flannel/cni-conf.json
        - /etc/cni/net.d/10-flannel.conflist
        volumeMounts:
        - name: cni
          mountPath: /etc/cni/net.d
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      containers:
      - name: kube-flannel
       #image: flannelcni/flannel:v0.18.1 for ppc64le and mips64le (dockerhub limitations may apply)
        image: rancher/mirrored-flannelcni-flannel:v0.18.1
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        resources:
          requests:
            cpu: "100m"
            memory: "50Mi"
          limits:
            cpu: "100m"
            memory: "50Mi"
        securityContext:
          privileged: false
          capabilities:
            add: ["NET_ADMIN", "NET_RAW"]
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: EVENT_QUEUE_DEPTH
          value: "5000"
        volumeMounts:
        - name: run
          mountPath: /run/flannel
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
        - name: xtables-lock
          mountPath: /run/xtables.lock
      volumes:
      - name: run
        hostPath:
          path: /run/flannel
      - name: cni-plugin
        hostPath:
          path: /opt/cni/bin
      - name: cni
        hostPath:
          path: /etc/cni/net.d
      - name: flannel-cfg
        configMap:
          name: kube-flannel-cfg
      - name: xtables-lock
        hostPath:
          path: /run/xtables.lock
          type: FileOrCreate
```

#### 5.测试 kubernetes 集群

在 Kubernetes 集群中创建一个 pod，验证是否正常运行：

```sh
$ kubectl create deployment nginx --image=nginx
$ kubectl expose deployment nginx --port=80 --type=NodePort # NodPort需要定义,此时为本pod的ip
$ kubectl get pod
```

访问地址：http://NodeIP:Port，在master节点上此时就是 `192.168.44.146:32753`,其中端口需要看上面命令(get pod)给出来的暴露端口

### 6.二进制源码搭建

#### 1.系统初始化

<img src="D:\07_GitHub\GitHub_Blogs\assets\images\20220217K8Sprofile\06-第二部分 搭建k8s集群（二进制方式）.png" style="zoom: 67%;" />

* 一台或者多台机器，操作系统centos7以上版本
* 硬件配置：RAM至少2GB，CPU至少2核，硬盘至少30GB
* 集群中所有机器之间网络互通
* 可以访问外网（需要拉取镜像）
* 进制swap分区

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
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
chmod +x cfssl_linux-amd64 cfssljson_linux-amd64 cfssl-certinfo_linux-amd64
mv cfssl_linux-amd64 /usr/local/bin/cfssl
mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
mv cfssl-certinfo_linux-amd64 /usr/bin/cfssl-certinfo
```

#### 3.下发证书

二进制包下载地址：`https://github.com/etcd-io/etcd/releases`

```sh
# 部署etcd
```

