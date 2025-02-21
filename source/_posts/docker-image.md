---
layout: blog
title: 解决docker国内源失效问题
date: 2025-02-21 12:08:39
categories: 环境搭建
---

最近在学习docker，由于经典原因，需要配置阿里云国内docker镜像源加速

然而，按照阿里云的操作文档，修改 `/etc/docker/daemon.json` 并重启服务以后，docker 还是从海外的仓库拉取：

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://xxxxxxxx.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker


docker info   
...
 Insecure Registries:
  127.0.0.0/8
 Registry Mirrors:
  https://docker.mirrors.ustc.edu.cn/
 Live Restore Enabled: false

docker run --rm hello-world      
Unable to find image 'hello-world:latest' locally
docker: Error response from daemon: Get "https://registry-1.docker.io/v2/": context deadline exceeded (Client.Timeout exceeded while awaiting headers).
```

查找这篇教程：https://blog.csdn.net/llllllllpc/article/details/143693832 以后，修改了 json 文件：

```json
{
  "registry-mirrors": [
    "https://xxxxxxxx.mirror.aliyuncs.com"
  ],
  "dns" : [
    "114.114.114.114",
    "8.8.8.8"
  ],
  "proxies": {
    "http-proxy": "http://127.0.0.1:7897",
    "https-proxy": "http://127.0.0.1:7897"
  },
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "200m"
  }
}

```

其中 7897 是 clash verge 的代理地址

然后保存和重启 docker 就可以了
