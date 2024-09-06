# net
用于创建基于 TCP 和 IPC（进程间通信）的服务器和客户端。提供了创建和操作网络连接的工具，适用于需要高效、低延迟通信的应用程序。

## Class:net.BlockList

BlockList 对象可与一些网络 API 一起使用，以指定用于禁用对特定 IP 地址、IP 范围或 IP 子网的入站或出站访问的规则。

### blockList.addAddress(address[, type])
将一个单独的 IP 地址添加到黑名单中
```txt
address：字符串，IP 地址（例如 '192.168.1.1' 或 '2001:0db8:85a3:0000:0000:8a2e:0370:7334'）。
type（可选）：字符串，指定地址类型。可以是 'ipv4' 或 'ipv6'。如果未指定，Node.js 会自动检测地址类型。
```
```js
blockList.addAddress('192.168.1.1');
blockList.addAddress('2001:0db8:85a3:0000:0000:8a2e:0370:7334', 'ipv6');
```

#### blockList.addRange(start, end[, type])
将一个 IP 地址范围添加到黑名单中

```txt
start：字符串，起始 IP 地址。
end：字符串，结束 IP 地址。
type（可选）：字符串，指定地址类型。可以是 'ipv4' 或 'ipv6'。如果未指定，Node.js 会自动检测地址类型。

```

### blockList.addSubnet(net, prefix[, type])
将一个子网添加到黑名单中
```txt
net：字符串，子网地址。
prefix：整数，子网前缀长度（CIDR 表示法）。
```
```js
blockList.addSubnet('192.168.1.0', 24);
blockList.addSubnet('2001:0db8:85a3::', 64);
```
### blockList.check(address)
检查一个 IP 地址是否在黑名单中。
```txt
address：字符串，IP 地址。
返回值：布尔值，如果 IP 地址在黑名单中则返回 true，否则返回 false。
```
```js
console.log(blockList.check('192.168.1.1'));  // 输出: true
console.log(blockList.check('192.168.2.1'));  // 输出: false
```
完整demo
```js
const net = require('net');

// 创建 BlockList 实例
const blockList = new net.BlockList();

// 添加单个 IP 地址
blockList.addAddress('192.168.1.1');
blockList.addAddress('2001:0db8:85a3:0000:0000:8a2e:0370:7334', 'ipv6');

// 添加 IP 地址范围
blockList.addRange('192.168.1.10', '192.168.1.20');
blockList.addRange('2001:0db8:85a3:0000:0000:8a2e:0000:0000', '2001:0db8:85a3:0000:0000:8a2e:ffff:ffff', 'ipv6');

// 添加子网
blockList.addSubnet('192.168.2.0', 24);
blockList.addSubnet('2001:0db8:85a3::', 64);

// 检查 IP 地址是否在黑名单中
console.log(blockList.check('192.168.1.1'));  // 输出: true
console.log(blockList.check('192.168.1.15')); // 输出: true
console.log(blockList.check('192.168.2.1'));  // 输出: true
console.log(blockList.check('192.168.3.1'));  // 输出: false
console.log(blockList.check('2001:0db8:85a3:0000:0000:8a2e:0370:7334'));  // 输出: true
console.log(blockList.check('2001:0db8:85a3::1234'));  // 输出: true
console.log(blockList.check('2001:0db8:85a3:ffff:ffff:ffff:ffff:ffff'));  // 输出: false

```

## Class:net.SocketAddress
用于表示网络套接字的地址信息,提供了一种结构化的方式来存储和访问网络套接字的地址信息。

### new net.SocketAddress([options])
创建 SocketAddress 实例
```txt
options：一个对象，包含以下属性：
  address：字符串，IP 地址。
  port：整数，端口号。
  family（可选）：字符串，指定地址族。可以是 'ipv4' 或 'ipv6'。如果未指定，Node.js 会自动检测地址类型。
  flowlabel <number> 仅当 family 为 'ipv6' 时才使用的 IPv6 流标签
```
```js
// 创建一个 IPv4 地址
const address = new net.SocketAddress({
  address: '192.168.1.1',
  port: 8080,
  family: 'ipv4'
});

// 创建一个 IPv6 地址
const address = new net.SocketAddress({
  address: '2001:0db8:85a3:0000:0000:8a2e:0370:7334',
  port: 8080,
  family: 'ipv6'
});
```

## Class：net.Server
此类用于创建 TCP 或 IPC 服务器。net.Server 提供了一组方法和事件，用于监听连接、处理数据、管理连接等

### new net.Server([options][, connectionListener])#
用于创建一个新的 TCP 服务器实例。这个构造函数接受一个可选的配置对象 options 和一个可选的连接监听器 connectionListener
```txt
options（可选）：一个对象，包含以下可选属性：
  allowHalfOpen：布尔值，表示是否允许半开连接（即服务器端的套接字可以在客户端关闭其写入端后仍然保持打开）。默认为 false。
  pauseOnConnect：布尔值，表示是否在连接建立时暂停套接字。默认为 false。
connectionListener（可选）：一个函数，当客户端连接到服务器时调用。该函数接受一个参数 socket，表示与客户端的连接。
```

### 事件 
#### close
服务器关闭时触发。如果连接存在，则在所有连接结束之前不会触发此事件。
#### connection
建立新连接时触发。socket 是 net.Socket 的实例。
#### error
发生错误时触发。与 net.Socket 不同的是，除非手动调用 server.close()，否则不会在该事件之后直接触发 'close' 事件。参见 server.listen() 讨论中的示例
#### listening
在调用 server.listen() 后绑定服务器时触发
#### drop
当连接数达到 server.maxConnections 的阈值时，服务器将丢弃新的连接并触发 'drop' 事件。如果是 TCP 服务器，参数如下，否则参数为 undefined。
```txt
data <Object> 传给事件监听器的参数。

  localAddress <string> 本地地址。

  localPort <number> 本地端口。

  localFamily <string> 本地族。

  remoteAddress <string> 远程地址。

  remotePort <number> 远程端口。

  remoteFamily <string> 远程 IP 系列。'IPv4' 或 'IPv6'
```
#### 完整demo
```js
const server = new net.Server(options, (socket) => {
  console.log('客户端已连接');

  // 监听客户端发送的数据
  socket.on('data', (data) => {
    console.log(`接收到的数据: ${data}`);
    // 回应客户端
    socket.write('服务器已收到数据');
  });

  // 监听客户端断开连接
  socket.on('end', () => {
    console.log('客户端已断开连接');
  });

  // 处理错误
  socket.on('error', (err) => {
    console.error(`套接字错误: ${err.message}`);
  });
});

// 监听连接事件
server.on('connection', (socket) => {
  console.log('新的客户端连接');
});

// 监听服务器开始监听事件
server.on('listening', () => {
  console.log('服务器正在监听');
});

// 监听服务器关闭事件
server.on('close', () => {
  console.log('服务器已关闭');
});

// 监听服务器drop事件
server.on('drop', (data) => {
  console.error(data);
});

// 监听服务器错误事件
server.on('error', (err) => {
  console.error(`服务器错误: ${err.message}`);
});

// 服务器开始监听指定端口
server.listen(8080, () => {
  console.log('服务器正在监听端口 8080');
});
```

### server.address()
如果监听 IP 套接字，则返回操作系统报告的绑定 address、地址 family 名称和服务器的 port（在获取操作系统分配的地址时有助于查找分配的端口）：{ port: 12346, family: 'IPv4', address: '127.0.0.1' }。

对于监听管道或 Unix 域套接字的服务器，名称作为字符串返回

```js
const server = net.createServer((socket) => {
  socket.end('goodbye\n');
}).on('error', (err) => {
  // Handle errors here.
  throw err;
});

// Grab an arbitrary unused port.
server.listen(() => {
  console.log('opened server on', server.address());
});
```
### server.getConnections(callback)
异步获取服务器上的并发连接数。当套接字被发送到复刻时工作。

回调应该采用两个参数 err 和 coun

### server.listen(port[, host][, backlog][, callback])
启动监听连接的服务器。net.Server 可以是 TCP 或 IPC 服务器，具体取决于它监听的内容。
```txt
port：整数，指定要监听的端口号。
host（可选）：字符串，指定要监听的主机名或 IP 地址。默认是 '0.0.0.0'。
backlog（可选）：整数，指定待处理连接队列的最大长度。默认是操作系统定义的值。
callback（可选）：函数，当服务器开始监听时调用。
```
### server.listen(options[, callback])
```txt
options：一个对象，包含以下属性：
  backlog <number> server.listen() 函数的通用参数。
  exclusive <boolean> 默认值：false
  host <string>
  ipv6Only <boolean> 对于 TCP 服务器，将 ipv6Only 设置为 true 将禁用双栈支持，即绑定到主机 :: 不会绑定 0.0.0.0。默认值：false。
  path <string> 如果指定了 port，则将被忽略。参见 识别 IPC 连接的路径。
  port <number>
  readableAll <boolean> 对于 IPC 服务器，使管道对所有用户都可读。默认值：false。
  signal <AbortSignal> 可用于关闭监听服务器的中止信号。
  writableAll <boolean> 对于 IPC 服务器，使管道对所有用户都可写。默认值：false
callback（可选）：函数，当服务器开始监听时调用。
```
### server.listen(path[, callback])
```txt
path：字符串，指定要监听的路径（用于 UNIX 域套接字）。
callback（可选）：函数，当服务器开始监听时调用。
```
### server.listen(handle[, callback])
```txt
handle：一个对象，表示要监听的句柄（例如，另一个服务器或套接字）。
callback（可选）：函数，当服务器开始监听时调用。
```
demo
```js
// 监听指定端口
server.listen(8080, () => {
  console.log('服务器正在监听端口 8080');
});

// 监听指定端口和主机
server.listen(8080, 'localhost', () => {
  console.log('服务器正在监听 localhost:8080');
});

// 使用 options 对象监听端口和主机
const options = {
  port: 8080,
  host: 'localhost',
  backlog: 511
};
// 服务器开始监听
server.listen(options, () => {
  console.log('服务器正在监听 localhost:8080');
});

// 监听 UNIX 域套接字
const path = '/tmp/echo.sock';
// 服务器开始监听 UNIX 域套接字
server.listen(path, () => {
  console.log(`服务器正在监听 ${path}`);
});
```

### server.listening
用于指示 TCP 服务器是否正在监听连接。它是一个布尔值，当服务器正在监听连接时，其值为 true，否则为 false。

### server.maxConnections
设置此属性以在服务器的连接计数变高时拒绝连接。

一旦将套接字发送给具有 child_process.fork() 的子级，不建议使用此选项
### server.ref()
用于将一个已经被调用了 unref() 方法的 TCP 服务器重新加入到事件循环中。

这个方法通常用于确保 Node.js 程序在服务器上有活动事件时不会退出。

### server.unref()
用于使 TCP 服务器不保持事件循环活动。调用这个方法后，如果事件循环中没有其他保持活动的事件，Node.js 进程将会退出。

这个方法通常用于在后台运行的服务器或守护进程中，当你希望服务器不会阻止 Node.js 进程退出时使用。

```js
// 使用 unref() 和 ref() 控制事件循环
const net = require('net');

// 创建一个 TCP 服务器
const server = net.createServer((socket) => {
  console.log('客户端已连接');

  socket.on('data', (data) => {
    console.log(`接收到的数据: ${data}`);
    socket.write('服务器已收到数据');
  });

  socket.on('end', () => {
    console.log('客户端已断开连接');
  });

  socket.on('error', (err) => {
    console.error(`套接字错误: ${err.message}`);
  });
});

// 监听服务器开始监听事件
server.on('listening', () => {
  console.log('服务器正在监听端口 8080');
  
  // 取消对事件循环的保持
  server.unref();
  console.log('服务器已取消对事件循环的保持');

  // 5秒后重新保持事件循环
  setTimeout(() => {
    server.ref();
    console.log('服务器重新保持对事件循环的保持');
  }, 5000);
});

// 服务器开始监听指定端口
server.listen(8080, () => {
  console.log('服务器开始监听');
});

```


## Class：net.Socket

### new net.Socket([options])
用于创建一个新的 TCP 或 IPC 套接字实例
```txt
options（可选）：一个对象，包含以下可选属性：
  fd：整数，指定要使用的现有文件描述符（文件描述符必须是一个有效的套接字）。
  allowHalfOpen：布尔值，表示是否允许半开连接（即在对方关闭其写入端后，仍然保持连接打开）。默认为 false。
  readable：布尔值，表示套接字是否应该是可读的。默认为 true。
  writable：布尔值，表示套接字是否应该是可写的。默认为 true。
  signal <AbortSignal> 可用于销毁套接字的中止信号
```
```js
// 1. 创建一个 TCP 套接字
const socket = new net.Socket();
// 使用文件描述符创建套接字
const net = require('net');
const fs = require('fs');

// 2.打开一个现有文件描述符
const fd = fs.openSync('/path/to/some/socket', 'r+');
// 使用文件描述符创建套接字
const socket = new net.Socket({ fd: fd });

// 3.创建一个允许半开连接的套接字
const socket = new net.Socket({ allowHalfOpen: true });
```
### 事件
#### close
套接字完全关闭后触发。参数 hadError 是一个布尔值，表示套接字是否由于传输错误而关闭

#### connect
成功建立套接字连接时触发

#### connectionAttempt
```txt
ip <string> 套接字尝试连接的 IP。

port <number> 套接字尝试连接的端口。

family <number> IP 家族。对于 IPv6 可以是 6，对于 IPv4 可以是 4
```
当开始新的连接尝试时触发。如果在 socket.connect(options) 中启用了系列自动选择算法，则可能会多次触发此消息。

#### connectionAttemptFailed
```txt
ip <string> 套接字尝试连接的 IP。

¥ip <string> The IP which the socket attempted to connect to.

port <number> 套接字尝试连接的端口。

family <number> IP 家族。对于 IPv6 可以是 6，对于 IPv4 可以是 4。

error <Error> 与失败相关的错误。
```
连接尝试失败时触发。如果在 socket.connect(options) 中启用了系列自动选择算法，则可能会多次触发此消息。

#### connectionAttemptTimeout
```txt
ip <string> 套接字尝试连接的 IP。

port <number> 套接字尝试连接的端口。

family <number> IP 家族。对于 IPv6 可以是 6，对于 IPv4 可以是 4
```
连接尝试超时时触发。仅当在 socket.connect(options) 中启用系列自动选择算法时，才会触发此消息（并且可能会触发多次）。

#### data
收到数据时触发。参数 data 将是 Buffer 或 String。数据编码由 socket.setEncoding() 设置。

如果 Socket 触发 'data' 事件时没有监听器，则数据将丢失

#### drain
当写缓冲区变空时触发。可用于限制上传。

也可以看看：socket.write() 的返回值

#### end
当套接字的另一端触发传输结束信号时触发，从而结束套接字的可读端。

默认情况下（allowHalfOpen 是 false）套接字将发送回传输结束数据包并在写完其待处理的写入队列后销毁其文件描述符。但是，如果 allowHalfOpen 设置为 true，套接字将不会自动 end() 其可写端，允许用户写入任意数量的数据。用户必须显式调用 end() 以关闭连接（即发回 FIN 数据包）。

#### error
发生错误时触发。'close' 事件将在此事件之后直接调用

#### lookup
在解析主机名之后但在连接之前触发。不适用于 Unix 套接字。
```txt
err <Error> | <null> 错误对象。参见 dns.lookup()。

address <string> IP 地址。

family <number> | <null> 地址类型。 'IPv4' 和 'IPv6' 分别被解释为 4 和 6。值 0 指示返回 IPv4 或 IPv6 地址

host <string> 主机名
```

#### ready
当套接字准备好使用时触发,在 'connect' 之后立即触发

#### timeout
如果套接字因不活动而超时则触发。这只是为了通知套接字已经空闲。用户必须手动关闭连接


### socket.address()
返回操作系统报告的绑定 address、地址 family 名称和套接字 port：{ port: 12346, family: 'IPv4', address: '127.0.0.1' }

### socket.bytesRead
接收到的字节数。

### socket.bytesWritten#
发送的字节数。

### socket.connect()
#### socket.connect(options[, connectListener])
```txt
options：一个对象，包含以下属性：
  port：整数，指定要连接的远程端口号。
  host（可选）：字符串，指定要连接的远程主机名或 IP 地址。默认是 'localhost'。
  localAddress（可选）：字符串，指定本地绑定的 IP 地址。
  localPort（可选）：整数，指定本地绑定的端口号。
  family（可选）：整数，指定 IP 地址族。可以是 4 或 6。
  hints（可选）：整数，指定用于 DNS 解析的提示。
  lookup（可选）：函数，自定义 DNS 解析函数。
connectListener（可选）：函数，当连接成功时调用。
```
```js
const net = require('net');

// 创建一个 TCP 套接字
const socket = new net.Socket();

// 使用 options 对象连接到服务器
const options = {
  port: 8080,
  host: 'localhost',
  localAddress: '127.0.0.1',
  localPort: 12345,
  family: 4
};

socket.connect(options, () => {
  console.log('已连接到服务器');
  // 发送数据到服务器
  socket.write('你好，服务器！');
});

// 监听服务器发送的数据
socket.on('data', (data) => {
  console.log(`接收到的数据: ${data}`);
  // 关闭连接
  socket.end();
});

// 监听连接关闭事件
socket.on('end', () => {
  console.log('已断开与服务器的连接');
});

// 处理错误
socket.on('error', (err) => {
  console.error(`客户端错误: ${err.message}`);
});
```
#### socket.connect(port[, host][, connectListener])
```txt
port：整数，指定要连接的远程端口号。
host（可选）：字符串，指定要连接的远程主机名或 IP 地址。默认是 'localhost'。
connectListener（可选）：函数，当连接成功时调用。
```
```js
const net = require('net');

// 创建一个 TCP 套接字
const socket = new net.Socket();

// 连接到服务器
socket.connect(8080, 'localhost', () => {
  console.log('已连接到服务器');
  // 发送数据到服务器
  socket.write('你好，服务器！');
});

// 监听服务器发送的数据
socket.on('data', (data) => {
  console.log(`接收到的数据: ${data}`);
  // 关闭连接
  socket.end();
});

// 监听连接关闭事件
socket.on('end', () => {
  console.log('已断开与服务器的连接');
});

// 处理错误
socket.on('error', (err) => {
  console.error(`客户端错误: ${err.message}`);
});

```
#### socket.connect(path[, connectListener])
```txt
path：字符串，指定要连接的 UNIX 域套接字路径。
connectListener（可选）：函数，当连接成功时调用
```
```js
const net = require('net');
const path = '/tmp/echo.sock';

// 创建一个 TCP 套接字
const socket = new net.Socket();

// 连接到 UNIX 域套接字
socket.connect(path, () => {
  console.log('已连接到 UNIX 域套接字');
  // 发送数据到服务器
  socket.write('你好，UNIX 域套接字服务器！');
});

// 监听服务器发送的数据
socket.on('data', (data) => {
  console.log(`接收到的数据: ${data}`);
  // 关闭连接
  socket.end();
});

// 监听连接关闭事件
socket.on('end', () => {
  console.log('已断开与 UNIX 域套接字服务器的连接');
});

// 处理错误
socket.on('error', (err) => {
  console.error(`客户端错误: ${err.message}`);
});

```

### socket.connecting
在连接尝试期间，socket.connecting 属性的值为 true。
当连接成功时，socket.connecting 属性的值为 false
如果 true，socket.connect(options[, connectListener]) 被调用但尚未完成。它将保持 true 直到套接字连接，然后将其设置为 false 并触发 'connect' 事件。请注意，socket.connect(options[, connectListener]) 回调是 'connect' 事件的监听器。
```js
// 在连接尝试期间检查连接状态
console.log(`socket.connecting: ${socket.connecting}`);  // 输出: true
```


### socket.destroy([error])
调用这个方法后，套接字将立即关闭，并且不会再触发任何新的事件。如果提供了 error 参数，该错误对象将作为参数传递给 'error' 事件监听器

### socket.destroyed
调用这个方法后，套接字将立即关闭，并且不会再触发任何新的事件

### socket.destroySoon()
写入所有数据后销毁套接字。如果 'finish' 事件已经触发，则立即销毁套接字。如果套接字仍然可写，它会隐式调用 socket.end()。

### socket.end([data[, encoding]][, callback])
```txt
data <string> | <Buffer> | <Uint8Array>

encoding <string> 仅当数据为 string 时使用。默认值：'utf8'。

callback <Function> 套接字完成时的可选回调。

返回：<net.Socket> 套接字自身
```
用于半关闭 TCP 套接字，即发送一个 FIN 数据包，表示发送端已经完成了数据发送。调用这个方法后，套接字的写入端将被关闭，不能再写入数据，但仍然可以读取数据，直到远程端关闭连接

### socket.localAddress
远程客户端正在连接的本地 IP 地址的字符串表示形式

例如，在 '0.0.0.0' 上监听的服务器中，如果客户端连接到 '192.168.1.1'，则 socket.localAddress 的值将为 '192.168.1.1'

### socket.localPort
本地端口的数字表示。例如，80 或 21

### socket.localFamily
本地 IP 系列的字符串表示形式。'IPv4' 或 'IPv6'

### socket.pause()
当调用 pause() 方法时，套接字将停止触发 'data' 事件，直到调用 socket.resume() 方法恢复数据接收

使用场景：

临时停止数据接收 ：在处理大量数据时，可以使用 pause() 方法临时停止数据接收，以避免过载。

控制数据流 ：在处理流式数据时，可以使用 pause() 和 resume() 方法来控制数据流，以便按需处理数据
### socket.resume()
方法恢复数据接收
### socket.pending
如果套接字尚未连接，则为 true，因为 .connect() 尚未被调用，或者因为它仍在连接过程中（请参阅 socket.connecting）。

### socket.ref()
用于将一个已经被调用了 unref() 方法的 TCP 套接字重新加入到事件循环中。这个方法通常用于确保 Node.js 程序在套接字上有活动事件时不会退出

### socket.unref()
取消对事件循环的保持

### socket.remoteAddress
远程 IP 地址的字符串表示形式。例如，'74.125.127.100' 或 '2001:4860:a005::68'。如果套接字被销毁（例如，如果客户端断开连接），则值可能是 undefined。
### socket.remoteFamily
远程 IP 系列的字符串表示形式。'IPv4' 或 'IPv6'。如果套接字被销毁（例如，如果客户端断开连接），则值可能是 undefined。
### socket.remotePort
远程端口的数字表示。例如，80 或 21。如果套接字被销毁（例如，如果客户端断开连接），则值可能是 undefined。

### socket.setEncoding([encoding])
用于设置 TCP 套接字的数据编码格式。调用这个方法后，来自套接字的数据将以指定的编码格式进行解码，并作为字符串传递给 'data' 事件的回调函数。默认情况下，套接字的数据是以 Buffer 对象的形式传递的。

encoding（可选）：一个字符串，指定数据的编码格式。可以是 'utf8'、'ascii'、'base64' 等。默认值是 null，表示数据将以 Buffer 对象的形式传递

返回当前的 net.Socket 对象，以便可以链式调用其他方法

### socket.setKeepAlive([enable][, initialDelay])#
用于启用或禁用 TCP 保活（Keep-Alive）功能。TCP 保活是一种机制，用于检测空闲连接是否仍然有效。启用保活功能可以帮助检测和关闭已经失效的连接，从而释放资源
```txt
enable（可选）：布尔值，表示是否启用保活功能。默认为 false。
initialDelay（可选）：整数，表示在连接空闲多长时间后发送第一个保活探测包（以毫秒为单位）。默认值和行为取决于操作系统。
```
使用场景

检测空闲连接 ：启用保活功能可以帮助检测和关闭已经失效的连接，从而释放资源。
网络环境 ：在不稳定的网络环境中，启用保活功能可以帮助保持连接的稳定性
```js
const net = require('net');

// 创建一个 TCP 套接字
const socket = new net.Socket();

// 连接到服务器
socket.connect(8080, 'localhost', () => {
  console.log('已连接到服务器');

  // 启用 TCP 保活功能，初始延迟为 10 秒
  socket.setKeepAlive(true, 10000);

  // 发送数据到服务器
  socket.write('你好，服务器！');
});

// 监听服务器发送的数据
socket.on('data', (data) => {
  console.log(`接收到的数据: ${data}`);
  // 关闭连接
  socket.end();
});

// 监听连接关闭事件
socket.on('end', () => {
  console.log('已断开与服务器的连接');
});

// 处理错误
socket.on('error', (err) => {
  console.error(`客户端错误: ${err.message}`);
});

```

### socket.setNoDelay([noDelay])
用于启用或禁用 Nagle 算法。Nagle 算法是一种用于减少小数据包数量的网络优化技术，通过将小数据包合并成更大的数据包来提高网络效率。

然而，在某些应用场景下，例如低延迟的实时通信，禁用 Nagle 算法可以减少延迟。

使用场景：

低延迟通信 ：在需要低延迟的实时通信场景中，例如在线游戏、金融交易等，禁用 Nagle 算法可以减少数据传输的延迟。

小数据包传输 ：在频繁发送小数据包的应用中，禁用 Nagle 算法可以提高数据传输的即时性

### socket.setTimeout(timeout[, callback])
将套接字设置为在套接字上处于非活动状态 timeout 毫秒后超时。默认情况下 net.Socket 没有超时。

当触发空闲超时时，套接字将收到 'timeout' 事件，但连接不会被切断。用户必须手动调用 socket.end() 或 socket.destroy() 来结束连接。

```js
socket.setTimeout(3000);
socket.on('timeout', () => {
  console.log('socket timeout');
  socket.end();
});
```
### socket.timeout
socket.setTimeout() 设置的套接字超时（以毫秒为单位）

### socket.write(data[, encoding][, callback])
```txt
data：要发送的数据，可以是一个字符串或一个 Buffer 对象。
encoding（可选）：如果 data 是字符串，指定字符串的编码。默认是 'utf8'。
callback（可选）：一个回调函数，当数据写入完成时调用。
```
用于通过 TCP 套接字发送数据。这个方法允许你将数据写入到套接字中，并可以选择指定数据的编码格式和写入完成后的回调函数
### socket.readyState
用于指示 TCP 套接字的当前状态。这个属性可以帮助你了解套接字的连接状态，从而进行相应的处理。readyState 属性的值可以是以下几个状态之一：

'opening': 套接字正在尝试连接到远程服务器。
'open': 套接字已成功连接到远程服务器。
'readOnly': 套接字的写入端已关闭，但仍然可以读取数据。
'closing': 套接字正在关闭。
'closed': 套接字已关闭，无法再进行读写操作。

## net.connect()
net.createConnection() 的别名
用于创建并连接到远程服务器的 TCP 套接字。这个方法是 net.Socket 类的便捷方法，允许你快速创建一个 TCP 客户端并连接到指定的端口和主机。
### net.connect(options[, connectListener])
```txt
options：一个对象，包含以下属性：
  port：整数，指定要连接的远程端口号。
  host（可选）：字符串，指定要连接的远程主机名或 IP 地址。默认是 'localhost'。
  localAddress（可选）：字符串，指定本地绑定的 IP 地址。
  localPort（可选）：整数，指定本地绑定的端口号。
  family（可选）：整数，指定 IP 地址族。可以是 4 或 6。
  hints（可选）：整数，指定用于 DNS 解析的提示。
  lookup（可选）：函数，自定义 DNS 解析函数。
connectListener（可选）：函数，当连接成功时调用。
```

### net.connect(port[, host][, connectListener])
```txt
port：整数，指定要连接的远程端口号。
host（可选）：字符串，指定要连接的远程主机名或 IP 地址。默认是 'localhost'。
connectListener（可选）：函数，当连接成功时调用。
```

### net.connect(path[, connectListener])
```txt
port：整数，指定要连接的远程端口号。
host（可选）：字符串，指定要连接的远程主机名或 IP 地址。默认是 'localhost'。
connectListener（可选）：函数，当连接成功时调用。
```

### net.createConnection()
见上

### net.createServer([options][, connectionListener])
用于创建一个新的 TCP 服务器实例。这个方法可以接受一个可选的配置对象 options 和一个可选的连接监听器 connectionListener。
```txt
options（可选）：一个对象，包含以下可选属性：
  allowHalfOpen：布尔值，表示是否允许半开连接（即服务器端的套接字可以在客户端关闭其写入端后仍然保持打开）。默认为 false。
  pauseOnConnect：布尔值，表示是否在连接建立时暂停套接字。默认为 false。
connectionListener（可选）：一个函数，当客户端连接到服务器时调用。该函数接受一个参数 socket，表示与客户端的连接。
```

### net.getDefaultAutoSelectFamily()
获取 socket.connect(options) 的 autoSelectFamily 选项的当前默认值。初始默认值为 true，除非提供了命令行选项 --no-network-family-autoselection

### net.setDefaultAutoSelectFamily(value)
设置 socket.connect(options) 的 autoSelectFamily 选项的默认值。

value <boolean> 新的默认值。初始默认值为 false

### net.getDefaultAutoSelectFamilyAttemptTimeout()#
获取 socket.connect(options) 的 autoSelectFamilyAttemptTimeout 选项的当前默认值。

初始默认值为 250 或通过命令行选项 --network-family-autoselection-attempt-timeout 指定的值

### net.setDefaultAutoSelectFamilyAttemptTimeout(value)
设置 socket.connect(options) 的 autoSelectFamilyAttemptTimeout 选项的默认值。

value <number> 新的默认值，必须是正数。如果数字小于 10，则使用值 10。初始默认值为 250 或通过命令行选项 --network-family-autoselection-attempt-timeout 指定的值

### net.isIP(input)
如果 input 是 IPv6 地址，则返回 6。如果 input 是 点十进制表示法 中没有前导零的 IPv4 地址，则返回 4。否则，返回 0。

```js
net.isIP('::1'); // returns 6
net.isIP('127.0.0.1'); // returns 4
net.isIP('127.000.000.001'); // returns 0
net.isIP('127.0.0.1/24'); // returns 0
net.isIP('fhqwhgads'); // returns 0
```
### net.isIPv4(input)
如果 input 是 点十进制表示法 中没有前导零的 IPv4 地址，则返回 true。否则，返回 false。

### net.isIPv6(input)
如果 input 是 IPv6 地址，则返回 true。否则，返回 false
