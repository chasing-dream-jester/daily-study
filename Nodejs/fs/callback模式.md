# callback 模式
回调的 API 异步地执行所有操作，不会阻塞事件循环，然后在完成或错误时调用回调函数。

回调的 API 使用底层 Node.js 线程池在事件循环线程之外执行文件系统操作。这些操作不是同步的也不是线程安全的。对同一文件执行多个并发修改时必须小心，否则可能会损坏数据

所有API请参考 [fsPromise](./fsPromises.md) ，api方法一样，只是使用上不一样,callback 模式所有方法从node:fs模块导入，然后使用时最后一个参数使用callback，callback采用错误优先，第一个参数是err

例如：fs.access(path[, mode], callback)
```js
const fs = require('fs');
const constants = require('fs').constants;

async function checkFileAccess() {
    // 检查文件是否存在以及是否可读和可写
  await fs.access('example.txt', constants.F_OK | constants.R_OK | constants.W_OK,(err)=>{
    console.error('文件不存在或不可读写:', err);
  });
}

checkFileAccess();
```
