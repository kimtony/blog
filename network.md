### webscoket与socket.io的区别
```
协议 vs. 库：WebSocket 是一个协议，而 Socket.IO 是一个库。
功能：WebSocket 提供了基本的双向通信功能，而 Socket.IO 提供了更多的高级功能，如自动重连、事件处理、命名空间和房间支持。
兼容性：Socket.IO 通过降级机制提供了更好的兼容性和稳定性。
使用场景：如果需要基本的实时通信并且网络环境稳定，可以使用 WebSocket。如果需要更强大的功能、更好的兼容性和更易用的 API，可以使用 Socket.IO。 
```


### TCP 的主要特性
* 面向连接：在传输数据之前，TCP 必须先在通信双方之间建立一个连接。
* 可靠传输：通过确认（ACK）、重传机制和序列号，确保数据不丢失、按序到达。
* 流量控制：防止发送方发送过多数据，使接收方来不及处理。
* 拥塞控制：防止网络过载，确保网络资源合理利用。



### WebSocket协议概述
```
WebSocket是一种基于TCP的协议，用于在客户端和服务器之间建立长连接，支持双向通信。这与传统的HTTP请求-响应模型不同，WebSocket允许服务器主动向客户端发送数据，实现实时通信
```
#### 一、建立连接
* 1、握手：客户端向服务器发送一个HTTP请求，带有Upgrade头部，表示请求协议升级到WebSocket。
```
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
```
* 2、握手应答：服务器接收到请求后，如果支持WebSocket，将返回101状态码响应，确认协议升级。
```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```
#### 二、数据传输
```
WebSocket连接建立后，客户端和服务器可以相互发送消息。每条消息都以帧的形式传输，帧包含以下部分：
```
* FIN和RSV：表示消息的结束标志和保留位。
* 操作码：表示消息类型，如文本数据（0x1）、二进制数据（0x2）、关闭连接（0x8）等。
* 掩码标志和有效载荷长度：表示是否对数据进行掩码和数据的长度。
* 掩码密钥和有效载荷数据：实际传输的数据及其掩码密钥（如果有）。

#### 三、关闭连接
WebSocket连接关闭过程如下：
* 关闭帧：一方（客户端或服务器）发送一个带有操作码0x8的关闭帧，表示关闭连接。
* 关闭应答：另一方接收到关闭帧后，发送一个关闭帧作为应答，然后双方关闭TCP连接。
