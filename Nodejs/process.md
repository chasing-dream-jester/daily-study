# process
process 对象是 Node.js 中的一个全局对象，提供了与当前 Node.js 进程相关的信息和控制功能。它是一个非常重要的对象，允许开发者访问运行时的环境、管理进程的输入输出、处理信号等

## 事件
### beforeExit
当 Node.js 清空其事件循环并且没有额外的工作要安排时，则会触发 'beforeExit' 事件。通常情况下，当没有工作要调度时，Node.js 进程会退出，但是注册在 'beforeExit' 事件上的监听器可以进行异步的调用，从而使 Node.js 进程继续。

调用监听器回调函数时将 process.exitCode 的值作为唯一的参数传入
```js
const process = require('node:process');

process.on('beforeExit', (code) => {
  console.log('Process beforeExit event with code: ', code);
});

process.on('exit', (code) => {
  console.log('Process exit event with code: ', code);
});

console.log('This message is displayed first.');

// Prints:
// This message is displayed first.
// Process beforeExit event with code: 0
// Process exit event with code: 0
```
### exit
当 Node.js 进程由于以下任一原因即将退出时，则会触发 'exit' 事件：

process.exit() 方法被显式调用

Node.js 事件循环不再需要执行任何额外的工作

监听器回调函数必须仅执行同步操作。Node.js 进程将在调用 'exit' 事件监听器之后立即退出，从而使任何仍在事件循环中排队的其他工作被丢弃
### disconnect
如果 Node.js 进程是使用 IPC 通道生成的（请参阅 子进程 和 集群 文档），则当 IPC 通道关闭时将触发 'disconnect' 事件。


### message
```txt
message <Object> | <boolean> | <number> | <string> | <null> 已解析的 JSON 对象或可序列化的原始值。

sendHandle <net.Server> | <net.Socket> net.Server 或 net.Socket 对象，或未定义
```
如果 Node.js 进程是使用 IPC 通道生成的（请参阅 子进程 和 集群 文档），则只要子进程收到父进程使用 childprocess.send() 发送的消息，就会触发 'message' 事件。

消息经过序列化和解析。结果消息可能与最初发送的消息不同

### rejectionHandled
表示一个先前未处理的 Promise 拒绝（即 unhandledRejection）现在已经被处理。这个事件通常用于跟踪 Promise 拒绝的状态变化。

使用场景：

当一个 Promise 被拒绝且没有被处理时，Node.js 会触发 unhandledRejection 事件。此后，如果这个未处理的拒绝被一个 .catch() 处理，rejectionHandled 事件就会被触发。通过监听这个事件，开发者可以了解哪些拒绝已经被处理，进而做出更好的错误监控和日志记录。


```js
const process = require('node:process');

// 监听未处理的拒绝
process.on('unhandledRejection', (reason, promise) => {
  console.log('未处理的拒绝:', reason);
});

// 监听拒绝被处理事件
process.on('rejectionHandled', (promise) => {
  console.log('拒绝已被处理:', promise);
});

// 创建一个未处理的 Promise 拒绝
const p = new Promise((_, reject) => {
  reject('这是一个未处理的拒绝');
});

// 模拟处理未处理的拒绝
setTimeout(() => {
  p.catch((reason) => {
    console.log('处理拒绝:', reason);
  });
}, 2000);

```

### uncaughtException
```txt
err <Error> 未捕获的异常。

origin <string> 指示异常是源自未处理的拒绝还是源自同步错误。可以是 'uncaughtException' 或 'unhandledRejection'。后者用于在基于 Promise 的异步上下文中发生异常（或者如果 Promise 被拒绝）并且 --unhandled-rejections 标志设置为 strict 或 throw（这是默认值）并且拒绝未被处理，或者当拒绝发生在命令行入口点的 ES 模块静态加载阶段

```
uncaughtException 是 Node.js 中 process 对象的一个事件，用于监听未捕获的异常。当在 Node.js 应用程序中发生未被捕获的异常时，uncaughtException 事件会被触发。这允许开发者在异常没有被处理时采取一些措施，比如记录错误信息、执行清理操作或发送通知。

触发条件 ：

当异常在代码中未被捕获（即没有使用 try/catch 块处理）时，Node.js 会触发 uncaughtException 事件。

处理未捕获异常 ：通过监听这个事件，开发者可以在程序崩溃之前进行某些操作，例如记录日志、关闭连接等。
```js
const process = require('node:process');
const fs = require('node:fs');

process.on('uncaughtException', (err, origin) => {
  fs.writeSync(
    process.stderr.fd,
    `Caught exception: ${err}\n` +
    `Exception origin: ${origin}\n`,
  );
});

setTimeout(() => {
  console.log('This will still run.');
}, 500);

// Intentionally cause an exception, but don't catch it.
nonexistentFunc();
console.log('This will not run.');
```

### uncaughtExceptionMonitor
uncaughtExceptionMonitor 是 Node.js 中 process 对象的一个事件，用于监听未捕获的异常。在 Node.js 中，如果发生未被捕获的异常，通常会触发 uncaughtException 事件，而 uncaughtExceptionMonitor 事件则是在 uncaughtException 事件之前触发的。

主要特点

事件顺序 ：uncaughtExceptionMonitor 事件在 uncaughtException 事件之前触发。这意味着你可以使用 uncaughtExceptionMonitor 来执行某些操作（例如日志记录或监控）而不干扰最终的错误处理。

监控未捕获的异常 ：这个事件的主要目的是提供一种方式来监控未捕获的异常，而不影响应用程序的正常错误处理流程。

使用场景

uncaughtExceptionMonitor 事件可以用于开发和生产环境中进行错误监控和记录。它允许开发者在未捕获的异常发生时执行一些操作，比如记录错误信息、发送通知等。

```js
// 监听 uncaughtExceptionMonitor 事件
process.on('uncaughtExceptionMonitor', (err, origin) => {
  console.error('监控到未捕获的异常:', err);
  console.error('异常来源:', origin);
});

// 监听 uncaughtException 事件
process.on('uncaughtException', (err) => {
  console.error('未捕获的异常:', err);
  // 这里可以进行清理操作
  process.exit(1); // 以错误代码退出
});

// 模拟未捕获的异常
setTimeout(() => {
  throw new Error('这是一个未捕获的异常');
}, 1000);

```
注意事项

不建议使用 ：虽然 uncaughtExceptionMonitor 提供了监控未捕获异常的能力，但在生产环境中，依赖于未捕获异常的处理不是一个好习惯。应当尽量确保所有可能的异常都得到妥善处理。

清理和退出 ：在 uncaughtException 事件的处理器中，通常建议进行必要的清理后退出进程，因为未捕获的异常可能导致程序处于不一致状态。

### unhandledRejection
```txt
reason <Error> | <any> Promise 被拒绝的对象（通常是 Error 对象）。

promise <Promise> 被拒绝的 promise
```
unhandledRejection 是 Node.js 中 process 对象的一个事件，用于监听未处理的 Promise 拒绝。当一个 Promise 被拒绝且没有相应的 .catch() 处理时，Node.js 会触发这个事件。

触发条件 ：

当 Promise 被拒绝但没有被处理（例如，没有 .catch() 或没有 try/catch）时，unhandledRejection 事件会被触发。

捕获未处理的拒绝 ：这个事件允许开发者在应用程序中捕获未处理的 Promise 拒绝，从而进行日志记录、通知、或其他错误处理。

建议的处理方式 ：在处理 unhandledRejection 事件时，通常建议记录错误并采取适当的操作，例如通知开发者或记录日志

```js
// 监听未处理的拒绝事件
process.on('unhandledRejection', (reason, promise) => {
  console.error('未处理的拒绝:', reason);
  // 可以在这里添加额外的日志记录或错误处理逻辑
});

// 模拟未处理的 Promise 拒绝
const promise = new Promise((_, reject) => {
  reject('这是一个未处理的拒绝');
});

// 模拟处理 Promise 拒绝
setTimeout(() => {
  promise.catch((reason) => {
    console.log('处理拒绝:', reason);
  });
}, 2000);

```

### warning
```txt
arning <Error> 警告的主要属性是：

name <string> 警告的名称。默认值：'Warning'。

message <string> 系统提供的警告描述。

stack <string> 代码中触发警告的位置的堆栈跟踪。
```
每当 Node.js 触发进程警告时，则会触发 'warning' 事件

进程警告类似于错误，因为其描述了引起用户注意的异常情况。但是，警告不是正常 Node.js 和 JavaScript 错误处理流程的一部分。Node.js 可以在检测到可能导致次优应用性能、错误或安全漏洞的不良编码实践时触发警告
```js
const process = require('node:process');

process.on('warning', (warning) => {
  console.warn(warning.name);    // Print the warning name
  console.warn(warning.message); // Print the warning message
  console.warn(warning.stack);   // Print the stack trace
});
```

### worker
创建新的 <Worker> 线程后会触发 'worker' 事件。

## 属性
### process.arch
为其编译 Node.js 二进制文件的操作系统 CPU 架构。可能的值是：'arm'、'arm64'、'ia32'、'loong64'、'mips'、'mipsel'、'ppc'、'ppc64'、'riscv64'、's390'、's390x' 和 'x64'。

### process.argv
process.argv 是 Node.js 中的一个全局属性，表示命令行中传递给 Node.js 进程的参数。它是一个数组，包含了启动 Node.js 进程时提供的所有参数

**process.argv 的结构**

第一个元素 ：Node.js 可执行文件的路径（例如 /usr/local/bin/node）。

第二个元素 ：正在执行的 JavaScript 文件的路径（例如 /path/to/your/script.js）。

后续元素 ：命令行中提供的其他参数，按顺序排列。

**使用场景**

process.argv 可以用于多种场景，包括但不限于：

命令行工具 ：构建命令行工具时，使用 process.argv 来解析用户输入的参数和选项。
配置参数 ：从命令行中传递配置参数，以便在运行时动态调整应用程序的行为。
调试和测试 ：在调试和测试过程中，可以通过命令行参数传递不同的输入，以便测试不同的场景。

### process.argv0
process.argv0 属性存储了 Node.js 启动时传入的 argv[0] 原始值的只读副本。

### process.channel
如果 Node.js 进程是使用 IPC 通道生成的（请参阅 子进程 文档），则 process.channel 属性是对 IPC 通道的引用。如果不存在 IPC 通道，则此属性为 undefined
### process.config
提供了与当前 Node.js 进程的构建和配置相关的信息。它是一个对象，包含了 Node.js 的编译时配置和环境设置的详细信息
```js
{
  target_defaults:
   { cflags: [],
     default_configuration: 'Release',
     defines: [],
     include_dirs: [],
     libraries: [] },
  variables:
   {
     host_arch: 'x64',
     napi_build_version: 5,
     node_install_npm: 'true',
     node_prefix: '',
     node_shared_cares: 'false',
     node_shared_http_parser: 'false',
     node_shared_libuv: 'false',
     node_shared_zlib: 'false',
     node_use_openssl: 'true',
     node_shared_openssl: 'false',
     target_arch: 'x64',
     v8_use_snapshot: 1
   }
}
```

### process.connected
如果 Node.js 进程是使用 IPC 通道生成的（请参阅 子进程 和 集群 文档），只要连接了 IPC 通道，process.connected 属性就会返回 true，并在调用 process.disconnect() 后返回 false。

一旦 process.connected 为 false，就不能再使用 process.send() 通过 IPC 通道发送消息。
### process.debugPort
启用时 Node.js 调试器使用的端口

### process.env
process.env 属性返回包含用户环境的对象
```js
// 可能是以下结构
{
  TERM: 'xterm-256color',
  SHELL: '/usr/local/bin/bash',
  USER: 'maciej',
  PATH: '~/.bin/:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin',
  PWD: '/Users/maciej',
  EDITOR: 'vim',
  SHLVL: '1',
  HOME: '/Users/maciej',
  LOGNAME: 'maciej',
  _: '/usr/local/bin/node'
}
```

### process.execArgv
process.execArgv 是 Node.js 中的一个属性，它是一个数组，包含了启动 Node.js 进程时传递给 Node.js 的命令行参数。这些参数通常是用于配置 Node.js 的运行时行为，比如调试选项、性能分析工具等,这些选项不会出现在 process.argv 属性返回的数组中，也不包括 Node.js 可执行文件、脚本名称或脚本名称后面的任何选项。这些选项可用于衍生与父进程具有相同执行环境的子进程
```bash
node --icu-data-dir=./foo --require ./bar.js script.js --version
# process.execArgv : ["--icu-data-dir=./foo", "--require", "./bar.js"]
# process.argv : ['/usr/local/bin/node', 'script.js', '--version'] 
```
### process.execPath
process.execPath 属性返回启动 Node.js 进程的可执行文件的绝对路径名。符号链接（如果有）会被解析。

### process.exitCode
当进程正常退出或通过 process.exit() 退出而不指定代码时，将作为进程退出码的数字。

将代码指定为 process.exit(code) 将覆盖 process.exitCode 的任何先前设置

### process.noDeprecation
process.noDeprecation 属性指示是否在当前 Node.js 进程上设置了 --no-deprecation 标志。有关此标志行为的更多信息，请参阅 'warning' 事件 和 emitWarning() 方法 的文档

process.noDeprecation 是 Node.js 中的一个全局属性，用于控制 Node.js 的弃用警告（deprecation warnings）。当设置为 true 时，Node.js 将不会显示任何弃用警告。这主要用于开发过程中，当你希望忽略所有的弃用警告时，可以临时使用这个属性

注意事项：

临时使用 ：虽然可以设置 process.noDeprecation 来禁用弃用警告，但通常不建议在生产环境中使用。弃用警告提供了重要的信息，帮助开发者识别可能在未来版本中移除的功能。

调试和开发 ：在开发过程中，如果你知道某些功能是弃用的并且计划在将来的版本中更新代码，临时禁用警告可能会使输出更清晰。

不影响全局设置 ：设置 process.noDeprecation 只会影响当前进程，不会影响其他运行的 Node.js 进程或全局环境

### process.pid
process.pid 属性返回进程的 PID

### process.platform
process.platform 属性返回用于标识编译 Node.js 二进制文件的操作系统平台的字符串。

可能的值

'aix'

'darwin'

'freebsd'

'linux'

'openbsd'

'sunos'

'win32'

### process.ppid
process.ppid 属性返回当前进程的父进程的 PID。

### process.report
process.report 是一个对象，其方法用于为当前进程生成诊断报告。报告文件 中提供了其他文档。


### process.report.compact
以紧凑的单行 JSON 格式编写报告，与专为人类使用而设计的默认多行格式相比，日志处理系统更易于使用。

### process.report.directory
写入报告的目录。默认值为空字符串，表示将报告写入 Node.js 进程的当前工作目录。

### process.report.filename
写入报告的文件名。如果设置为空字符串，则输出文件名将由时间戳、PID 和序列号组成。默认值为空字符串。
### process.report.reportOnFatalError
如果为 true，则会生成有关致命错误（例如内存不足错误或 C++ 断言失败）的诊断报告。

### process.report.reportOnSignal
如果为 true，则当进程接收到 process.report.signal 指定的信号时生成诊断报告。

### process.report.reportOnUncaughtException
如果为 true，则针对未捕获的异常生成诊断报告

### process.report.signal
用于触发诊断报告创建的信号。默认为 'SIGUSR2'

### process.stderr
process.stderr 属性返回连接到 stderr (文件描述符 2) 的流。它是一个 net.Socket（它是一个 双工 流）除非 fd 2 引用一个文件，在这种情况下它是一个 可写 流

### process.stderr.fd
该属性指的是 process.stderr 的底层文件描述符的值。该值固定为 2。在 Worker 线程中，该字段不存在。

### process.stdin
process.stdin 属性返回连接到 stdin (文件描述符 0) 的流。它是一个 net.Socket（它是一个 双工 流）除非 fd 0 引用一个文件，在这种情况下它是一个 可读 流。

有关如何从 stdin 读取的详细信息，请参阅 readable.read()。

作为 双工 流，process.stdin 也可以在 "old" 模式下使用，与 v0.10 之前为 Node.js 编写的脚本兼容。有关详细信息，请参阅 流兼容性。

在 "old" 流模式下，stdin 流默认暂停，因此必须调用 process.stdin.resume() 才能从中读取。另请注意，调用 process.stdin.resume() 本身会将流切换到 "old" 模式
### process.stdin.fd
该属性指的是 process.stdin 的底层文件描述符的值。该值固定为 0。在 Worker 线程中，该字段不存在。

### process.stdout
process.stdout 属性返回连接到 stdout (文件描述符 1) 的流。它是一个 net.Socket（它是一个 双工 流）除非 fd 1 引用一个文件，在这种情况下它是一个 可写 流。

### process.stdout.fd
该属性指的是 process.stdout 的底层文件描述符的值。该值固定为 1。在 Worker 线程中，该字段不存在。

### process.versions
rocess.versions 属性返回对象，其中列出了 Node.js 的版本字符串及其依赖。process.versions.modules 表示当前的 ABI 版本，每当 C++ API 更改时都会增加。Node.js 将拒绝加载针对不同模块 ABI 版本编译的模块
```js
const { versions } = require('node:process');

console.log(versions);
/* 
{ 
    node: '23.0.0',
    acorn: '8.11.3',
    ada: '2.7.8',
    ares: '1.28.1',
    base64: '0.5.2',
    brotli: '1.1.0',
    cjs_module_lexer: '1.2.2',
    cldr: '45.0',
    icu: '75.1',
    llhttp: '9.2.1',
    modules: '127',
    napi: '9',
    nghttp2: '1.61.0',
    nghttp3: '0.7.0',
    ngtcp2: '1.3.0',
    openssl: '3.0.13+quic',
    simdjson: '3.8.0',
    simdutf: '5.2.4',
    sqlite: '3.46.0',
    tz: '2024a',
    undici: '6.13.0',
    unicode: '15.1',
    uv: '1.48.0',
    uvwasi: '0.0.20',
    v8: '12.4.254.14-node.11',
    zlib: '1.3.0.1-motley-7d77fb7' 
}
*/
```
### process.version
process.version 属性包含 Node.js 版本字符串。
## 方法
### process.abort()
process.abort() 方法会导致 Node.js 进程立即退出并生成一个核心文件

### process.channel.ref()
如果之前已调用过 .unref()，则此方法使 IPC 通道保持进程的事件循环运行。

通常，这是通过 process 对象上的 'disconnect' 和 'message' 监听器的数量来管理的。但是，此方法可用于显式请求特定行为。

### process.channel.unref()
此方法使 IPC 通道不会保持进程的事件循环运行，并且即使在通道打开时也让它完成

### process.chdir(directory)
process.chdir() 方法更改 Node.js 进程的当前工作目录，如果失败则抛出异常（例如，如果指定的 directory 不存在）。

```js
const { chdir, cwd } = require('node:process');

console.log(`Starting directory: ${cwd()}`);
try {
  chdir('/tmp');
  console.log(`New directory: ${cwd()}`);
} catch (err) {
  console.error(`chdir: ${err}`);
}
```
### process.cpuUsage([previousValue])
process.cpuUsage() 方法在具有属性 user 和 system 的对象中返回当前进程的用户和系统 CPU 时间使用情况，其值为微秒值（百万分之一秒）。这些值分别测量在用户和系统代码中花费的时间，如果多个 CPU 内核为此进程执行工作，则最终可能会大于实际经过的时间。

### process.cwd()
process.cwd() 方法返回 Node.js 进程的当前工作目录

### process.disconnect()
如果 Node.js 进程是使用 IPC 通道生成的（请参阅 子进程 和 集群 文档），process.disconnect() 方法将关闭到父进程的 IPC 通道，允许子进程在没有其他连接保持活动时正常退出

### process.dlopen(module, filename[, flags])
```txt
module <Object>

filename <string>

flags <os.constants.dlopen> 默认值：os.constants.dlopen.RTLD_LAZY
```
process.dlopen() 方法允许动态加载共享对象。require() 主要用于加载 C++ 插件，除非特殊情况，否则不应直接使用,换句话说，require() 应该优先于 process.dlopen()，除非有特定的原因，例如自定义 dlopen 标志或从 ES 模块加载

### process.emitWarning(warning[, options])
```txt
warning <string> | <Error> 要触发的警告。

options <Object>

  type <string> 当 warning 是 String 时，type 是用于触发的警告类型的名称。默认值：'Warning'。

  code <string> 触发的警告实例的唯一标识符。

  ctor <Function> 当 warning 为 String 时，ctor 是可选函数，用于限制生成的堆栈跟踪。默认值：process.emitWarning。

  detail <string> 要包含在错误中的额外文本
```
process.emitWarning() 方法可用于触发自定义或特定于应用的进程警告。这些可以通过向 'warning' 事件添加句柄来监听

```js
const { emitWarning } = require('node:process');

// Emit a warning with a code and additional detail.
emitWarning('Something happened!', {
  code: 'MY_WARNING',
  detail: 'This is some additional information',
});
// Emits:
// (node:56338) [MY_WARNING] Warning: Something happened!
// This is some additional information
```

### process.exit([code])
```txt
code <integer> | <string> | <null> | <undefined> 退出码。对于字符串类型，仅允许整数字符串（例如，'1'）。默认值：0。
```
process.exit() 方法指示 Node.js 以 code 的退出状态同步终止进程。如果省略 code，则退出使用 'success' 代码 0 或 process.exitCode 的值（如果已设置）。直到所有 'exit' 事件监听器都被调用，Node.js 才会终止。


### process.kill(pid[, signal])
```txt
pid <number> 进程标识

signal <string> | <number> 要发送的信号，可以是字符串或数字。默认值：'SIGTERM'。

```

常用信号

SIGTERM ：请求进程终止。默认信号。

SIGKILL ：强制终止进程，无法被捕获或忽略。

SIGINT ：中断进程（通常是 Ctrl+C）。

SIGHUP ：挂起进程，通常用于重启。

```js
// 创建一个子进程
const { spawn } = require('child_process');

// 启动一个长时间运行的进程
const child = spawn('node', ['-e', 'setInterval(() => console.log("running..."), 1000)']);

// 在 5 秒后发送 SIGTERM 信号来终止子进程
setTimeout(() => {
  console.log(`终止进程 ${child.pid}`);
  process.kill(child.pid); // 默认发送 SIGTERM
}, 5000);

```

### process.memoryUsage()
```txt
返回：<Object>

  rss ：常驻集大小（Resident Set Size），表示进程占用的物理内存量，包括代码、数据和堆栈。

  heapTotal ：V8 堆的总大小，表示分配给堆的内存总量。

  heapUsed ：V8 堆的已使用大小，表示当前已使用的堆内存量。

  external ：V8 管理的 C++ 对象占用的内存量。

  arrayBuffers <integer>
```
返回描述 Node.js 进程的内存使用量（以字节为单位）的对象
```js
const { memoryUsage } = require('node:process');

console.log(memoryUsage());
// Prints:
// {
//  rss: 4935680,
//  heapTotal: 1826816,
//  heapUsed: 650472,
//  external: 49879,
//  arrayBuffers: 9386
// }
```
### process.memoryUsage.rss()
process.memoryUsage.rss() 方法返回以字节为单位表示驻留集大小的整数 (RSS)。

### process.nextTick(callback[, ...args])
process.nextTick() 将 callback 添加到 "下一个滴答队列"。在 JavaScript 堆栈上的当前操作运行完成之后，且在允许事件循环继续之前，此队列将被完全排空。如果递归地调用 process.nextTick()，则可能会创建无限的循环。有关更多背景信息，请参阅 事件循环 指南。


### process.permission.has(scope[, reference])
process.permission.has(scope[, reference]) 是 Node.js 中的一个方法，用于检查当前进程是否具有特定的权限。这个方法是 Node.js 的权限管理功能的一部分，旨在提供更细粒度的权限控制

```txt
scope ：一个字符串，表示要检查的权限范围。这可以是一些操作或功能的名称，例如 read, write, execute 等。
reference （可选）：一个对象，表示与权限相关的上下文或参考信息。这个参数通常用于提供额外的信息，以帮助判断是否授予了特定的权限
```
```js
// 检查当前进程是否具有某个权限
const hasReadPermission = process.permission.has('read');

if (hasReadPermission) {
  console.log('当前进程具有读取权限');
} else {
  console.log('当前进程不具有读取权限');
}

```

#### 注意事项

权限管理 ：process.permission.has 方法是 Node.js 的权限管理的一部分，主要用于确保应用程序在执行特定操作时具有必要的权限。

上下文依赖 ：权限的检查可能依赖于上下文，因此在某些情况下，提供 reference 参数是有意义的。

应用场景 ：这种权限检查通常在需要安全控制的应用程序中使用，例如文件操作、网络请求等。
### process.report.getReport([err])
```txt
err <Error> 用于报告 JavaScript 堆栈的自定义错误
```
返回正在运行的进程的诊断报告的 JavaScript 对象表示形式。报告的 JavaScript 堆栈跟踪取自 err（如果存在）。
```js
const { report } = require('node:process');
const util = require('node:util');

const data = report.getReport();
console.log(data.header.nodejsVersion);

// Similar to process.report.writeReport()
const fs = require('node:fs');
fs.writeFileSync('my-report.log', util.inspect(data), 'utf8');
```

### process.report.writeReport([filename][, err])
将诊断报告写入文件。如果未提供 filename，则默认文件名包括日期、时间、PID 和序列号。报告的 JavaScript 堆栈跟踪取自 err（如果存在）。

如果 filename 的值设置为 'stdout' 或 'stderr'，则报告分别写入进程的 stdout 或 stderr。

### process.resourceUsage()
process.resourceUsage() 是 Node.js 中的一个方法，用于获取当前 Node.js 进程的资源使用情况。它返回一个对象，提供了有关进程资源使用的详细信息，包括内存、CPU 时间等。这对于性能监控和调试非常有用
```txt
返回：<Object> 当前进程的资源使用情况。所有这些值都来自返回 uv_rusage_t 结构 的 uv_getrusage 调用。

    userCPUTime <integer> 映射到以微秒计算的 ru_utime。它与 process.cpuUsage().user 的值相同。

    systemCPUTime <integer> 映射到以微秒计算的 ru_stime。它与 process.cpuUsage().system 的值相同。

    maxRSS <integer> 映射到 ru_maxrss，这是使用的最大驻留集大小（以千字节为单位）。

    sharedMemorySize <integer> 映射到 ru_ixrss 但不受任何平台支持。

    unsharedDataSize <integer> 映射到 ru_idrss 但不受任何平台支持。

    unsharedStackSize <integer> 映射到 ru_isrss 但不受任何平台支持。

    minorPageFault <integer> 映射到 ru_minflt，它是进程的次要页面错误数，请参阅 这篇文章了解更多详情。

    majorPageFault <integer> 映射到 ru_majflt，这是进程的主要页面错误数，请参阅 这篇文章了解更多详情。Windows 不支持此字段。

    swappedOut <integer> 映射到 ru_nswap 但不受任何平台支持。

    fsRead <integer> 映射到 ru_inblock，这是文件系统必须执行输入的次数。

    fsWrite <integer> 映射到 ru_oublock，这是文件系统必须执行输出的次数。

    ipcSent <integer> 映射到 ru_msgsnd 但不受任何平台支持。

    ipcReceived <integer> 映射到 ru_msgrcv 但不受任何平台支持。

    signalsCount <integer> 映射到 ru_nsignals 但不受任何平台支持。

    voluntaryContextSwitches <integer> 映射到 ru_nvcsw，这是由于进程在其时间片完成之前（通常是为了等待资源可用）自愿放弃处理器而导致 CPU 上下文切换的次数。Windows 不支持此字段。

    involuntaryContextSwitches <integer> 映射到 ru_nivcsw，这是由于较高优先级进程变得可运行或因为当前进程超出其时间片而导致 CPU 上下文切换的次数。Windows 不支持此字段。
```
```js
const { resourceUsage } = require('node:process');

console.log(resourceUsage());
/*
  Will output:
  {
    userCPUTime: 82872,
    systemCPUTime: 4143,
    maxRSS: 33164,
    sharedMemorySize: 0,
    unsharedDataSize: 0,
    unsharedStackSize: 0,
    minorPageFault: 2469,
    majorPageFault: 0,
    swappedOut: 0,
    fsRead: 0,
    fsWrite: 8,
    ipcSent: 0,
    ipcReceived: 0,
    signalsCount: 0,
    voluntaryContextSwitches: 79,
    involuntaryContextSwitches: 1
  }
*/
```

### process.send(message[, sendHandle[, options]][, callback])
```txt
message ：要发送的消息，可以是字符串、对象（通常是 JSON 可序列化的）等。
sendHandle （可选）：一个可选的句柄，通常是一个 TCP 或 IPC（进程间通信）套接字。这个句柄可以被发送到另一个进程，以便在那个进程中使用。
options （可选）：一个对象，用于指定发送消息的选项，例如：
  retries: 发送失败时的重试次数。
  timeout: 发送超时时间。
callback （可选）：一个可选的回调函数，当消息成功发送后调用。
```
process.send(message[, sendHandle[, options]][, callback]) 是 Node.js 中的一个方法，用于在父进程和子进程之间发送消息。这个方法主要在使用 child_process.fork() 创建的子进程中使用，允许进程间进行通信

### process.uptime()
process.version 属性包含 Node.js 版本字符串。
