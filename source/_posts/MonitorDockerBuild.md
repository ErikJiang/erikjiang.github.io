---
title: "Docker 构建小记"
date: 2018-12-14 19:32:00
categories: "技术" 
tags: 
  - go
  - docker
type: "tags"

---

关于在项目 market_monitor 尝试进行 Docker 构建的过程中的一些记录整理；

<!--more-->

### 构建应用服务 docker 镜像

> 本项目见：[market_monitor](https://github.com/ErikJiang/market_monitor)

#### docker 构建操作步骤

编写镜像构建文件：
``` dockerfile

############################
# STEP 1 构建可执行文件
############################

# 指定 GO 版本号
ARG GO_VERSION=1.11.1

# 指定构建环境
FROM golang:${GO_VERSION}-alpine AS builder


# ca-certificates is required to call HTTPS endpoints.
# tzdata is required to time zone info.
RUN apk update && apk upgrade && apk add --no-cache ca-certificates tzdata && update-ca-certificates

# 创建用户 appuser
RUN adduser -D -g '' appuser

# 复制源码并指定工作目录
RUN mkdir -p /src/myapp
COPY ./src/ /src/myapp
WORKDIR /src/myapp

# 为 go build 设置环境变量:
# * CGO_ENABLED=0 表示构建一个静态链接的可执行程序
# * GOOS=linux GOARCH=amd64 表示指定linux 64位的运行环境
# * GOPROXY=https://goproxy.io 指定代理地址
# * GOFLAGS=-mod=vendor 在执行 `go build` 强制查看 `/vendor` 目录
ENV CGO_ENABLED=0 GOOS=linux GOARCH=amd64 GOFLAGS=-mod=vendor

# 构建可执行文件
RUN go build -a -installsuffix cgo -ldflags="-w -s" -o /src/myapp/monitor

############################
# STEP 2 构建镜像
############################

# 指定最小镜像源
FROM scratch AS final

# 设置系统语言
ENV LANG en_US.UTF-8

# 从 builder 中导入时区信息
COPY --from=builder /usr/share/zoneinfo /usr/share/zoneinfo

# 从 builder 中导入证书
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

# 从 builder 中导入用户及组相关文件
COPY --from=builder /etc/passwd /etc/passwd

# 将构建的可执行文件复制到新镜像中
COPY --from=builder /src/myapp/config /config
COPY --from=builder /src/myapp/monitor /monitor

# 端口申明
EXPOSE 8000

# 运行
ENTRYPOINT [ "/monitor" ]

```

在构建镜像前，为了保证效率，我们最好配置一个国内的镜像地址，可以采用阿里云的镜像加速器地址；

执行构建命令：
```
# 指定镜像名为 monitor
docker build -t monitor .
```

构建成功，查看镜像列表：

发现多了一个`monitor`镜像
``` bash
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
monitor             latest              3cce1a5a8514        12 minutes ago      34.6MB
golang              1.11.1-alpine       95ec94706ff6        3 months ago        310MB
```

运行容器服务：
``` bash
docker run -p 8000:8000 monitor -d monitor
```

#### 镜像的构建流程

这里使用到 Docker 的多阶段构建(multi-stage builds)方式;

按步骤主要两个阶段：
1. 源码编译阶段，镜像使用：golang:1.11.1-alpine
2. 环境构建阶段，镜像使用：scratch

大致过程，
我们会在第一个镜像中安装运行所需的依赖、创建用户、将源码编译为可执行文件等，
然后再将运行所需的所有文件拷贝至第二个镜像中，第二个镜像最终会作为容器的运行镜像；

#### 为什么不直接用一个镜像完成所有操作？
``` bash
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
monitor             latest              3cce1a5a8514        12 minutes ago      34.6MB
golang              1.11.1-alpine       95ec94706ff6        3 months ago        310MB
```
如上就可以说明问题，`monitor`是采用`scratch`最终生成的镜像，而`golang-1.11.1-alpine`是用于编译源码的基础镜像，如果采用`golang-alpine`作为基础镜像，再加上源码及依赖包文件，总大小可能会在500MB左右，这对于运维部署是不能接受的；而[`scratch`](https://hub.docker.com/_/scratch/)能够很好的控制镜像大小，它本身即是一个精简的最小镜像，再加上golang编译过程中的优化，最终镜像尺寸会很小；


#### 处理构建过程的依赖包问题
本来一开始是打算采用`go mod download`的方式，在镜像环境中下载依赖包以满足构建编译，但是没走通，故转而尝试`vendor`的方式；
也就是说我们会首先将所有依赖包存入项目根目录的`vendor/`下，然后连同项目源码一起拷贝至源码编译镜像环境，再进行编译操作；

首先，要保证你曾成功执行过`go build`或`go run main.go`操作，只有这样，才能迅速将依赖包的缓存文件汇总到`vendor/`目录下；

当然，你要去执行`go mod vendor`，它才会生成`vendor/`目录；

然后，需要在`dockerfile`的golang编译命令前添加环境变量，指定依赖从`vendor/`目录读取；
``` dockerfile
ENV GOFLAGS=-mod=vendor
```
如此，依赖问题就解决了；

#### HTTPS X509 问题

如果项目中有包含对第三方HTTPS接口的请求，那么多少都会遇到如下报错：
```
x509: certificate signed by unknown authority
```
这个问题，是没有安装导入CA证书的原因；

解决方法分两步：
1. 编译源码阶段，需要安装`ca-certificates`
``` dockerfile
RUN apk update && apk upgrade && apk add --no-cache ca-certificates && update-ca-certificates
```
2. 构建最小环境阶段，需要将证书拷入镜像环境
``` dockerfile
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
```

#### 关于编排

ok，经过如上步骤的操作尝试，会发现虽然服务已运行，但是 mysql、redis 无法正常连接，噢，mysql、redis 容器服务还没有跑起来 ...

是不是有些烦躁，因为 mysql、redis 容器服务的构建还会涉及到一些初始化操作、账号权限配置、密码更改等很多自定义设置，对于一介开发，有些凌乱；

但是，别忘了 `docker compose` 容器编排，它将为你解决余下的事情，请见：

[Compose 容器编排部署](https://github.com/ErikJiang/market_monitor/wiki/ComposeDeployTutorials)

---

* [create the smallest and secured golang docker image based on scratch](https://medium.com/@chemidy/create-the-smallest-and-secured-golang-docker-image-based-on-scratch-4752223b7324)