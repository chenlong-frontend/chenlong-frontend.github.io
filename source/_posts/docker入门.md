---
title: docker入门笔记
author: 陈龙
date: 2019-06-28 19:40:20
tags: [docker]
categories: [docker]
---

认识 docker 至今也有一年多一点了，在此之前对 docker 或多或少已经有了一些了解，这里主要记录一下安装已经打包镜像。

内容基本都是摘录自网上，方便以后查找。这里以 centos7 为例。

## 安装

首先安装必要环境

```shell
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum makecache fast
```

安装 docker

```shell
yum -y install docker-ce
```

启动 docker

```shell
systemctl start docker
```

## 打包 node 服务镜像

首先进到 node 项目根目录下，执行`touch Dockerfile`，写入如下配置：

```shell
## 拉取官方镜像10-alpine版本
FROM node:10-alpine
## 工作路径
WORKDIR /usr/src/app
## 将package.json拷贝到docker中
COPY package*.json ./
## 安装yarn
RUN npm install -g yarn
## 设置淘宝镜像
RUN yarn config set registry https://registry.npm.taobao.org -g
## 安装依赖
RUN yarn
## 将本地项目代码拷贝至docker
COPY . .
## 暴露3333端口
EXPOSE 3333
## 执行yarn start启动服务
CMD [ "yarn", "start" ]
```

执行`touch .dockerignore`，这里放入需要忽略的文件，例如：

```shell
node_modules
```

之后执行`docker build -t l1016482011/prisma-rest .`打包镜像。可以执行`docker images`查看当前镜像信息。

通过执行`docker run -p 3333:3333 -d l1016482011/prisma-rest`启动进行，访问本机`3333`端口即可。

## 一些常用信息

[docker hub](https://cloud.docker.com/)

[docker 命名大全](https://www.runoob.com/docker/docker-command-manual.html)
