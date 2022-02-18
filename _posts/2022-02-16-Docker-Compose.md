---
title: Docker-Compose
layout: post
tags: DevOps
categories: '边缘计算'
---

本文主要对docker-compose相关的知识进行归纳，针对docker-compose的yaml文件编写和集群管理进行整理

官网文档：https://docs.docker.com/compose/compose-file/；https://docs.docker.com/compose/

### 1. Docker-compose

Docker-Compose是一个用来定义和运行复杂应用的Docker工具。一个使用Docker容器的应用，通常由多个容器组成。

**使用Docker Compose不再需要使用shell脚本来启动容器**。 

Docker-Compose 通过一个配置文件来管理多个Docker容器，在配置文件中，**所有的容器通过services来定义**，然后使用docker-compose脚本编排容器应用，控制容器的启动、停止和重启以及应用中的服务和所有依赖服务的容器，非常适合组合使用多个容器进行开发的场景。

### 2. Dokcer-compose安装

在安装`docker-compose`时，需要注意版本与`docker`版本的匹配

### 3. 基础使用步骤

首先保证计算机中安装了`Docker`和`Docker-Compose`，然后官网给出了3个步骤：

```sh
1. 在 Dockerfile 中定义 app ，这样可以在任何地方进行app的生成
2. 编写 docker-compose.yml 文件，这样可以在独立环境中启动所有应用
3. 使用 docker-compose up 命令启动所有的app
```

附`docker-compose.yml`示例文件

```yaml
version: "3.9"  # optional since v1.27.0
services:
  web:
    build: .
    ports:
      - "8000:5000"
    volumes:
      - .:/code
      - logvolume01:/var/log
    links:
      - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```

具体操作，按照官方文档

1. 创建项目文件夹

   ```sh
    mkdir composetest
    cd composetest
   ```

2. 创建项目文件`app.py`和依赖`requirements.txt`

   `app.py`

   ```python
   import time
   
   import redis
   from flask import Flask
   
   app = Flask(__name__)
   cache = redis.Redis(host='redis', port=6379)
   
   def get_hit_count():
       retries = 5
       while True:
           try:
               return cache.incr('hits')
           except redis.exceptions.ConnectionError as exc:
               if retries == 0:
                   raise exc
               retries -= 1
               time.sleep(0.5)
   
   @app.route('/')
   def hello():
       count = get_hit_count()
       return 'Hello World! I have been seen {} times.\n'.format(count)
   
   ```

   `requirements.txt`

   ```tex
   flask
   redis
   ```

3. 创建`Dockerfile`和`docker-compose.yml`

   `Dockerfile`

   ```dockerfile
   # syntax=docker/dockerfile:1
   FROM python:3.7-alpine
   WORKDIR /code
   ENV FLASK_APP=app.py
   ENV FLASK_RUN_HOST=0.0.0.0
   COPY requirements.txt requirements.txt
   RUN pip install -r requirements.txt
   EXPOSE 5000
   COPY . .
   CMD ["flask", "run"]
   ```

   `docker-compose.yml`

   ```yaml
   version: "3.5"
   services:
     web:
       build: .
       ports:
         - "8000:5000"
     redis:
       image: "redis"
   ```

   `compose`文件中包含了两个服务`web`和`redis`，其中**`web`服务**中`build`后面的`.`表示使用当前文件夹下的`Dockerfile`创建镜像，端口将镜像内部的`5000`端口映射出来，映射到宿主机的`8000`端口，这样访问宿主机的`8000`端口就可以访问该web服务了。**`redis`服务**直接使用本地的`redis`镜像创建。

4. 创建生成服务

   ```sh
   docker-compose up
   ```

   启动服务后，在宿主机的`http://localhost:8000/`访问服务页面，可以看到

   ```sh
   Hello World! I have been seen 1 times.
   ```

   刷新后，会有数据的增长

   ```sh
   # 停止应用
   docker-compose down
   ```

5. 编辑Compose文件并重生成

   ```yaml
   version: "3.5"
   services:
     web:
       build: .
       ports:
         - "8000:5000"
       volumes:
         - .:/code
       environment:
         FLASK_ENV: development
     redis:
       image: "redis"
   ```

   * 新增一个卷，将项目中的所有内容（宿主机内容）挂载到容器的`/code`文件夹，这样就可以在本地直接编辑容器内的代码了
   * 设置运行环境`environment`，设置`FLASK_ENV`为开发者环境，这样`flask run`就可以重加载（仅用于开发模式）

   ```sh
   docker-compose up
   Recreating composetest_redis_1 ... done
   Recreating composetest_web_1   ... done
   Attaching to composetest_redis_1, composetest_web_1
   redis_1  | 1:C 18 Feb 2022 00:40:52.396 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
   redis_1  | 1:C 18 Feb 2022 00:40:52.396 # Redis version=6.2.6, bits=64, commit=00000000, modified=0, pid=1, just started
   redis_1  | 1:C 18 Feb 2022 00:40:52.396 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
   redis_1  | 1:M 18 Feb 2022 00:40:52.396 * monotonic clock: POSIX clock_gettime
   redis_1  | 1:M 18 Feb 2022 00:40:52.397 * Running mode=standalone, port=6379.
   redis_1  | 1:M 18 Feb 2022 00:40:52.397 # Server initialized
   redis_1  | 1:M 18 Feb 2022 00:40:52.397 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
   redis_1  | 1:M 18 Feb 2022 00:40:52.397 * Loading RDB produced by version 6.2.6
   redis_1  | 1:M 18 Feb 2022 00:40:52.397 * RDB age 4 seconds
   redis_1  | 1:M 18 Feb 2022 00:40:52.397 * RDB memory usage when created 0.79 Mb
   ...
   ```

6. 修改宿主机内容，验证挂载

   `app.py`

   ```sh
   ...
   # 将app.py中最后一句话修改为如下所示
   return 'Hello from Docker! I have been seen {} times.\n'.format(count)
   ```

   刷新页面，可以看到

   ```html
   Hello from Docker! I have been seen 37 times.
   ```

### 4. docker-compose数据卷

