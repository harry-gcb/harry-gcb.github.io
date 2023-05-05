---
layout: title
title: docker常用命令
date: 2023-04-25 23:21:40
tags: [docker]
---

## docker删除命令

删除镜像之前，必须先停止并删除运行的容器，或者删除镜像时添加`-f`选项进行强制删除

###  删除单个容器或镜像

```shell
docker rm 容器ID/容器名字
docker rmi 镜像ID/镜像名字:TAG
```

### 停止所有容器

```shell
docker stop $(docker ps -a -q)
```

### 删除所有untagged images

```shell
docker rmi $(docker images | grep "<none>" | awk '{print $3}')
```

### 删除所有镜像

```shell
docker rmi $(docker images -q)
docker image prune
```



