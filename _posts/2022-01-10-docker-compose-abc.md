---
title: Docker Compose 入门
key: 2022-01-10
tags: docker-compose
---



Docker Compose 是一个用来定义和运行复杂应用的 Docker 工具。一个使用 Docker 容器的应用，通常由多个容器组成。使用 Docker Compose 不再需要使用 shel l脚本来启动容器。 

Docker Compose 通过一个配置文件来管理多个 Docker 容器，在配置文件中，所有的容器通过 services 来定义，然后使用 `docker-compose` 脚本来启动、停止和重启应用，和应用中的服务以及所有依赖服务的容器，非常适合组合使用多个容器进行开发的场景。

<!--more-->

## Docker Compose 和 Docker 的兼容性

| compose文件格式版本 | docker版本 |
| ------------------- | ---------- |
| 3.4                 | 17.09.0+   |
| 3.3                 | 17.06.0+   |
| 3.2                 | 17.04.0+   |
| 3.1                 | 1.13.1+    |
| 3.0                 | 1.13.0+    |
| 2.3                 | 17.06.0+   |
| 2.2                 | 1.13.0+    |
| 2.1                 | 1.12.0+    |
| 2.0                 | 1.10.0+    |
| 1.0                 | 1.9.1.+    |

Docker 版本变化说明：

Docker 从 1.13.x 版本开始，版本分为企业版 EE 和社区版 CE，版本号也改为按照时间线来发布，比如 17.03 就是 2017 年 3 月。

Docker 的 linux 发行版的软件仓库从以前的 https://apt.dockerproject.org 和 https://yum.dockerproject.org 变更为目前的 https://download.docker.com, 软件包名字改为 docker-ce 和 docker-ee。




## 安装

安装文档： [https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose --version
```



## docker-compose 文件结构示例



docker-compose.yml

```yaml
version: "3"
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
    image: postgres:9.5
    values:
      - db-data:/var/lib/postgresql/data
    networks:
      -backend
    deploy:
      placement:
        constraints: [node.role == manager]
  vote:
    image: dockersamples/examplevotingapp_vote:before
    ports:
      - 5000:80
    networks:
      - fronted
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
      - 5001:80
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
      labels: [app=voting]
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints: [node.role=manager]
        
  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]
        
networks:
  frontend:
  backend:
  
volumes:
  db-data:
```



## docker-compose 使用

通过 docker-compose 构建一个在 docker 中运行的基于 python flask 框架的 web 应用。



### 1， 定义 python 应用

#### 1.1 创建工程目录

```bash
mkdir compose_test
cd compose_test
mkdir src
mkdir docker
touch Dockerfile
touch docker/docker-compose.yml
touch src/app.py
touch src/requirements.txt
```



#### 1.2 编写 app.py 文件

```python
from flask import Flask
from redis import Redis

app = Flask(__name__)
redis = Redis(host='redis', port=6379)

@app.route("/")
def hello():
  count = redis.incr("hits")
  return 'hello world! I have been seen {} times.\n'.format(count)

if __name__ == "__main__":
  app.run(host="0.0.0.0", debug=True)
```



#### 1.3 编写 requirements.txt

```sh
flask
redis
```



#### 1.4 编写 Dockerfile 文件

```dockerfile
FROM python:3.7

COPY src /opt/src
WORKDIR /opt/src

RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```



#### 1.5 编写 docker-compose 文件

```yaml
version: "3"
services:
  web:
    build: ../
    ports:
      - "5000:5000"
  redis:
    image: redis:3.0.7
```



这个 compose 文件定义了两个服务，web 和 redis 两个容器。

##### web 容器

- 使用当前 docker-compose.yml 文件所在的目录的上级目录中的 Dockerfile 构建。
- 将容器的 5000 端口映射到主机的 5000 端口

##### Redis 容器

- 使用官方的 redis 镜像 3.0.7 版本。



#### 运行



```bash
cd docker
docker-compose -f docker-compose.yml up -d

```



#### 编辑 compose 文件以添加文件绑定挂载

上面的代码是在构建时静态复制到容器中的，即通过 Dockerfile 文件中的 `COPY src /opt/src` 命令实现物理主机中的源码复制到容器中，这样在后续物理主机 src 目录中代码的更改不会反应到容器中。 可以通过 `volumes `关键字实现物理主机目录挂载到容器中的功能

**同时删除 Dockerfile 中的 COPY 指令，不需要创建镜像时将代码打包进镜像，而是通过 volums 动态挂载，容器和物理 host 共享数据卷**

后来我事件证明挂载的 valumes 在 RUN 命令执行的时候找不到 requirements.txt 文件。

Dockerfile:

```dockerfile
FROM python:3.7

#COPY src /opt/src
COPY src/requirements.txt /opt/
WORKDIR /opt/src

RUN cd ../ ; pip install -r requirements.txt; cd /opt/src
CMD ["python", "app.py"]
```



docker-compose.yml:

```yaml
version: "3"
services:
  web:
    build: ../
    ports:
      - "5000:5000"
    volumes:
      - ../src:/opt/src
  redis:
    image: redis:3.0.7
```

然后重新运行：

```bash
## 构建和启动
docker-compose -f docker-compose.yml up -d
## 重启
docker-compose -f docker-compose.yml restart
## 一次性运行
## 看一下环境变量
docker-compsoe run web env
## 停止
docker-compose stop
```



## 常用服务配置参考



docker compose 文件是一个用来定义服务、网络和卷的 yaml 文件。默认的文件名是 docker-compose.yml 。

与 Docker 运行一样，默认情况下，Dockerfile 中指定的选项（例如 CMD, EXPOSE, VOLUME, ENV) 都被遵守，不需要在 docker-compose.yml 中在此指定它们。

同时，可以使用类似 BASH 的 `${VARIABLE}` 语法在配置文件中使用环境变量。

> 有关详细信息，请参阅[变量替换](https://docs.docker.com/compose/compose-file/#variable-substitution)。

本节包含版本3中服务定义支持的所有配置选项。

### build

build 可以指定包含构建上下文的路径：

```yaml
version: '2'
services:
  webapp:
    build: ./dir
```

或者，作为一个对象，该对象具有上下文路径和制定的 Dockerfile 文件以及 args 参数值：

```yaml
version: '2'
services:
  webapp:
    build:
      context: ./dir
      dockerfile: DockerFile-alternate
      args:
        buildno: 1
```

webapp服务将会通过./dir目录下的Dockerfile-alternate文件构建容器镜像。 

如果你同时指定了 `image` 和 `build`，则 compose 会通过 `build` 指定的目录构建容器镜像，而构建的镜像名为 `image` 中指定的镜像名和标签。

```yaml
build: ./dir
image: webapp:tag
```



#### context

包含 Dockerfile 文件的目录路径，红哦这是 git 仓库的 URL。

当提供的值是相对路径时，会被解释为相对当前 compose 文件的位置，该目录也是发送到 Docker 守护进程构建镜像的上下文。

#### dockerfile

备用 Docker 文件。 Compose 将使用备用文件来构建，还必须制定构建路径。

#### args

添加构建镜像的参数，环境变量只能在构建过程中访问。

首先，在 Dockerfile 中指定要使用的参数：

```dockerfile
ARG buildno
ARG password

RUN echo "Build number: $buildno"
RUN script-requiring-password.sh "$password"
```

> 注意： yaml 中的 布尔值 (true, false, yes, no, on, off) 都必须用引号括起来，以便解析器将它们解析为字符串

#### image

制定启动容器的镜像，可以是镜像仓库/标签或者镜像 ID（或者ID 的前一部分）

```yaml
image: redis
image: ubuntu:14.04
image: tutum/influxdb
image: example-registry.com:4000/postgresql
image: a4bcd65fd
```

如果镜像不存在，Compose 将尝试从官方镜像仓库将其 pull 下来，如果你还指定了 build，在这种情况下，它将使用指定的 build 选项构建它，并使用 image 指定的名字和标记对其进行标记（前面说过了)。

#### container_name

指定一个自定义容器的名称，而不是生成的默认名称。

```yaml
container_name: my-web-container
```

> 由于 Docker 容器的名称必须是唯一的，因此如果指定了自定义名称，则无法将服务扩展到多个容器。

#### volumes

卷挂在路径设置

可以设置宿主机路径（HOST:CONTAINER) 或者加上访问模式（HOST:CONTAINER:ro），挂在数据卷的默认权限是读写`rw`，可以通过 `ro` 指定为只读。

可以在主机上挂载相对路径，该路径将相对于当前正在使用的 compose 配置文件的目录进行扩展。相对路径使用以 `.` 或者 `..` 开始。

```yaml
volumes:
  # 只指定一个路径，让引擎创建一个卷
  - /var/lib/mysql
  # 指定绝对路径映射
  - /opt/data:/var/lib/mysql
  # 用户主目录相对路径
  - ~/configs:/etc/configs/:ro
  # 命名卷
  - datavolume:/var/lib/mysql
```

但是，如果要跨多个服务并重用挂载卷，请在顶级volumes关键字中命名挂载卷，但是并不强制，如下的示例亦有重用挂载卷的功能，但是不提倡。

```yaml
version: "3"

services:
  web1:
    build: ./web1/
    volumes:
      - ../code:/opt/web/code
  web2:
    build: ./web2/
    volumes:
      - ../code:/opt/web/code
```

> **注意**：通过顶级 `volumes` 定义一个挂载卷，并从每个服务的卷列表中引用它， 这会替换早期版本的 Compose 文件格式中 `volumes_from`。

```yaml
sersion: "3"

services:
  db:
    image: db
    volumes:
      - data-volume:/var/lib/db
  backup:
    image: backup-service
    volumes:
      - data-volume:/var/lib/backup/data
      
volumes:
  data-volume:
```



#### command

覆盖容器启动后默认执行的命令。

```yaml
command: bundle exec thin -p 3000
```

也可以是一个类似于 dockerfile 的列表：

```yaml
command: ["bundle", "exec", "thin", "-p", "3000"]
```



#### links

链接到另一个服务中的容器。

请指定服务名称和链接别名（SERVICE：ALIAS），或者仅指定服务名称。

```yaml
web:
  links:
    - db
    - db:database
    - redis
```

在当前的web服务的容器中可以通过链接的 db 服务的别名 database 访问 db 容器中的数据库应用，如果没有指定别名，则可直接使用服务名访问。

链接不需要启用服务进行通信。

默认情况下，任何服务都可以以该服务的名称到达任何其他服务。  （实际是通过设置 /etc/hosts 的域名解析，从而实现容器间的通信。故可以像在应用中使用 localhost 一样使用服务的别名链接其他容器的服务，前提是多个服务容器在一个网络中可路由联通）

`links` 也可以起到和 `depends_on` 相似的功能，即定义服务之间的依赖关系，从而确定服务启动的顺序。



#### external_links

链接到 docker-compose.yml 外部的容器，甚至是非 compose 管理的容器。参数格式跟 `links` 类似。

```yaml
external_links:
  - redis_1
  - project_db_1:mysql
  - project_db_1:postgresql
```



#### expose

暴露端口，但是不映射到宿主机，只被连接的服务访问。尽可以指定内部端口为参数。

```yaml
expose:
  - "3000"
  - "8000"
```



#### ports

暴露端口，使用 `宿主机端口:容器端口` （host:container) 格式或者仅仅指定容器的端口（宿主将会随机选择端口）都可以。

> **注意**：当使用 `HOST:CONTAINER` 格式来映射端口时，如果你使用的容器端口小于 60 你可能会得到错误得结果，因为 YAML 将会解析 xx:yy 这种数字格式为 60 进制。所以建议采用字符串格式。

简单的短格式：

```yaml
ports:
  - "3000"
  - "3000-3005"
  - "8000:8000"
  - "9090-9091:8080-8081"
  - "49100:22"
  - "127.0.0.1:8001:8001"
  - "127.0.0.1:5000-5010:5000-5010"
  - "6060:6060/udp"
```

在 v3.2 中，ports 的长格式语法允许配置不能用短格式表示的附加字段。

```yaml
ports:
  - target: 80       # 容器内的端口
    published: 8080  # 物理主机的端口
    protocol: tcp    # 端口的协议 （tcp / udp）
    mode: host       # host 和 ingress 两总模式，
                     # host 用于在每个节点上发布主机端口，
                     # ingress 用于被负载平衡的 swarm 模式端口。
```



#### restart

重启策略，默认值是 `no`，在任何情况下都不会重启容器。指定为 `always` 时，容器总是重新启动。 如果退出代码指示出现故障错误，则 `on-failure` 将重新启动容器。

```yaml
restart: "no"
restart: always
restart: on-failure
restart: unless-stopped
```



#### envirnment

添加环境变量。 可以使用数组或字典两种形式。 任何布尔值 `true`，`false`，`yes`，`no` 需要用引号括起来，以确保它们不被YML解析器转换为 True 或 False。 

只给定名称的变量会自动获取它在 Compose 主机上的值，可以用来防止泄露不必要的数据。

```yaml
environment:
  RACK_ENV: development
  SHOW: 'true'
  SESSION_SECRET: 
  
environment:
  - RACK_ENV=development
  - SHOW=true
  - SESSION_SECRET
```

> **注意**：如果你的服务指定了 `build` 选项，那么在构建过程中通过 `environment` 定义的环境变量将不会起作用。 将使用 `build` 的 `args` 子选项来定义构建时的环境变量。



#### pid

将 PID 模式设置为主机 PID 模式。

这就打开了容器与主机操作系统之间的共享 PID 地址空间。 使用此标志启动的容器将能够访问和操作裸机的命名空间中的其他容器，反之亦然。即打开该选项的容器可以相互通过进程 ID 来访问和操作。

```yaml
pid: "host"
```



#### dns

配置 DNS 服务器。可以是一个值，也可以是一个列表。

```yaml
dsn: 8.8.8.8
dns: 
  - 8.8.8.8
  - 9.9.9.9
```







照猫画虎的猫：

- 原文链接：[https://blog.csdn.net/pushiqiang/article/details/78682323](https://blog.csdn.net/pushiqiang/article/details/78682323)





