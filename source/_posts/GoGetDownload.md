title: "长城下的Go Get, 被安排的明明白白"
date: 2018-07-20 12:47:02
categories: "技术" 
tags: 
  - go
type: "tags"

---

在`nodejs`开发中，包的安装，习惯了`npm install`的流畅，再不济也可以用`cnpm`代替；

但是，当进行`golang`开发时，由于golang.org及其他相关开发站点被GFW屏蔽，就不再那么好运了，而这里主要记录了一些方法，希望能让`go get`安装时显得不那么痛苦；

<!--more-->

### 人肉搬运大法

比如 `golang.org/x/xxx` 这种包，其实已被`golang`团队托管到了`github`;

需要做的就是在 `github.com/golang` 找到对应的源码并手动下载安装，
下载的方式有多种，可以用`git clone`或者直接下载源码压缩文件，
下载完的源码，需要放入`GOPATH`的src对应路径中；

比方说`golang.org/x/crypto`这个包，当源码下载完成后，需要创建目录`$GOPATH/src/golang.org/x/`，并将该源码包放入即完成；

当然有些网站的源码包需要翻墙才能下载，如果有代理或VPN则直接访问下载即可，但当没有这些工具时，该怎么办呢？其实还是有一些办法的，比如，使用第三方网站提供的下载服务，这里列出两个：

* [gopm.io package](https://gopm.io/download)
* [golangtc package](https://golangtc.com/download/package)


### 代理安装大法

首先，需要用到`shadowsocks`翻墙，但是由于`shadowsocks`基于`socks5`协议，而我们的`go get`基于`http`协议，故即使开启`shadowsocks`也依然无法安装下载golang包，所以还需要一个可以将socks5转为http的代理工具；

这里我们使用的是[`COW`](https://github.com/cyfdecyf/cow)，即通过它完成socks5转为http；

安装方法见该github中`REAMD.md`说明；

#### 安装完成后，需要在其配置`$HOME/.cow/rc`中修改如下：
``` bash
listen = http://127.0.0.1:7777  #默认已存在
proxy = socks5://127.0.0.1:1080
```

#### 然后在`~/.profile`中添加环境变量：
``` bash
export http_proxy=http://127.0.0.1:7777
export https_proxy=http://127.0.0.1:7777
```

#### 再然后启用环境变量：
``` bash
$ source ~/.profile # 启用环境变量
$ echo $http_proxy  # 查看是否生效
```

#### 最后，后台启动cow：
```bash
$ cow &
```

一切配置完毕，就可以愉快的使用`go get`安装下载需要的依赖包了；

### 总结最后

说了这么多，其实还是希望不要这么麻烦，但愿谷歌早日重返中国吧，至于李彦宏能不能赢就不知道了....

---
