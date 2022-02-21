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

供参考的`dokcer-compose.yml`文件，配置项参考 https://docs.docker.com/compose/compose-file/compose-file-v3/#volume-configuration-reference

```yaml
version: "3.9"
services:

  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        max_replicas_per_node: 1
        constraints:
          - "node.role==manager"

  vote:
    image: dockersamples/examplevotingapp_vote:before
    ports:
      - "5000:80"
    networks:
      - frontend
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
      restart_policy:
        condition: on-failure

  result:
    image: dockersamples/examplevotingapp_result:before
    ports:
      - "5001:80"
    networks:
      - backend
    depends_on:
      - db
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 1
      labels: [APP=VOTING]
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints:
          - "node.role==manager"

  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints:
          - "node.role==manager"

networks:
  frontend:
  backend:

volumes:
  db-data:
```

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

1. #### `.env`设置

   在`docker-compose.yml`文件中，可以使用环境变量。这些环境变量定义在与`yaml`文件同文件夹下的`.env`文件中。

   举例说明

   `.env`

   ```sh
   TAG=v1.5
   ```

   `docker-compose.yml`

   ```yaml
   version: '3'
   services: 
   	web:
   		image: "webapp:$(TAG)"
   ```

   使用`docker-compose config`查看

   ```sh
   version: '3'
   services: 
   	web:
   		image: "webapp:v1.5"
   ```

   除此之外，还可以在`docker-compose.yml`文件中指定环境变量文件，具体参考：https://docs.docker.com/compose/environment-variables/

   ```yaml
   web:
     env_file:
       - web-variables.env
   ```

2. #### 数据持久化

   在docker中，数据持久化的方法有三种：`volumes（数据卷）`、`bind mount`和`tmpfs`

   ![types of mounts and where they live on the Docker host](../../assets/images/20220218docker-compose/types-of-mounts.png)

   三种方式当中，最优先推荐也最常用的方式是`volumes（数据卷）`，简单介绍几种的区别：

   `Volumes`：将docker容器中的数据持久化保存到宿主机文件系统中，Linux系统中保存到`/var/lib/docker/volumes/`,非docker进行不能修改这部分文件

   `Bind mounts`：能够保存docker容器中的文件到宿主机文件系统中的任意位置，非docker进程可以随意修改该部分文件

   `Tmpfs mounts`：保存数据到宿主机的内存中，不能够保存数据到宿主机文件系统中，不能持久化保存数据

   ```sh
   # 使用 volume 和 bind mount 需要注意的地方
   1. 挂载空卷（宿主机）--> 容器（有文件和文件夹），那么这些文件和文件夹会被复制--> 宿主机空卷
   2. 挂载数据卷（宿主机，但该卷不存在于宿主机）--> 容器，这个空卷将被创建（宿主机）
   3. 挂载数据卷（宿主机）-->容器特定文件夹，那么容器特定文件夹中的内容会被覆盖但不会删除和被修改，直到该数据卷卸载(相当于U盘挂载到Linux系统)
   ```

   ```yaml
volumes:
     # Just specify a path and let the Engine create a volume
     - /var/lib/mysql
   
     # Specify an absolute path mapping
     - /opt/data:/var/lib/mysql
   
     # Path on the host, relative to the Compose file
     - ./cache:/tmp/cache
   
    # User-relative path
   	 - ~/configs:/etc/configs/:ro
   
    # Named volume
    - datavolume:/var/lib/mysql
   ```
   
     下面着重记录`Volumes`数据持久化，以`mysql`为例：
   
   ```sh
      文件目录 参考链接：https://github.com/treetips/docker-compose-all-mysql，仅供参考，尝试了下，感觉无用
      mysql
      	|------ docker-compose.yml
      	|------ mysql
      			|----- config
      					|----- mysqld.cnf
      			|----- data			
   ```
   
   
   
      `docker-compose.yml`
   
   ```yaml
   version: "3.7"
      services:
        mysql:
          container_name: mysql
          image: mysql:8.0                            #从私有仓库拉镜像
          restart: always     
          command: --default-authentication-plugin=mysql_native_password #这行代码解决无法访问的问题
          volumes:
            - ./mysql/data/:/var/lib/mysql/                            #映射mysql的数据目录到宿主机，保存数据
            - ./mysql/config/mysqld.cnf:/etc/mysql/mysql.conf.d/mysqld.cnf   #把mysql的配置文件映射到容器的相应目录
          ports:
             - "3305:3306"
          environment:
            - MYSQL_ROOT_PASSWORD=123456
            - LANG=C.UTF-8
   ```
   
   此时运行`docker-compose up -d`，启动服务，可以看到在`./mysql/data`文件夹下有数据库中的数据被挂载到本地
   
   如果需要通过`Navicat`远程连接到这个mysql，需要等几分钟才行

3. 注意事项

   ```yaml
   # 外部卷
   # 如果设置为true，则指定该卷已在Compose外部创建。 docker-compose up不会尝试创建它，并且如果它不存在将会引发一个错误。
   version: '2'
   
   services:
     db:
       image: postgres
       volumes:
         - data:/var/lib/postgresql/data     
   
   volumes:
     data:							# 这里的data必须是已经在外部创建好的卷，不然external为true不能用，就出现错误
       external: true
   ```


### 5. docker-compose网络

​	docker-compose基于docker的网络，启动后会默认创建网络，通过`docker network ls`查看

```sh
docker network ls
NETWORK ID     NAME                               DRIVER    SCOPE
88d2242caf7b   bridge                             bridge    local
147d30941ecb   composetest_default                bridge    local
e2523367faaa   docker-compose-all-mysql_default   bridge    local
db2e9c36669e   host                               host      local
d7e251a21e14   mysql_default                      bridge    local
7016f0320535   none                               null      local
```

1. 使用已存在的网络

   ```yaml
   version: '2'
   
   services:
     proxy:
       build: ./proxy
       networks:
         - outside
         - default
     app:
       build: ./app
       networks:
         - default
   
   networks:
     outside:			# 这里的outside网络必须是已经在外部创建好的卷，不然external为true不能用，就出现错误
       external: true
   ```

2. 创建指定网络

   ```yaml
   version: '2.1'
   services:
     app:
       image: busybox
       networks:		#  服务级别(service-level)的networks用来定义网络名字的列表，供顶层级别的networks参考配置
         - app_net
   
   networks:            # 顶层级别(top-level)的networks关键字用来指定自定义网络，用于创建复杂的网络
     app_net:
       driver: bridge   # 指定网络模式为bridge
   ```

3. 指定默认网络

   Instead of (or as well as) specifying your own networks, you can also change the settings of the app-wide default network by defining an entry under `networks` named `default`:

   ```yaml
   version: "3.9"
   services:
     web:
       build: .
       ports:
         - "8000:8000"
     db:
       image: postgres
   
   networks:
     default:
       # Use a custom driver
       driver: custom-driver-1
   ```

   ```yaml
   # 将容器加入外部预定网络
   services:
     # ...
   networks:
     default:
       external: true
       name: my-pre-existing-network
   ```

   



