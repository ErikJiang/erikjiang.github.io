title: "Python开发环境预备"
date: 2018-01-28 16:10:05
categories: "技术" 
tags: 
  - pyenv
  - pipenv
  - autoenv 
type: "tags"
---

> 以`pyenv`及`pipenv + autoenv`的方式开发Python;

<!--more-->

所涉及到的工具有：
1. [pyenv 版本管理](https://github.com/pyenv/pyenv)
2. [pipenv 开发工作流管理](https://github.com/pypa/pipenv)
3. [autoenv 基于目录的环境自动构建](https://github.com/kennethreitz/autoenv)

## Python版本管理

`Pyenv` 是一种可以实现安装不同版本`Python`并可在不同版本间任意切换的管理工具。
其功能类似于在`Node.JS`中的`NVM`版本管理工具；

## Python项目的环境及包管理

`Pipenv`是python项目中的包管理工具，通过`pipenv`安装的pyhon包将被记录于`pipfile`当中，方便项目中包的依赖管理及安装部署;
`Pipenv`工具是集pip, pipfile, virtualenv于一身，故它也实现了项目目录的虚拟环境创建，使得其项目的运行环境能够满足独立且隔离，不受系统及其他环境的影响;
`Pipenv`功能类似于在`Node.JS`中的`NPM`包管理工具;
``` shell

# 1. 在第一次执行`pipenv install`时，会在该项目创建对应的虚拟环境;
# 2. 同时生成Pipfile及Pipfile.lock文件，用于管理项目中Python包依赖;
# 3. 第一次安装依赖包,需要指定`--two`或`--three`参数指定python2/3.x的运行环境

$ pipenv install --three <packageName>
```

## Python项目环境的自启动

`Autoenv`可以实现在进入项目目录时，将自启动需要的运行环境;

1 . 安装autoenv
使用pip方式安装存在问题，会导致terminal异常退出，之后的版本可能会有更新;
目前是以`git`方式进行安装：
``` shell
$ git clone git://github.com/kennethreitz/autoenv.git ~/.autoenv
$ echo 'source ~/.autoenv/activate.sh' >> ~/.bashrc
```

2 . 创建`.env`文件
在已有的pipenv项目中运行`pipenv shell`,
会得到虚拟环境路径信息，通过信息创建`.env`运行环境文件;

``` shell
$ echo "source /home/{userName}/.local/share/virtualenvs/{projectName}/bin/activate" > {projectName}/.env
```

3 . 再次进入项目目录，将自启动该项目的虚拟环境

---

