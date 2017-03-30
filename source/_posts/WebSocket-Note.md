title: "WebSocket学习笔记"
date: 2016-01-19 13:54:31
categories: "技术" 
tags: [websocket,socket.io]
---

HTTP 协议是一种无状态协议，服务器端本身不具备识别客户端的能力，必须借助外部机制，比如 session 和 cookie 才能与特定客户端保持对话。那么如果我们遇到一种需求，它要求保证服务器端与客户端持续的数据交换，比如网络聊天，那再使用 HTTP 协议就会有不便，这时我们就需要用到 Websocket API了。

<!--more-->

---

###  概述 ：

WebSocket 的主要作用：允许服务器端与客户端进行全双工（full-duplex）的通信。
举例来说，
HTTP 协议有点像发电子邮件，发出后必须等待对方回信；
WebSocket 则是像打电话，服务器端和客户端可以同时向对方发送数据，它们之间存着一条持续打开的数据通道。

WebSocket 协议完全可以取代 Ajax 方法，用来向服务器端发送文本和二进制数据，而且还没有“同域限制”。

WebSocket 不使用 HTTP 协议，浏览器在发送 Websocket 请求时，会将 HTTP 协议升级（Upgrade）为 webSocket 自己的协议。

> WebSocket 协议用 ws 表示。此外，还有 wss 协议，表示加密的 WebSocket 协议，对应 HTTPs 协议。

在完成TCP协议三次握手之后，WebSocket 协议就在 TCP 协议之上，开始数据的传送。

WebSocket 协议需要服务器支持，目前比较流行的实现是基于 node.js 的 socket.io ，至于浏览器端，目前主流浏览器都支持 WebSocket 协议。

 
###  客户端 ：

浏览器端对WebSocket协议的处理，主要归纳为三件事：

* 建立连接和断开连接
* 发送数据和接收数据
* 处理错误

##### 建立连接和断开连接

首先，客户端要检查浏览器是否支持 WebSocket ，使用的方法是查看 window 对象是否具有 WebSocket 属性。

	if(window.WebSocket != undefined) {
    	// WebSocket代码
	}

然后，开始与服务器建立连接（这里假定服务器就是本机的1740端口，需要使用ws协议）。

	if(window.WebSocket != undefined) {
    	var connection = new WebSocket('ws://localhost:1740');
	}

建立连接以后的 WebSocket 实例对象（即上面代码中的connection），有一个readyState属性，表示目前的状态，可以取4个值：

	0： 正在连接
	1： 连接成功
	2： 正在关闭
	3： 连接关闭

握手协议成功以后，readyState 就从 0 变为 1 ，并触发 open 事件，这时就可以向服务器发送信息了。我们可以指定 open 事件的回调函数。

	connection.onopen = wsOpen;

	function wsOpen (event) {
    	console.log('Connected to: ' + event.currentTarget.URL);
	}
	
关闭 WebSocket 连接，会触发 close 事件。

	connection.onclose = wsClose;

	function wsClose () {
    	console.log("Closed");
	}

	connection.close();


##### 发送数据和接收数据

连接建立后，客户端通过 send 方法向服务器端发送数据。

	connection.send(message);
	
除了发送字符串，也可以使用 Blob 或 ArrayBuffer 对象发送二进制数据。

	// 使用ArrayBuffer发送canvas图像数据
	var img = canvas_context.getImageData(0, 0, 400, 320);
	var binary = new Uint8Array(img.data.length);
	for (var i = 0; i < img.data.length; i++) {
    	binary[i] = img.data[i];
	}
	connection.send(binary.buffer);

	// 使用Blob发送文件
	var file = document.querySelector('input[type="file"]').files[0];
	connection.send(file);
	
客户端收到服务器发送的数据，会触发 message 事件。可以通过定义 message 事件的回调函数，来处理服务端返回的数据。

	connection.onmessage = wsMessage;

	function wsMessage (event) {
    	console.log(event.data);
	}
	
上面代码的回调函数 wsMessage 的参数为事件对象 event ，该对象的 data 属性包含了服务器返回的数据。

如果接收的是二进制数据，需要将连接对象的格式设为 blob 或 arraybuffer。

	connection.binaryType = 'arraybuffer';

	connection.onmessage = function(e) {
  		// ArrayBuffer对象有byteLength属性
  		console.log(e.data.byteLength); 
	};

##### 处理错误

如果出现错误，浏览器会触发WebSocket实例对象的error事件。

	connection.onerror = wsError;

	function wsError(event) {
    	console.log("Error: " + event.data);
	}
	

###  服务端 ：

服务器端需要单独部署处理 WebSocket 的代码。下面用 node.js 搭建一个服务器环境。

	var http = require('http');
	var server = http.createServer(function(request, response) {});

假设监听 1740 端口。

	server.listen(1740, function() {
    	console.log((new Date()) + ' Server is listening on port 1740');
	});
	
接着启动 WebSocket 服务器。这需要加载websocket库，如果没有安装，可以先使用npm命令安装。

	var WebSocketServer = require('websocket').server;
	var wsServer = new WebSocketServer({
    	httpServer: server
	});
WebSocket服务器建立 request 事件的回调函数。

	var connection;

	wsServer.on('request', function(req){
    	connection = req.accept('echo-protocol', req.origin);
	});
	
上面代码的回调函数接受一个参数 req ，表示 request 请求对象。然后，在回调函数内部，建立WebSocket连接connection。接着，就要对connection的message事件指定回调函数。

	wsServer.on('request', function(r){
    	connection = req.accept('echo-protocol', req.origin);

    	connection.on('message', function(message) {
        	var msgString = message.utf8Data;
        	connection.sendUTF(msgString);
    	});
	});
	
最后，监听用户的 disconnect 事件。

	connection.on('close', function(reasonCode, description) {
    	console.log(connection.remoteAddress + ' disconnected.');
	});
	
使用 [ws](https://github.com/websockets/ws) 模块，部署一个简单的 WebSocket 服务器非常容易。

	var WebSocketServer = require('ws').Server;
	var wss = new WebSocketServer({ port: 8080 });

	wss.on('connection', function connection(ws) {
  		ws.on('message', function incoming(message) {
    		console.log('received: %s', message);
  		});

  		ws.send('something');
	});

###  Socket.IO ：

Socket.io 是目前最流行的 WebSocket 实现，包括服务器和客户端两个部分。它不仅简化了接口，使得操作更容易，而且对于那些不支持 WebSocket 的浏览器，会自动降为 Ajax 连接，最大限度地保证了兼容性。它的目标是统一通信机制，使得所有浏览器和移动设备都可以进行实时通信。

第一步，在服务器端的项目根目录下，安装 socket.io 模块。

	$ npm install socket.io
	
第二步，在根目录下建立 app.js ，并写入以下代码（假定使用了Express框架）。

	var app = require('express')();
	var server = require('http').createServer(app);
	var io = require('socket.io').listen(server);

	server.listen(80);

	app.get('/', function (req, res) {
  		res.sendfile(__dirname + '/index.html');
	});
	
上面代码表示，先建立并运行 HTTP 服务器。Socket.io 的运行建立在 HTTP 服务器之上。

第三步，将 Socket.io 插入客户端网页。

	<script src="/socket.io/socket.io.js"></script>
	
然后，在客户端脚本中，建立 WebSocket 连接。

	var socket = io.connect('http://localhost');
	
由于本例假定 WebSocket 主机与客户端是同一台机器，所以 connect 方法的参数是 http://localhost。接着，指定 news 事件（即服务器端发送news）的回调函数。

	socket.on('news', function (data){
   		console.log(data);
	});
	
最后，用 emit 方法向服务器端发送信号，触发服务器端的 anotherNews 事件。

	socket.emit('anotherNews');
	
> 请注意，emit 方法可以取代 Ajax 请求，而on方法指定的回调函数，也等同于 Ajax 的回调函数。

第四步，在服务器端的 app.js，加入以下代码。

	//广播信息给除当前用户之外的用户
    socket.broadcast.emit('user connected');
    
    //广播给全体客户端
	io.sockets.on('connection', function (socket) {
  		socket.emit('news', { hello: 'world' });
  		socket.on('anotherNews', function (data) {
    		console.log(data);
  		});
	});
	
上面代码的 io.sockets.on 方法指定 connection 事件（ WebSocket 连接建立）的回调函数。在回调函数中，用 emit 方法向客户端发送数据，触发客户端的 news 事件。然后，再用 on 方法指定服务器端anotherNews 事件的回调函数。

> 不管是服务器还是客户端，socket.io 提供两个核心方法：
emit方法用于发送消息，on方法用于监听对方发送的消息。


---
参考：

* [JavaScript标准参考教程](http://javascript.ruanyifeng.com/htmlapi/websocket.html "JavaScript标准参考教程")
