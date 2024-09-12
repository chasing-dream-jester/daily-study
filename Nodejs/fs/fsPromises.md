# fsPromises
fsPromises是 Node.js 文件系统（File System）模块的一个部分，提供了一组基于 Promise 的 API，用于进行文件和目录的操作。相比传统的回调（callback）方式，使用 Promise 可以让异步代码更加简洁和易读

## fsPromises.access(path[, mode])
用于检查文件或目录的访问权限
```txt
path: 一个字符串或 Buffer 对象，表示要检查的文件或目录的路径。
mode: 一个可选的整数，表示要检查的访问权限。默认值是 fs.constants.F_OK，表示检查文件是否存在。其他可选值包括：
fs.constants.R_OK: 检查是否可读。
fs.constants.W_OK: 检查是否可写。
fs.constants.X_OK: 检查是否可执行（在 Windows 上无效）。
```
```js
const fs = require('fs').promises;
const constants = require('fs').constants;

async function checkFileAccess() {
  try {
    // 检查文件是否存在以及是否可读和可写
    await fs.access('example.txt', constants.F_OK | constants.R_OK | constants.W_OK);
    console.log('文件存在且可读可写');
  } catch (err) {
    console.error('文件不存在或不可读写:', err);
  }
}

checkFileAccess();

```

## fsPromises.appendFile(path, data[, options])
用于将数据追加到文件末尾。如果文件不存在，则会创建该文件。

```txt
path: 一个字符串、Buffer 对象、URL 对象或文件描述符，表示要追加数据的文件路径。
data: 要追加到文件的数据，可以是字符串、Buffer、TypedArray 或 DataView。
options: 一个可选的对象，用于指定配置选项。常用选项包括：
  encoding: 指定字符编码，默认为 'utf8'。
  mode: 文件的权限（如 0o666），但在大多数情况下会被文件系统的默认权限覆盖。
  flag: 文件系统标志，默认为 'a'（追加）。
```

## fsPromises.chmod(path, mode)
更改文件的权限
```js
const fs = require('fs').promises;

async function example() {
  try {
    await fs.chmod('example.txt', 0o600);
    console.log('文件权限已更改');
  } catch (err) {
    console.error('更改文件权限时出错:', err);
  }
}

example();

```

## fsPromises.chown(path, uid, gid)
更改文件的所有权
```txt
path <string> | <Buffer> | <URL>

uid <integer>

gid <integer>

返回：<Promise> 成功时将使用 undefined 履行。
```

## fsPromises.copyFile(src, dest[, mode])
用于将文件从一个位置复制到另一个位置。该方法返回一个 Promise，当复制操作完成时，Promise 会被解决。
```txt
src: 一个字符串、Buffer 对象或 URL 对象，表示源文件的路径。
dest: 一个字符串、Buffer 对象或 URL 对象，表示目标文件的路径。
mode: 一个可选的整数，指定复制操作的行为，可以是以下常量之一：
  fs.constants.COPYFILE_EXCL: 如果目标文件已存在，则操作失败。
  fs.constants.COPYFILE_FICLONE: 如果可能，创建源文件的副本，而不是复制文件数据。
  fs.constants.COPYFILE_FICLONE_FORCE: 如果可能，创建源文件的副本；如果失败，则复制文件数据。
```
```js
const fs = require('fs').promises;

async function copyFileExample() {
  try {
    // 将文件从 'source.txt' 复制到 'destination.txt'
    await fs.copyFile('source.txt', 'destination.txt');
    console.log('文件已复制');
  } catch (err) {
    console.error('复制文件时出错:', err);
  }
}

copyFileExample();

```

## fsPromises.cp(src, dest[, options])
将整个目录结构从 src 异步地复制到 dest，包括子目录和文件。

当将目录复制到另一个目录时，不支持 globs，并且行为类似于 cp dir1/ dir2/。

```txt
src <string> | <URL> 要复制的源路径。

dest <string> | <URL> 要复制到的目标路径。

options <Object>

  dereference <boolean> 取消引用符号链接。默认值：false。

  errorOnExist <boolean> 当 force 是 false 且目标存在时，抛出错误。默认值：false。

  filter <Function> 过滤复制文件/目录的函数。返回 true 则复制条目，返回 false 则忽略它。忽略目录时，其所有内容也将被跳过。还可以返回解析为 true 或 false 默认值的 Promise：undefined。

      src <string> 要复制的源路径。

      dest <string> 要复制到的目标路径。

      返回：<boolean> | <Promise> 可强制转换为 boolean 的值或满足该值的 Promise。

  force <boolean> 覆盖现有文件或目录。如果将此设置为 false 并且目标存在，则复制操作将忽略错误。使用 errorOnExist 选项更改此行为。默认值：true。

  mode <integer> 复制操作的修饰符。默认值：0。参见 fsPromises.copyFile() 的 mode 标志。

  preserveTimestamps <boolean> 当为 true 时，则 src 的时间戳将被保留。默认值：false。

  recursive <boolean> 递归复制目录默认：false

  verbatimSymlinks <boolean> 当为 true 时，则符号链接的路径解析将被跳过。默认值：false
```

## fsPromises.lchmod(path, mode)
更改符号链接的权限

## fsPromises.lchown(path, uid, gid)
更改符号链接上的所有权

## fsPromises.lutimes(path, atime, mtime)
以与 fsPromises.utimes() 相同的方式更改文件的访问和修改时间，不同之处在于如果路径引用符号链接，则该链接不会取消引用：相反，符号链接本身的时间戳被改变了

## fsPromises.link(existingPath, newPath)
用于创建一个硬链接（hard link）。硬链接是指多个文件名指向同一个文件数据块。
```txt
existingPath: 一个字符串、Buffer 对象或 URL 对象，表示现有文件的路径。
newPath: 一个字符串、Buffer 对象或 URL 对象，表示要创建的硬链接的路径。
```
```js
const fs = require('fs').promises;

async function createHardLink() {
  try {
    // 创建硬链接，将 'existing.txt' 链接到 'link.txt'
    await fs.link('existing.txt', 'link.txt');
    console.log('硬链接已创建');
  } catch (err) {
    console.error('创建硬链接时出错:', err);
  }
}

createHardLink();

```
使用场景

- 备份和恢复 ：硬链接可以用于创建文件的备份，而不占用额外的磁盘空间。
- 文件共享 ：多个文件名指向同一个文件数据块，可以在不同位置共享同一个文件内容。
- 版本控制 ：在某些版本控制系统中，硬链接可以用于管理文件的不同版本。
  
硬链接与符号链接的区别
- 硬链接（Hard Link） ：多个文件名指向同一个文件数据块。删除一个硬链接不会影响其他硬链接。
- 符号链接（Symbolic Link） ：一个文件名指向另一个文件名。删除符号链接不会影响目标文件，但删除目标文件会导致符号链接失效。

## fsPromises.lstat(path[, options])
用于获取文件或目录的状态信息，与 fsPromises.stat 类似，但不同之处在于，如果路径是一个符号链接（symbolic link），lstat 会返回符号链接本身的信息，而不是它指向的目标文件的信息。该方法返回一个 Promise，当获取操作完成时，Promise 会被解决，并返回一个包含文件状态信息的 fs.Stats 对象。
```js
const fs = require('fs').promises;

async function getFileStats() {
  try {
    // 获取文件状态信息
    const stats = await fs.lstat('example.txt');
    
    // 打印文件状态信息
    console.log('文件大小:', stats.size);
    console.log('文件创建时间:', stats.birthtime);
    console.log('文件最后修改时间:', stats.mtime);
    console.log('是否是符号链接:', stats.isSymbolicLink());
    
  } catch (err) {
    console.error('获取文件状态时出错:', err);
  }
}

getFileStats();

```
## fsPromises.mkdir(path[, options])
用于创建目录。该方法返回一个 Promise，当目录创建操作完成时，Promise 会被解决。
```txt
path: 一个字符串或 Buffer 对象，表示要创建的目录的路径。
options: 一个可选的对象，用于指定配置选项。常用选项包括：
  recursive: 一个布尔值，表示是否递归地创建目录。如果设置为 true，则会递归创建所有不存在的上级目录。默认值为 false。
  mode: 一个整数，表示新目录的权限（如 0o777）。在大多数情况下会被文件系统的默认权限覆盖。
```
```js
// 创建一个单一目录
const fs = require('fs').promises;

async function createDirectory() {
  try {
    await fs.mkdir('new_directory');
    console.log('目录已创建');
  } catch (err) {
    console.error('创建目录时出错:', err);
  }
}

createDirectory();

```
```js
// 递归创建目录
const fs = require('fs').promises;

async function createNestedDirectories() {
  try {
    await fs.mkdir('parent_directory/child_directory', { recursive: true });
    console.log('递归目录已创建');
  } catch (err) {
    console.error('创建目录时出错:', err);
  }
}

createNestedDirectories();

```
```js
// 设置目录权限
const fs = require('fs').promises;

async function createDirectoryWithMode() {
  try {
    await fs.mkdir('secure_directory', { mode: 0o755 });
    console.log('目录已创建，权限已设置');
  } catch (err) {
    console.error('创建目录时出错:', err);
  }
}

createDirectoryWithMode();

```
## fsPromises.open(path, flags[, mode])

用于打开文件并返回一个 FileHandle 对象
```txt
path: 一个字符串、Buffer 对象或 URL 对象，表示要打开的文件路径。
flags: 一个字符串，表示打开文件的模式。常用的模式包括：
  'r': 以读取模式打开文件。如果文件不存在，则抛出异常。
  'r+': 以读写模式打开文件。如果文件不存在，则抛出异常。
  'rs+': 以同步读写模式打开文件。
  'w': 以写入模式打开文件。如果文件不存在则创建文件，如果文件存在则截断文件。
  'wx': 类似于 'w'，但如果文件存在则失败。
  'w+': 以读写模式打开文件。如果文件不存在则创建文件，如果文件存在则截断文件。
  'wx+': 类似于 'w+'，但如果文件存在则失败。
  'a': 以追加模式打开文件。如果文件不存在则创建文件。
  'ax': 类似于 'a'，但如果文件存在则失败。
  'a+': 以读写追加模式打开文件。如果文件不存在则创建文件。
  'ax+': 类似于 'a+'，但如果文件存在则失败。
mode: 一个可选的整数，表示文件的权限（如 0o666），在大多数情况下会被文件系统的默认权限覆盖。
```
```js
const fs = require('fs').promises;

async function readFile() {
  let fileHandle;
  try {
    // 以读取模式打开文件
    fileHandle = await fs.open('example.txt', 'r');
    
    // 读取文件内容
    const buffer = Buffer.alloc(1024);
    const { bytesRead, buffer: readBuffer } = await fileHandle.read(buffer, 0, 1024, 0);
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
## fsPromises.opendir(path[, options])
用于打开一个目录并返回一个 fs.Dir 对象。该方法返回一个 Promise，当打开操作完成时，Promise 会被解决，并返回一个 fs.Dir 对象，表示打开的目录。通过 fs.Dir 对象，你可以逐个读取目录中的条目
```txt
path: 一个字符串或 Buffer 对象，表示要打开的目录路径。
options: 一个可选的对象，用于指定配置选项。常用选项包括：
encoding: 指定返回的文件名的编码类型。默认为 'utf8'。

返回一个包含 fs.Dir 对象的 Promise。fs.Dir 对象表示打开的目录，包含用于逐个读取目录条目的方法。
```
```js
const fs = require('fs').promises;

async function readDirectoryEntries() {
  let dir;
  try {
    // 打开目录
    dir = await fs.opendir('.');
    
    // 逐个读取目录条目
    for await (const dirent of dir) {
      console.log(dirent.name, dirent.isDirectory() ? '目录' : '文件');
    }
  } catch (err) {
    console.error('读取目录时出错:', err);
  } finally {
    // 确保目录在操作完成后关闭
    if (dir) {
      await dir.close();
    }
  }
}

readDirectoryEntries();

```
## fsPromises.readdir(path[, options])
用于读取目录的内容
```txt
path: 一个字符串、Buffer 对象或 URL 对象，表示要读取的目录路径。
options: 一个可选的对象或字符串，用于指定配置选项。常用选项包括：
  encoding: 指定返回的文件名的编码类型。默认为 'utf8'。
  withFileTypes: 一个布尔值，如果设置为 true，则返回的数组包含 fs.Dirent 对象，否则返回文件名字符串。默认为 false。

返回一个包含目录内容的 Promise 对象。解决时返回一个包含目录中所有文件和子目录名称的数组。如果 options.withFileTypes 设置为 true，则返回的数组包含 fs.Dirent 对象
```

```js
const fs = require('fs').promises;

async function readDirectory() {
  try {
    const files = await fs.readdir('.');
    console.log('目录内容:', files);
  } catch (err) {
    console.error('读取目录时出错:', err);
  }
}

readDirectory();

```

## fsPromises.readFile(path[, options])
用于读取文件的内容
```txt
path: 一个字符串、Buffer 对象或 URL 对象，表示要读取的文件路径。
options: 一个可选的对象或字符串，用于指定配置选项。常用选项包括：
  encoding: 指定文件内容的编码类型。如果不指定，默认返回 Buffer 对象。
  flag: 指定文件系统标志，默认值是 'r'（读取模式）
```
```js
const fs = require('fs').promises;

async function readFileExample() {
  try {
    const data = await fs.readFile('example.txt', 'utf8');
    console.log('文件内容:', data);
  } catch (err) {
    console.error('读取文件时出错:', err);
  }
}

readFileExample();

```

## fsPromises.readlink(path[, options])
用于读取符号链接（symbolic link）指向的路径
```txt
path: 一个字符串、Buffer 对象或 URL 对象，表示要读取的符号链接的路径。
options: 一个可选的对象或字符串，用于指定配置选项。常用选项包括：
  encoding: 指定返回路径的编码类型。默认为 'utf8'。
```
```js
const fs = require('fs').promises;

async function readSymbolicLink() {
  try {
    const targetPath = await fs.readlink('symlink.txt');
    console.log('符号链接指向的路径:', targetPath);
  } catch (err) {
    console.error('读取符号链接时出错:', err);
  }
}

readSymbolicLink();

```

## fsPromises.rename(oldPath, newPath)
将 oldPath 重命名为 newPath

## fsPromises.rmdir(path[, options])
用于删除目录
```txt
path <string> | <Buffer> | <URL>

options <Object>

  maxRetries <integer> 如果遇到 EBUSY、EMFILE、ENFILE、ENOTEMPTY 或 EPERM 错误，Node.js 将在每次尝试时以 retryDelay 毫秒的线性退避等待时间重试该操作。此选项表示重试次数。如果 recursive 选项不为 true，则忽略此选项。默认值：0。

  recursive <boolean> 如果为 true，则执行递归目录删除。在递归模式下，操作将在失败时重试。默认值：false。已弃用。

  retryDelay <integer> 重试之间等待的时间（以毫秒为单位）。如果 recursive 选项不为 true，则忽略此选项。默认值：100。

返回：<Promise> 成功时将使用 undefined 履行
```


## fsPromises.rm(path[, options])
删除文件和目录
```txt
path <string> | <Buffer> | <URL>

options <Object>

force <boolean> 当为 true 时，如果 path 不存在，则异常将被忽略。默认值：false。

maxRetries <integer> 如果遇到 EBUSY、EMFILE、ENFILE、ENOTEMPTY 或 EPERM 错误，Node.js 将在每次尝试时以 retryDelay 毫秒的线性退避等待时间重试该操作。此选项表示重试次数。如果 recursive 选项不为 true，则忽略此选项。默认值：0。

recursive <boolean> 如果为 true，则执行递归目录删除。在递归模式下，操作将在失败时重试。默认值：false。

retryDelay <integer> 重试之间等待的时间（以毫秒为单位）。如果 recursive 选项不为 true，则忽略此选项。默认值：100。

```

## fsPromises.stat(path[, options])
用于获取文件或目录的状态信息
```js
const fs = require('fs').promises;

async function getFileStats() {
  try {
    const stats = await fs.stat('example.txt');
    console.log('文件大小:', stats.size);
    console.log('文件创建时间:', stats.birthtime);
    console.log('文件最后修改时间:', stats.mtime);
  } catch (err) {
    console.error('获取文件状态时出错:', err);
  }
}

getFileStats();

```
fs.Stats 对象
```txt
size: 文件的大小（以字节为单位）。
birthtime: 文件的创建时间。
mtime: 文件的最后修改时间。
ctime: 文件的状态更改时间。
atime: 文件的最后访问时间。
mode: 文件的权限和类型。
isFile() : 判断是否为普通文件。
isDirectory() : 判断是否为目录。
isSymbolicLink() : 判断是否为符号链接。
isBlockDevice() : 判断是否为块设备。
isCharacterDevice() : 判断是否为字符设备。
isFIFO() : 判断是否为 FIFO。
isSocket() : 判断是否为 socket
```

## fsPromises.constants
返回一个包含文件系统操作常用常量的对象。对象与 fs.constants 相同。有关详细信息，请参阅 文件系统常量。


## fsPromises.statfs(path[, options])


## fsPromises.symlink(target, path[, type])
创建符号链接

## fsPromises.truncate(path[, len])
将 path 上的内容截断（缩短或延长长度）到 len 个字节

## fsPromises.unlink(path)

如果 path 指向符号链接，则删除该链接，但不影响链接所指向的文件或目录。如果 path 指向的文件路径不是符号链接，则删除文件。有关更多详细信息，请参阅 POSIX unlink(2) 文档

## fsPromises.utimes(path, atime, mtime)
更改 path 引用的对象的文件系统时间戳
```txt
path <string> | <Buffer> | <URL>

atime <number> | <string> | <Date>

mtime <number> | <string> | <Date>

```

time 和 mtime 参数遵循以下规则：

值可以是代表 Unix 纪元时间的数字，Date，或像 '123456789.0' 这样的数字字符串。

如果该值不能转换为数字，或者是 NaN、Infinity 或 -Infinity，则会抛出 Error。


## fsPromises.watch(filename[, options])
```txt
filename <string> | <Buffer> | <URL>

options <string> | <Object>

  persistent <boolean> 指示只要正在监视文件，进程是否应继续运行。默认值：true。

  recursive <boolean> 指示是应监视所有子目录，还是仅监视当前目录。这在指定目录时适用，并且仅适用于受支持的平台（参见 caveats）。默认值：false。

  encoding <string> 指定用于传给监听器的文件名的字符编码。默认值：'utf8'。

  signal <AbortSignal> 用于指示监视器何时应停止的 <AbortSignal>。

返回：<AsyncIterator> 个具有以下属性的对象：

  eventType <string> 变更类型

  filename <string> | <Buffer> | <null> 变更的文件的名称
```
```js
const { watch } = require('node:fs/promises');

const ac = new AbortController();
const { signal } = ac;
setTimeout(() => ac.abort(), 10000);

(async () => {
  try {
    const watcher = watch(__filename, { signal });
    for await (const event of watcher)
      console.log(event);
  } catch (err) {
    if (err.name === 'AbortError')
      return;
    throw err;
  }
})();
```

## fsPromises.writeFile(file, data[, options])
异步地将数据写入文件，如果文件已经存在，则替换该文件。data 可以是字符串、缓冲区、AsyncIterable、或 Iterable 对象。

如果 data 是缓冲区，则忽略 encoding 选项。

如果 options 是字符串，则它指定编码
```txt
file <string> | <Buffer> | <URL> | <FileHandle> 文件名或 FileHandle

data <string> | <Buffer> | <TypedArray> | <DataView> | <AsyncIterable> | <Iterable> | <Stream>

options <Object> | <string>

  encoding <string> | <null> 默认值：'utf8'

  mode <integer> 默认值：0o666

  flag <string> 参见 支持文件系统 flags。默认值：'w'。

  flush <boolean> 如果所有数据都成功写入文件，并且 flush 是 true，则使用 filehandle.sync() 来刷新数据。默认值：false。

  signal <AbortSignal> 允许中止正在进行的 writeFile

返回：<Promise> 成功时将使用 undefined 履行。
```

```js
import { writeFile } from 'node:fs/promises';
import { Buffer } from 'node:buffer';

try {
  const controller = new AbortController();
  const { signal } = controller;
  const data = new Uint8Array(Buffer.from('Hello Node.js'));
  const promise = writeFile('message.txt', data, { signal });

  // Abort the request before the promise settles.
  controller.abort();

  await promise;
} catch (err) {
  // When a request is aborted - err is an AbortError
  console.error(err);
}
```
