---
title: Docker-Compose
layout: post
tags: DevOps
categories: '边缘计算'
---

### 1. Docker Compose简介

​		Docker Compose是一个用来定义和运行复杂应用的Docker工具。一个使用Docker容器的应用，通常由多个容器组成。使用Docker Compose不再需要使用shell脚本来启动容器。 
​		Docker Compose 通过一个配置文件（yml文件）来管理多个Docker容器，在配置文件中，所有的容器通过services来定义，然后使用docker-compose脚本来启动，停止和重启应用，和应用中的服务以及所有依赖服务的容器，非常适合组合使用多个容器进行开发的场景。
`原文链接：https://blog.csdn.net/pushiqiang/article/details/78682323`

版本需求

| **Compose file format** | **Docker Engine release** |
| :---------------------- | :------------------------ |
| Compose specification   | 19.03.0+                  |
| 3.8                     | 19.03.0+                  |
| 3.7                     | 18.06.0+                  |
| 3.6                     | 18.02.0+                  |
| 3.5                     | 17.12.0+                  |
| 3.4                     | 17.09.0+                  |
| 3.3                     | 17.06.0+                  |
| 3.2                     | 17.04.0+                  |
| 3.1                     | 1.13.1+                   |
| 3.0                     | 1.13.0+                   |
| 2.4                     | 17.12.0+                  |
| 2.3                     | 17.06.0+                  |
| 2.2                     | 1.13.0+                   |
| 2.1                     | 1.12.0+                   |
| 2.0                     | 1.10.0+                   |

### 2. 安装

​	见`Ubuntu装机`专栏的内容

### 3. 使用

​	参考`Nginx`专栏的内容

#### 	1.yml文件格式

```sh
# 1. 基本语法
1、yml文件以缩进代表层级关系
2、缩进不允许使用tab只能使用空格
3、空格的个数不重要，只要相同层级的元素左对齐即可
4、大小写敏感
5、数据格式为，名称:(空格)值
```

```sh
# 数据格式
1、对象：键值对的集合(key:value)
         字符串不用使用双引号或单引号圈起来
         双引号圈住时不会转义字符串中的特殊字符
         单引号圈住时会转义字符串中的特殊字符
         
2、数组：一组按顺序排列的值
         数组名:
              -元素1
              -元素2
         行内写法：
           数组名:[元素1,元素2,元素3]
           
3、字面量：单个的、不可再分的值（数字、字符串、布尔值）
```

#### 2.docker-compose CLI

```SH
Define and run multi-container applications with Docker.  # 基于Docker定义和运行多容器

Usage:
  docker-compose [-f <arg>...] [--profile <name>...] [options] [COMMAND] [ARGS...]
  docker-compose -h|--help

Options:
  -f, --file FILE             Specify an alternate compose file    # 指定运行compose文件
                              (default: docker-compose.yml)
  -p, --project-name NAME     Specify an alternate project name   # 指定项目名
                              (default: directory name)
  --profile NAME              Specify a profile to enable
  --verbose                   Show more output
  --log-level LEVEL           DEPRECATED and not working from 2.0 - Set log level (DEBUG, INFO, WARNING, ERROR, CRITICAL)
  --no-ansi                   Do not print ANSI control characters
  -v, --version               Print version and exit
  -H, --host HOST             Daemon socket to connect to

  --tls                       Use TLS; implied by --tlsverify
  --tlscacert CA_PATH         Trust certs signed only by this CA
  --tlscert CLIENT_CERT_PATH  Path to TLS certificate file
  --tlskey TLS_KEY_PATH       Path to TLS key file
  --tlsverify                 Use TLS and verify the remote
  --skip-hostname-check       Don't check the daemon's hostname against the
                              name specified in the client certificate
  --project-directory PATH    Specify an alternate working directory
                              (default: the path of the Compose file)
  --compatibility             If set, Compose will attempt to convert deploy
                              keys in v3 files to their non-Swarm equivalent

Commands:
  build              Build or rebuild services
  bundle             Generate a Docker bundle from the Compose file   
  config             Validate and view the Compose file
  create             Create services
  down               Stop and remove containers, networks, images, and volumes
  events             Receive real time events from containers
  exec               Execute a command in a running container
  help               Get help on a command
  images             List images
  kill               Kill containers
  logs               View output from containers
  pause              Pause services
  port               Print the public port for a port binding
  ps                 List containers
  pull               Pull service images
  push               Push service images
  restart            Restart services
  rm                 Remove stopped containers
  run                Run a one-off command
  scale              Set number of containers for a service
  start              Start services
  stop               Stop services
  top                Display the running processes
  unpause            Unpause services
  up                 Create and start containers
  version            Show the Docker-Compose version information
```

常用命令

```sh
docker-compose up -d 			# 后台运行,拉起服务
dokcer-compose down 			# 停止服务，并移除容器、网络、镜像和挂载卷
docker-compose version 			# 查看版本
docker-compose start [服务名]	  # 启动服务
docker-compose stop [服务名]	   # 停止服务
docker-compose top				# 查看运行的进程
docker-compose logs [服务名]	  # 查看日志
```

#### 3.环境变量

​		和`docker`一样，`docker-compose`环境变量写在同目录下的`.env`文件中。也可以写在别的目录中，启动时候通过`--env-file`指定路径，一般写在同一目录下。在`docker-compose.yml`文件中通过`${变量名}`引用变量。

`.env`

```sh
TAG=v1.5
```

`docker-compose.yml`

```sh
version:'3'
services:
	web:
	   images:"webapp:${TAG}"
```

```sh
$ docker config
version:'3'
services:
	web:
	   images:"webapp:v1.5"
```

