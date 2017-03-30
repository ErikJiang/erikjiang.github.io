title: "MEAN学习笔记"
date: 2015-04-08 17:22:59
categories: "技术" 
tags: [MEAN]
---

(M)ongoDB——NoSQL的文档数据库，使用JSON风格来存储数据，甚至也是使用JS来进行sql查询；
(E)xpress——基于Node的Web开发框架；
(A)agular——JS的前端开发框架，提供了声明式的双向数据绑定；
(N)ode——基于V8的运行时环境（JS语言开发），可以构建快速响应、可扩展的网络应用。

<!--more-->

---
####  Express安装：

1、如果需要安装express 3.x版本，可直接在待安装的包名后添加@版本数，命令行如下：
``` bash
$ npm install -g express-generator@3
```

2、Express 4.x版本后，将express命令行工具分离了出来，我们使用全局方式安装时，其命令行如下：
``` bash
$ npm install -g express-generator
```

3、在安装结束，我们想要创建一个应用app并进入工程目录：
``` bash
$ express -e app && cd app
```

4、初始化依赖环境：
``` bash
$ npm install
```

5、运行这个应用：
``` bash
$ npm start
```

####  MongoDB安装：

这里没有用源码进行安装，用的是HomeBrew，它是一个缺省包管理器，类似于apt-get、yum这样的工具。

1、我们首先需要更新一下homebrew的包数据库，这个过程可能需要几分钟，终端命令如下：
``` bash    
$ brew update
```

2、可以安装MongoDB了，命令行如下：
``` bash
$ brew install mongodb
```

3、安装成功后，终端会提示如何启动，大概命令如下：
``` bash
$ mongod --config /usr/local/etc/mongod.conf
```

4、若已启动，可以再启一个终端，命令行中键入“mongo”即可直接连接到MongoDB；

---

####  问题汇总：
######1、新工程创建标准：
    
创建一个标准的package.json文件：
``` bash
$ npm init
```
该命令将以文本互动的方式生成一份简易的 package.json 文件；

工程内模块的依赖生成；
``` bash
$ npm install modules_name --save
```
通过 --save 参数，就能够在你安装模块的同时，自动将模块的依赖写入 package.json。
安装完成后，查看 package.json，会多了一项 dependencies 字段，该字段内将罗列具体的依赖模块；

######2、package.json中的dependencies字段与devDependencies字段：

node package存在两种依赖项，一种是dependencies，而另一种是devDependencies，
其中前者的依赖项是正常运行该包时所需要的依赖项，而后者则是进行开发时所需要的依赖项，如一些单元测试的模块包。
``` javascript
"dependencies": {}      //生产环境
"devDependencies": {}   //开发环境
```
在package.json所在目录执行npm install的时候，devDependencies里的模块同样会被安装。
如果我们只想安装dependencies里面的包，可以执行
``` bash
npm install –production
```
如果只安装devDependencies，可以执行
``` bash
npm install –dev
```
同理
自动更新dependencies字段值的依赖模块：
``` bash
npm install node_module –save
```
自动更新devDependencies字段值的依赖模块：
``` bash
npm install node_module –save-dev。
```

######3、Most middleware (like session) is no longer bundled with Express and must be installed ……
解决方法：
大多数的中间件如session，在Express4.x版本之后不会再随着Express包本身一并被安装，而是被分离出来；原本3.x版本的写法，在这里将不再适用：
``` javascript
var express = require('express');
var MongoStore = require('connect-mongo')(express);

app.use(express.session({
    secret: settings.cookie_secret,
    store: new MongoStore({
        db: settings.db
    })
}));
```
正确的做法应该首先安装中间件express-session，在package.json文件中添加中间件包名及版本，
然后，命令行通过 npm install 来安装；
最后修改代码：
``` javascript
var session    = require('express-session');
var MongoStore = require('connect-mongo')(session);

app.use(session({
    secret: settings.cookie_secret,
    store: new MongoStore({
        db: settings.db
    })
}));
```

######4、Redis的安装：
到[官网](http://redis.io/download)下载最新源码，或者通过命令下载：
```bash
$ wget http://download.redis.io/releases/redis-3.0.1.tar.gz
```

解压源码包并编译安装：
```bash
$ tar xzf redis-3.0.1.tar.gz
$ cd redis-3.0.1
$ make
```

这里比较有意思的是，redis的源码编译与安装只需“make”即可，
不像之前编译其他源码要再进行一步“make install”才能安装；
安装好后将会在/usr/local/bin/路径下看得到几个命令执行程序：
```bash
redis-benchmark	
redis-cli
redis-check-aof	
redis-sentinel
redis-check-dump	
redis-server
```

---


