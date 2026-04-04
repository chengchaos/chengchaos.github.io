---
title: A cheat sheet of docker commands
key: 2022-08-19
tags: cheat-sheet docker kubectl
---

默认情况下，容器以隔离方式运行。它们对同一台计算机上的其他进程或容器一无所知。若要允许容器间进行通信，请使用网络。

如果两个容器在同一网络上，那么它们可彼此通信。如果没在同一网络上，则没法通信。

<!--more-->

有两种方法可将容器放在网络上：在启动时进行分配，或者连接现有容器。 在此示例中，先创建网络，然后在启动时附加 MySQL 容器。

```bash
## 创建 Docker 网络
docker network create todo-app
## 使用这个 Docker 网络
## 命令指定了网络别名 mysql
docker run -d \
    --network todo-app --network-alias mysql \
    -v todo-mysql-data:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=<your-password> \
    -e MYSQL_DATABASE=todos \
    mysql:5.7
## 
```

## 使用 MySQL 运行应用

待办事项应用支持设置环境变量来指定 MySQL 连接设置。

- MYSQL_HOST MySQL 服务器的主机名。
- MYSQL_USER 要用于连接的用户名。
- MYSQL_PASSWORD 要用于连接的密码。
- MYSQL_DB 连接后要使用的数据库。

> 警告
> 对于开发，可以使用环境变量来设置连接设置。 在生产环境中运行应用程序时，不建议采用此做法。 有关详细信息，请参阅为何不该对机密数据使用环境变量。
>
> 更安全的机制是使用容器编排框架提供的机密支持。 在大多数情况下，这些机密作为文件装载到正在运行的容器中。
> [Why you shouldn't use ENV variables for secret data](https://blog.diogomonica.com//2017/03/27/why-you-shouldnt-use-env-variables-for-secret-data/)

使用以下 docker run 命令。 该命令指定上面的环境变量。

```bash
docker run -dp 3000:3000 \
  -w /app -v ${PWD}:/app \
  --network todo-app \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=<your-password> \
  -e MYSQL_DB=todos \
  node:12-alpine \
  sh -c "yarn install && yarn run dev"
```

## 创建 Docker Compose 文件

Docker Compose 有助于定义和共享多容器应用程序。 使用 Docker Compose，你可以创建用于定义服务的文件。 使用单个命令，可以启动所有内容，也可以将其全部销毁。

可以在文件中定义应用程序堆栈，并将该文件保存在项目存储库的根目录下，受版本控制。 利用此方法，其他人也可以参与你的项目。 他们只需克隆你的存储库。

1. 在应用项目的根目录中，创建名为 `docker-compose.yml` 的文件。
2. 在 Compose 文件中，首先定义架构版本。

```yaml
version: "3.7"
```

在大多数情况下，最好使用支持的最新版本。 有关当前架构版本和兼容性矩阵，请参阅 [Compose 文件](https://docs.docker.com/compose/compose-file/)。

3.定义要作为应用程序的一部分运行的服务（或容器）列表。

```yaml
version: "3.7"

services:
```

4.下面是用于应用容器的命令。 将此信息添加到 .yml 文件。

```Bash
docker run -dp 3000:3000
  -w /app -v ${PWD}:/app
  --network todo-app
  -e MYSQL_HOST=mysql
  -e MYSQL_USER=root
  -e MYSQL_PASSWORD=<your-password>
  -e MYSQL_DB=todos
  node:12-alpine
  sh -c "yarn install && yarn run dev"
```

定义容器的服务项和映像。

```YAML

version: "3.7"

services:
  app:
    image: node:12-alpine
```

你可为服务选择任何名称。 该名称会自动成为网络别名，这在定义 MySQL 服务时非常有用。

5.添加命令。

```YAML

version: "3.7"

services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
```

6.指定用于服务的端口，该端口对应于上述命令中的 -p 3000:3000。

```YAML

version: "3.7"

services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
```

7.指定工作目录和卷映射

```YAML

version: "3.7"

services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
    working_dir: /app
    volumes:
      - ./:/app
```

在 Docker Compose 卷定义中，你可以使用当前目录中的相对路径。

8.指定环境变量定义。

```YAML

version: "3.7"

services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
    working_dir: /app
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: <your-password>
      MYSQL_DB: todos
```

9.添加 MySQL 服务的定义。 下面是你在上面所使用的命令：

```Bash

docker run -d
  --network todo-app --network-alias mysql
  -v todo-mysql-data:/var/lib/mysql
  -e MYSQL_ROOT_PASSWORD=<your-password>
  -e MYSQL_DATABASE=todos
  mysql:5.7
```

定义新服务，并将其命名为 mysql。 在 app 定义后以相同的缩进级别添加文本。

```YAML

version: "3.7"

services:
  app:
    # The app service definition
  mysql:
    image: mysql:5.7
```

该服务会自动获取网络别名。 指定要使用的映像。

`0.定义卷映射。

使用与 services: 相同级别的 volumes: 部分指定卷。 在映像下指定卷映射。

```YAML

version: "3.7"

services:
  app:
    # The app service definition
  mysql:
    image: mysql:5.7
    volumes:
      - todo-mysql-data:/var/lib/mysql

volumes:
  todo-mysql-data:
```

``.指定环境变量。

```YAML

    version: "3.7"

    services:
      app:
        # The app service definition
      mysql:
        image: mysql:5.7
        volumes:
          - todo-mysql-data:/var/lib/mysql
        environment: 
          MYSQL_ROOT_PASSWORD: <your-password>
          MYSQL_DATABASE: todos

    volumes:
      todo-mysql-data:
```

此时，完整的 docker-compose.yml 如下所示：

```YAML

version: "3.7"

services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
    working_dir: /app
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: <your-password>
      MYSQL_DB: todos

  mysql:
    image: mysql:5.7
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: <your-password>
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```

## 运行应用程序堆栈

现在，已拥有 docker-compose.yml 文件，请试运行看看。

1.确保应用和数据库的其他副本未运行。 在 Docker 扩展中，右键单击任何正在运行的容器，然后选择“移除”。 或者，在命令行中，使用 docker rm 命令，如前面的示例所示。

2.在“VS Code 资源管理器”中，右键单击 docker-compose.yml，然后选择“Compose Up”。 或者，在命令行中，使用此 docker 命令。

```Bash
docker-compose up -d
```

利用参数 `-d`，可使命令在后台运行。

你应该看到类似于以下结果的输出。

Creating network "app_default" with the default driver
Creating volume "app_todo-mysql-data" with default driver
Creating app_app_1   ... done
Creating app_mysql_1 ... done

已创建卷以及网络。 默认情况下，Docker Compose 会专门为应用程序堆栈创建网络。

3.删除

```bash
docker-compose down
```

EOF

---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
