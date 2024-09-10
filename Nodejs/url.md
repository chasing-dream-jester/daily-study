# url
WHATWG URL API

## Class:URL

### new URL(input[, base])
用于创建url对象
```txt
input：要解析的 URL 字符串。
base（可选）：一个基本 URL。如果提供了这个参数，input 将相对于这个基本 URL 进行解析。
```
```js
// 解析绝对 URL
const myURL = new URL('https://example.com:8080/path/name?query=string#hash');

// 解析相对 URL：
const baseURL = 'https://example.com/path/';
const myURL = new URL('name?query=string#hash', baseURL);
```

#### url.hash
获取和设置网址的片段部分 
```js
const myURL = new URL('https://example.org/foo#bar');
console.log(myURL.hash);
// Prints #bar

myURL.hash = 'baz';
console.log(myURL.href);
// Prints https://example.org/foo#baz
```

#### url.host
获取和设置网址的主机部分
```js
const myURL = new URL('https://example.org:81/foo');
console.log(myURL.host);
// Prints example.org:81

myURL.host = 'example.com:82';
console.log(myURL.href);
// Prints https://example.com:82/foo
```

#### url.hostname
获取和设置网址的主机名部分。url.host 和 url.hostname 之间的主要区别是 url.hostname 不包括端口。
```js
const myURL = new URL('https://example.org:81/foo');
console.log(myURL.hostname);
// Prints example.org

// Setting the hostname does not change the port
myURL.hostname = 'example.com';
console.log(myURL.href);
// Prints https://example.com:81/foo

// Use myURL.host to change the hostname and port
myURL.host = 'example.org:82';
console.log(myURL.href);
// Prints https://example.org:82/foo
```

#### url.href
获取和设置序列化的网址

获取 href 属性的值相当于调用 url.toString()

将此属性的值设置为新值相当于使用 new URL(value) 创建新的 URL 对象。URL 对象的每个属性都将被修改。

```js
const myURL = new URL('https://example.org/foo');
console.log(myURL.href);
// Prints https://example.org/foo

myURL.href = 'https://example.com/bar';
console.log(myURL.href);
// Prints https://example.com/bar
```

#### url.origin
获取网址的源的只读的序列化
```js
const myURL = new URL('https://example.org/foo/bar?baz');
console.log(myURL.origin);
// Prints https://example.org
```

#### url.password
获取和设置网址的密码部分
```js
const myURL = new URL('https://abc:xyz@example.com');
console.log(myURL.password);
// Prints xyz

myURL.password = '123';
console.log(myURL.href);
// Prints https://abc:123@example.com/
```

#### url.pathname
获取和设置网址的路径部分
```js
const myURL = new URL('https://example.org/abc/xyz?123');
console.log(myURL.pathname);
// Prints /abc/xyz

myURL.pathname = '/abcdef';
console.log(myURL.href);
// Prints https://example.org/abcdef?123
```

#### url.port
获取和设置网址的端口部分

端口值可以是数字，也可以是包含 0 到 65535（含）范围内的数字的字符串。将值设置为给定 protocol 的 URL 对象的默认端口将导致 port 值成为空字符串 ('')

端口值可以是空字符串，在这种情况下端口取决于协议/方案：

|   协议    |   端口  |
|-------|----------|
|ftp    |21        |
|file   |          |
|http   |80        |
|https  |443       |
|ws     |80        |
|wss    |443       |

为端口分配值后，该值将首先使用 .toString() 转换为字符串。

如果该字符串无效但以数字开头，则将前导数字分配给 port。如果数字在上述范围之外，则将其忽略。


#### url.protocol
获取和设置网址的协议部分,分配给 protocol 属性的无效的网址协议值将被忽略。

```js
const myURL = new URL('https://example.org');
console.log(myURL.protocol);
// Prints https:

myURL.protocol = 'ftp';
console.log(myURL.href);
// Prints ftp://example.org/
```

#### url.search
获取和设置网址的序列化的查询部分
```js
const myURL = new URL('https://example.org/abc?123');
console.log(myURL.search);
// Prints ?123

myURL.search = 'abc=xyz';
console.log(myURL.href);
// Prints https://example.org/abc?abc=xyz
```

#### url.searchParams
获取表示网址查询参数的 URLSearchParams 对象。该属性是只读的，但它提供的 URLSearchParams 对象可用于改变 URL 实例；要替换 URL 的全部查询参数，请使用 url.search 设置器。有关详细信息，请参阅 [URLSearchParams](https://nodejs.cn/api/url.html#class-urlsearchparams) 文档。

当使用 .searchParams 修改 URL 时要小心，因为根据 WHATWG 规范，URLSearchParams 对象使用不同的规则来确定要对哪些字符进行百分比编码。例如，URL 对象不会对 ASCII 波浪号 (~) 字符进行百分比编码，而 URLSearchParams 将始终对其进行编码：

```js
const myURL = new URL('https://example.org/abc?foo=~bar');

console.log(myURL.search);  // prints ?foo=~bar

// Modify the URL via searchParams...
myURL.searchParams.sort();

console.log(myURL.search);  // prints ?foo=%7Ebar
```

#### url.username
获取和设置网址的用户名部分
```js
const myURL = new URL('https://abc:xyz@example.com');
console.log(myURL.username);
// Prints abc

myURL.username = '123';
console.log(myURL.href);
// Prints https://123:xyz@example.com/
```

#### url.toString()
URL 对象上的 toString() 方法返回序列化的网址。返回值等同于 url.href 和 url.toJSON() 的值。

#### url.toJSON()
URL 对象上的 toJSON() 方法返回序列化的网址。返回值等同于 url.href 和 url.toString() 的值。

当 URL 对象用 JSON.stringify() 序列化时，会自动调用此方法
```js
const myURLs = [
  new URL('https://www.example.com'),
  new URL('https://test.example.org'),
];
console.log(JSON.stringify(myURLs));
// Prints ["https://www.example.com/","https://test.example.org/"]
```

### URL.parse(input[, base])
```txt
input <string> 要解析的绝对或相对的输入网址。如果 input 是相对的，则需要 base。如果 input 是绝对的，则忽略 base。如果 input 不是字符串，则首先是 转换为字符串。

base <string> 如果 input 不是绝对的，则为要解析的基本网址。如果 base 不是字符串，则首先是 转换为字符串。

返回：<URL> | <null>
```

## Class:URLSearchParams

URLSearchParams API 提供对 URL 查询的读写访问。URLSearchParams 类也可以与以下四个构造函数之一单独使用。URLSearchParams 类也在全局对象上可用。

### new URLSearchParams()
实例化新的空 URLSearchParams 对象

### new URLSearchParams(string)
将 string 解析为查询字符串，并使用它来实例化新的 URLSearchParams 对象。前导 '?'（如果存在）将被忽略。

```js
let params;

params = new URLSearchParams('user=abc&query=xyz');
console.log(params.get('user'));
// Prints 'abc'
console.log(params.toString());
// Prints 'user=abc&query=xyz'

params = new URLSearchParams('?user=abc&query=xyz');
console.log(params.toString());
// Prints 'user=abc&query=xyz'
```

### new URLSearchParams(obj)
使用查询哈希映射实例化新的 URLSearchParams 对象。obj 的每个属性的键和值总是被强制转换为字符串。

与 querystring 模块不同，不允许以数组值的形式出现重复的键。数组使用 array.toString() 字符串化，它简单地用逗号连接所有数组元素。

```js
const params = new URLSearchParams({
  user: 'abc',
  query: ['first', 'second'],
});
console.log(params.getAll('query'));
// Prints [ 'first,second' ]
console.log(params.toString());
// Prints 'user=abc&query=first%2Csecond'
```

### new URLSearchParams(iterable)
iterable <Iterable> 元素为键值对的可迭代对象

以类似于 Map 的构造函数的方式使用可迭代映射实例化新的 URLSearchParams 对象。iterable 可以是 Array 或任何可迭代对象。这意味着 iterable 可以是另一个 URLSearchParams，在这种情况下，构造函数将简单地创建提供的 URLSearchParams 的克隆。iterable 的元素是键值对，并且本身可以是任何可迭代对象

允许重复的键
```js
let params;

// Using an array
params = new URLSearchParams([
  ['user', 'abc'],
  ['query', 'first'],
  ['query', 'second'],
]);
console.log(params.toString());
// Prints 'user=abc&query=first&query=second'

// Using a Map object
const map = new Map();
map.set('user', 'abc');
map.set('query', 'xyz');
params = new URLSearchParams(map);
console.log(params.toString());
// Prints 'user=abc&query=xyz'

// Using a generator function
function* getQueryPairs() {
  yield ['user', 'abc'];
  yield ['query', 'first'];
  yield ['query', 'second'];
}
params = new URLSearchParams(getQueryPairs());
console.log(params.toString());
// Prints 'user=abc&query=first&query=second'

// Each key-value pair must have exactly two elements
new URLSearchParams([
  ['user', 'abc', 'error'],
]);
// Throws TypeError [ERR_INVALID_TUPLE]:
// Each query pair must be an iterable [name, value] tuple
```

### urlSearchParams.append(name, value)
将新的名称-值对追加到查询字符串

### urlSearchParams.delete(name[, value])
如果提供了 value，则删除名称为 name 且值为 value 的所有名称-值对。

如果未提供 value，则删除名称为 name 的所有名称-值对

### urlSearchParams.entries()
在查询中的每个名称-值对上返回 ES6 Iterator。迭代器的每一项都是 JavaScript Array。Array 的第一项是 name，Array 的第二项是 value。

```js
const params = new URLSearchParams('key1=value1&key2=value2&key3=value3');

for (const [key, value] of params.entries()) {
  console.log(`${key}: ${value}`);
}

```
### urlSearchParams.forEach(fn[, thisArg])

```js
const myURL = new URL('https://example.org/?a=b&c=d');
myURL.searchParams.forEach((value, name, searchParams) => {
  console.log(name, value, myURL.searchParams === searchParams);
});
// Prints:
//   a b true
//   c d true
```

### urlSearchParams.get(name)
返回名称为 name 的第一个名称-值对的值。如果没有这样的对，则返回 null

### urlSearchParams.getAll(name)
返回名称为 name 的所有名称-值对的值。如果没有这样的对，则返回空数组

### urlSearchParams.has(name[, value])
```txt
name <string>

value <string>

返回：<boolean>
```
检查 URLSearchParams 对象是否包含基于 name 和可选的 value 参数的键值对。

如果提供了 value，则当存在具有相同 name 和 value 的名称-值对时返回 true。

如果未提供 value，则如果至少有一个名称为 name 的名称-值对，则返回 true。

### urlSearchParams.keys()
在每个名称-值对的名称上返回 ES6 Iterator。

```js
const params = new URLSearchParams('foo=bar&foo=baz');
for (const name of params.keys()) {
  console.log(name);
}
// Prints:
//   foo
//   foo
```

### urlSearchParams.set(name, value)
```js
const params = new URLSearchParams();
params.append('foo', 'bar');
params.append('foo', 'baz');
params.append('abc', 'def');
console.log(params.toString());
// Prints foo=bar&foo=baz&abc=def

params.set('foo', 'def');
params.set('xyz', 'opq');
console.log(params.toString());
// Prints foo=def&abc=def&xyz=opq
```

### urlSearchParams.size
参数条目的总数。

### urlSearchParams.sort()
按名称对所有现有的名称-值对进行就地排序。排序是使用 稳定排序算法 完成的，因此保留了具有相同名称的名称-值对之间的相对顺序。

该方法尤其可用于增加缓存命中

```js
const params = new URLSearchParams('query[]=abc&type=search&query[]=123');
params.sort();
console.log(params.toString());
// Prints query%5B%5D=abc&query%5B%5D=123&type=search
```

### urlSearchParams.toString()
返回序列化为字符串的搜索参数，必要时使用百分比编码的字符

### urlSearchParams.values()
在每个名称-值对的值上返回 ES6 Iterator。

### urlSearchParams[Symbol.iterator]()
在查询字符串中的每个名称-值对上返回 ES6 Iterator。迭代器的每一项都是 JavaScript Array。Array 的第一项是 name，Array 的第二项是 value。

```js
const params = new URLSearchParams('foo=bar&xyz=baz');
for (const [name, value] of params) {
  console.log(name, value);
}
// Prints:
//   foo bar
//   xyz baz
```

### url.domainToASCII(domain)
返回 domain 的 Punycode ASCII 序列化。如果 domain 是无效域，则返回空字符串。

它执行与 url.domainToUnicode() 相反的操作
```js
const url = require('node:url');

console.log(url.domainToASCII('español.com'));
// Prints xn--espaol-zwa.com
console.log(url.domainToASCII('中文.com'));
// Prints xn--fiq228c.com
console.log(url.domainToASCII('xn--iñvalid.com'));
// Prints an empty string
```

### url.domainToUnicode(domain)
返回 domain 的 Unicode 序列化。如果 domain 是无效域，则返回空字符串。
```js
const url = require('node:url');

console.log(url.domainToUnicode('xn--espaol-zwa.com'));
// Prints español.com
console.log(url.domainToUnicode('xn--fiq228c.com'));
// Prints 中文.com
console.log(url.domainToUnicode('xn--iñvalid.com'));
// Prints an empty string
```

### url.format(URL[, options])
```txt
URL <URL> 一个 WHATWG URL 对象

options <Object>

  auth <boolean> 如果序列化 URL 字符串应包含用户名和密码，则为 true，否则为 false。默认值：true。

  fragment <boolean> 如果序列化 URL 字符串应包含片段，则为 true，否则为 false。默认值：true。

  search <boolean> 如果序列化 URL 字符串应包含搜索查询，则为 true，否则为 false。默认值：true。

  unicode <boolean> true 如果 URL 字符串的主机组件中出现的 Unicode 字符应直接编码，而不是 Punycode 编码。默认值：false。

返回：<string>
```
```js
const url = require('node:url');
const myURL = new URL('https://a:b@測試?abc#foo');

console.log(myURL.href);
// Prints https://a:b@xn--g6w251d/?abc#foo

console.log(myURL.toString());
// Prints https://a:b@xn--g6w251d/?abc#foo

console.log(url.format(myURL, { fragment: false, unicode: true, auth: false }));
// Prints 'https://測試/?abc'
```
