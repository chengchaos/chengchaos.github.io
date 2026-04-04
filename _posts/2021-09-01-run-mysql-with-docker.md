---
title: 使用 Docker 运行 MySQL
key: 2021-09-01
tags: docker mysql
---

记一下免得忘.

完整版本看[这里 https://hub.docker.com/_/mysql](https://hub.docker.com/_/mysql)

<!--more-->

```bash
docker pull mysql

mkdir -p /works/data/mysql/datadir

docker run -d \
    --name cc-mysql \
    --net host \
    -v /works/data/mysql/datadir:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD='Mq*h4Oy[D{vi=(aE7pJYCD#6' \
    -p 3306:3306 mysql
```

EOF
