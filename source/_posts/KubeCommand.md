---
title: "Kubenetes 常用命令"
date: 2019-03-10 18:45:11
categories: "技术" 
tags: 
  - kube
type: "tags"
---


## 查看 Node

``` sh
$ kubectl get node -o wide
```
`-o wide`：该参数表示输出更加详细的信息



## 创建 Deployments

这里创建了一个名为 hello-test 的 Deployment
``` sh
$kubectl run hello-test --image=alpine --replicas=2 sleep 360000
```

## 查看 Deployments
查看所有的 Namespaces:
``` sh
kubectl get deployments --all-namespaces -o wide
```

## 查看 Deployments 详情
``` sh
$ kubectl describe deployment hello-test
```

## 删除 Deployments
``` sh
$ kubectl delete deployment hello-test -n default
```



## 查看 Pod
``` sh
$ kubectl get pod -o wide
```

## 查看具体某一个 Pod
``` sh
$ kubectl get pod <pod-name> -o yaml
```
`-o yaml`: 以 yaml 格式显示详细信息
`-o json`: 以 json 格式显示详细信息


## 查看 Pod 的详细信息
`kubectl describe` 不支持 `-o` 输出参数；
``` sh
$ kubectl describe pod <pod-name>
```

## 查看 Pod 容器内标准输出的日志信息
``` sh
# -f 随日志信息增长而输出
$ kubectl logs <pod-name> -f
```


# 命令式配置文件操作

## 基于文件创建集群资源

``` sh
$ kubectl create -f xxx.yaml
```

## 基于文件对资源进行更新、替换操作
比如修改了 xxx.yaml 中的副本数、镜像版本、端口等, 此时需要更新资源：
``` sh
$ kubectl replace -f xxx.yaml
```

## 删除配置文件对应的资源
``` sh
$ kubectl delete -f xxx.yaml
```

# 声明式API操作

## 创建及更新资源

创建以及修改更新对应资源都可以执行同样的命令：
``` sh
$ kubectl apply -f xxx.yaml
```
`kubectl replace` 操作相当于新建一个 API 对象来替换原有 API 对象；
`kubectl apply` 操作则是对原有的 API 对象进行了 Patch 操作；
