title: "govendor 升级小记"
date: 2019-06-11 15:48:54
categories: "技术" 
tags: 
  - go
  - vendor
type: "tags"

---

老项目要升级 k8s 的 client-go 包，而这个项目由于历史原因，

使用的是 govendor 这样一个包管理工具，于是就捣鼓了一下 ...

<!--more-->

首先，我看了一下 vendor/vendor.json, 发现涉及到 client-go 的包有很多：
``` json
...
{
    "checksumSHA1": "Ovxe5OrkHu7tTYVE/XbTD5+DvF4=",
    "path": "k8s.io/client-go/discovery",
    ...
},
{
    "checksumSHA1": "mgVdewcg4jTq0piouybeQoXd/yQ=",
    "path": "k8s.io/client-go/kubernetes",
    ...
},
{
    "checksumSHA1": "Jmg4wTN/9ztjnzQgADNhFALddv8=",
    "path": "k8s.io/client-go/kubernetes/scheme",
    ...
},
{
    "checksumSHA1": "pmmsHaBIeDlD1LSw/KM2wVdYh2I=",
    "path": "k8s.io/client-go/kubernetes/typed/admissionregistration/v1beta1",
    ...
},
{
    "checksumSHA1": "t7UdbYz8ir1FewYR53Izec0Sivw=",
    "path": "k8s.io/client-go/kubernetes/typed/apps/v1",
    ...
},
{
    "checksumSHA1": "JU//A2Vhtt2lmEJvjPSzotPbax4=",
    "path": "k8s.io/client-go/kubernetes/typed/apps/v1beta1",
    ...
},
...
```
如果我逐个对每个 package 进行 `govendor update` 也没什么问题，

但是效率会很低，且升级下来估计要吐血~

于是，这里用到了通配符 `/...`, 将通配符追加到 `k8s.io/client-go` 包之后，

表示的是，将与 `k8s.io/client-go` 包相关的所有依赖一并操作处理；

这样就一劳永逸了。

如下具体操作，将 client-go 升级到 v11.0.0 版本：
注：`@=` 表示指定固定的 tag 版本

##### 1. 更新 k8s.io/client-go 相关包
``` sh
$ govendor remove k8s.io/client-go/...
$ govendor fetch -v k8s.io/client-go/...@=v11.0.0
```

##### 2. 更新 k8s.io/apimachinery 相关包
``` sh
$ govendor remove k8s.io/apimachinery/...
$ govendor fetch -v k8s.io/apimachinery/...@=kubernetes-1.14.3
```

##### 3. 更新 k8s.io/api 相关包
``` sh
$ govendor remove k8s.io/api/...
$ govendor fetch -v k8s.io/api/...@=kubernetes-1.14.3 
```

当然，也可以尝试使用直接更新的方式来处理升级，比如：
注：`待验证`

``` sh
$ govendor update -v k8s.io/client-go/...@=v11.0.0
$ govendor update -v k8s.io/apimachinery/...@=kubernetes-1.14.3
$ govendor update -v k8s.io/api/...@=kubernetes-1.14.3
```

---
