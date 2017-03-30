title: "NodeJS应用部署初探"
date: 2016-01-13 21:13:51
categories: "技术"
tags: [MongoLab,Redis4You,Heroku]
---

尝试使用 MongoLab、Redis4You、Heroku 来部署一个 node 应用。

<!--more-->

---

###  首先，使用 MongoLab ：

MongoLab 与 MongoHQ 是当今比较流行的 MongoDB 云数据库提供商。MongoLab 的免费套餐提供了 500MB 的存储空间，MongoLab 的使用比较简单明了，这里简要概述一二。

##### 注册

[mongolab.com/signup](https://mongolab.com/signup/ "signup")

##### 创建数据库

注册成功后，此时进入到自己的控制面板页，点击 `Create new` 创建一个数据库。在 `Plan` 选项选择 `Single-node` 的 `Sandbox` 免费套餐。在 `Darebase name` 除填写数据库的名字，点击 `Create new MongoDB deployment` 完成创建。

创建成功后跳转到了控制面板页，点击刚才创建的数据库进入该数据库配置页面。我们需要先为数据库创建一个拥有读写权限的用户，点击 `Click here` 添加一个用户。

打开 mongodb.js ，将：

	mongoose.connect('mongodb://localhost/drifter', {server: {poolSize: 10}});
	
修改为：

	mongoose.connect('your_mongolab_url', {server: {poolSize: 10}});
	
注意：将 `<dbuser>` 和 `<dbpassword>` 分别替换为刚才为数据库创建的用户名和密码。

###  然后，使用 Redis4You ：

redis4you 是一家 Redis 云服务提供商，提供多个档次的 Redis 实例供选择，所有类型的服务包括免费的 5M 试用版，都提供诸如多 DB，持久化等功能。这里我们仅供测试使用。

##### 注册

[redis4you.com/login_new](http://redis4you.com/login_new.php "redis4you.com")

##### 创建数据库

验证邮箱完成注册。登录 Redis4You，点击右上角的 `My Redis Instanses` 进入 `Redis` 管理页面，点击 `Provision new instance` 创建一个新的数据库。点击 5MB 的免费套餐右边的 `Select this instance` 完成创建。

此时跳转到 Redis 管理页面，点击 `Information` 查看数据库详情。我们只会用到 Host、Port 和 Auth 这三项，其中 Auth 为随机生成的访问数据库的凭证。

注意：一定要点击左上角的 `start` 启动数据库。

打开 redis.js ，将：

	var client = redis.createClient();
	
修改为：

	var client = redis.createClient(Port, Host, {auth_pass: Auth});
	
注意：将 Host、Port 和 Auth 分别替换为给定的值。
	
###  最后，部署到 HeroKu ：

Heroku 是一个主流的 PaaS 提供商，在开发人员中广受欢迎。这个服务围绕着基于 Git 的工作流设计，假如你熟悉 Git ，那部署就十分简单。这个服务原本是为托管 Ruby 应用程序而设计的，但 Heroku 之后加入了对 Node.js 、Clojure 、Scala 、Python 和 Java 等语言的支持。Heroku 的基础服务是免费的。

##### 注册

[heroku.com](https://www.heroku.com/ "heroku.com")

##### 创建一个应用

右上角 “＋” 中点击 `Create a new app` ，填写独一无二的应用名称后，点击 `creat app` 即创建成功，然后点击 `Open app` 即可打开应用 URL 链接 。

##### 安装 Heroku Toolbelt

Heroku 官方提供了 Heroku Toolbelt 工具更方便地部署和管理应用。它包含三个部分：

* Heroku client ：创建和管理 Heroku 应用的命令行工具
* Foreman ：一个在本地运行你的 app 的不错的选择
* Git ：分布式版本控制工具，用来把应用推送到 Heroku

Heroku Toolbelt 下载地址：[toolbelt.heroku.com](https://toolbelt.heroku.com/ "toolbelt.heroku.com") 。

注意：假如你的电脑上已经安装了 Git ，那么在安装的时候选择 `Custom Installation` 并去掉安装 Git 的选项，否则选择 `Full Installation` 。

安装成功后，打开终端 shell ，输入 `heroku login` ，然后输入在 Heroku 注册的帐号和密码进行登录。Git 会检测是否有 SSH 密钥，如果有，则使用此密钥并上传，如果没有，则创建一个密钥并上传。

Tips：SSH 密钥通常用于授予用户访问服务器的权限。可将它们用于某些配置中，以便无需密码即可访问服务器。许多 PaaS 提供商都使用了此功能。

##### Procfile

在工程的根目录下新建一个 Procfile 文件，添加如下内容：

	web: ./bin/www
	
Procfile 文件告诉了服务器该使用什么命令启动一个 web 服务，这里我们通过 `./bin/www` 执行 Node 脚本。为什么这里声明了一个 web 类型呢？官方解释为：

> The name “web” is important here. It declares that this process type will be attached to the HTTP routing stack of Heroku, and receive web traffic when deployed.


##### 上传应用

首先 clone 一份 git 仓库源码到你的本地机器中；

	$ heroku git:clone -a web_app_name
	$ cd web_app_name
	
使用 git 将源码变动的部分更新以及部署到 heroku 当中.

	$ git add .
	$ git commit -am "make it better"
	$ git push heroku master

在 push 到 heroku 服务器之前，我们还需要做一个工作。由于我国某些政策的原因，我们需到 ~/.ssh/ 目录下，新建一个 config 文件，内容如下：

	Host git.heroku.com
	User yourName
	Hostname 107.21.95.3
	PreferredAuthentications publickey
	IdentityFile ~/.ssh/id_rsa
	port 22
	
然后回到 Git Bash ，输入：

	$ git push heroku master
	
稍等片刻即上传成功。

---

参考：

* [nswbmw github](https://github.com/nswbmw "nswbmw")
