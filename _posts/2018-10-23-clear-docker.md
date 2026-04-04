---
title: 如何彻底清除 Docker 占用的存储空间
key: 201810123
tags: Docker
---

原文地址：http://www.microsofttranslator.com/bv.aspx?from=&to=en&a=http%3A%2F%2Fwww.cnblogs.com%2Ffundebug%2Fp%2F8353158.html

如何彻底清除 Docker 占用的存储空间呢

<!--more-->


How do I clean up the disk space occupied by Docker?


As a faith-based technology company, our fundebug background uses a cool, fully docker architecture, with all services, including databases, running in Docker. This is certainly not for the sake of dazzle skill, the benefit that sees clearly is still a lot of:

- The configuration of all servers is very simple, only Docker installed, so the new server is much simpler.
- It's easy to move a variety of services between servers, download the Docker mirror and run without having to manually configure the running environment.
- The development/testing environment is in strict conformity with the production environment, without fear of a deployment failure due to environmental problems.

At least, on the line for more than a year, Docker has been very stable, there is no problem. However, it has a modest problem that consumes more disk space.

If Docker accidentally fills up the disk space and your service is over, all Docker users need to be vigilant . Of course, we should not be nervous, this problem is very good solution.

## 1. Docker system command

In who ran out of disk? In the Docker system command detailed, we detail the Docker System command, which can be used to manage disk space.

Docker System DF command, similar to the DF command on Linux, to view disk usage for Docker:

```sh
docker system df
TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              147                 36                  7.204GB             3.887GB (53%)
Containers          37                  10                  104.8MB             102.6MB (97%)
Local Volumes       3                   3                   1.421GB             0B (0%)
Build Cache                                                 0B                  0B
```

The Docker Mirror uses the 7.2GB disk, the Docker container occupies the 104.8MB disk, and the Docker data volume occupies the 1.4GB disk.

The Docker system Prune command can be used to clean up disks, remove closed containers, useless data volumes and networks, and dangling mirrors (that is, mirrors without tag). The Docker system prune-a command cleans more thoroughly, eliminating the use of Docker mirrors with no containers. Note that these two commands delete the container you have temporarily closed, and the Docker mirror image that you have temporarily not used ... So you must think clearly before using.

After the Docker system prune-a command is executed, Docker consumes a lot less disk space:

```sh
docker system df
TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              10                  10                  2.271GB             630.7MB (27%)
Containers          10                  10                  2.211MB             0B (0%)
Local Volumes       3                   3                   1.421GB             0B (0%)
Build Cache                                                 0B                  0B
```

## 2. Manually clean docker mirrors/containers/data volumes

For older versions of Docker (prior to version 1.13), there is no Docker system command, so manual cleanup is required. Here are some common life
Remove all closed containers

```sh
docker ps -a | grep Exit | cut -d ' ' -f 1 | xargs docker rm
```

Delete all dangling mirrors (that is, mirror without tag):

```sh
docker rmi $(docker images | grep "^<none>" | awk "{print $3}")
```

Delete all dangling data volumes (that is, useless volume):

```sh
docker volume rm $(docker volume ls -qf dangling=true)
```

Fundebug provides real-time, professional error monitoring services, for your online code escort, welcome everyone to use FREE!


## 3. Limit container Log Small

Once, when I cleaned up the disk using the methods mentioned in 1 and 2, I did a series of analyses.

On Ubuntu, all related files for Docker, including mirrors, containers, and so on, are stored in the /var/lib/docker/directory:

```sh
du -hs /var/lib/docker/
97G	/var/lib/docker/
```

It's enough that Docker used nearly 100GB disks. To continue with the du command, you can navigate to a directory that really consumes so many disks:

```sh
92G	/var/lib/docker/containers/a376aa694b22ee497f6fc9f7d15d943de91c853284f8f105ff5ad6c7ddae7a53
```

It is known by Docker PS that the ID of the Nginx container is exactly a376aa694b22, with the above directory /var/lib/docker/containers/a376aa694b22 the prefix for the is consistent:

```sh
docker ps
CONTAINER ID        IMAGE                                       COMMAND                  CREATED             STATUS              PORTS               NAMES
a376aa694b22        192.168.59.224:5000/nginx:1.12.1            "nginx -g 'daemon off"   9 weeks ago         Up 10 minutes                           nginx
```

As a result, the Nginx container occupies a disk of 92GB . Further analysis shows that the real disk space consumption is nginx log files. Then it is not difficult to understand. Our fundebug daily data requests are millions, so the log data is naturally very large.

You can use the truncate command to "clear 0" of the Nginx container's log file:

```sh
truncate -s 0 /var/lib/docker/containers/a376aa694b22ee497f6fc9f7d15d943de91c853284f8f105ff5ad6c7ddae7a53/*-json.log
```

Of course, this command is only temporary effect, the log file will rise back sooner or later. To solve the problem fundamentally, you need to limit the log file size of the Nginx container . This can be done by configuring the log's max-size , which is the Docker-compose configuration file for the Nginx container:

```yaml
nginx:
  image: nginx:1.12.1
  restart: always
  logging:
    driver: "json-file"
    options:
      max-size: "5g"
```

After restarting the Nginx container, the size of its log file is limited to 5GB, so don't worry anymore ~

## 4. Restart Docker

Also once , when I cleaned out mirrors, containers, and data volumes, I found that there was no reduction in disk space. Based on the recommendations mentioned in Docker disk usage , I restarted Docker and found that disk usage dropped from 83% to 19%. Depending on the master pointing , this should be a bug related to kernel 3.13, causing Docker to be unable to clean up some unwanted directories:

> It's quite likely that for some reason when those container shutdown, Docker couldn ' t remove the directory because the SHM Device was busy. This tends to happen often on 3.13 kernel. You may want to update it to the 4.4 version supported on trusty 14.04.5 LTS.

> The reason it disappeared after a restart, is that daemon probably tried and succeeded to clean up left over data from Sto pped containers.

I looked at the kernel version and found it was 3.13:

```sh
uname -r
3.13.0-86-generic
```

If your kernel version is also 3.13, and the disk is not successful, you may wish to reboot the Docker. Of course, this evening operation is more reliable.


---

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
