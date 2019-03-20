---
title: "Minikube MacOS 安装搭建"
date: 2019-03-09 20:53:18
categories: "技术" 
tags: 
  - kube
type: "tags"

---

要完成 minikube 环境的整体安装搭建，我们需要：
1. docker
2. hyperkit
3. minikube
4. kubectl

## 安装 docker

docker 有针对 mac 系统的安装包，只需要下载并安装即可，[Get started with Docker Desktop for Mac](https://docs.docker.com/docker-for-mac/install/) ;

当然安装运行好后，需要查看一下运行状态是否良好，这个官方文档上也写的很明了；

## 安装 hyperkit
Mac 系统下支持 VirtualBox、 VMware Fusion、 Hyperkit 三种虚拟机管理程序，这里虚拟机的作用是为 kubenetes 提供基本环境，由于官方推荐了 Hyperkit，于是选择 Hyperkit 的驱动程序来进行安装：

``` sh
$ brew install docker-machine-driver-hyperkit

# docker-machine-driver-hyperkit need root owner and uid 
$ sudo chown root:wheel /usr/local/opt/docker-machine-driver-hyperkit/bin/docker-machine-driver-hyperkit
$ sudo chmod u+s /usr/local/opt/docker-machine-driver-hyperkit/bin/docker-machine-driver-hyperkit
```

## 安装 kubectl

使用 homebrew 直接安装即可：
``` sh
$ brew install kubectl
```

## 安装 minikube

采用手动方式安装 minikube：
``` sh
$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64 && sudo install minikube-darwin-amd64 /usr/local/bin/minikube
```

以上执行完成后，就到了较为关键的部分，即使用 `minikube start` 开启并运行一个集群；

首先，如果直接运行 `minikube start` 会因为执行过程中，在下载相应依赖时报错；

原因是这些相关的下载项被 GFW 封阻，如果要做到正常下载访问，就需要使用[http代理](https://kubernetes.io/docs/setup/minikube/#using-minikube-with-an-http-proxy)；

具体操作命令如下：
``` sh
$ minikube start --vm-driver=hyperkit \
    --docker-env http_proxy=http://$YOURPROXY:PORT \
    --docker-env https_proxy=https://$YOURPROXY:PORT
```

那么问题来了，如何才能拥有一个可访问的 http 代理服务呢？
考虑到有 shadowsocks，但是基于 socks 又不能直接使用，于是问题又变成如何将 socks 转为 http proxy 呢？
OK，找到了一个转换工具，喔嘈，这配置工作能不能简单点？MMP ...

在经过一番挣扎之后，终于发现，原来[ shadowsocks 客户端](https://github.com/shadowsocks/ShadowsocksX-NG)提供了 HTTP 代理功能，晕~

在这个客户端的 `偏好设置` -> `HTTP` 里面可以直接配置HTTP代理监听地址及端口，爽~

这里需要注意的是，由于上述 `minikube start` 是在虚拟机上进行依赖项下载的，故在命令中的代理监听地址不能使用127.0.0.1，而应该是宿主机的IP地址，否则无论如何虚拟机上都无法访问到宿主机的代理服务的。

比方说，宿主机IP是：192.168.1.101，shadowsocks 上设置的 HTTP 代理监听端口设置为1087，那么 `minikube start` 的实际执行命令应该是：
``` sh
$ minikube start --vm-driver=hyperkit \
    --docker-env HTTP_PROXY=http://192.168.1.101:1087 \
    --docker-env HTTPS_PROXY=http://192.168.1.101:1087
```

此外，在 shadowsocks 上设置 HTTP 代理的监听地址时，由于我们上述命令使用的是宿主机的 IP 地址，所以如果这里 HTTP 代理监听地址还是使用 127.0.0.1 这个默认的回环地址的话，上述命令的宿主机IP地址将无法访问到代理服务，于是需要将 shadowsocks 上的代理地址由 127.0.0.1 改为 0.0.0.0，这样代理服务就可以接收任何IP地址的请求了；

HTTP 代理设置完成并执行 `minikube start` 之后，OK，集群启动成功；

此时，配置 kubectl 使用 minikube 上下文，这样使得 bubectl 能够与 minikube 集群进行交互：
``` sh
$ kubectl config use-context minikube
```

验证查看 bubectl 已配置的集群信息：
``` sh
$ kubectl cluster-info
```

此时，可以使用 `kubectl` 与 k8s 集群交互，并启动一个服务：
``` sh
$ kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.4 --port=8080
```

公开该服务为 NodePort：
``` sh
$ kubectl expose deployment hello-minikube --type=NodePort
```

minikube 可以轻松的使用你的浏览器打开这个公开的端点（endpoint）：
``` sh
$ minikube service hello-minikube
```

开启第二个本地集群：
``` sh
$ minikube start -p cluster2
```

停止你的本地集群：
``` sh
$ minikube stop
```

删除你的本地集群：
``` sh
$ minikube delete
```

## 重启系统后的问题

minikube 正常安装完了之后，以为一切归于平静，没想到系统重启后，又出了幺蛾子：
``` sh
$ kubectl get node
The connection to the server xx.xx.xx.xx:8443 was refused - did you specify the right host or port?
```
关于这个问题，minikube 的 github issue 中有提及，但作者并没有很好的解决，最终给出的意见是建议使用 docker for mac 中提供的 K8S 功能；

感觉是绕了一圈，最终又回到了原点，人生有时就是如此之骚~

好吧，于是我放弃了minikube，转向 docker for mac 提供的 kubenetes 功能；

由于这其中下载安装仍然困难重重，这里提供一个安装方法，见：[k8s-for-docker-desktop](https://github.com/AliyunContainerService/k8s-for-docker-desktop)

---

参考：

* http://blog.samemoment.com/articles/kubernetes/

* https://k8smeetup.github.io/docs/tutorials/stateless-application/hello-minikube/
