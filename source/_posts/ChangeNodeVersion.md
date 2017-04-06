title: "关于Node版本切换"
date: 2015-12-22 12:06:55
categories: "技术"
tags: [NVM] 
---

今天在安装yeoman时，警告提示node版本过低，那么好吧，咱们就来更新node版本。
顺带把整个过程梳理一遍，做一个有节操的程序猿。

<!--more-->

---

提到node的安装，就不得不说这个 NVM （ Node Version Manager ）Node版本管理器，
一般情况下，不建议直接下载node安装程序进行安装，为了便于管理更新频繁的node，
比较妥当的做法是通过nvm工具进行安装，而nvm工具本身的安装就不多加细究，
在Mac上的方式是通过homebrew管理工具进行安装的，其他平台可Google寻找答案。

#### NVM的安装及简单使用：
Mac上安装nvm的命令：
``` bash
$ brew install nvm
```

我们可以查看nvm是否正常安装：
``` bash
$ nvm —version
```

如果安装正常，我们可以通过如下命令，查看当前哪些版本可供安装：
``` bash
$ nvm ls-remote
```

然后可以通过下面命令安装指定版本的node：
``` bash
$ nvm install <version>
```

同样我们可以卸载指定版本的node：
``` bash
$ nvm uninstall <version>
```
查看当前系统中可管理的所有版本node信息：
``` bash
$ nvm ls
```

---

#### Node安装及版本切换：

我这里安装了v0.12.9版本：
``` bash
$ nvm install v0.12.9
#################################################### 100.0%
Now using node v0.12.9
```

当前我使用v0.12.9版本即可，但如果此时我需要切换至其它版本，就可以使用如下命令：
``` bash
$ nvm use <version>
```

但是使用 nvm use 命令有一个问题，即只对当前shell窗口生效，
如果我们此时关闭该shell窗口，重新打开命令行就会发现node的版本又恢复到之前的版本状态；
所以针对这种状况，我们需要设置一个默认node版本，命令如下：
``` bash
$ nvm alias default <version>
```
此时打开shell窗口，就会使用默认版本的node执行；

噢，对了，更新了node版本过后，别忘记更新npm：
``` bash
npm install -g npm
```



