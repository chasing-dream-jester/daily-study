# buffer
Buffer 是一个全局对象，用于处理二进制数据。它在处理文件I/O、网络通信以及其他需要操作二进制数据的场景中非常有用。
它主要用于处理那些不能直接被 JavaScript 处理的二进制数据，比如文件读写、网络通信等场景

## 使用场景
1. **文件系统操作** 在读取或写入文件时，特别是在涉及到非文本文件（如图片、视频等）时，会使用 Buffer 来处理二进制数据
2. **网络通信**  在 HTTP 服务器或客户端、WebSocket 等网络应用中，Buffer 用于处理网络传输中的二进制数据
3. **数据转换** 在处理不同编码的数据时，Buffer 可以用于转换数据格式，比如将字符串转换为二进制数据，或者将二进制数据解码为字符串
4. **性能优化** 在某些情况下，使用 Buffer 可以提高数据处理的性能，因为它允许直接操作内存中的二进制数据，而不是 JavaScript 的字符串或对象
5. **流处理** Buffer 可以与 Node.js 的流（Streams）API 配合使用，用于构建高效处理大量数据的管道

## Class:Buffer
### 静态方法
#### 创建 Buffer
1. Buffer.alloc(size[, fill[, encoding]])  
   
   size ：长度； fill：填充值，默认值0；encoding：编码，默认值：'utf8'
2. Buffer.allocUnsafe(size) 
   
   size：要分配的Buffer的大小（以字节为单位）。这个方法返回一个新的Buffer，其中的内容需要被初始化，以避免敏感数据泄露。
3. Buffer.from(array) 
   
   array：一个用于创建Buffer实例的八位字节的数组。
4. Buffer.from(buffer) 
   
   buffer：一个现有的Buffer，用于拷贝数据创建新的Buffer。
5. Buffer.from(string[, encoding])
   
   string：用于创建Buffer的字符串。encoding：（可选）字符编码，默认是utf8。

#### 检测Buffer 对象
Buffer.isBuffer(obj) 用于检测给定的对象是否是一个 Buffer 对象

#### 检查的字符编码
Buffer.isEncoding(encoding)

### buffer对象
1. buf[index]
   
   可以使用数组索引语法来访问 Buffer 对象中的特定字节。索引从 0 开始，范围是 [0, buffer.length - 1]
   ```js
    const buf = Buffer.from('Hello, World!', 'utf8');
    console.log(buf[0]); // 输出: 72 (ASCII 码 'H')
    console.log(buf[1]); // 输出: 101 (ASCII 码 'e')
    console.log(buf[2]); // 输出: 108 (ASCII 码 'l')
   ```
2. buf.buffer
   
   buf.buffer 返回一个 ArrayBuffer 对象，这个对象表示 Buffer 实例的底层内存。通过这个属性，你可以在需要时直接操作底层的 ArrayBuffer
   ```js
    const { Buffer } = require('node:buffer');
    const arrayBuffer = new ArrayBuffer(16);
    const buffer = Buffer.from(arrayBuffer);
    console.log(buffer.buffer === arrayBuffer);
    // Prints: true
   ```
3. buf.compare(target[, targetStart[, targetEnd[, sourceStart[, sourceEnd]]]]) 
   
   ```txt
   target <Buffer> | <Uint8Array> 用于比较 buf 的 Buffer 或 Uint8Array

   targetStart <integer> target 内开始比较的偏移量。默认值：0

   targetEnd <integer> target 中结束比较（不包括）的偏移量。默认值：target length

   sourceStart <integer> buf 内开始比较的偏移量。默认值：0

   sourceEnd <integer> buf 中结束比较（不包括）的偏移量。默认值：buf.length

   返回：<integer>
   ```
   ```js
    const { Buffer } = require('node:buffer');
    const buf1 = Buffer.from('ABC');
    const buf2 = Buffer.from('BCD');
    const buf3 = Buffer.from('ABCD');
    console.log(buf1.compare(buf1));
    // Prints: 0
    console.log(buf1.compare(buf2));
    // Prints: -1
    console.log(buf1.compare(buf3));
    // Prints: -1
    console.log(buf2.compare(buf1));
    // Prints: 1
    console.log(buf2.compare(buf3));
    // Prints: 1
    console.log([buf1, buf2, buf3].sort(Buffer.compare));
    // Prints: [ <Buffer 41 42 43>, <Buffer 41 42 43 44>, <Buffer 42 43 44> ]
    // (This result is equal to: [buf1, buf3, buf2].)
   ```
4. buf.copy(target[, targetStart[, sourceStart[, sourceEnd]]])
   ```txt
   target <Buffer> | <Uint8Array> 要复制到的 Buffer 或 Uint8Array

   targetStart <integer> target 内开始写入的偏移量。默认值：0

   sourceStart <integer> buf 内开始复制的偏移量。默认值：0

   sourceEnd <integer> buf 内停止复制的偏移量（不包括）。默认值：buf.length

   返回：<integer> 复制的字节数
   ```
5. buf.entries() 
   
   根据 buf 的内容创建并返回 迭代器 对 [index, byte] 
6. buf.equals(otherBuffer)
   
   用于比较 buf 的 Buffer 或 Uint8Array
7. buf.fill(value[, offset[, end]][, encoding])
   ```txt
    value <string> | <Buffer> | <Uint8Array> | <integer> 用于填充 buf 的值。空值（字符串、Uint8Array、缓冲区）被强制转换为 0。

    offset <integer> 在开始填充 buf 之前要跳过的字节数。默认值：0。

    end <integer> 停止填充 buf（不包括在内）的位置。默认值：buf.length。

    encoding <string> 如果 value 是字符串，则为 value 的编码。默认值：'utf8'。

    返回：<Buffer> buf 的引用。

    用指定的 value 填充 buf。如果没有给定 offset 和 end，则整个 buf 都会被填满
   ```
8. buf.includes(value[, byteOffset][, encoding])
   ```txt
    value <string> | <Buffer> | <Uint8Array> | <integer> 要搜索的内容。

    byteOffset <integer> 开始搜索 buf 的位置。如果为负数，则从 buf 的末尾开始计算偏移量。默认值：0。

    encoding <string> 如果 value 是字符串，则这就是它的编码。默认值：'utf8'。

    返回：<boolean> 如果在 buf 中找到 value，则为 true，否则为 false
   ```
9. buf.indexOf(value[, byteOffset][, encoding]) (参数含义同上)
10. buf.lastIndexOf(value[, byteOffset][, encoding])
    ```js
      const { Buffer } = require('node:buffer');

      const buf = Buffer.from('this buffer is a buffer');

      console.log(buf.lastIndexOf('this'));
      // Prints: 0
      console.log(buf.lastIndexOf('buffer'));
      // Prints: 17
      console.log(buf.lastIndexOf(Buffer.from('buffer')));
      // Prints: 17
      console.log(buf.lastIndexOf(97));
      // Prints: 15 (97 is the decimal ASCII value for 'a')
      console.log(buf.lastIndexOf(Buffer.from('yolo')));
      // Prints: -1
      console.log(buf.lastIndexOf('buffer', 5));
      // Prints: 5
      console.log(buf.lastIndexOf('buffer', 4));
      // Prints: -1

      const utf16Buffer = Buffer.from('\u039a\u0391\u03a3\u03a3\u0395', 'utf16le');

      console.log(utf16Buffer.lastIndexOf('\u03a3', undefined, 'utf16le'));
      // Prints: 6
      console.log(utf16Buffer.lastIndexOf('\u03a3', -5, 'utf16le'));
      // Prints: 4
    ```
11. buf.keys() 
    创建并返回 迭代器 个 buf 个键（索引）
    ```js
      const { Buffer } = require('node:buffer');
      const buf = Buffer.from('buffer');
      for (const key of buf.keys()) {
        console.log(key);
      }
    ```
12. buf.length
    返回 buf 中的字节数
13. buf.subarray([start[, end]])
    ```txt
      start <integer> 新的 Buffer 将开始的位置。默认值：0。

      end <integer> 新的 Buffer 将结束的位置（不包括在内）。默认值：buf.length。

      返回：<Buffer>
    ```
    注意：该操作返回一个新的 Buffer，它引用与原始内存相同的内存，但由 start 和 end 索引进行偏移和裁剪。修改新的 Buffer 切片会修改原来 Buffer 中的内存，因为两个对象分配的内存是重叠的
14. buf.slice([start[, end]]) 已弃用 使用上面b uf.subarray([start[, end]])
15. buf.swap16()
    返回：<Buffer> buf 的引用 ，将 buf 解释为无符号 16 位整数数组，并就地交换字节顺序。如果 buf.length 不是 2 的倍数，则抛出 ERR_INVALID_BUFFER_SIZE。
    ```js
       const { Buffer } = require('node:buffer');

      const buf1 = Buffer.from([0x1, 0x2, 0x3, 0x4, 0x5, 0x6, 0x7, 0x8]);

      console.log(buf1);
      // Prints: <Buffer 01 02 03 04 05 06 07 08>

      buf1.swap16();

      console.log(buf1);
      // Prints: <Buffer 02 01 04 03 06 05 08 07>

      const buf2 = Buffer.from([0x1, 0x2, 0x3]);

      buf2.swap16();
      // Throws ERR_INVALID_BUFFER_SIZE.
    ```
16. buf.swap32()
    将 buf 解释为无符号 32 位整数数组，并就地交换字节顺序。如果 buf.length 不是 4 的倍数，则抛出 ERR_INVALID_BUFFER_SIZE。
18. buf.swap64()
    将 buf 解释为无符号 64 位整数数组，并就地交换字节顺序。如果 buf.length 不是 8 的倍数，则抛出 ERR_INVALID_BUFFER_SIZE。
19. buf.toJSON() 返回 buf 的 JSON 表示。JSON.stringify() 在字符串化 Buffer 实例时隐式调用此函数。
    ```js
       const { Buffer } = require('node:buffer');

      const buf = Buffer.from([0x1, 0x2, 0x3, 0x4, 0x5]);
      const json = JSON.stringify(buf);

      console.log(json);
      // Prints: {"type":"Buffer","data":[1,2,3,4,5]}

      const copy = JSON.parse(json, (key, value) => {
      return value && value.type === 'Buffer' ?
         Buffer.from(value) :
         value;
      });

      console.log(copy);
      // Prints: <Buffer 01 02 03 04 05>
    ```

20. buf.toString([encoding[, start[, end]]])
    ```txt
      encoding <string> 要使用的字符编码。默认值：'utf8'。

      start <integer> 开始解码的字节偏移量。默认值：0。

      end <integer> 停止解码的字节偏移量（不包括在内）。默认值：buf.length。

      返回：<string>
    ```
    ```js
      const { Buffer } = require('node:buffer');

      const buf1 = Buffer.allocUnsafe(26);

      for (let i = 0; i < 26; i++) {
      // 97 is the decimal ASCII value for 'a'.
      buf1[i] = i + 97;
      }

      console.log(buf1.toString('utf8'));
      // Prints: abcdefghijklmnopqrstuvwxyz
      console.log(buf1.toString('utf8', 0, 5));
      // Prints: abcde

      const buf2 = Buffer.from('tést');

      console.log(buf2.toString('hex'));
      // Prints: 74c3a97374

    ```
21. buf.values()
    为 buf 值（字节）创建并返回 迭代器。当在 for..of 语句中使用 Buffer 时，会自动调用此函数。
    ```js
      const { Buffer } = require('node:buffer');

      const buf = Buffer.from('buffer');

      for (const value of buf.values()) {
      console.log(value);
      }
      // Prints: 98 117 102 102 101 114

      for (const value of buf) {
      console.log(value);
      }
      // Prints: 98 117 102 102 101 114
    ```
22.  buf.write(string[, offset[, length]][, encoding])
     ```txt
      string <string> 要写入 buf 的字符串。

      offset <integer> 开始写入 string 之前要跳过的字节数。默认值：0。

      length <integer> 要写入的最大字节数（写入的字节数不会超过 buf.length - offset）。默认值：buf.length - offset。

      encoding <string> string 的字符编码。默认值：'utf8'。

      返回：<integer> 写入的字节数
     ```
     ```js
      const { Buffer } = require('node:buffer');

      const buf = Buffer.alloc(256);

      const len = buf.write('\u00bd + \u00bc = \u00be', 0);

      console.log(`${len} bytes: ${buf.toString('utf8', 0, len)}`);
      // Prints: 12 bytes: ½ + ¼ = ¾

      const buffer = Buffer.alloc(10);

      const length = buffer.write('abcd', 8);

      console.log(`${length} bytes: ${buffer.toString('utf8', 8, 10)}`);
      // Prints: 2 bytes : ab
     ```

## Class:Blob
Blob 封装了不可变的原始数据，可以在多个工作线程之间安全地共享。
### 常见方法
1. new buffer.Blob([sources[, options]])
   ```txt
   sources <string[]> | <ArrayBuffer[]> | <TypedArray[]> | <DataView[]> | <Blob[]> 将存储在 Blob 中的字符串数组、<ArrayBuffer>、<TypedArray>、<DataView> 或 <Blob> 对象、或此类对象的任何组合。

   options <Object>

   endings <string> 'transparent' 或 'native' 之一。设置为 'native' 时，字符串源部分中的行结束将转换为 require('node:os').EOL 指定的平台原生行结束。

   type <string> Blob 内容类型。type 的目的是传达数据的 MIME 媒体类型，但是不执行类型格式的验证。

   创建新的 Blob 对象，其中包含给定源的串接
   ```
   ```js
    const { Blob } = require('buffer');
    const blob = new Blob(['Hello, world!'], { type: 'text/plain' });
    console.log(blob);  // 输出: Blob { size: 13, type: 'text/plain' }
   ```
2. blob.arrayBuffer()
   返回使用包含 Blob 数据副本的 <ArrayBuffer> 履行的 promise,可以使用其方法来读取其内容
   ```js
    blob.arrayBuffer().then(buffer => {
      console.log(Buffer.from(buffer).toString('utf8'));  // 输出: Hello, world!
    });
   ```
3. blob.bytes()
   blob.bytes() 方法将 Blob 对象的字节作为 Promise<Uint8Array> 返回。
   ```js
    const blob = new Blob(['hello']);
    blob.bytes().then((bytes) => {
      console.log(bytes); // Outputs: Uint8Array(5) [ 104, 101, 108, 108, 111 ]
    }); 
   ```
4. blob.size
   blob.bytes() 方法将 Blob 对象的字节作为 Promise<Uint8Array> 返回
   ```js
    const blob = new Blob(['hello']);
    blob.bytes().then((bytes) => {
      console.log(bytes); // Outputs: Uint8Array(5) [ 104, 101, 108, 108, 111 ]
    }); 
   ```
5. blob.slice([start[, end[, type]]])
   ```txt
    start <number> 起始索引。

    end <number> 结束索引。

    type <string> 新 Blob 对象的 MIME 类型。默认为空字符串
   ```
   ```js
    const { Blob } = require('buffer');

    // 创建一个包含字符串数据的 Blob 对象
    const blob = new Blob(['Hello, world!'], { type: 'text/plain' });

    // 提取前5个字节的数据
    const slicedBlob = blob.slice(0, 5);

    slicedBlob.text().then(text => {
      console.log(text);  // 输出: Hello
    });
   ```
6. blob.stream()
   返回允许读取 Blob 内容的新 ReadableStream。
   ```js
    const { Blob } = require('buffer');
    // 创建一个包含字符串数据的 Blob 对象
    const blob = new Blob(['Hello, world!'], { type: 'text/plain' });
    // 获取 Blob 的 ReadableStream
    const stream = blob.stream();
    // 创建一个读取器
    const reader = stream.getReader();
    // 逐步读取 Blob 的内容
    reader.read().then(function processText({ done, value }) {
      if (done) {
        console.log('Stream complete');
        return;
      }
      // 将 Uint8Array 转换为字符串并输出
      console.log(new TextDecoder("utf-8").decode(value));
      // 继续读取
      return reader.read().then(processText);
    });

   ```
7. blob.text()
   返回使用解码为 UTF-8 字符串的 Blob 的内容履行的 promise, 你可以使用 text() 方法将其转换为文本
   ```js
    const { Blob } = require('buffer');

    // 创建一个包含二进制数据的 Blob 对象
    const binaryData = new Uint8Array([72, 101, 108, 108, 111, 44, 32, 119, 111, 114, 108, 100, 33]);
    const blob = new Blob([binaryData], { type: 'text/plain' });

    // 使用 text() 方法读取 Blob 的内容并转换为文本
    blob.text().then(text => {
      console.log(text);  // 输出: Hello, world!
    });

   ```
8. blob.type
   Blob 的内容类型
   ```js
    const { Blob } = require('buffer');

    // 创建一个包含字符串数据的 Blob 对象，并指定 MIME 类型为 "text/plain"
    const blob = new Blob(['Hello, world!'], { type: 'text/plain' });

    // 读取 Blob 的 MIME 类型
    console.log(blob.type);  // 输出: text/plain

   ```

## Class:File
file 继承：Blob
### 常用方法
new buffer.File(sources, fileName[, options])
   ```txt
    sources：一个包含要放入 File 的数据的数组。数据可以是 ArrayBuffer、ArrayBufferView、Blob、DOMString 等。
    fileName：一个字符串，表示文件的名称。
    options（可选）：一个包含属性的对象，可以指定：
      type：文件的 MIME 类型（默认为空字符串）。
      lastModified：文件的最后修改时间戳（以毫秒为单位，默认为 Date.now()）。
   ```
   ```js
    const { File } = require('buffer');

    // 创建一个包含字符串数据的 File 对象
    const file = new File(['Hello, world!'], 'hello.txt', { type: 'text/plain' });

    console.log(file.name);        // 输出: hello.txt
    console.log(file.type);        // 输出: text/plain
    console.log(file.lastModified); // 输出: 当前时间戳
   ```
### 使用案例
```js
const { File } = require('buffer');
const fs = require('fs');

// 创建一个包含字符串数据的 File 对象
const file = new File(['Hello, world!'], 'hello.txt', { type: 'text/plain' });

// 使用 Node.js 的文件系统模块将 File 对象的内容写入文件
file.arrayBuffer().then(buffer => {
  fs.writeFileSync('output.txt', Buffer.from(buffer));
  console.log('File has been written');
});
```
## 缓冲区常量
`buffer.constants.MAX_LENGTH` 单个 Buffer 实例允许的最大大小 在 32 位架构上，该值当前为 230 - 1（约 1 GiB）。在 64 位架构上，此值当前为 253 - 1（约 8 PiB）

`buffer.constants.MAX_STRING_LENGTH` 表示 string 基础类型可以拥有的最大 length，以 UTF-16 代码单元计算
### QA
Q:是什么让 Buffer.allocUnsafe() 和 Buffer.allocUnsafeSlow() "不安全"？

A:调用 Buffer.allocUnsafe() 和 Buffer.allocUnsafeSlow() 时，分配的内存段未初始化（未清零）。虽然这种设计使内存分配速度非常快，但分配的内存段可能包含可能敏感的旧数据。使用由 Buffer.allocUnsafe() 创建的 Buffer 而没有完全覆盖内存可以让旧数据在读取 Buffer 内存时泄漏。虽然使用 Buffer.allocUnsafe() 有明显的性能优势，但必须格外小心以避免将安全漏洞引入应
