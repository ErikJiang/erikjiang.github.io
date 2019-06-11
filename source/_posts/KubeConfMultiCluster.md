---
title: "kubectl config 管理多集群"
date: 2019-05-02 19:47:11
categories: "技术" 
tags: 
  - kube
  - cluster
type: "tags"
---


我们日常开发中会不可避免的使用到多个 Kubernetes 集群，一般比较传统的做法，我们会使用 ssh 远程到集群节点主机上，然后再进行 kubectl 相关命令的操作，这样做也没什么问题，但是当集群数增多以后，就会发现切换集群的操作略显麻烦，且无法管理，于是就有了一个需求，怎样能够更加优雅的组织管理这些不同的集群接入场景？

<!--more-->

在了解 kubectl config 提供的功能时发现，它本身就能够清晰的组织管理不同集群的上下文，且能够无缝切换集群环境。

在这篇文章中，我将介绍如何使用 kubectl config 组织管理日常工作中的四个不同的集群环境。

我们先列出需要组织管理的集群列表：

* `docker for desktop`: 本地 Docker Desktop 提供的 k8s 单节点环境
* `work-test`:          k8s 多节点测试环境
* `work-dev`:           k8s 多节点开发环境
* `work-prod`:          k8s 多节点生产环境

首先，当你本地成功安装了 kubectl，会存在一个 `~/.kube/` 的目录，该目录用于存放所有集群的配置文件；

由于我本地安装了 Docker Desktop Kubernetes 环境，
所以现在可以通过 `vim ~/.kube/config` 查看一下本地 k8s 单节点配置信息：

``` yaml
apiVersion: v1
clusters:
- cluster:
    insecure-skip-tls-verify: true
    server: https://localhost:6443
  name: docker-for-desktop-cluster
contexts:
- context:
    cluster: docker-for-desktop-cluster
    user: docker-for-desktop
  name: docker-for-desktop
current-context: docker-for-desktop
kind: Config
preferences: {}
users:
- name: docker-for-desktop
  user:
    client-certificate-data: ...
    client-key-data: ...
```

从该配置能够看出，一个完整的集群配置至少要包含三个要素：

* clusters 集群
* contexts 上下文
* users 用户

接下来，我们需要分别将 `work-test`、`work-dev`、`work-prod` 三个环境中的配置添加到本地；

通常 Kubernetes 集群在安装时会生成一个管理配置文件，该配置文件位于每个集群的主节点上，具体目录：

``` sh
/etc/kubernetes/admin.conf
```

了解到了这一点，我们就可以使用 scp 命令将各个集群主节点中的配置拷贝到本地 .kube 目录中了：

``` sh
# copy the Kubernetes admin config for work test
scp root@SERV_MASTER_TEST:/etc/kubernetes/admin.conf ~/.kube/config-work-test`

# copy the Kubernetes admin config for work dev
scp root@SERV_MASTER_DEV:/etc/kubernetes/admin.conf ~/.kube/config-work-dev`

# copy the Kubernetes admin config for work prod
scp root@SERV_MASTER_PROD:/etc/kubernetes/admin.conf ~/.kube/config-work-prod`
```

将如上命令中 `SERV_MASTER_TEST`、`SERV_MASTER_DEV`、`SERV_MASTER_PROD` 分别替换为对应集群主节点 IP;

注：配置文件中包含敏感信息，请妥善保管！

现在，我们已经完成了配置的拷贝，但是为了便于管理，我们需要对这些配置中的命名进行修改规范；

我们要修改这三个配置文件，并着重关注三大要素：集群、上下文、用户；

以 work-dev 为例：

##### 1. 集群 clusters

``` yaml
# 修改集群名称
- cluster:
    server: https://xx.xx.xx.xx:6443
  name: work-dev-cluster

```

##### 2. 用户 users

``` yaml
# 修改用户名称
users:
- name: work-dev-admin
```

##### 3. 上下文 contexts
```
# 更新上下文名称，关联对应用户及集群
contexts:
- context:
    cluster: work-dev-cluster
    user: work-dev-admin
  name: work-dev
```

三个配置文件都修改完成后，我们需要将这些配置全部加载到 kube config 中；

具体这里将会使用到名为 KUBECONFIG 的环境变量，该环境变量用于保存以冒号分隔的配置文件路径列表；

kubectl config 将通过 KUBECONFIG 环境变量加载并组合所有的集群配置信息；

我们可以将 KUBECONFIG 环境变量配置添加到 .bash_profile 或 .zshrc 当中：

```
export KUBECONFIG=$HOME/.kube/config-work-test:$HOME/.kube/config-work-dev:$HOME/.kube/config-work-prod:$HOME/.kube/config
```

设置完后，执行命令以使配置生效：
```
$ source ~/.bash_profile
# 或者
$ source ~/.zshrc
```

也可以打开一个新终端，验证环境变量是否生效：
```
$ echo $KUBECONFIG
```

至此，kubectl config 集群配置已经完成;

让我们验证一下！

##### 1. 获取所有的集群上下文列表：
```
$ kubectl config get-contexts
CURRENT   NAME                 CLUSTER                      AUTHINFO             NAMESPACE
*         docker-for-desktop   docker-for-desktop-cluster   docker-for-desktop
          work-test            work-test-cluster            work-test-admin
          work-dev             work-dev-cluster             work-dev-admin
          work-prod            work-prod-cluster            work-prod-admin
```

##### 2. 获取展示当前的上下文：
```
$ kubectl config current-context
docker-for-desktop
```

##### 3. 将当前上下文设置为开发环境：
```
$ kubectl config use-context work-dev
Switched to context "work-dev".
```
此时，即可使用 `kubectl get nodes` 等命令验证一下是否已切换到对应集群环境。


##### 4. 更多的 kubectl config 命令见帮助信息：
``` sh
  current-context 显示 current_context
  delete-cluster  删除 kubeconfig 文件中指定的集群
  delete-context  删除 kubeconfig 文件中指定的 context
  get-clusters    显示 kubeconfig 文件中定义的集群
  get-contexts    描述一个或多个 contexts
  rename-context  Renames a context from the kubeconfig file.
  set             设置 kubeconfig 文件中的一个单个值
  set-cluster     设置 kubeconfig 文件中的一个集群条目
  set-context     设置 kubeconfig 文件中的一个 context 条目
  set-credentials 设置 kubeconfig 文件中的一个用户条目
  unset           取消设置 kubeconfig 文件中的一个单个值
  use-context     设置 kubeconfig 文件中的当前上下文
  view            显示合并的 kubeconfig 配置或一个指定的 kubeconfig 文件
```

---
