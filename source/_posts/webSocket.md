---
title: webSocket
cover: /covers/webSocket.jpg
date: 2020-04-04 17:32:39
category: 前端
tags: [webSocket, 后端]
excerpt: webSocket 简介及其实践
---

## 前言

&emsp;&emsp;现在很多网站的推送服务用的都是 ajax 轮询，有些还是每隔 1s 的短轮询，由浏览器对服务器发出 HTTP 请求，然后服务器返回最新的消息数据给浏览器端，这种模式的缺点很明显，就是浏览器要不停地向服务器发送请求，这样会浪费很多带宽等资源。而 HTML5 里新增了`webSocket`的技术,`webSocket`是单个 TCP 连接上进行全双工通讯的协议，这样服务器端也能主动向客户端推送数据了，弥补了 HTTP 协议的缺陷。而且浏览器和服务器只要建立了一次握手，就可以建立持久性的通信通道，可以进行双向数据传输。

## 特点

1.建立在 TCP 协议之上，服务器端的实现比较容易。

2.与 HTTP 协议有着良好的兼容性。默认端口也是 80 和 443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。

3.数据格式比较轻量，性能开销小，通信高效。

4.可以发送文本，也可以发送二进制数据。

5.没有同源限制，客户端可以与任意服务器通信。

6.协议标识符是 ws（如果加密，则为 wss），服务器网址就是 URL。

## 实践

&emsp;&emsp;而最近我结合`webSocket`，`koa`，`socket.io`和`mysql`做了一个聊天室的 web 页面，用到了`webSocket`，<a href="https://github.com/RuanXuSong/webSocket/tree/feat/chatRoom">聊天室 github 地址</a>其中有`React`版和`h5`版，目前`React`版还在做，小伙伴们可以先根据 readme 里的教程跑一下看看`h5`版，最近会更新`React`版哈。

&emsp;&emsp;以下为聊天室初版截图，由于还是初版所以样式还会在`React`版里改进：
![聊天室截图](/images/posts/webSocket/chatRoom.jpg)

## 代码及原理

> Talk is cheap. Show me the code.
> <span style="float:right">—— Linus Torvalds</span>

&emsp;&emsp;OK，主要的还是代码及原理部分。源代码都在 github 上了，我就不再全部贴出来了。先讲引入部分，我用的是`socket.io`框架，也是基于`webSocket`的，只是经过了一些封装。

### 定义

`socket.io`是一个允许实时，全双工，基于事件的，在服务器和客户端建立连接沟通的库，且基于`Node.js`。他有着`webSocket`类似的特性，而且他支持自动连接，也就是客户端会自动连接上服务器，除非进行配置`io({autoConnect: false})`。

```js
// 服务器端
// 引入koa
const Koa = require("koa");
const app = new Koa();
// 创建http服务
var server = require("http").createServer(app.callback());
// 给http封装成io对象
var io = require("socket.io")(server);
```

以上就是定义部分的代码，这个 io 对象里包含了监听事件的方法，在服务器端定义以后将通过引入服务器地址下的`/socket.io/socket.io.js`的`script`脚本暴露给客户端，相当于沟通服务器和客户端的桥梁。

```js
// 客户端
// 引入io对象
<script src="http://localhost:3000/socket.io/socket.io.js"></script>
```

以上为客户端代码，这样在客户端就可以调用服务器端暴露的`io`对象的`connect`方法，代码如下:

```js
const BASE_URL = "http://localhost:3000";

class Websocket {
  constructor() {
    this.socket;
  }
  // 连接到服务器
  handleConnect(callback) {
    try {
      const _this = this;
      this.socket = io.connect(BASE_URL);
      this.socket.on("connect", function() {
        const userName = localStorage.getItem("chatRoom-userName");
        const userStatus = localStorage.getItem("chatRoom-userStatus");
        chatRoom.userName = userName;
        $("#name-input").val(userName);
        if (userName && userStatus !== "online") {
          _this.handleSendSystemMessage(userName, "online");
        }
        // 执行回调
        callback();
      });
      this.socket.on("getMessage", function(data) {
        // 服务端收到信息回调
        callback();
      });
      this.socket.on("disconnect", function() {
        alert("服务器已断开连接");
        this.userName = "";
        this.userContent = "";
        this.socket = null;
      });
    } catch (err) {
      alert("服务器未启动!");
    }
  }
}

// 连接服务器
const handleConnect = content => {
  // 发送成功回调
  const callback = () => {
    const scrollHeight = $(".chat-list").prop("scrollHeight");
    // 获取聊天室数据
    chatRoom.fetchData();
    // 滚动到底部
    $(".chat-list").scrollTop(scrollHeight, 1000);
  };
  websocket.handleConnect(callback);
};
```

&emsp;&emsp;刚打开页面就执行`handleConnect()`,在以上代码中 `io.connect(BASE_URL)`是为了连接上服务器，并使用`socket`进行监听客户端的事件，并触发函数，比如:`socket.on("connect",func)`是监听客户端连接上服务器的事件，然后执行回调函数，`socket.on("getMessage",func)`是监听客户端接收到服务器端发出的`getMessage`事件，这个`getMessage`是自己定义的，后面会说到，`socket.on("disconnect",func)`是监听客户端断开连接的事件，其他的都是业务代码。

```js
// 客户端
// 点击发送内容给服务器
handleSendMessage(content) {
  if (this.socket) {
    this.socket.emit('sendMessage', content);
  } else {
    alert('请先连接到服务器！');
  }
}
```

&emsp;&emsp;如果客户端想要发送信息给服务器端，就可以触发事件`socket.emit`表示私发，在客户端只能私发给服务器端，而在服务器端则可以用`io.emit`表示群发给客户端，`socket.emit`表示私发给客户端。这里的`sendMessage`跟前面的`getMessage`一样都是自定义的。

```js
// 服务器端
// 建立链接
io.on("connection", function(socket) {
  // io.emit代表广播，socket.emit代表私发
  socket.on("sendMessage", async function(content) {
    const result = await mysql.addChatData(content);
    io.emit("getMessage", content);
  });
});
```

&emsp;&emsp;此时，服务器端收到了客户端的`sendMessage`事件后进行回应。服务器端的`io`对象有一个全局监听事件`connect`，当有客户端连接上以后，回调函数的传参会返回连接上的客户端的`socket`对象，就可以在回调函数里监听该客户端的对应事件，这里是`sendMessage`也就是前面说到的客户端发起的事件。在回调函数在会返回客户端的传参`content`，接着用这个`content`进行`mysql`的增删改查操作，然后服务器端调用`io.emit`进行群发(也可以用`socket.emit`私发)，所有的客户端上都会触发`getMessage`事件，并进行后续操作。

<p style="color:#F0412C">注:这里`sendMessage`和`getMessage`也可以叫别的名字，这个前端和后端需要约定好保持一致。</p>

&emsp;&emsp;至此，`webSocket`的运作原理已经讲完了，也可以配合常规的`HTTP`请求，如本例中的`fetchData`方法就是正常的`ajax`请求。
