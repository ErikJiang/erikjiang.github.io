title: "Node调试工具的使用"
date: 2015-05-11 22:22:57
categories: "技术" 
tags: [Node,Debug] 
---

今天发现一个在浏览器上调试node代码的工具，分享一下.
这个工具名叫：[Node Inspector](https://github.com/node-inspector/node-inspector)
感兴趣的话可以点链接到Github看看详细文档说明，这里就简要介绍一二；
<!--more-->

#### 1、安装步骤
安装命令：
``` bash
npm install –g node-inspector
```

#### 2、运行步骤
1）首先，需要启动node.js应用
命令如下：
``` bash
node --debug-brk=[app port] [app name]   
```
或者
``` bash
node --debug=[app port] [app name]
```
例如：
``` bash
node --debug-brk=3001 bin/www
```
[debug与debug-brk两种配参见[说明](https://github.com/node-inspector/node-inspector#troubleshooting)]

2）然后，启动node-inspector调试工具：
``` bash
node-inspector --web-port [web port] --debug-port [app port]
```

例如：
``` bash
node-inspector --web-port 8081 --debug-port 3001
```
[注：8081为web访问端口；3001为应用调试端口]


3)最后，访问chrome浏览器：
具体访问的网址，其实在启动调试工具后会有输出提示，大致如下：
``` bash
http://127.0.0.1:8081/debug?ws=127.0.0.1:8081&port=3001
```

---
具体也可参考如下：
1. [百度经验](http://jingyan.baidu.com/article/dca1fa6fbd580ff1a44052de.html)
2. [用node-inspector调试Node.js](http://www.noanylove.com/2011/12/node-the-inspector-debugging-node-js/)



