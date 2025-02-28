---
layout: blog
title: docker与compose部署分布式网络
date: 2025-02-25 14:13:52
categories: 环境搭建
---

最近手里有一个go写的区块链工程，之前都是在本机上部署节点：写一个shell脚本，循环为每个节点go run启动

为了后续在真实分布式环境上的部署方便，所以在项目中加入docker来方便部署

## 基础：Go程序创建docker容器

这一步比较简单，在Dockerfile中把官方的Go容器拉过来，然后把代码放进去，下载必要的软件和依赖库，设置启动脚本为 entrypoint：

``` dockerfile
FROM golang:1.23.4-alpine AS builder

# the default user in golang image is root
WORKDIR /root/Chamael

# copy the chamael source code to the image
COPY . .

# enable cgo
ENV CGO_ENABLED=1

# add the aliyun mirror
RUN echo "https://mirrors.aliyun.com/alpine/v3.21/main" > /etc/apk/repositories && \
    echo "https://mirrors.aliyun.com/alpine/v3.21/community" >> /etc/apk/repositories && \
    apk add --no-cache gcc musl-dev bash

# add the goproxy   
ENV GOPROXY=https://goproxy.cn,direct

# download the dependencies
RUN go mod download

# make the start_all.sh executable
RUN chmod +x start_all.sh && chmod +x start_one.sh

ENTRYPOINT ["./start_all.sh"]
```

然后build镜像和运行就ok了：

```bash
docker build -t chamael:latest .
docker run -it --rm chamael:latest 4 3 1000 1
```

这样的代码跑起来和在本机跑也没有区别，仅仅是挪到了容器里面

## 进阶：Docker compose编排分布式容器

把所有节点挪到一个docker容器里面跑显然是无意义的，让一个节点运行在一个容器中，同时启动多个容器交互才是我们的最终目的，因此就需要用到Docker compose

首先是docker-compose.yml本身：

```yaml
version: '3'
services:
  node0:
    build: .
    entrypoint: ['./start_one.sh', '0', '1000', '1', '1']
    container_name: node0
    ports:
      - 9233:9233
    networks:
      - tcp_net

  node1:
    build: .
    entrypoint: ['./start_one.sh', '1', '1000', '1', '1']
    container_name: node1
    ports:
      - 9234:9234
    networks:
      - tcp_net
  
  ...
  
networks:
  tcp_net:
    driver: bridge
```

写一个docker-compose.yml同时启动多个容器并不难，难在网络的配置：

一开始以为的方案是每个容器打开一个端口映射到主机的端口上，然后其他容器去连接的端口即可，经过查证这个方法是可行的，docker容器中主机的IP地址一般固定是172.17.0.1，不确定的话可以ifconfig看一下docker虚拟网卡的地址

然而这种方法老是出bug，最后发现docker compose会给容器创建一个内部网络，如果其他节点要去连接node0的话，只需要直接连接其container_name即可：`net.Dial('tcp', 'node0:8080')`，这样一来就可以直接连接到其他容器，而不需要通过主机的端口映射了

写好compose文件以后 `docker compose up` 即可

注意，如果需要删掉节点镜像的话，需要先 `docker compose down` 关闭所有节点容器才可以删除
