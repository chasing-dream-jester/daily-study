# Http

##  Class:Agent
Agent 负责管理 HTTP 客户端连接的持久性和重用。它维护一个给定主机和端口的待处理请求队列，为每个请求重用单个套接字连接，直到队列为空，此时套接字要么被销毁，要么放入池中，在那里它会被再次用于请求到相同的主机和端口，它负责处理连接的重用、连接池和请求的并发性。通过使用 http.Agent，你可以更好地控制 HTTP 请求的行为，尤其是在处理大量请求时


### new Agent([options])
用于创建一个新的 HTTP 或 HTTPS 代理对象，这个代理对象用于管理 HTTP 请求的连接，提供了一些配置选项来优化连接的使用。
```txt
options 是一个可选的对象，用于配置代理的行为。以下是一些常用的选项：

  keepAlive (boolean)：如果为 true，则启用 Keep-Alive 功能，允许连接在请求之间保持打开状态。默认值为 false。
  maxSockets (number)：允许的最大并发连接数。默认值为 Infinity。可以设置为一个具体的数字来限制并发连接数。
  maxFreeSockets (number)：最大空闲连接数。如果空闲连接数超过这个值，代理会关闭多余的空闲连接。默认值为 256。
  timeout (number)：连接的超时时间（以毫秒为单位）。如果连接在指定时间内没有活动，则会被关闭。
  socketActiveTTL (number)：活动的 socket 的生存时间（以毫秒为单位）。如果 socket 在此时间内没有被使用，它将被关闭。
  scheduling <string> 选择下一个要使用的空闲套接字时应用的调度策略。它可以是 'fifo' 或 'lifo'。两种调度策略的主要区别在于 'lifo' 选择最近使用的套接字，而 'fifo' 选择最近最少使用的套接字。在每秒请求率较低的情况下，'lifo' 调度将降低选择可能因不活动而被服务器关闭的套接字的风险。在每秒请求率较高的情况下，'fifo' 调度将最大化打开套接字的数量，而 'lifo' 调度将保持尽可能低。默认值：'lifo'。

```
```js
// 创建一个自定义的 HTTP Agent
const agent = new http.Agent({
  keepAlive: true,
  maxSockets: 10,
  maxFreeSockets: 5,
  timeout: 60000
});

// 使用 Agent 发起 HTTP 请求
const options = {
  hostname: 'example.com',
  port: 80,
  path: '/',
  method: 'GET',
  agent: agent // 将自定义 Agent 传递给请求
};

const req = http.request(options, (res) => {
  console.log(`状态码: ${res.statusCode}`);

  res.on('data', (d) => {
    process.stdout.write(d);
  });
});
```

###  agent.createConnection(options[, callback])
```txt
options:一个对象，包含连接所需的配置选项,查看 net.createConnection() 以获取选项的格式
  host ：要连接的主机名。
  port ：要连接的端口号。
  localAddress ：本地地址。
  timeout ：连接超时时间（以毫秒为单位）。
  family ：IP 地址的版本（如 4 或 6）。
callback <Function> 接收创建的套接字的回调函数
返回一个 net.Socket 对象，表示与远程服务器的连接
```

###  agent.keepSocketAlive(socket)
agent.keepSocketAlive(socket) 是一个方法，用于将一个特定的 socket 标记为可重用（即保持活跃状态）。这意味着该 socket 可以在多个 HTTP 请求之间保持打开状态，从而减少连接的建立和关闭所带来的开销

功能
连接重用 ：通过将 socket 标记为活跃，http.Agent 或 https.Agent 可以在后续请求中重用该连接，而无需重新建立连接。
性能优化 ：在高并发的场景中，保持活跃的 socket 可以显著提高性能，减少延迟和资源消耗。


###  agent.reuseSocket(socket, request)
用于将现有的 socket 重新用于新的请求。这可以提高性能，因为它避免了建立新的连接的开销
```txt
socket ：要重用的 net.Socket 或 tls.TLSSocket 对象。通常是已经建立的连接。
request ：发起的 HTTP 请求对象。这个对象通常是通过 http.request 或 https.request 创建的。
```
一般情况下，reuseSocket 是由 http.Agent 或 https.Agent 自动处理的，开发者不需要手动调用它
###  agent.destroy()
销毁代理当前正在使用的所有套接字。通常没有必要这样做。但是，如果使用启用了 keepAlive 的代理，则最好在不再需要代理时显式关闭该代理。否则，套接字可能会在服务器终止它们之前保持打开很长时间

###  agent.freeSockets
当启用 keepAlive 时，包含当前等待代理使用的套接字数组的对象。不要修改。

freeSockets 列表中的套接字将被自动销毁并从 'timeout' 上的数组中删除。

###  agent.getName([options])
```txt
options <Object> 一组提供名称生成信息的选项

host <string> 向其触发请求的服务器的域名或 IP 地址

port <number> 远程服务器端口

localAddress <string> 触发请求时绑定网络连接的本地接口

family <integer> 如果这不等于 undefined，则必须是 4 或 6。

返回：<string>
```
获取一组请求选项的唯一名称,以确定是否可以重用连接

###  agent.maxFreeSockets
默认设置为 256。对于启用了 keepAlive 的代理，这将设置在空闲状态下将保持打开的最大套接字数量。

###  agent.maxSockets
默认设置为 Infinity。确定代理可以为每个来源打开多少个并发套接字。来源是 agent.getName() 的返回值。

###  agent.maxTotalSockets
默认设置为 Infinity。确定代理可以打开多少个并发套接字。与 maxSockets 不同，此参数适用于所有来源。


###  agent.requests
包含尚未分配给套接字的请求队列的对象。不要修改。

###  agent.sockets
包含代理当前正在使用的套接字数组的对象。不要修改

##  http.ClientRequest

http.ClientRequest 是 Node.js 中 http 模块的一部分，表示一个 HTTP 请求的实例。它是通过 http.request() 方法创建的，用于向服务器发送请求并接收响应
主要属性和方法

### 事件：
#### close：
表示请求已完成，或者其底层连接提前终止（在响应完成之前）

#### connect：
每次服务器使用 CONNECT 方法响应请求时触发。如果未监听此事件，则接收 CONNECT 方法的客户端将关闭其连接。

#### continue：
当服务器发送 '100 继续' HTTP 响应时触发，通常是因为请求包含“Expect:100-继续'。这是客户端应该发送请求正文的指令。

information：当服务器发送 1xx 中间响应（不包括 101 升级）时触发。此事件的监听器将接收一个对象，其中包含 HTTP 版本、状态码、状态消息、键值标头对象和带有原始标头名称及其各自值的数组
```txt
info <Object>

  httpVersion <string>

  httpVersionMajor <integer>

  httpVersionMinor <integer>

  statusCode <integer>

  statusMessage <string>

  headers <Object>

  rawHeaders <string[]>
```

#### response ：
当收到响应时触发。

#### error ：
当请求过程中发生错误时触发。

#### socket ：
当请求的 socket 连接建立时触发。

#### 
timeout ：当请求超时时触发（需要设置超时）

#### upgrade:
每次服务器响应升级请求时触发。如果未监听此事件且响应状态码为 101 Switching Protocols，则接收升级标头的客户端将关闭其连接。

### 属性 ：
#### maxHeadersCount:
限制最大响应头计数。如果设置为 0，则不会应用任何限制

#### path:
请求路径

#### host:
请求主机

#### protocol:
请求协议

#### method:
请求方法

#### reusedSocket:
请求是否通过重用套接字发送

#### destroyed:
在调用 request.destroy() 之后是 true

#### socket:
对底层套接字的引用。通常用户不会想要访问这个属性

#### writableEnded:
在调用 request.end() 之后是 true

#### writableFinished:
如果所有数据都已在 'finish' 事件触发之前立即刷新到底层系统，则为 true
### 方法 ：
#### abort() ：
中止请求。

#### setHeader(name, value) ：
设置请求头。

#### getHeader(name) ：
获取请求头的值。

#### getHeaderNames()：
返回包含当前传出标头的唯一名称的数组。所有标头名称均为小写

#### getHeaders：
返回当前传出标头的浅拷贝

#### getRawHeaderNames()：
返回包含当前传出原始标头的唯一名称的数组。标头名称返回并设置了它们的确切大小写

#### hasHeader(name)：
如果 name 标识的标头当前设置在传出标头中，则返回 true。标头名称匹配不区分大小写。

#### removeHeader(name) ：
移除请求头。

#### write(chunk[, encoding][, callback])：
向请求主体写入数据（适用于 POST 或其他有请求体的请求）。

#### end([data[, encoding]][, callback]) ：
结束请求，可以选择性地发送最后的数据块。

#### destroy([error]):
error <Error> 可选，与 'error' 事件一起触发的错误

#### flushHeaders():
刷新请求头

#### setSocketKeepAlive([enable][, initialDelay])：
一旦套接字被分配给这个请求并被连接，则 socket.setKeepAlive() 将被调用。
```txt
enable <boolean> 
initialDelay <number>
```
#### setTimeout(timeout[, callback]):
一旦套接字被分配给这个请求并被连接，则 socket.setTimeout() 将被调用

## Class:http.Server
http.Server 是 Node.js 的内置模块 http 中的一个类，用于创建 HTTP 服务器。这个服务器可以接收客户端的请求并返回响应

### 事件：
checkContinue：每次收到带有 HTTP Expect: 100-continue 的请求时触发。如果未监听此事件，则服务器将根据需要自动响应 100 Continue。

#### checkExpectation
每次收到带有 HTTP Expect 标头的请求时触发，其中值不是 100-continue。如果未监听此事件，则服务器将根据需要自动响应 417 Expectation Failed。

#### clientError：如果客户端连接触发 'error' 事件，则会在此处转发。此事件的监听器负责关闭/销毁底层套接字

#### close
服务器关闭时触发

#### connect
```txt
request <http.IncomingMessage> HTTP 请求的参数，如它在 'request' 事件中

socket <stream.Duplex> 服务器和客户端之间的网络套接字

head <Buffer> 隧道流的第一个数据包（可能为空）
```
每次客户端请求 HTTP CONNECT 方法时触发。如果未监听此事件，则请求 CONNECT 方法的客户端将关闭其连接。


#### connection
当建立新的 TCP 流时会触发此事件。socket 通常是 net.Socket 类型的对象。通常用户不会想访问这个事件

#### dropRequest
```txt
request <http.IncomingMessage> HTTP 请求的参数，如它在 'request' 事件中

socket <stream.Duplex> 服务器和客户端之间的网络套接字
```
当套接字上的请求数达到 server.maxRequestsPerSocket 的阈值时，服务器会丢弃新的请求并触发 'dropRequest' 事件，然后将 503 发送给客户端。

#### request
每次有请求时触发。每个连接可能有多个请求（在 HTTP Keep-Alive 连接的情况下）。

#### upgrade
```txt
request <http.IncomingMessage> HTTP 请求的参数，如它在 'request' 事件中

socket <stream.Duplex> 服务器和客户端之间的网络套接字

head <Buffer> 升级流的第一个数据包（可能为空）
```
每次客户端请求 HTTP 升级时触发。监听此事件是可选的，客户端不能坚持协议更改。

触发此事件后，请求的套接字将没有 'data' 事件监听器，这意味着需要绑定它才能处理发送到该套接字上的服务器的数据。


### 属性
#### listening
指示服务器是否正在监听连接，返回boolean

#### server.requestTimeout
设置从客户端接收整个请求的超时值（以毫秒为单位）。

如果超时到期，则服务器以状态 408 响应而不将请求转发给请求监听器，然后关闭连接。

必须将其设置为非零值（例如 120 秒）以防止潜在的拒绝服务攻击，以防在部署服务器之前没有反向代理的情况下。

#### server.maxRequestsPerSocket
关闭保持活动的连接之前，套接字可以处理的最大请求数。0 值将禁用限制。

当达到限制时，则它会将 Connection 标头值设置为 close，但不会实际地关闭连接，达到限制后发送的后续请求将获得 503 Service Unavailable 作为响应。

#### server.timeout
超时（以毫秒为单位）。默认值：0（无超时）
假定套接字超时之前不活动的毫秒数

#### server.keepAliveTimeout
超时（以毫秒为单位）。默认值：5000（5 秒）

### 方法
#### server.close([callback])
停止服务器接受新连接并关闭连接到该服务器的所有未发送请求或等待响应的连接

#### server.closeAllConnections()
关闭连接到此服务器的所有已建立的 HTTP(S) 连接，包括连接到此服务器的正在发送请求或等待响应的活动连接

#### server.closeIdleConnections()
关闭连接到此服务器的所有未发送请求或等待响应的连接。

#### server.headersTimeout
限制解析器等待接收完整 HTTP 标头的时间。

如果超时到期，则服务器以状态 408 响应而不将请求转发给请求监听器，然后关闭连接。

必须将其设置为非零值（例如 120 秒）以防止潜在的拒绝服务攻击，以防在部署服务器之前没有反向代理的情况下。

#### server.listen()
启动 HTTP 服务器监听连接。此方法与 net.Server 中的 server.listen() 相同。

#### server.setTimeout([msecs][, callback])
```txt
msecs <number> 默认值：0（无超时）

callback <Function>

返回：<http.Server>
```
设置套接字的超时值，并在服务器对象上触发 'timeout' 事件，如果发生超时，则将套接字作为参数传入。

如果 Server 对象上有 'timeout' 事件监听器，则将使用超时套接字作为参数调用它。

默认情况下，服务器不会超时套接字。但是，如果将回调分配给服务器的 'timeout' 事件，则必须显式处理超时。


## Class：http.OutgoingMessage
http.OutgoingMessage 是 Node.js 中 http 模块的一个类，它是 http.ClientRequest 和 http.ServerResponse 的基类。这个类用于处理 HTTP 请求和响应的基本功能，提供了一些方法和属性来管理 HTTP 消息的发送
### 事件

#### drain
当消息的缓冲区再次空闲时触发

#### finish
当传输成功完成时触发

#### prefinish
调用 outgoingMessage.end() 后触发。触发事件时，所有数据都已处理，但不一定完全刷新。


### 属性
#### outgoingMessage.socket
对底层套接字的引用。通常，用户不会希望访问此属性。

调用 outgoingMessage.end() 后，该属性将被清空
#### outgoingMessage.headersSent
只读。如果标头已发送，则为 true，否则为 false。

#### outgoingMessage.writableCorked
调用 outgoingMessage.cork() 的次数

#### outgoingMessage.writableEnded
如果调用了 outgoingMessage.end()，则为 true。该属性不表示数据是否被刷新。为此目的，则改用 message.writableFinished。

#### outgoingMessage.writableFinished
如果所有数据都已刷新到底层系统，则为 true

#### outgoingMessage.writableHighWaterMark
如果分配了底层套接字的 highWaterMark。否则，writable.write() 开始时的默认缓冲级别返回 false（16384）。

#### outgoingMessage.writableObjectMode
始终为 false


### 方法
#### outgoingMessage.addTrailers(headers)
添加 HTTP 尾标（标头，但在消息末尾）到消息。

仅当消息经过分块编码时才会触发标尾。如果没有，则块头将被默默丢弃,尝试设置包含无效字符的标头字段名称或值将导致抛出 TypeError。
```js
message.writeHead(200, { 'Content-Type': 'text/plain','Trailer': 'Content-MD5' });
message.write(fileData);
message.addTrailers({ 'Content-MD5': '7895bf4b8828b55ceaf47747b4bca667' });
message.end();
```

#### outgoingMessage.appendHeader(name, value)
```txt
name <string> 标头名称

value <string> | <string[]> 标头值

返回：<this>
```
将单个标头值附加到标头对象。

如果该值为数组，则相当于多次调用该方法。

如果标头没有先前的值，则相当于调用 outgoingMessage.setHeader(name, value)。

根据创建客户端请求或服务器时 options.uniqueHeaders 的值，这将导致标头多次发送或单次发送，并使用 ; 连接值。

#### outgoingMessage.destroy([error])
销毁消息。一旦套接字与消息关联并连接，则该套接字也将被销毁

#### outgoingMessage.end(chunk[, encoding][, callback])
```txt
chunk <string> | <Buffer> | <Uint8Array>

encoding <string> 可选，默认：utf8

callback <Function> 可选的

返回：<this>
```
完成传出消息。如果正文的任何部分未发送，则会将它们刷新到底层系统。如果消息被分块，则它将发送终止块 0\r\n\r\n，并发送块尾（如果有）。

如果指定了 chunk，则相当于调用 outgoingMessage.write(chunk, encoding)，后跟 outgoingMessage.end(callback)。

如果提供了 callback，则在消息结束时调用（相当于 'finish' 事件的监听器）。

#### outgoingMessage.flushHeaders()
刷新消息标头

#### outgoingMessage.getHeader(name)
获取具有给定名称的 HTTP 标头的值。如果未设置该标头，则返回值为 undefined

#### outgoingMessage.getHeaderNames()
返回包含当前传出标头的唯一名称的数组。所有名称均为小写

#### outgoingMessage.getHeaders()
返回当前传出标头的浅拷贝。由于使用了浅拷贝，因此无需额外调用各种与标头相关的 HTTP 模块方法即可更改数组值。返回对象的键是标头名称，值是相应的标头值。所有标头名称均为小写


#### outgoingMessage.hasHeader(name)
如果 name 标识的标头当前设置在传出标头中，则返回 true。标头名称不区分大小写。

#### outgoingMessage.removeHeader(name)
删除排队等待隐式发送的标头

#### outgoingMessage.setHeader(name, value)
设置单个标头值。如果待发送标头中已经存在该标头，则替换其值。使用字符串数组发送多个同名标头

#### outgoingMessage.setHeaders(headers)
为隐式标头设置多个标头值。headers 必须是 Headers 或 Map 的一个实例，如果一个标头已经存在于待发送的标头中，它的值将被替换。当标头已使用 outgoingMessage.setHeaders() 设置时，则它们将与任何传给 response.writeHead() 的标头合并，其中传给 response.writeHead() 的标头优先

#### outgoingMessage.pipe()
覆盖继承自旧版的 Stream 类（http.OutgoingMessage 的父类）的 stream.pipe() 方法,调用此方法将抛出 Error，因为 outgoingMessage 是只写流

#### outgoingMessage.setTimeout(msesc[, callback])
一旦套接字与消息关联并连接，则 socket.setTimeout() 将被调用，msecs 作为第一个参数


#### outgoingMessage.write(chunk[, encoding][, callback])
```txt
chunk <string> | <Buffer> | <Uint8Array>

encoding <string> 默认值：utf8

callback <Function>

返回：<boolean>
```
发送一块正文。此方法可以被多次调用,encoding 参数仅在 chunk 是字符串时才相关。默认为 'utf8'。

callback 参数是可选的，当此数据被刷新时将被调用。

如果整个数据被成功刷新到内核缓冲区，则返回 true。如果所有或部分数据在用户内存中排队，则返回 false。当缓冲区再次空闲时将触发 'drain' 事件。


## Class:http.IncomingMessage
http.IncomingMessage 是 Node.js 中 http 模块的一个类，用于表示来自客户端的 HTTP 请求。它是 http.ServerRequest 的基类，并在 HTTP 请求被接收时创建。通过这个对象，你可以访问请求的各种信息，如请求头、请求方法、请求 URL 等

### 事件
#### close
当请求完成时触发

### 属性
#### message.aborted
如果请求已中止，则 message.aborted 属性将为 true

#### message.complete
如果已接收并成功解析完整的 HTTP 消息，则 message.complete 属性将为 true。

此属性作为一种确定客户端或服务器是否在连接终止之前完全传输消息的方法特别有用

```js
const req = http.request({
  host: '127.0.0.1',
  port: 8080,
  method: 'POST',
}, (res) => {
  res.resume();
  res.on('end', () => {
    if (!res.complete)
      console.error(
        'The connection was terminated while the message was still being sent');
  });
});
```
#### message.headers
请求/响应头对象。

标头名称和值的键值对。标头名称是小写的

原始标头中的重复项按以下方式处理，具体取决于标头名称：

1. 重复的 age、authorization、content-length、content-type、etag、expires、from、host、if-modified-since、if-unmodified-since、last-modified、location、max-forwards、proxy-authorization、referer、retry-after、server 或 user-agent 被丢弃。要允许合并上面列出的标头的重复值，请在 http.request() 和 http.createServer() 中使用选项 joinDuplicateHeaders。有关详细信息，请参阅 RFC 9110 第 5.3 节。

2. set-cookie 始终是数组。重复项被添加到数组中。

3. 对于重复的 cookie 标头，值使用 ; 连接。

4. 对于所有其他标头，值使用 , 连接。

#### message.headersDistinct
类似于 message.headers，但没有连接逻辑，并且值始终是字符串数组，即使对于仅收到一次的标头也是如此。

```js
// Prints something like:
//
// { 'user-agent': ['curl/7.22.0'],
//   host: ['127.0.0.1:8000'],
//   accept: ['*/*'] }
console.log(request.headersDistinct);
```

#### message.httpVersion
在服务器请求的情况下，客户端发送的 HTTP 版本。在客户端响应的情况下，连接到服务器的 HTTP 版本。可能是 '1.1' 或 '1.0'。

message.httpVersionMajor 是第一个整数，message.httpVersionMinor 是第二个

#### message.method
请求方法作为字符串。只读。示例：'GET', 'DELETE'

#### message.rawHeaders
原始请求/响应头完全按照收到的方式列出。

键和值在同一个列表中。它不是元组列表。因此，偶数偏移是键值，奇数偏移是关联的值。

标头名称不小写，重复项不合并

#### message.rawTrailers
原始请求/响应尾标的键和值与收到的完全一样。仅在 'end' 事件中填充。
#### message.socket
与连接关联的 net.Socket 对象。
#### message.statusCode
3 位 HTTP 响应状态码。E.G.404
#### message.statusMessage
仅对从 http.ClientRequest 获得的响应有效。

HTTP 响应状态消息（原因短语）。E.G.OK 或 Internal Server Error。

#### message.trailers
请求/响应尾标对象。仅在 'end' 事件中填充

#### message.trailersDistinct
类似于 message.trailers，但没有连接逻辑，并且值始终是字符串数组，即使对于仅收到一次的标头也是如此。仅在 'end' 事件中填充。

#### message.url
仅对从 http.Server 获得的请求有效

### 方法
#### message.destroy([error])
在接收到 IncomingMessage 的套接字上调用 destroy()。如果提供了 error，则在套接字上触发 'error' 事件，并将 error 作为参数传给该事件的任何监听器
#### message.setTimeout(msecs[, callback])


## Class:http.ServerResponse
继承：http.OutgoingMessage

http.ServerResponse 是 Node.js 的 http 模块中的一个类，表示 HTTP 服务器的响应对象。它用于向客户端发送 HTTP 响应，并提供了多种方法和属性来设置响应的状态、头部、主体等

### 事件
#### close
表示响应已完成，或者其底层连接提前终止（在响应完成之前）。

#### finish
发送响应时触发。更具体地说，当响应头和正文的最后一段已移交给操作系统以通过网络传输时，则将触发此事件。这并不意味着客户端已收到任何东西。

### 方法
#### response.addTrailers(headers)
此方法向响应添加 HTTP 尾随标头（标头，但位于消息末尾）

仅当响应使用分块编码时才会触发标尾；如果不是（例如，如果请求是 HTTP/1.0），它们将被静默丢弃

HTTP 要求发送 Trailer 标头以触发尾标，其值中包含标头字段列表。例如
```js
response.writeHead(200, { 'Content-Type': 'text/plain',
                          'Trailer': 'Content-MD5' });
response.write(fileData);
response.addTrailers({ 'Content-MD5': '7895bf4b8828b55ceaf47747b4bca667' });
response.end();
```
#### response.end([data[, encoding]][, callback])
```txt
data <string> | <Buffer> | <Uint8Array>

encoding <string>

callback <Function>

返回：<this>
```
此方法向服务器触发信号，表明已发送所有响应标头和正文；该服务器应认为此消息已完成。response.end() 方法必须在每个响应上调用。

如果指定了 data，则其效果类似于调用 response.write(data, encoding) 后跟 response.end(callback)。

如果指定了 callback，则将在响应流完成时调用

#### response.flushHeaders()
刷新响应头

#### response.getHeader(name)
读取已排队但未发送到客户端的标头,返回值的类型取决于提供给 response.setHeader() 的参数

#### response.getHeaderNames()
返回包含当前传出标头的唯一名称的数组。所有标头名称均为小写
```js
response.setHeader('Foo', 'bar');
response.setHeader('Set-Cookie', ['foo=bar', 'bar=baz']);

const headerNames = response.getHeaderNames();
// headerNames === ['foo', 'set-cookie']
```
#### response.getHeaders()
返回当前传出标头的浅拷贝。由于使用了浅拷贝，因此无需额外调用各种与标头相关的 http 模块方法即可更改数组值。返回对象的键是标头名称，值是相应的标头值。所有标头名称均为小写.

#### response.hasHeader(name)
如果 name 标识的标头当前设置在传出标头中，则返回 true。标头名称匹配不区分大小写。
#### response.removeHeader(name)
删除排队等待隐式发送的标头
#### response.setHeader(name, value)
为隐式标头设置单个标头值。如果该标头已经存在于待发送的标头中，则其值将被替换。在此处使用字符串数组发送具有相同名称的多个标头。非字符串值将不加修改地存储。因此，response.getHeader() 可能返回非字符串值。但是，非字符串值将转换为字符串以进行网络传输。相同的响应对象返回给调用者，以启用调用链.

#### response.setTimeout(msecs[, callback])
将套接字的超时值设置为 msecs。如果提供了回调，则将其添加为响应对象上 'timeout' 事件的监听器。

如果没有向请求、响应或服务器添加 'timeout' 监听器，则套接字在超时时会被销毁。如果将句柄分配给请求、响应或服务器的 'timeout' 事件，则必须显式处理超时套接字
#### response.write(chunk[, encoding][, callback])
```txt
chunk <string> | <Buffer> | <Uint8Array>

encoding <string> 默认值：'utf8'

callback <Function>
```
发送数据
#### response.writeContinue()
向客户端发送 HTTP/1.1 100 Continue 消息，指示应发送请求正文。请参阅 Server 上的 'checkContinue' 事件。

#### response.writeHead(statusCode[, statusMessage][, headers])
```txt
statusCode <number>

statusMessage <string>

headers <Object> | <Array>

返回：<http.ServerResponse>
```
向请求发送响应头。状态码是 3 位的 HTTP 状态码，如 404。最后一个参数 headers 是响应头。可选地给定人类可读的 statusMessage 作为第二个参数。

headers 可以是 Array，其中键和值在同一个列表中。它不是元组列表。因此，偶数偏移是键值，奇数偏移是关联的值。该数组的格式与 request.rawHeaders 相同。

#### response.writeProcessing()
向客户端发送 HTTP/1.1 102 Processing 消息，表示应发送请求正文
### 属性
#### response.headersSent
布尔值（只读）。如果标头被发送，则为真，否则为假

#### response.req
对原始的 HTTP request 对象的引用

#### response.sendDate
如果为真，则 Date 标头将自动生成并在响应中发送，如果它尚未出现在标头中。默认为真。

这应该只在测试时被禁用；HTTP 要求在响应中使用 Date 标头

#### response.socket
对底层套接字的引用,通常用户不会想要访问这个属

#### response.statusCode
响应头发送到客户端后，该属性表示发送出去的状态码

#### response.statusMessage
响应头发送到客户端后，该属性表示发送出去的状态消息

#### response.strictContentLength
strictContentLength 属性是一个布尔值，默认情况下为 false。当它设置为 true 时，Node.js 将严格检查 Content-Length 响应头的值。如果响应的实际内容长度与 Content-Length 的值不匹配，Node.js 将会抛出错误。这个属性的主要目的是提供更严格的响应验证，以确保客户端能够正确解析响应内容
```js
const http = require('http');

// 创建 HTTP 服务器
const server = http.createServer((req, res) => {
  // 设置严格内容长度检查
  res.strictContentLength = true; // 启用严格检查

  // 设置响应头和状态码
  res.writeHead(200, { 'Content-Type': 'text/plain', 'Content-Length': '12' });

  // 写入响应数据
  res.write('Hello, World!'); // 注意：这里的内容长度是 13，而 Content-Length 是 12

  // 结束响应
  res.end(); // 这将抛出错误，因为内容长度不匹配
});

// 服务器监听端口
const PORT = 3000;
server.listen(PORT, () => {
  console.log(`服务器正在运行，访问 http://localhost:${PORT}/`);
});

```
#### response.writableEnded
在调用 response.end() 之后是 true。此属性不指示数据是否已刷新，为此则使用 response.writableFinished 代替。

#### response.writableFinished
如果所有数据都已在 'finish' 事件触发之前立即刷新到底层系统，则为 true。

