title: "Go Micro 入门指南"
date: 2018-07-05 20:10:05
categories: "技术" 
tags: 
  - grpc
  - go
type: "tags"

---

`go-micro`是一种可插拔的RPC分布式系统微服务开发框架，这一章主要介绍一下入门的相关内容。

<!--more-->

### 服务发现依赖项
首先 `micro` 这个框架需要且依赖于服务发现工具(service discovery)，框架默认的服务发现工具是 `Consul` ，同时框架的插拔机制也可确保能够切换到其他的服务发现工具上，如Etcd、NATS等，详见 [`micro plugins`](https://github.com/micro/go-plugins)；

关于`consul`的安装与运行：
``` bash
$ brew install consul   # 安装
$ consul agent -dev     # 运行
```

### gRPC的安装
由于 `micro` 框架基于 `gRPC`，故需要安装相关依赖项，同时安装时确保 `go version` 版本在`1.6`以上；

#### 安装`gRPC`
``` bash
$ go get -u google.golang.org/grpc
```
#### 安装Protocol Buffers v3

##### 安装用于生成`gRPC`服务代码的`protoc`编译器工具；

最简单的方式是通过在[protobuf工具文件列表](https://github.com/google/protobuf/releases)中，以查看`protoc-<version>-<platform>.zip`文件名格式的方式找出符合你当前版本及系统平台的安装包，找到后下载解压放置于`bin`目录、修改环境变量PATH等；

##### 同时安装protoc go插件
``` bash
$ go get -u github.com/golang/protobuf/protoc-gen-go
```

### Micro相关安装

#### 框架包的安装
``` bash
$ go get github.com/micro/go-micro
```

#### protoc micro插件安装
用于通过`.proto`文件生成`.micro.go`代码文件
``` bash
$ go get github.com/micro/protoc-gen-micro
```

#### micro工具包的安装
``` bash
$ go get github.com/micro/micro
```
micro 命令行工具可以提供诸如服务列表查看、服务详情查看、调用服务接口等功能；

### 实践一下

#### 首先，创建一个`user.proto`结构文件
``` proto
syntax = "proto3";

service User {
    rpc Hello(Request) returns (Response) {}
}

message Request {
    string name = 1;
}

message Response {
    string msg = 1;
}
```
#### 其次，通过proto文件编译生成代码
``` bash
protoc --proto_path=$GOPATH/src:. --micro_out=. --go_out=. user.proto
```
此时，通过`protoc`工具对`user.proto`文件的编译，生成了两个go文件:

* user.micro.go
* user.pb.go

#### 然后，分别编写服务端及客户端代码

##### 服务端 `service.go`
``` go
package main

import (
	"context"
	"fmt"

	proto "github.com/ErikJiang/coin_exchange/ms-user/proto"
	micro "github.com/micro/go-micro"
)

type User struct{}

func (u *User) Hello(ctx context.Context, req *proto.Request, res *proto.Response) error {
	res.Msg = "Hello " + req.Name
	return nil
}

func main() {
	service := micro.NewService(
		micro.Name("user"),
	)

	service.Init()

	proto.RegisterUserHandler(service.Server(), new(User))

	if err := service.Run(); err != nil {
		fmt.Println(err)
	}
}
```

##### 客户端 `client.go`
``` go
package main

import (
	"context"
	"fmt"

	proto "github.com/ErikJiang/coin_exchange/ms-user/proto"
	micro "github.com/micro/go-micro"
)

func main() {
	service := micro.NewService(micro.Name("user.client"))
	service.Init()

	user := proto.NewUserService("user", service.Client())

	res, err := user.Hello(context.TODO(), &proto.Request{Name: "World ^_^"})
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println(res.Msg)
}
```

#### 最后，运行起来

##### 首先启动服务发现
``` bash
$ consul agent -dev
```
可通过访问：http://127.0.0.1:8500/ui/ 查看服务；

##### 启动服务端程序
``` bash
$ go run service.go
```
此时刷新：http://127.0.0.1:8500/ui/ 页面会发现新服务`user`；

##### 执行客户端程序，调用接口
``` bash
$ go run client.go
```

---

* [GRPC Quick Start](https://grpc.io/docs/quickstart/go.html#install-protocol-buffers-v3)
* [micro install guide](https://micro.mu/docs/install-guide.html)