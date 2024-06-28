---
title: 从Websocket到内网穿透工具
date: 2024-06-27 13:39:11
tags:计算机网络
---



最近工作上接触到了Websocket这个东西，由此引发了一系列的问题，写篇文章记录一下涉及的技术。

## 为什么有Websocket？

初见Websocket可能会联想到socket套接字，用我看到文章里的说法来解释两者的关系，大概就是雷锋和雷锋塔的关系，两者几乎没有关系。

其实Wesocket是一个协议，属于应用层，提到应用层协议首先想到的是HTTP，对于我来说，我就只知道HTTP这一个应用层协议，既然有了HTTP，为什么还需要Websocket呢？

因为HTTP有一个缺陷，通信只能由客户端发起。

客户端发送请求后，服务器才能给客户端响应消息，这种单向性导致如果客户端要持续获得服务器的状态变化，就得每隔一段时间向服务器发送请求，这种方式称作轮询。

轮询的弊端显而易见，需要额外的资源来进行轮询，即使服务器状态没有发生变化。

众所周知，HTTP是基于TCP的，而TCP是全双工的，但HTTP是半双工的，这是因为HTTP起初是为看网页文本的场景设计的，没有考虑客户端和服务器之间需要大量互发数据。

为了支持这种场景，就有了Websocket。

## 什么是Websocket?

基于TCP的应用层协议，支持全双工通信，协议标识符是`ws`(加密的话是`wss`)。

```
ws://example.com:80/some/path
```

其报文结构我就不细说（不太关心，也不太了解），可以讲讲连接建立的方式。

首先肯定是通过TCP三次握手建立连接，然后使用HTTP协议进行通信，如果想要建立Websocket通信，在这次HTTP通信中，就会带上特殊header头，接着就会升级到Websocket协议。

```
Connection: Upgrade
Upgrade: WebSocket
Sec-WebSocket-Key: T2a6wZlAwhgQNqruZ2YUyg==\r\n //随机生成的 base64 码
```

## 如何搭建一个Websocket服务器

工作需要，需要搭建一个Websocket服务器，在GPT帮助下也是易如反掌。

1. 安装Node.js

   ```bash
   sudo apt update
   sudo apt install nodejs npm
   ```

2. 创建服务器工程

   ```bash
   mkdir websocket-server
   cd websocket-server
   npm init -y
   ```

3. 安装ws库

   ```bash
   npm install ws
   ```

4. 创建服务器文件

   在你的项目目录中创建一个名为 server.js 的文件，并添加以下代码：

   ```js
   const WebSocket = require('ws');
   const wss = new WebSocket.Server({ host: 'localhost', port: 8080 });
   wss.on('connection', function connection(ws) {
   console.log('Client connected');
   ws.on('message', function incoming(message) {
   console.log('received: %s', message);
   // Echo the message back to the client
   ws.send(`Echo: ${message}`);
   });
   ws.on('close', function close() {
   console.log('Client disconnected');
   });
   });
   console.log('WebSocket server is running');
   ```

5. 运行服务器

   ```bash
   node server.js
   ```

   现在，你的WebSocket服务器应该在端口8080上运行。你可以使用任何支持WebSocket的客户端来连 接到这个服务器，例如WebSocket客户端或者浏览器。

## 如何在公网上访问本地Websocket服务器

使用上述流程搭建的服务器是在本地的8080端口上，如何让其他人也能访问到你的服务器呢？

当然可以搭建一个公网服务器，但是为了工作还要付出额外开销是不可能的，于是可以采用内网穿透的方式来解决这个问题。

> 内网穿透，也称为NAT穿透，是一种网络技术，它允许位于私有网络（内网）中的计算机或设备上的服务可以被外部网络（如互联网）上的其他计算机访问。这通常用于绕过NAT（网络地址转换）和防火墙的限制，因为大多数家庭和小型办公室网络都使用NAT来共享一个公共IP地址。

我们可以使用 `localtunnel `工具来实现内网穿透。

```bash
npm i -g localtunnel
lt -p 8080
```

-p 参数后面跟着你的本地服务器端口，运行后，会分配一个随机唯一的公网URL，如果想要固定的 URL，请使用 -s 参数，后面跟着你设置的域名。

```bash
lt -p 8080 -s forwstest
```

GPT提示我说，这样方式暴露的服务器的安全性没法得到保障，很有可能被人反向工具，只建议测试使用。

### 参考

[3.9 既然有 HTTP 协议，为什么还要有 WebSocket？ | 小林coding (xiaolincoding.com)](https://xiaolincoding.com/network/2_http/http_websocket.html#使用-http-不断轮询)

[WebSocket 教程 - 阮一峰的网络日志 (ruanyifeng.com)](https://www.ruanyifeng.com/blog/2017/05/websocket.html)

[Localtunnel ~ Expose yourself to the world](https://localtunnel.github.io/www/)