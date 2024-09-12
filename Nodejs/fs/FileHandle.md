# Class:FileHandle
FileHandle 是 Node.js 中 fsPromises API 提供的一种对象，用于表示文件句柄。通过 FileHandle 对象，你可以对文件执行一系列操作，如读取、写入、关闭等。

## 文件句柄
文件句柄（File Handle）是操作系统用来标识和管理打开文件的一个抽象标识符。它可以被看作是文件的一个引用或指针，允许应用程序执行各种文件操作，如读取、写入、关闭等。文件句柄由操作系统分配，通常在打开文件时创建，并在关闭文件时释放。

在 Node.js 中，文件句柄是通过 `fsPromises.open()` 方法获取的，返回一个 `FileHandle` 对象。这个对象提供了一系列方法，用于对文件进行操作。

### 文件句柄的作用

1. **标识文件**：文件句柄是一个唯一标识符，用于标识当前进程中打开的文件。
2. **管理文件操作**：通过文件句柄，可以进行文件的读取、写入、追加、截断等操作。
3. **资源管理**：文件句柄需要在不再使用时关闭，以释放系统资源。

#### 示例

下面是一个示例，展示如何使用文件句柄在 Node.js 中进行文件操作：

```javascript
const fs = require('fs').promises;

async function fileOperations() {
  let fileHandle;
  try {
    // 打开文件并获取文件句柄
    fileHandle = await fs.open('example.txt', 'r+');
    
    // 读取文件内容
    const buffer = Buffer.alloc(1024);
    const { bytesRead, buffer: readBuffer } = await fileHandle.read(buffer, 0, 1024, 0);
    console.log('文件内容:', readBuffer.toString('utf8', 0, bytesRead));
    
    // 写入文件内容
    await fileHandle.write('新的内容');
    console.log('文件已写入');
    
    // 追加文件内容
    await fileHandle.appendFile('\n追加的内容');
    console.log('内容已追加');
    
    // 截断文件
    await fileHandle.truncate(20);
    console.log('文件已截断');
    
  } catch (err) {
    console.error('文件操作出错:', err);
  } finally {
    // 确保文件句柄在操作完成后关闭
    if (fileHandle) {
      await fileHandle.close();
      console.log('文件句柄已关闭');
    }
  }
}

fileOperations();
```
##### 为什么要关闭文件句柄

关闭文件句柄是非常重要的操作，因为每个打开的文件句柄都会占用系统资源。如果不及时关闭文件句柄，可能会导致资源泄漏，进而影响系统性能，甚至导致文件操作失败，因为系统可以打开的文件句柄数量是有限的。



## 事件 close
当 FileHandle 已关闭且不再可用时，则触发 'close' 事件

### filehandle.appendFile(data[, options])
```txt
data: 要追加到文件的数据，可以是字符串、Buffer、TypedArray 或 DataView。
options: 可选参数，是一个对象，用于指定编码等选项。常用选项包括：
  encoding: 指定字符编码，默认为 'utf8'。
  mode: 文件的权限（如 0o666），但在大多数情况下会被文件系统的默认权限覆盖。
  flag: 文件系统标志，默认为 'a'（追加）。
```
```js
const fs = require('fs').promises;

async function appendToFile() {
  let fileHandle;
  try {
    // 打开文件，获取文件句柄
    fileHandle = await fs.open('example.txt', 'a');
    
    // 追加数据到文件末尾
    await fileHandle.appendFile('这是追加的内容\n', { encoding: 'utf8' });
    console.log('内容已追加到文件');
    
  } catch (err) {
    console.error('追加文件时出错:', err);
  } finally {
    // 关闭文件句柄
    if (fileHandle) {
      await fileHandle.close();
      console.log('文件句柄已关闭');
    }
  }
}

appendToFile();

```
### filehandle.writeFile()

### filehandle.chmod(mode)
mode: 一个表示文件权限的整数。文件权限通常以八进制表示，例如 0o777 表示文件所有者、组和其他用户都有读、写和执行权限。

文件权限模式通常以八进制表示，由三个八进制数字组成，每个数字代表文件所有者、组成员和其他用户的权限。每个八进制数字可以是以下值的组合：

读权限 (r) : 4

写权限 (w) : 2

执行权限 (x) : 1

例如：

0o777 表示文件所有者、组成员和其他用户都有读、写和执行权限。
0o644 表示文件所有者有读写权限，组成员和其他用户只有读权限。
0o600 表示只有文件所有者有读写权限


### filehandle.chown(uid, gid)
```txt
uid <integer> 文件的新所有者的用户 ID。

gid <integer> 文件的新群组的组 ID。

返回：<Promise> 成功时将使用 undefined 履行。

```
为什么要更改文件所有者和组

更改文件的所有者和组可以控制谁有权访问和操作文件。这在多用户系统中尤其重要，可以确保文件的安全性和隐私性。通过适当设置文件的所有者和组，可以防止未授权的用户对文件进行操作

### filehandle.close()
等待句柄上的任何未决操作完成后，关闭文件句柄
```js
import { open } from 'node:fs/promises';

let filehandle;
try {
  filehandle = await open('thefile.txt', 'r');
} finally {
  await filehandle?.close();
}
```

### filehandle.createReadStream([options])
用于创建一个可读流（Readable Stream），可以从文件中读取数据。这个方法非常适合处理大文件或需要逐步读取文件内容的场景
```txt
options: 一个可选的对象，用于指定流的配置选项。常用选项包括：
  start: 流的起始位置（字节数）。
  end: 流的结束位置（字节数）。
  highWaterMark: 流的内部缓冲区大小（以字节为单位），默认是 64 KB。
  encoding: 指定字符编码，例如 'utf8'。

  返回：<fs.ReadStream>
```
```js
const fs = require('fs').promises;

async function readFileStream() {
  let fileHandle;
  try {
    // 打开文件并获取文件句柄
    fileHandle = await fs.open('example.txt', 'r');
    
    // 创建可读流
    const readStream = fileHandle.createReadStream({ encoding: 'utf8' });
    
    // 监听 data 事件，逐步读取文件内容
    readStream.on('data', (chunk) => {
      console.log('读取到的数据块:', chunk);
    });
    
    // 监听 end 事件，文件读取完成
    readStream.on('end', () => {
      console.log('文件读取完成');
    });
    
    // 监听 error 事件，处理错误
    readStream.on('error', (err) => {
      console.error('读取文件时出错:', err);
    });
    
  } catch (err) {
    console.error('打开文件时出错:', err);
  } finally {
    // 确保文件句柄在操作完成后关闭
    if (fileHandle) {
      await fileHandle.close();
      console.log('文件句柄已关闭');
    }
  }
}

readFileStream();
```

### filehandle.createWriteStream([options])
```txt
options <Object>

  autoClose <boolean> 默认值：true

  emitClose <boolean> 默认值：true

  start: 流的起始位置（字节数）。

  highWaterMark: 流的内部缓冲区大小（以字节为单位），默认是 16 KB。

  encoding: 指定字符编码，例如 'utf8'。

  flags: 文件系统标志，默认为 'w'（写入模式）。

  flush <boolean> 如果是 true，则在关闭基础文件描述符之前将其刷新。默认值：false。

返回：<fs.WriteStream>
```
```js
const fs = require('fs').promises;

async function writeFileStream() {
  let fileHandle;
  try {
    // 打开文件并获取文件句柄
    fileHandle = await fs.open('example.txt', 'w');
    
    // 创建可写流
    const writeStream = fileHandle.createWriteStream({ encoding: 'utf8' });
    
    // 写入数据
    writeStream.write('这是第一行数据\n');
    writeStream.write('这是第二行数据\n');
    
    // 结束写入
    writeStream.end('这是最后一行数据\n');
    
    // 监听 finish 事件
    writeStream.on('finish', () => {
      console.log('文件写入完成');
    });

    // 监听 error 事件，处理错误
    writeStream.on('error', (err) => {
      console.error('写入文件时出错:', err);
    });
    
  } catch (err) {
    console.error('打开文件时出错:', err);
  } finally {
    // 确保文件句柄在操作完成后关闭
    if (fileHandle) {
      await fileHandle.close();
      console.log('文件句柄已关闭');
    }
  }
}

writeFileStream();

```

### fileHandle.datasync()
用于将文件的所有已修改数据（不包括元数据）写入存储设备。这个方法确保文件内容在系统崩溃或断电等情况下不会丢失。该方法返回一个 Promise，当同步操作完成时，Promise 会被解决

与 fileHandle.sync() 的区别

fileHandle.datasync() : 只同步文件数据，不包括文件的元数据（如修改时间、权限等）。

fileHandle.sync() : 同步文件数据和文件的元数据。

```js
// 同步文件数据到存储设备
......
await fileHandle.datasync();
```

### filehandle.read(buffer, offset, length, position)
用于从文件中读取数据到指定的缓冲区。该方法返回一个 Promise，当读取操作完成时，Promise 会被解决，并返回一个对象，包含读取的字节数和缓冲区
```txt
buffer: 一个 Buffer 对象，表示读取的数据将被写入的缓冲区。
offset: 一个整数，表示开始写入到 buffer 的位置（以字节为单位）。
length: 一个整数，表示要读取的字节数。
position: 一个整数，表示文件中开始读取的位置。如果为 null，则从当前文件位置读取，并更新文件位置。

返回一个包含 bytesRead 和 buffer 属性的 Promise 对象：
bytesRead: 表示实际读取的字节数。
buffer: 表示传递给方法的缓冲区对象。

```
```js
const fs = require('fs').promises;

async function readFile() {
  let fileHandle;
  try {
    // 打开文件并获取文件句柄
    fileHandle = await fs.open('example.txt', 'r');
    
    // 创建一个缓冲区
    const buffer = Buffer.alloc(1024);
    
    // 从文件中读取数据到缓冲区
    const { bytesRead, buffer: readBuffer } = await fileHandle.read(buffer, 0, 1024, 0);
    
    // 打印读取的数据
    console.log('读取到的数据:', readBuffer.toString('utf8', 0, bytesRead));
    
  } catch (err) {
    console.error('读取文件时出错:', err);
  } finally {
    // 确保文件句柄在操作完成后关闭
    if (fileHandle) {
      await fileHandle.close();
      console.log('文件句柄已关闭');
    }
  }
}

readFile();

```
### filehandle.read([options])
```txt
options <Object>

  buffer <Buffer> | <TypedArray> | <DataView> 将填充读取的文件数据的缓冲区。默认值：Buffer.alloc(16384)

  offset <integer> 缓冲区中开始填充的位置。默认值：0

  length <integer> 读取的字节数。默认值：buffer.byteLength - offset

  position <integer> | <bigint> | <null> 从文件开始读取数据的位置。如果是 null 或 -1，将从当前文件位置读取数据，并更新该位置。如果 position 是非负整数，则当前文件位置将保持不变。Default::null
```

### filehandle.read(buffer[, options]) 见上

### fileHandle.readFile(options)
用于读取整个文件的内容
```txt
options: 一个可选的对象，用于指定读取操作的配置选项。常用选项包括：
encoding: 指定字符编码，例如 'utf8'。如果不指定，则返回 Buffer 对象。
flag: 文件系统标志，默认为 'r'（读取模式）。
```
```js
const fs = require('fs').promises;

async function readFile() {
  let fileHandle;
  try {
    // 打开文件并获取文件句柄
    fileHandle = await fs.open('example.txt', 'r');
    
    // 读取整个文件内容
    const data = await fileHandle.readFile({ encoding: 'utf8' });
    
    // 打印文件内容
    console.log('文件内容:', data);
    
  } catch (err) {
    console.error('读取文件时出错:', err);
  } finally {
    // 确保文件句柄在操作完成后关闭
    if (fileHandle) {
      await fileHandle.close();
      console.log('文件句柄已关闭');
    }
  }
}

readFile();
```

### filehandle.readLines([options])
按行读取
```txt
options <Object>

  encoding <string> 默认值：null

  autoClose <boolean> 默认值：true

  emitClose <boolean> 默认值：true

  start <integer>

  end <integer> 默认值：Infinity

  highWaterMark <integer> 默认值：64 * 1024

返回：<readline.InterfaceConstructor>
```
```js
const { open } = require('node:fs/promises');

(async () => {
  const file = await open('./some/file/to/read');

  for await (const line of file.readLines()) {
    console.log(line);
  }
})();
```

### filehandle.readv(buffers[, position])
用于从文件中读取数据到多个缓冲区（buffers）。该方法利用了分散/聚集（scatter/gather）I/O 的概念，可以一次性读取数据到多个缓冲区中。这个方法返回一个 Promise，当读取操作完成时，Promise 会被解决，并返回一个对象，包含读取的字节数和缓冲区。
```txt
buffers: 一个 Buffer 对象的数组，每个 Buffer 用于存储读取的数据。
position: 一个可选的整数，表示文件中开始读取的位置。如果为 null，则从当前文件位置读取，并更新文件位置。
```
```js
const fs = require('fs').promises;

async function readFile() {
  let fileHandle;
  try {
    // 打开文件并获取文件句柄
    fileHandle = await fs.open('example.txt', 'r');
    
    // 创建多个缓冲区
    const buffers = [Buffer.alloc(10), Buffer.alloc(10)];
    
    // 从文件中读取数据到多个缓冲区
    const { bytesRead, buffers: readBuffers } = await fileHandle.readv(buffers, 0);
    
    // 打印读取的数据
    for (const buffer of readBuffers) {
      console.log('读取到的数据:', buffer.toString('utf8'));
    }
    console.log('总共读取的字节数:', bytesRead);
    
  } catch (err) {
    console.error('读取文件时出错:', err);
  } finally {
    // 确保文件句柄在操作完成后关闭
    if (fileHandle) {
      await fileHandle.close();
      console.log('文件句柄已关闭');
    }
  }
}

readFile();

```

### fileHandle.stat
用于获取文件的状态信息（如大小、权限、创建时间等）。该方法返回一个 Promise，当获取操作完成时，Promise 会被解决，并返回一个包含文件状态信息的 fs.Stats 对象。
```txt
options: 一个可选的对象，用于指定配置选项。常用选项包括：
  bigint: 如果设置为 true，将文件大小和时间戳作为 BigInt 类型返回，而不是默认的 number 类型。
```
```js
const fs = require('fs').promises;

async function getFileStats() {
  let fileHandle;
  try {
    // 打开文件并获取文件句柄
    fileHandle = await fs.open('example.txt', 'r');
    
    // 获取文件状态信息
    const stats = await fileHandle.stat();
    
    // 打印文件状态信息
    console.log('文件大小:', stats.size);
    console.log('文件创建时间:', stats.birthtime);
    console.log('文件最后修改时间:', stats.mtime);
    
  } catch (err) {
    console.error('获取文件状态时出错:', err);
  } finally {
    // 确保文件句柄在操作完成后关闭
    if (fileHandle) {
      await fileHandle.close();
      console.log('文件句柄已关闭');
    }
  }
}

getFileStats();

```



### filehandle.sync() 见上fileHandle.datasync()

### filehandle.truncate(len)
用于截断文件。

len: 一个整数，表示文件的新长度（以字节为单位）。

```js
// 下面的示例仅保留文件的前四个字节：
import { open } from 'node:fs/promises';

let filehandle = null;
try {
  filehandle = await open('temp.txt', 'r+');
  await filehandle.truncate(4);
} finally {
  await filehandle?.close();
}
```

### filehandle.utimes(atime, mtime)
用于更新文件的访问时间（atime）和修改时间（mtime）。该方法返回一个 Promise，当更新操作完成时，Promise 会被解决。
```txt
atime: 文件的访问时间，可以是一个 Date 对象、时间戳（秒）或 number 类型的毫秒数。
mtime: 文件的修改时间，可以是一个 Date 对象、时间戳（秒）或 number 类型的毫秒数。
```
```js
const fs = require('fs').promises;

async function updateFileTimes() {
  let fileHandle;
  try {
    // 打开文件并获取文件句柄
    fileHandle = await fs.open('example.txt', 'r+');
    
    // 获取当前时间
    const currentTime = new Date();
    
    // 更新文件的访问时间和修改时间
    await fileHandle.utimes(currentTime, currentTime);
    console.log('文件时间已更新');
    
  } catch (err) {
    console.error('更新文件时间时出错:', err);
  } finally {
    // 确保文件句柄在操作完成后关闭
    if (fileHandle) {
      await fileHandle.close();
      console.log('文件句柄已关闭');
    }
  }
}

updateFileTimes();

```

### filehandle.write(buffer, offset[, length[, position]])
### filehandle.write(buffer[, options])
### filehandle.write(string[, position[, encoding]])
### filehandle.writeFile(data, options)
异步地将数据写入文件，如果文件已经存在，则替换该文件

### filehandle.writev(buffers[, position])
用于将多个缓冲区（buffers）中的数据写入文件。这个方法利用了分散/聚集（scatter/gather）I/O 的概念，可以一次性将多个缓冲区的数据写入文件。
```js
const fs = require('fs').promises;

async function writeFile() {
  let fileHandle;
  try {
    // 打开文件并获取文件句柄
    fileHandle = await fs.open('example.txt', 'w');
    
    // 创建多个缓冲区
    const buffers = [Buffer.from('第一部分数据\n'), Buffer.from('第二部分数据\n')];
    
    // 将多个缓冲区的数据写入文件
    const { bytesWritten, buffers: writtenBuffers } = await fileHandle.writev(buffers, 0);
    
    // 打印写入的信息
    console.log('总共写入的字节数:', bytesWritten);
    for (const buffer of writtenBuffers) {
      console.log('写入的数据:', buffer.toString('utf8'));
    }
    
  } catch (err) {
    console.error('写入文件时出错:', err);
  } finally {
    // 确保文件句柄在操作完成后关闭
    if (fileHandle) {
      await fileHandle.close();
      console.log('文件句柄已关闭');
    }
  }
}

writeFile();

```
