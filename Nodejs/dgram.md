# dgram
node:dgram 模块提供了 UDP 数据报套接字的实现,UDP（User Datagram Protocol，用户数据报协议）是一种无连接、简单且高效的传输层协议，适用于需要快速传输但对可靠性要求不高的场景，如视频流、在线游戏等。

## node:dgram 模块函数

### dgram.createSocket(options[, callback])
```txt
options {Object} - 用来指定 socket 选项的配置对象，可以包含以下字段：
  type {string} - 指定 socket 类型，udp4 或 udp6。
  reuseAddr {boolean} - 允许绑定到其他进程正在使用的端口（默认为 false）。
  ipv6Only {boolean} - 启用或禁用双栈支持（默认为 true）。如果设置为 false，则 udp6 套接字可以接收 IPv4 数据包。
  recvBufferSize {number} - 接收缓冲区的大小（以字节为单位）。
  sendBufferSize {number} - 发送缓冲区的大小（以字节为单位）。
  lookup {Function} - 自定义 DNS 解析函数。

callback {Function} - 当 socket 被创建并准备好使用时，会调用此回调函数。

返回值:{dgram.Socket} - 返回一个新的 dgram.Socket 对象。
```
创建 dgram.Socket 对象。一旦创建了套接字，则调用 socket.bind() 将指示套接字开始监听数据报消息,当 address 和 port 未传递给 socket.bind() 时，该方法会将套接字绑定到随机端口上的 "所有接口" 地址（它对 udp4 和 udp6 套接字都做正确的事情）。可以使用 socket.address().address 和 socket.address().port 检索绑定的地址和端口
```js
// 创建一个 UDP 套接字（IPv4）
const socket = dgram.createSocket('udp4');
```
```js
// 创建一个 UDP 套接字（IPv4），并设置接收和发送缓冲区大小
const options = {
  type: 'udp4',
  recvBufferSize: 1024 * 1024,  // 接收缓冲区大小为 1MB
  sendBufferSize: 512 * 1024    // 发送缓冲区大小为 512KB
};

const socket = dgram.createSocket(options);
```
```js
//创建一个 UDP 套接字（IPv4），并设置回调函数
const socket = dgram.createSocket('udp4', (msg, rinfo) => {
  console.log(`接收到来自 ${rinfo.address}:${rinfo.port} 的消息: ${msg}`);
});
```

如果启用了 signal 选项，则在对应的 AbortController 上调用 .abort() 与在套接字上调用 .close() 类似：
```js
const controller = new AbortController();
const { signal } = controller;
const server = dgram.createSocket({ type: 'udp4', signal });
server.on('message', (msg, rinfo) => {
  console.log(`server got: ${msg} from ${rinfo.address}:${rinfo.port}`);
});
// Later, when you want to close the server.
controller.abort();
```

### dgram.createSocket(type[, callback])
```txt
type <string> 'udp4' 或 'udp6'。

callback <Function> 绑定作为 'message' 事件的监听器。

返回：<dgram.Socket>
```


## Class:dgram.Socket
dgram.Socket  继承：`EventEmitter`,封装数据报功能

### 事件
#### close
在使用 close() 关闭套接字后会触发 'close' 事件。一旦触发，则此套接字上将不会触发新的 'message' 事件。

#### connect
connect 事件在套接字关联到远程地址作为成功的 connect() 调用的结果之后触发。

#### error
每当发生任何错误时都会触发 'error' 事件。事件句柄被传入单一的 Error 对象。


#### listening
一旦 dgram.Socket 可寻址并且可以接收数据，则会触发 'listening' 事件。这会在 socket.bind() 中显式地发生，或者在第一次使用 socket.send() 发送数据时隐式地发生。直到 dgram.Socket 正在监听，底层系统资源不存在，则调用 socket.address() 和 socket.setTTL() 等将失败。


#### message
当套接字上有新的数据报可用时，则会触发 'message' 事件。事件处理函数传递了两个参数：msg 和 rinfo。

```txt
msg <Buffer> 消息。

rinfo <Object> 远程地址信息。
  address <string> 发送者地址。
  family <string> 地址族（'IPv4' 或 'IPv6'）。
  port <number> 发送者端口。
  size <number> 消息大小
```

demo
```js
// 服务端
const dgram = require('dgram');
const server = dgram.createSocket('udp4');

server.on('error', (err) => {
  console.error(`服务器错误:\n${err.stack}`);
  server.close();
});

server.on('message', (msg, rinfo) => {
  console.log(`服务器接收到来自 ${rinfo.address}:${rinfo.port} 的消息: ${msg}`);
  
  const response = Buffer.from('Message received');
  server.send(response, rinfo.port, rinfo.address, (err) => {
    if (err) {
      console.error(err);
    } else {
      console.log('响应已发送');
    }
  });
});

server.on('listening', () => {
  const address = server.address();
  console.log(`服务器正在监听 ${address.address}:${address.port}`);
});

server.bind(41234);

// 客户端
const dgram = require('dgram');
const client = dgram.createSocket('udp4');

const message = Buffer.from('Hello, UDP server!');

client.send(message, 41234, 'localhost', (err) => {
  if (err) {
    console.error(err);
    client.close();
  } else {
    console.log('消息已发送');
  }
});

client.on('message', (msg, rinfo) => {
  console.log(`客户端接收到来自 ${rinfo.address}:${rinfo.port} 的消息: ${msg}`);
  client.close();
});

```


#### socket.addMembership(multicastAddress[, multicastInterface])socket.addMembership(multicastAddress[, multicastInterface])
```txt
<!-- 参数含义 -->
multicastAddress：要加入的多播组的 IP 地址。IPv4 地址通常在 224.0.0.0 到 239.255.255.255 之间。
multicastInterface（可选）：指定要使用的网络接口的地址。如果不提供，操作系统将选择一个默认的接口
```
使用 IP_ADD_MEMBERSHIP 套接字选项告诉内核在给定的 multicastAddress 和 multicastInterface 上加入多播组。如果未指定 multicastInterface 参数，则操作系统将选择一个接口并为其添加成员资格。要为每个可用接口添加成员资格，则多次调用 addMembership，每个接口一次。

当在未绑定的套接字上调用时，则此方法将隐式地绑定到随机端口，监听所有接口。
当在多个 cluster 工作进程共享 UDP 套接字时，则必须只调用一次 socket.addMembership() 函数，否则会发生 EADDRINUSE 错误：
```js
const cluster = require('node:cluster');
const dgram = require('node:dgram');

if (cluster.isPrimary) {
  cluster.fork(); // Works ok.
  cluster.fork(); // Fails with EADDRINUSE.
} else {
  const s = dgram.createSocket('udp4');
  s.bind(1234, () => {
    s.addMembership('224.0.0.114');
  });
}
```

以下是如何使用 `socket.addMembership` 将套接字加入到一个多播组的示例。

##### UDP 服务器（多播接收器）

```javascript
const dgram = require('dgram');
const server = dgram.createSocket('udp4');

server.on('error', (err) => {
  console.error(`服务器错误:\n${err.stack}`);
  server.close();
});

server.on('message', (msg, rinfo) => {
  console.log(`服务器接收到来自 ${rinfo.address}:${rinfo.port} 的消息: ${msg}`);
});

server.on('listening', () => {
  const address = server.address();
  console.log(`服务器正在监听 ${address.address}:${address.port}`);
  
  // 加入多播组
  server.addMembership('224.0.0.114');
});

server.bind(41234);  // 绑定到指定端口和所有可用的网络接口
```

##### UDP 客户端（多播发送器）

```javascript
const dgram = require('dgram');
const client = dgram.createSocket('udp4');

const message = Buffer.from('Hello, Multicast!');

// 发送消息到多播组
client.send(message, 41234, '224.0.0.114', (err) => {
  if (err) {
    console.error(err);
    client.close();
  } else {
    console.log('消息已发送到多播组');
    client.close();
  }
});
```

##### 注意事项

1. **多播地址范围**：确保使用的多播地址在 `224.0.0.0` 到 `239.255.255.255` 之间。
2. **网络配置**：某些网络配置可能会阻止多播流量。确保网络设备（如路由器和交换机）支持并配置了多播。
3. **接口选择**：如果你的机器有多个网络接口，可以使用 `multicastInterface` 参数指定接口。例如：

    ```javascript
    server.addMembership('224.0.0.114', '192.168.1.1');
    ```

4. **离开多播组**：你可以使用 `socket.dropMembership(multicastAddress[, multicastInterface])` 方法将套接字从多播组中移除。

##### 总结

`socket.addMembership(multicastAddress[, multicastInterface])` 方法提供了一种简单的方式将 UDP 套接字加入到一个多播组，以便接收多播消息。这在需要广播消息给多个接收者的场景中非常有用，如实时数据分发、视频流等。

- **加入多播组**：使用 `addMembership` 方法将套接字加入到指定的多播组。
- **接收多播消息**：在加入多播组后，套接字可以接收发送到该多播组的消息。
- **网络配置**：确保网络设备和配置支持多播通信。

通过合理使用多播，可以高效地实现一对多的消息传递。


#### socket.addSourceSpecificMembership(sourceAddress, groupAddress[, multicastInterface])
socket.addSourceSpecificMembership(sourceAddress, groupAddress[, multicastInterface])用于将套接字加入到一个源特定多播组（Source-Specific Multicast，SSM）。在 SSM 中，不仅可以指定接收哪个多播组的数据，还可以指定接收来自哪个源地址的数据。这种方式可以提高多播的安全性和控制性
```txt
sourceAddress：指定发送多播数据包的源 IP 地址。
groupAddress：要加入的多播组的 IP 地址。IPv4 地址通常在 232.0.0.0 到 232.255.255.255 之间。
multicastInterface（可选）：指定要使用的网络接口的地址。如果不提供，操作系统将选择一个默认的接口。
```
加入源特定多播组并接收消息
```js
// service
const dgram = require('dgram');
const server = dgram.createSocket('udp4');

server.on('error', (err) => {
  console.error(`服务器错误:\n${err.stack}`);
  server.close();
});

server.on('message', (msg, rinfo) => {
  console.log(`服务器接收到来自 ${rinfo.address}:${rinfo.port} 的消息: ${msg}`);
});

server.on('listening', () => {
  const address = server.address();
  console.log(`服务器正在监听 ${address.address}:${address.port}`);
  
  // 加入源特定多播组
  server.addSourceSpecificMembership('192.168.1.1', '232.0.0.114');
});

server.bind(41234);  // 绑定到指定端口和所有可用的网络接口


// client
const dgram = require('dgram');
const client = dgram.createSocket('udp4');

const message = Buffer.from('Hello, Source-Specific Multicast!');

// 发送消息到多播组
client.send(message, 41234, '232.0.0.114', (err) => {
  if (err) {
    console.error(err);
    client.close();
  } else {
    console.log('消息已发送到源特定多播组');
    client.close();
  }
});
```


#### socket.address()
返回包含套接字地址信息的对象。对于 UDP 套接字，此对象将包含 address、family 和 port 属性。 {"address":"0.0.0.0","family":"IPv4","port":41234}

如果在未绑定的套接字上调用此方法将抛出 EBADF。

#### socket.bind([port][, address][, callback])
```txt
port（可选）：绑定的端口号。如果未指定，将由操作系统自动分配一个可用端口。
address（可选）：绑定的 IP 地址。如果未指定，将绑定到所有可用的网络接口。
callback（可选）：绑定完成后调用的回调函数
```
```js
// 绑定到端口 41234 和所有可用的网络接口
server.bind(41234);
// 绑定到端口 41234 和本地主机地址
server.bind(41234, '127.0.0.1');
// 绑定到端口 41234 和所有可用的网络接口，绑定完成后调用回调函数
server.bind(41234, () => {
  console.log('绑定完成');
});
// 绑定到所有可用的网络接口，端口由操作系统自动分配
server.bind(() => {
  console.log('绑定完成，端口由操作系统分配');
});
```
####  socket.bind(options[, callback])
```txt
options：一个包含以下属性的对象：
  port：要绑定的端口号。如果未指定，将由操作系统自动分配一个可用端口。
  address：要绑定的 IP 地址。如果未指定，将绑定到所有可用的网络接口。
  exclusive：布尔值。如果为 true，则独占端口，其他进程无法绑定到相同的端口。默认为 false。 
    当将 dgram.Socket 对象与 cluster 模块一起使用时会使用该属性。当 exclusive 设置为 false（默认）时，集群工作进程将使用相同的底层套接字句柄，允许共享连接处理职责。
    但是，当 exclusive 为 true 时，则句柄未共享，尝试共享端口会导致错误
callback（可选）：绑定完成后调用的回调函数
```
```js
socket.bind({
  address: 'localhost',
  port: 8000,
  exclusive: true,
}); 
```

#### socket.close([callback])
关闭底层套接字并停止监听其上的数据。如果提供回调，则将其添加为 'close' 事件的监听器。

#### socket.connect(port[, address][, callback])
用于将 UDP 套接字连接到指定的远程端口和地址。尽管 UDP 是无连接的协议，但这个方法允许你指定一个默认的目标地址和端口，使得后续的 send 操作可以省略这些参数
```txt
port：要连接的远程端口号。
address（可选）：要连接的远程 IP 地址。如果未指定，默认值是 '127.0.0.1'。
callback（可选）：连接完成后调用的回调函数
```
```js
const dgram = require('dgram');
const client = dgram.createSocket('udp4');

// 连接到远程服务器，连接完成后调用回调函数
client.connect(41234, 'localhost', () => {
  console.log('已连接到远程服务器');

  // 发送消息到已连接的远程服务器
  const message = Buffer.from('Hello, UDP server with callback!');
  client.send(message, (err) => {
    if (err) {
      console.error(err);
    } else {
      console.log('消息已发送');
    }
  });
});

// 接收来自服务器的消息
client.on('message', (msg, rinfo) => {
  console.log(`客户端接收到来自 ${rinfo.address}:${rinfo.port} 的消息: ${msg}`);
});

```

#### socket.disconnect()
将连接的 dgram.Socket 与其远程地址分离的同步函数。尝试在未绑定或已断开连接的套接字上调用 disconnect() 将导致 ERR_SOCKET_DGRAM_NOT_CONNECTED 异常
```js
// service
const dgram = require('dgram');
const server = dgram.createSocket('udp4');

server.on('error', (err) => {
  console.error(`服务器错误:\n${err.stack}`);
  server.close();
});

server.on('message', (msg, rinfo) => {
  console.log(`服务器接收到来自 ${rinfo.address}:${rinfo.port} 的消息: ${msg}`);
});

server.on('listening', () => {
  const address = server.address();
  console.log(`服务器正在监听 ${address.address}:${address.port}`);
});

server.bind(41234);  // 绑定到指定端口和所有可用的网络接口

// client
const dgram = require('dgram');
const client = dgram.createSocket('udp4');

// 连接到远程服务器
client.connect(41234, 'localhost', () => {
  console.log('已连接到远程服务器');

  // 发送消息到已连接的远程服务器
  const message = Buffer.from('Hello, UDP server!');
  client.send(message, (err) => {
    if (err) {
      console.error(err);
    } else {
      console.log('消息已发送');
    }

    // 解除连接
    client.disconnect();
    console.log('已解除连接');

    // 解除连接后再次发送消息，需要显式指定目标地址和端口
    const newMessage = Buffer.from('Hello again, UDP server!');
    client.send(newMessage, 41234, 'localhost', (err) => {
      if (err) {
        console.error(err);
      } else {
        console.log('解除连接后消息已发送');
      }

      // 关闭客户端
      client.close();
    });
  });
});

// 接收来自服务器的消息
client.on('message', (msg, rinfo) => {
  console.log(`客户端接收到来自 ${rinfo.address}:${rinfo.port} 的消息: ${msg}`);
});
```
##### 注意事项
无连接协议 ：尽管 UDP 是无连接的协议，connect 方法允许你指定一个默认的目标地址和端口，disconnect 方法则用于解除这种默认目标设置。
恢复到未连接状态 ：调用 disconnect 方法后，套接字恢复到未连接状态，后续的 send 操作需要显式指定目标地址和端口。
错误处理 ：确保在 error 事件中处理可能的错误。

#### socket.dropMembership(multicastAddress[, multicastInterface])
```txt
multicastAddress：要移除的多播组的 IP 地址。
multicastInterface（可选）：指定要使用的网络接口的地址。如果不提供，操作系统将选择一个默认的接口。
```
指示内核使用 IP_DROP_MEMBERSHIP 套接字选项离开 multicastAddress 处的多播组。当套接字关闭或进程终止时，内核会自动调用此方法，因此大多数应用永远没有理由调用此方法。

如果未指定 multicastInterface，则操作系统将尝试删除所有有效接口上的成员资格。

#### socket.dropSourceSpecificMembership(sourceAddress, groupAddress[, multicastInterface])
```txt
sourceAddress：指定发送多播数据包的源 IP 地址。
groupAddress：要移除的多播组的 IP 地址（通常在 232.0.0.0 到 232.255.255.255 之间）。
multicastInterface（可选）：指定要使用的网络接口的地址。如果不提供，操作系统将选择一个默认的接口。

```
用于将套接字从一个源特定多播组（Source-Specific Multicast，SSM）中移除。这个方法与 socket.addSourceSpecificMembership(sourceAddress, groupAddress[, multicastInterface]) 方法相对应，用于取消之前加入的源特定多播组

#### socket.getRecvBufferSize()
用于获取当前套接字的**接收缓冲区**大小。接收缓冲区大小决定了套接字在接收数据时可以存储的最大数据量。在高流量的网络应用中，调整接收缓冲区大小可以帮助优化性能

获取接收缓冲区大小 ：通过 getRecvBufferSize() 方法获取当前接收缓冲区大小。

调整接收缓冲区大小 ：通过 setRecvBufferSize(size) 方法调整接收缓冲区大小，以优化性能
```js
server.on('listening', () => {
  const address = server.address();
  console.log(`服务器正在监听 ${address.address}:${address.port}`);
  
  // 获取接收缓冲区大小
  const recvBufferSize = server.getRecvBufferSize();
  console.log(`接收缓冲区大小: ${recvBufferSize} 字节`);
});
```

#### socket.getSendBufferSize()  
用于获取当前套接字的**发送缓冲区**大小。发送缓冲区大小决定了套接字在发送数据时可以存储的最大数据量。在高流量的网络应用中，调整发送缓冲区大小可以帮助优化性能

获取发送缓冲区大小 ：通过 getSendBufferSize() 方法获取当前发送缓冲区大小。

调整发送缓冲区大小 ：通过 setSendBufferSize(size) 方法调整发送缓冲区大小，以优化性能。
```js
server.on('listening', () => {
  const address = server.address();
  console.log(`服务器正在监听 ${address.address}:${address.port}`);
  
  // 获取发送缓冲区大小
  const sendBufferSize = server.getSendBufferSize();
  console.log(`发送缓冲区大小: ${sendBufferSize} 字节`);
});
```

#### socket.getSendQueueSize()
用于获取当前套接字的发送队列大小。发送队列大小表示在发送缓冲区中等待发送的数据量（以字节为单位）

```js
const size = socket.getSendQueueSize();
```

#### socket.getSendQueueCount()

用于获取当前套接字当前队列中等待处理的发送请求数
```js
const count = socket.getSendQueueCount();
```

#### socket.ref()
用于将一个已经被调用了 unref() 方法的套接字重新加入到事件循环中。这个方法通常用于确保 Node.js 程序在套接字上有活动事件时不会退出

返回当前的 dgram.Socket 对象，以便可以链式调用其他方法。

#### socket.unref()
socket.unref() 方法可用于将套接字从保持 Node.js 进程处于活动状态的引用计数中排除，即使套接字仍在监听，也允许进程退出

```js
const dgram = require('dgram');
const server = dgram.createSocket('udp4');

server.on('error', (err) => {
  console.error(`服务器错误:\n${err.stack}`);
  server.close();
});

server.on('message', (msg, rinfo) => {
  console.log(`服务器接收到来自 ${rinfo.address}:${rinfo.port} 的消息: ${msg}`);
});

server.on('listening', () => {
  const address = server.address();
  console.log(`服务器正在监听 ${address.address}:${address.port}`);
  
  // 取消对事件循环的保持
  server.unref();
  console.log('服务器已取消对事件循环的保持');

  // 5秒后重新保持事件循环
  setTimeout(() => {
    server.ref();
    console.log('服务器重新保持对事件循环的保持');
  }, 5000);
});

server.bind(41234);  // 绑定到指定端口和所有可用的网络接口

```

#### socket.remoteAddress()
<!-- TODO: -->
返回包含远程端点的 address、family 和 port 的对象。如果套接字未连接，则此方法将抛出 ERR_SOCKET_DGRAM_NOT_CONNECTED 异常。

#### socket.send(msg[, offset, length][, port][, address][, callback])
```txt
msg：要发送的消息。可以是 Buffer、Uint8Array 或字符串。
offset（可选）：消息中要发送的起始字节的偏移量。默认为 0。
length（可选）：要发送的字节数。默认为 msg.length - offset。
port（可选）：目标端口号。
address（可选）：目标 IP 地址。默认为 '127.0.0.1'。
callback（可选）：消息发送完成后的回调函数。回调函数的参数为一个可能的错误对象
```
```js
client.send(message, 7, 11, 41234, 'localhost', (err) => {
  if (err) {
    console.error(err);
    client.close();
  } else {
    console.log('部分消息已发送');
    client.close();
  }
});
```
#### socket.setBroadcast(flag)
用于设置 UDP 套接字的广播标志。广播是一种网络通信方法，允许数据包发送到子网内的所有设备
```txt
flag：布尔值。如果为 true，则启用广播；如果为 false，则禁用广播
```
```js
const dgram = require('dgram')

const server = dgram.createSocket('udp4')

server.on('listening', () => {
  const address = server.address()

  console.log(`server running ${address.address}:${address.port}`)

  server.setBroadcast(true) // 开启广播模式

  server.send('hello', 8000, '255.255.255.255')

  // 每隔2秒发送一条广播消息
  setInterval(function () {
    // 直接地址 192.168.10.255
    // 受限地址 255.255.255.255
    // server.send('hello', 8000, '192.168.10.255')
    server.send('hello', 8000, '255.255.255.255')
  }, 2000)
})

server.on('message', (msg, remoteInfo) => {
  console.log(`server got ${msg} from ${remoteInfo.address}:${remoteInfo.port}`)
  server.send('world', remoteInfo.port, remoteInfo.address)
})

server.on('error', err => {
  console.log('server error', err)
})

server.bind(3000)

```

#### socket.setMulticastInterface(multicastInterface)
multicastInterface：指定要用于多播的网络接口的 IP 地址。如果设置为空字符串（''），则操作系统将选择一个默认的接口。

用于设置发送和接收多播数据包的网络接口。多播是一种网络通信方法，允许单个数据包被发送到多个接收者，而不需要为每个接收者单独发送数据包

```js
const dgram = require('dgram');
const server = dgram.createSocket('udp4');

server.on('error', (err) => {
  console.error(`服务器错误:\n${err.stack}`);
  server.close();
});

server.on('message', (msg, rinfo) => {
  console.log(`服务器接收到来自 ${rinfo.address}:${rinfo.port} 的消息: ${msg}`);
});

server.on('listening', () => {
  const address = server.address();
  console.log(`服务器正在监听 ${address.address}:${address.port}`);
  
  // 加入多播组
  server.addMembership('224.0.0.114');
  
  // 设置用于多播的网络接口
  server.setMulticastInterface('192.168.1.100');
});

server.bind(41234);  // 绑定到指定端口和所有可用的网络接口

```

#### socket.setMulticastLoopback(flag)
用于设置或禁用多播环回功能,多播环回功能允许发送到多播组的数据包在本地回送，即发送者可以接收到自己发送的多播数据包,这在调试和某些特定应用场景中非常有用

flag：布尔值。如果为 true，启用多播环回；如果为 false，禁用多播环回
```js
const dgram = require('dgram');
const socket = dgram.createSocket('udp4');

socket.on('error', (err) => {
  console.error(`套接字错误:\n${err.stack}`);
  socket.close();
});

socket.on('message', (msg, rinfo) => {
  console.log(`套接字接收到来自 ${rinfo.address}:${rinfo.port} 的消息: ${msg}`);
});

socket.on('listening', () => {
  const address = socket.address();
  console.log(`套接字正在监听 ${address.address}:${address.port}`);

  // 加入多播组
  socket.addMembership('224.0.0.114');

  // 设置是否启用多播环回
  socket.setMulticastLoopback(false);

  const message = Buffer.from('Hello, Multicast with Loopback!');

  // 定时发送多播消息到多播组
  setInterval(() => {
    socket.send(message, 0, message.length, 41234, '224.0.0.114', (err) => {
      if (err) {
        console.error(err);
      } else {
        console.log('多播消息已发送');
      }
    });
  }, 3000); // 每3秒发送一次消息
});

socket.bind(41234);  // 绑定到指定端口和所有可用的网络接口

```

#### socket.setMulticastTTL(ttl)
用于设置多播数据包的生存时间（TTL，Time To Live）。TTL 值决定了多播数据包可以经过的最大路由器数量
```txt
ttl：一个整数，表示多播数据包的生存时间。有效值范围是 0 到 255
```

#### socket.setTTL(ttl)
用于来设置 UDP 套接字的 TTL 值 （参考上面）
