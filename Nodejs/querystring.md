# querystring
node:querystring 模块提供了用于解析和格式化网址查询字符串的实用工具

querystring 比 `URLSearchParams` 性能更高，但不是标准化的 API。当性能不重要或需要与浏览器代码兼容时使用 `URLSearchParams`。

查询字符串是 URL 中以 ? 开头，包含键值对的部分，通常用于传递参数。例如，在 URL http://example.com/page?name=John&age=30 中，name=John&age=30 就是查询字符串。

querystring 模块提供了解析和格式化查询字符串的方法，可以将查询字符串解析为对象，或者将对象格式化为查询字符串

## querystring.decode()
querystring.parse() 方法的别名。它们的功能完全相同，都是用于将查询字符串解析为对象

默认情况下，querystring.decode 方法会对 URL 编码的字符串进行解码

## querystring.parse(str[, sep[, eq[, options]]])
```txt
str：一个字符串，表示需要解析的查询字符串。
sep：一个字符串，表示查询字符串中键值对之间的分隔符。默认值是 &。
eq：一个字符串，表示查询字符串中键和值之间的分隔符。默认值是 =。
options：一个对象，包含解析选项。可选。

返回一个对象，表示解析后的键值对
```
```js
const querystring = require('querystring');
const queryString = 'name=John&age=30';
const parsed = querystring.parse(queryString);
console.log(parsed); // 输出: { name: 'John', age: '30' }

// 处理 URL 编码的查询字符串 
const querystring = require('querystring');
const queryString = 'name=John%20Doe&city=New%20York';
const parsed = querystring.decode(queryString);
console.log(parsed); // 输出: { name: 'John Doe', city: 'New York' }

// 自定义分隔符和等号 ：
const querystring = require('querystring');
const queryString = 'name:John;age:30';
const parsed = querystring.decode(queryString, ';', ':');
console.log(parsed); // 输出: { name: 'John', age: '30' }

```

## querystring.escape(str)
用于对字符串进行 URL 编码。它的行为类似于 JavaScript 全局对象中的 encodeURIComponent 方法，主要用于将字符串中的特殊字符转换为百分号编码格式
```js
// 基本用法
const querystring = require('querystring');
const str = "Hello World!";
const encodedStr = querystring.escape(str);
console.log(encodedStr); // 输出: Hello%20World%21

// 处理特殊字符 ：
const querystring = require('querystring');
const str = "name=John Doe&city=New York";
const encodedStr = querystring.escape(str);
console.log(encodedStr); // 输出: name%3DJohn%20Doe%26city%3DNew%20York
```
uerystring.escape 与 encodeURIComponent 的行为非常相似，但 querystring.escape 是 Node.js 环境中的方法，而 encodeURIComponent 是 JavaScript 全局对象中的方法，适用于浏览器和 Node.js 环境

使用场景
- 处理 URL 查询参数 ：在构建带有查询参数的 URL 时，对参数值进行编码，以确保它们在 URL 中正确传递。
- 处理表单数据 ：在处理以查询字符串格式提交的表单数据时，对数据进行编码，确保特殊字符不会引起问题。

## querystring.unescape(str)
用于对 URL 编码的字符串进行解码。它的行为类似于 JavaScript 全局对象中的 decodeURIComponent 方法，主要用于将百分号编码的字符串还原为原始字符串。
```js
const querystring = require('querystring');
const encodedStr = "Hello%20World%21";
const decodedStr = querystring.unescape(encodedStr);
console.log(decodedStr); // 输出: Hello World!

// 处理特殊字符 ：
const querystring = require('querystring');
const encodedStr = "name%3DJohn%20Doe%26city%3DNew%20York";
const decodedStr = querystring.unescape(encodedStr);
console.log(decodedStr); // 输出: name=John Doe&city=New York

```
querystring.unescape 与 decodeURIComponent 的行为非常相似，但 querystring.unescape 是 Node.js 环境中的方法，而 decodeURIComponent 是 JavaScript 全局对象中的方法，适用于浏览器和 Node.js 环境

## querystring.encode()
querystring.encode() 函数是 querystring.stringify() 的别名。

## querystring.stringify(obj[, sep[, eq[, options]]])
```txt
obj：一个对象，表示需要转换为查询字符串的对象。
sep：一个字符串，表示查询字符串中键值对之间的分隔符。默认值是 &。
eq：一个字符串，表示查询字符串中键和值之间的分隔符。默认值是 =。
options：一个对象，包含格式化选项。可选。

返回一个字符串，表示格式化后的查询字符串
```

```js
// 基本用法
const querystring = require('querystring');
const obj = { name: 'John', age: 30 };
const queryString = querystring.stringify(obj);
console.log(queryString); // 输出: 'name=John&age=30'

// 处理复杂对象
const querystring = require('querystring');
const obj = { name: 'John Doe', city: 'New York' };
const queryString = querystring.stringify(obj);
console.log(queryString); // 输出: 'name=John%20Doe&city=New%20York'

// 自定义分隔符和等号 
const querystring = require('querystring');
const obj = { name: 'John', age: 30 };
const queryString = querystring.stringify(obj, ';', ':');
console.log(queryString); // 输出: 'name:John;age:30'

// 处理嵌套对象
const querystring = require('querystring');
const obj = { user: { name: 'John', age: 30 }, city: 'New York' };
const queryString = querystring.stringify(obj);
console.log(queryString); // 输出: 'user=%5Bobject%20Object%5D&city=New%20York'
```
querystring.stringify 方法会对特殊字符进行 URL 编码，例如空格会被编码为 %20。

默认情况下，键值对之间使用 & 作为分隔符，键和值之间使用 = 作为分隔符。

对于嵌套对象，querystring.stringify 会将其转换为 [object Object]，这通常不是期望的行为。如果需要更复杂的查询字符串处理，可以考虑使用第三方库如 qs
