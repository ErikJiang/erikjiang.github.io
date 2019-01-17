---
title: "CentOS7 安装 Docker 及 Docker Compose"
date: 2018-11-13 18:12:30
categories: "技术" 
tags: 
  - docker
type: "tags"

---

记录 CentOS 环境下 Docker 相关的安装过程，备忘~

<!--more-->

### Step 1 — 安装 Docker

安装需要的依赖包：
``` bash
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

配置 docker-ce repo：
``` bash
$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

安装 docker-ce：
``` bash
$ sudo yum install docker-ce
```

将当前用户添加至 Docker 用户组当中：
``` bash
$ sudo usermod -aG docker $(whoami)
```

设置 Docker 在引导时自启动：
``` bash
$ sudo systemctl enable docker.service
```

最后运行 Docker 服务：
``` bash
$ sudo systemctl start docker.service
```


### Step 2 — 安装 Docker Compose

为企业版linux安装额外依赖包：
``` bash
$ sudo yum install epel-release
```

安装 python-pip：
``` bash
$ sudo yum install -y python-pip
```

然后安装 Docker Compose：
``` bash
$ sudo pip install docker-compose
```

同时需要为你在 CentOS 7 下的 Python 进行升级，以便能够成功运行 docker-compose：
``` bash
$ sudo yum upgrade python*
```

验证 Docker Compose 是否成功安装完成：
``` bash
$ docker-compose version
```
