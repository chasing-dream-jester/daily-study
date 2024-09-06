# path
node:path 模块提供了用于处理文件和目录的路径的实用工具
## path.basename(path[, suffix])
```txt
path <string>

suffix <string> 要删除的可选后缀

返回：<string>
```
```js
path.basename('/foo/bar/baz/asdf/quux.html');
// Returns: 'quux.html'

path.basename('/foo/bar/baz/asdf/quux.html', '.html');
// Returns: 'quux'
```

## path.delimiter
它用于提供平台特定的路径分隔符。不同操作系统使用不同的路径分隔符来分隔多个路径：

在 Unix 和类 Unix 系统（如 Linux 和 macOS）中，路径分隔符是冒号 (:)。

在 Windows 系统中，路径分隔符是分号 (;)。

path.delimiter 通常用于需要处理包含多个路径的环境变量（如 PATH）的情况。例如，假设你需要将多个路径添加到环境变量 PATH 中，可以使用 path.delimiter 来确保代码在不同操作系统上都能正确运行：

```js
const path = require('path');

// Example paths to add
const additionalPaths = ['/usr/local/bin', '/opt/bin'];

// Join the paths using the platform-specific delimiter
const newPath = additionalPaths.join(path.delimiter);

// Append the new paths to the existing PATH environment variable
process.env.PATH += path.delimiter + newPath;

console.log(`Updated PATH: ${process.env.PATH}`);

```

## path.dirname(path)
path.dirname() 方法返回 path 的目录名
```js
path.dirname('/foo/bar/baz/asdf/quux');
// Returns: '/foo/bar/baz/asdf'
```
### path.extname(path)
path.extname() 方法返回 path 的扩展名，即 path 的最后一部分中从最后一次出现的 .（句点）字符到字符串的结尾。

如果 path 的最后一部分中没有 .，或者除了 path 的基本名称（参见 path.basename()）的第一个字符之外没有 . 个字符，则返回空字符串

```js
path.extname('index.html');
// Returns: '.html'

path.extname('index.coffee.md');
// Returns: '.md'

path.extname('index.');
// Returns: '.'

path.extname('index');
// Returns: ''

path.extname('.index');
// Returns: ''

path.extname('.index.md');
// Returns: '.md'
```

## path.format(pathObject)
```txt
pathObject <Object> 任何具有以下属性的 JavaScript 对象：
  dir：路径的目录部分。
  root：路径的根部分（如果提供了 dir，则 root 会被忽略）。
  base：文件的完整名称（包括扩展名）。如果提供了 base，则 name 和 ext 会被忽略。
  name：文件名（不包括扩展名）。
  ext：文件的扩展名
```
```js
const path = require('path');

const pathObject = {
  dir: '/home/user',
  base: 'file.txt'
};

const formattedPath = path.format(pathObject);
console.log(formattedPath); // 输出: /home/user/file.txt

const path = require('path');

const pathObject = {
  root: '/',
  name: 'file',
  ext: '.txt'
};

const formattedPath = path.format(pathObject);
console.log(formattedPath); // 输出: /file.txt

```

## path.isAbsolute(path)
path.isAbsolute() 方法确定 path 是否为绝对路径。

## path.join([...paths])
path.join() 方法使用特定于平台的分隔符作为定界符将所有给定的 path 片段连接在一起，然后规范化生成的路径。

```js
path.join('/foo', 'bar', 'baz/asdf', 'quux', '..');
// Returns: '/foo/bar/baz/asdf'

path.join('foo', {}, 'bar');
// Throws 'TypeError: Path must be a string. Received {}'
```

## path.normalize(path)
path.normalize() 方法规范化给定的 path，解析 '..' 和 '.' 片段,规范化的过程包括解析 . 和 .. 这样的相对路径标记，以及去除多余的斜杠等。这个方法返回一个平台特定的标准化路径
```js
const path = require('path');

const inputPath = 'foo/bar//baz/asdf/quux/..';
const normalizedPath = path.normalize(inputPath);
console.log(normalizedPath); // 输出: 'foo/bar/baz/asdf'

// 去除多余/
const path = require('path');
const inputPath = 'foo///bar/baz';
const normalizedPath = path.normalize(inputPath);
console.log(normalizedPath); // 输出: 'foo/bar/baz'

// 处理 . ..
const path = require('path');
const inputPath = 'foo/bar/./baz/../quux';
const normalizedPath = path.normalize(inputPath);
console.log(normalizedPath); // 输出: 'foo/bar/quux'

```
## path.parse(path)
path.parse用于将路径字符串解析为一个对象，该对象包含路径的各个组成部分。通过解析路径，可以方便地访问路径中的不同部分，如根目录、目录名、文件名和扩展名

返回一个对象，包含路径的各个组成部分。这个对象包含以下属性：
```txt
root：根目录。
dir：目录路径。
base：包含文件名和扩展名的文件部分。
ext：文件扩展名。
name：不包含扩展名的文件名。
```
```js
const path = require('path');

const filePath = '/home/user/dir/file.txt';
const parsedPath = path.parse(filePath);

console.log(parsedPath);
// 输出:
// {
//   root: '/',
//   dir: '/home/user/dir',
//   base: 'file.txt',
//   ext: '.txt',
//   name: 'file'
// }

```
## path.posix
path.posix 属性提供对 path 方法的 POSIX 特定实现的访问,用于处理 POSIX（Portable Operating System Interface）风格的路径。POSIX 路径主要使用在 Unix 和类 Unix 系统（如 Linux 和 macOS）中，路径分隔符是正斜杠 (/)
```js
// 将多个路径片段连接成一个路径字符串
const path = require('path');
const joinedPath = path.posix.join('/foo', 'bar', 'baz/asdf', 'quux', '..');
console.log(joinedPath); // 输出: '/foo/bar/baz/asdf'

// 将路径片段解析为绝对路径
const path = require('path');
const resolvedPath = path.posix.resolve('/foo/bar', './baz');
console.log(resolvedPath); // 输出: '/foo/bar/baz'
// ...... 
```

## path.relative(from, to)
用于计算从一个路径到另一个路径的相对路径。这个方法在处理文件系统路径时非常有用，特别是当你需要在两个路径之间进行导航时
```txt
from：一个字符串，表示起始路径。
to：一个字符串，表示目标路径。
```
返回一个字符串，表示从 from 路径到 to 路径的相对路径
```js
const path = require('path');

const fromPath = '/home/user/dir';
const toPath = '/home/user/dir/subdir/file.txt';

const relativePath = path.relative(fromPath, toPath);
console.log(relativePath); // 输出: 'subdir/file.txt'

```
## path.resolve([...paths])
用于将路径片段解析为一个绝对路径。它从右到左处理传入的路径片段，直到构建出一个绝对路径。如果没有传入绝对路径片段，则使用当前工作目录作为起点。

行为：
从右到左依次处理每个路径片段。

如果遇到绝对路径，则忽略其左边的所有路径片段。

如果没有遇到绝对路径，则使用当前工作目录作为起点。

规范化路径，解析 . 和 .. 片段

```js
const path = require('path');
const resolvedPath = path.resolve('foo/bar', './baz');
console.log(resolvedPath); // 输出: /当前工作目录/foo/bar/baz

// 处理绝对路径
const path = require('path');
const resolvedPath = path.resolve('/foo/bar', './baz');
console.log(resolvedPath); // 输出: /foo/bar/baz

// 遇到绝对路径后忽略左边的路径片段
const path = require('path');
const resolvedPath = path.resolve('/foo/bar', '/tmp/file/', 'baz');
console.log(resolvedPath); // 输出: /tmp/file/baz

// 没有绝对路径时使用当前工作目录 
const path = require('path');
const resolvedPath = path.resolve('foo', 'bar', 'baz');
console.log(resolvedPath); // 输出: /当前工作目录/foo/bar/baz

// 处理 .. 和 . 片段 
const path = require('path');
const resolvedPath = path.resolve('foo/bar', '../baz');
console.log(resolvedPath); // 输出: /当前工作目录/foo/baz

```
## path.sep
它提供了平台特定的路径分隔符（path separator）。路径分隔符用于分隔路径中的各个部分，不同操作系统使用不同的路径分隔符：

在 Unix 和类 Unix 系统（如 Linux 和 macOS）中，路径分隔符是正斜杠 (/)。

在 Windows 系统中，路径分隔符是反斜杠 (\)。

通过 path.sep，你可以编写跨平台代码，而不需要手动处理不同操作系统的路径分隔符。

```js
const path = require('path');
console.log(`Path separator in this OS is: ${path.sep}`);
// 在 Unix 系统上输出: Path separator in this OS is: /
// 在 Windows 系统上输出: Path separator in this OS is: \

// 分割路径字符串 ：
const path = require('path');
const filePath = 'home/user/docs/file.txt';
const segments = filePath.split(path.sep);
console.log(segments); // 在 Unix 系统上输出: [ 'home', 'user', 'docs', 'file.txt' ]

const windowsFilePath = 'C:\\Users\\user\\docs\\file.txt';
const windowsSegments = windowsFilePath.split(path.sep);
console.log(windowsSegments); // 在 Windows 系统上输出: [ 'C:', 'Users', 'user', 'docs', 'file.txt' ]

```

## path.toNamespacedPath(path)
用于将给定的路径转换为命名空间路径（Namespaced Path）。这个方法主要用于 Windows 系统，因为 Windows 有一个特殊的命名空间路径格式，可以处理较长的路径和某些特殊的文件系统操作。

```js
const path = require('path');

const normalPath = 'C:\\path\\to\\file.txt';
const namespacePath = path.toNamespacedPath(normalPath);
console.log(namespacePath); // 输出: \\?\C:\path\to\file.txt

```
## path.win32
path.win32 属性提供对 path 方法的 Windows 特定实现的访问。
