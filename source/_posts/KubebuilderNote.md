---
title: "kubebuilder 入门实践"
date: 2021-02-03 19:04:21
categories: "技术" 
tags: 
  - kube
  - kubebuilder
type: "tags"
---

通过本文的讲解, 你将大体了解 Kubebuilder 的开发流程, 及其中的关注要点;

> 本文实践的代码示例请参考: [kube-falcon](https://github.com/ErikJiang/kube-falcon)

<!--more-->

### 下载与安装

我们可以到 kubebuilder 在 github 上的 releases 页面, 下载支持当前系统的二进制包;

* [kubebuilder releases 下载页面]( https://github.com/kubernetes-sigs/kubebuilder/releases)

 
以 mac 系统为例, 具体的安装步骤如下:

```bash
# 解压安装包
$ tar -zxvf kubebuilder_2.3.1_darwin_amd64.tar.gz

# 本地创建安装目录
$ sudo mkdir /usr/local/kubebuilder/

# 将解压文件移动到安装目录
$ sudo mv kubebuilder_2.3.1_darwin_amd64/* /usr/local/kubebuilder/

# 为 PATH 环境变量追加 kubebuilder 二进制路径
$ export PATH=$PATH:/usr/local/kubebuilder/bin
```

### 创建一个项目

```bash
# 创建项目目录
$ mkdir /src/kube-falcon && cd /src/kube-falcon

# 使用 go mod 初始化项目
$ go mod init kube-falcon

# 使用 kubebuilder 脚手架初始化项目目录结构
$ kubebuilder init --domain kubebuilder.io

```


### 创建一个 API

这里我们创建一个 group 为 app, version 为 v1, kind 为 DeployObject 的 api:
```
# 创建 API
$ kubebuilder create api --group app --version v1 --kind DeployObject
```
执行后, 项目中的 `api/` `controllers/` 会分别生成对应的文件;


### 目录结构

当我们使用 kubebuilder 脚手架初始化了一个项目目录, 并创建了一个 API 后, 可以大致看到如下结构:
```
.
├── Dockerfile          # 用于构建控制器镜像的 Dockerfile
├── Makefile            # 用于控制器构建及部署的 Makefile
├── PROJECT             # 勇于生成组件的 kubebuilder 元数据
├── README.md
├── api                                 # API 模板代码所在目录
│   └── v1
│       ├── deployobject_types.go       # API 类型文件, 主要关注 Spec 与 Status 结构体
│       ├── groupversion_info.go        # 此文件包含了 Group Version 的一些元信息
│       └── zz_generated.deepcopy.go    # 自动生成的 runtime.Object 实现
├── bin
│   └── manager
├── config              # 采用 Kustomize YAML 定义的配置
│   ├── certmanager/    # 证书管理相关
│   ├── crd/            # CRD 相关, 当 make install 将 apply 此目录 yaml 
│   ├── default/        # 控制器相关, 当 make deploy 将 apply 此目录 yaml
│   ├── manager/
│   ├── prometheus/     # 监控相关
│   ├── rbac/           # RBAC 权限管理
│   ├── samples/        # CR 样例
│   └── webhook/        # webhook相关
├── controllers                     # 控制器逻辑所在目录
│   ├── deployobject_controller.go  # 控制器 reconcile 逻辑实现所在文件 
│   └── suite_test.go               # 测试文件
├── cover.out
├── go.mod              # Go Mod 配置文件,记录依赖信息
├── go.sum
├── hack
│   └── boilerplate.go.txt
└── main.go             # 程序入口
```

### 开发阶段

我们使用 kubebuilder 的目的是为了扩展 Kubernetes API, 实现我们的自定义需求;

换言之, 我们是要利用声明式 API 的方式, 扩展及构建属于我们自己的自定义资源(CRD);

那么什么是声明式 API 呢? 

其实通俗的讲就是"告知处理者, 你的最终需求是什么, 而不是告知处理者, 他该怎么做";

即我们定义一个 K8S 对象的终态(Custom Resource), 让处理者自行处理实现, 其中的处理过程不需要关心, 这里的处理者就是控制器(Controller);

所以对于开发者来讲, 实现自定义资源的扩展, 需要进行两个步骤:

1. 编写 CRD 并将其部署至 K8S 集群中;
2. 编写 Controller 并将其部署至 K8S 集群中;

以上提及的这两个步骤, 在 kubebuilder 的开发流程中都有所对应;

#### 1. 编写 CRD

编写 CRD, 本质上是设计及定义声明式 API 的属性字段;

这里面着重关注`${ProjectName}/api/${Version}/${Kind}_types.go` 这个文件;

在这个文件中, 我们着重关注 `${Kind}Spec` 与 `${Kind}Status` 这两个结构体;

这个两个结构体分别代表着一个 k8s 自定义资源所必须的两个部分`spec`及`status`;

我们需要将设计好的属性字段添加到所提及的上述结构体中,
然后在此之后执行 `make` 将会更新 `config/crd/` 内的 yaml 定义;

如下是实例展示:

```bash
$ vi api/v1/deployobject_types.go
```

```go
// DeployObjectSpec defines the desired state of DeployObject
type DeployObjectSpec struct {
	// INSERT ADDITIONAL SPEC FIELDS - desired state of cluster
	// Important: Run "make" to regenerate code after modifying this file

	// +kubebuilder:validation:Minimum=0

	// Replicas is pod replica num
	Replicas *int32 `json:"replicas"`

	// +kubebuilder:validation:MinLength=0

	// Image is container image address
	Image string `json:"image"`

	// Resources describes the compute resource requirements
	// +optional
	Resources corev1.ResourceRequirements `json:"resources,omitempty"`

	// EnvVar represents an environment variable present in a Container.
	// +optional
	Env []corev1.EnvVar `json:"env,omitempty"`

	// ServicePort contains information on service's port.
	Ports []corev1.ServicePort `json:"ports"`
}

// DeployObjectStatus defines the observed state of DeployObject
type DeployObjectStatus struct {
	// INSERT ADDITIONAL STATUS FIELD - define observed state of cluster
	// Important: Run "make" to regenerate code after modifying this file

	// DeploymentStatus is the most recently observed status of the Deployment.
	appsv1.DeploymentStatus `json:",inline"`
}
```

#### 2. 编写 Controller

我们知道, Controller 的工作原理其实就是将自定义资源(CR)定义的预期状态与当前资源的实际状态进行比对, 若无差异则略过, 若有差异则需要将实际状态更新为预期的状态值;

这个比对更新的过程在 Controller 中被称为调谐(Reconcile);

而我们编写 Controller 的主要逻辑, 大部分都集中在这个调谐的逻辑上;

在生成的脚手架目录中, 我们主要关注 `${ProjectName}/controllers/${Kind}_controller.go` 这个文件;

再此文件中, 我们要关注 `Reconcile()` 这个方法, 此方法中的实现即是调谐的整个逻辑;

这里展示的是一段示例的调谐逻辑, 
主要功能实现了通过一个自定义资源(CR)来管理 Deployment 以及 Service 的逻辑:

```bash
$ vi controllers/deployobject_controller.go
```

```go
func (r *DeployObjectReconciler) Reconcile(req ctrl.Request) (ctrl.Result, error) {
	ctx := context.Background()
	log := r.Log.WithValues("deployobject", req.NamespacedName)

	// 1. 获取 NS.DeployObject 实例
	var deployObject apiv1.DeployObject
	if err := r.Get(ctx, req.NamespacedName, &deployObject); err != nil {
		log.Info("unable to fetch DeployObject")
		return ctrl.Result{}, client.IgnoreNotFound(err)
	}

	// 2. DeployObject 实例处于删除状态, 则退出
	if deployObject.DeletionTimestamp != nil {
		log.Info("DeployObject/%s is deleting", req.Name)
		return ctrl.Result{}, nil
	}

	// 3. 检测 NS 下是否有已创建的关联 Deployment
	var deployment appsv1.Deployment
	if err := r.Get(ctx, req.NamespacedName, &deployment); err != nil && apierrors.IsNotFound(err) {
		// create deployment
		deployment := newDeployment(&deployObject)
		if err := r.Create(ctx, deployment); err != nil {
			log.Error(err, "create deployment failed")
			return ctrl.Result{}, err
		}

		// create service
		service := newService(&deployObject)
		if err := r.Create(ctx, service); err != nil {
			log.Error(err, "create service failed")
			return ctrl.Result{}, err
		}

		// update DeployObject spec annotation
		setDeployObjectSpecAnnotation(&deployObject)
		if err := r.Update(ctx, &deployObject); err != nil {
			log.Error(err, "update deployobject failed")
			return ctrl.Result{}, err
		}
		return ctrl.Result{}, nil
	}

	// 若 deployment 已存在, 则取出 DeployObject.Annotation 中之前的 DeployObject.Spec, 与当前DeployObject.Spec对比
	var prevSpec apiv1.DeployObjectSpec
	if err := json.Unmarshal([]byte(deployObject.Annotations["spec"]), &prevSpec); err != nil {
		log.Error(err, "parse spec annotation failed")
		return ctrl.Result{}, err
	}

	if !reflect.DeepEqual(deployObject.Spec, prevSpec) {
		// 与上一次的 Spec 有差异, 则需要更新 deployment & service
		newDeploy := newDeployment(&deployObject)
		var curDeploy appsv1.Deployment
		if err := r.Get(ctx, req.NamespacedName, &curDeploy); err != nil {
			log.Error(err, "get current deployment failed")
			return ctrl.Result{}, err
		}
		curDeploy.Spec = newDeploy.Spec
		if err := r.Update(ctx, &curDeploy); err != nil {
			log.Error(err, "update current deployment failed")
			return ctrl.Result{}, err
		}

		newService := newService(&deployObject)
		var curService corev1.Service
		if err := r.Get(ctx, req.NamespacedName, &curService); err != nil {
			log.Error(err, "get service failed")
			return ctrl.Result{}, err
		}
		clusterIP := curService.Spec.ClusterIP
		curService.Spec = newService.Spec
		curService.Spec.ClusterIP = clusterIP
		if err := r.Update(ctx, &curService); err != nil {
			log.Error(err, "update service failed")
			return ctrl.Result{}, err
		}

		// update DeployObject spec annotation
		setDeployObjectSpecAnnotation(&deployObject)
		if err := r.Update(ctx, &deployObject); err != nil {
			log.Error(err, "update deployobject failed")
			return ctrl.Result{}, err
		}
	} else {
		// 新旧 DeployObject.Spec 相同, 但 DeployObject 与 Deployment & Service 的设置有不同, 则需纠正 Deployment & Service 的设置
		// 对比 replicas / image / port/
		newDeploy := newDeployment(&deployObject)
		var curDeploy appsv1.Deployment
		if err := r.Get(ctx, req.NamespacedName, &curDeploy); err != nil {
			log.Error(err, "same spec, get current deployment failed")
			return ctrl.Result{}, err
		}

		newService := newService(&deployObject)
		var curService corev1.Service
		if err := r.Get(ctx, req.NamespacedName, &curService); err != nil {
			log.Error(err, "same spec, get current service failed")
			return ctrl.Result{}, err
		}

		if (*deployObject.Spec.Replicas != *curDeploy.Spec.Replicas) ||
			(deployObject.Spec.Image != curDeploy.Spec.Template.Spec.Containers[0].Image) {
			log.Info("same spec, update current deployment ...")
			curDeploy.Spec = newDeploy.Spec
			if err := r.Update(ctx, &curDeploy); err != nil {
				log.Error(err, "same spec, update current deployment failed")
				return ctrl.Result{}, err
			}
		}
		if !reflect.DeepEqual(deployObject.Spec.Ports, curService.Spec.Ports) {
			log.Info("same spec, update current service ...")
			clusterIP := curService.Spec.ClusterIP
			curService.Spec = newService.Spec
			curService.Spec.ClusterIP = clusterIP
			if err := r.Update(ctx, &curService); err != nil {
				log.Error(err, "same spec, update service deployment failed")
				return ctrl.Result{}, err
			}
		}
	}

	return ctrl.Result{}, nil
}

```

### 测试运行

执行如下命令将 `config/crd` 中 CRD 部署到 k8s 集群中:
```bash
$ make install
```

本地运行 controller:
```bash
$ make run
```

安装自定义资源(CR):
```bash
$ kubectl apply -f config/samples/
```

### 构建及部署

构建 docker 镜像及将镜像推送镜像仓库:
```bash
$ make docker-build docker-push IMG=<some-registry>/<project-name>:tag
```

使用 docker 镜像, 部署 controller 到 k8s 集群:
```bash
$ make deploy IMG=<some-registry>/<project-name>:tag
```

卸载 CRDs:
```bash
$ make uninstall
```

---

参考:

* https://book.kubebuilder.io/
* https://cloudnative.to/kubebuilder/
* https://juejin.cn/post/6844903952241131534