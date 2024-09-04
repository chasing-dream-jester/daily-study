# events
events用于创建和处理事件驱动的编程模型。events 模块提供了 EventEmitter 类，这个类可以用于创建、发射和监听事件。事件驱动的编程模型使得处理异步操作变得更加直观和高效。
## Class:EventEmitter
### 事件
#### newListener
```txt
eventName <string> | <symbol> 正在监听的事件的名称

listener <Function> 事件处理函数
```
在 Node.js 的 events 模块中，newListener 是一个特殊的事件。当一个新的事件监听器被添加到 EventEmitter 实例时，会触发 newListener 事件。通过监听 newListener 事件，你可以在监听器被添加时执行一些自定义逻辑。
```js
const EventEmitter = require('node:events');
class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();
// Only do this once so we don't loop forever
myEmitter.once('newListener', (event, listener) => {
  if (event === 'event') {
    // Insert a new listener in front
    myEmitter.on('event', () => {
      console.log('B');
    });
  }
});
myEmitter.on('event', () => {
  console.log('A');
});
myEmitter.emit('event');
// Prints:
//   B
//   A
```
#### removeListener
```txt
eventName <string> | <symbol> 事件名称

listener <Function> 事件处理函数
```
'removeListener' 事件在 listener 被删除后触发 见下面

### emitter
#### emitter.addListener(eventName, listener)
添加监听事件
#### emitter.emit(eventName[, ...args])
按注册顺序同步地调用为名为 eventName 的事件注册的每个监听器，并将提供的参数传给每个监听器。
如果事件有监听器，则返回 true，否则返回 false
```js
const EventEmitter = require('node:events');
const myEmitter = new EventEmitter();

// First listener
myEmitter.on('event', function firstListener() {
  console.log('Helloooo! first listener');
});
// Second listener
myEmitter.on('event', function secondListener(arg1, arg2) {
  console.log(`event with parameters ${arg1}, ${arg2} in second listener`);
});
// Third listener
myEmitter.on('event', function thirdListener(...args) {
  const parameters = args.join(', ');
  console.log(`event with parameters ${parameters} in third listener`);
});

console.log(myEmitter.listeners('event'));

myEmitter.emit('event', 1, 2, 3, 4, 5);

// Prints:
// [
//   [Function: firstListener],
//   [Function: secondListener],
//   [Function: thirdListener]
// ]
// Helloooo! first listener
// event with parameters 1, 2 in second listener
// event with parameters 1, 2, 3, 4, 5 in third listener
```
#### emitter.eventNames()
返回列出触发器已为其注册监听器的事件的数组。数组中的值是字符串或 Symbol
```js
const EventEmitter = require('node:events');

const myEE = new EventEmitter();
myEE.on('foo', () => {});
myEE.on('bar', () => {});

const sym = Symbol('symbol');
myEE.on(sym, () => {});

console.log(myEE.eventNames());
// Prints: [ 'foo', 'bar', Symbol(symbol) ]
```

#### emitter.getMaxListeners()
返回 EventEmitter 的当前最大监听器数的值，该值由 emitter.setMaxListeners(n) 设置或默认为 events.defaultMaxListeners

#### emitter.listenerCount(eventName[, listener])
返回监听名为 eventName 的事件的监听器数量。如果提供了 listener，它将返回在事件的监听器列表中找到监听器的次数。

#### emitter.listeners(eventName)
返回指定事件名称的所有监听器函数数组。这个方法在调试、监控和动态管理事件监听器时非常有用
```js
const EventEmitter = require('events');
const myEmitter = new EventEmitter();

function listener1() {
  console.log('Listener 1');
}

function listener2() {
  console.log('Listener 2');
}

// 添加事件监听器
myEmitter.on('event', listener1);
myEmitter.on('event', listener2);

// 获取所有监听器
const listeners = myEmitter.listeners('event');
console.log(listeners); // 输出: [ [Function: listener1], [Function: listener2] ]

// 触发事件
myEmitter.emit('event'); // 输出: Listener 1 // 输出: Listener 2
```
#### emitter.rawListeners(eventName)
用于返回指定事件名称的所有监听器函数数组，包括通过 once 方法添加的原始监听器。与 emitter.listeners(eventName) 不同，rawListeners 返回的是原始监听器，而 listeners 返回的是包装后的监听器。
```js
const EventEmitter = require('events');
const myEmitter = new EventEmitter();

function listener() {
  console.log('Listener');
}

// 使用 once 添加监听器
myEmitter.once('event', listener);

console.log(myEmitter.listeners('event')); // 输出: [ [Function: listener] ]
console.log(myEmitter.rawListeners('event')); // 输出: [ [Function: bound onceWrapper] { listener: [Function: listener] } ]
```

#### emitter.off(eventName, listener) / emitter.removeListener(eventName, listener)
从名为 eventName 的事件的监听器数组中移除指定的 listener
```js
const callback = (stream) => {
  console.log('someone connected!');
};
server.on('connection', callback);
// ...
server.removeListener('connection', callback); // server.off('connection', callback); 
``` 
#### emitter.removeAllListeners([eventName])
删除所有监听器，或指定 eventName 的监听器

#### emitter.on(eventName, listener)
返回对 EventEmitter 的引用，以便可以链式调用
将 listener 函数添加到名为 eventName 的事件的监听器数组的末尾。不检查是否已添加 listener。多次调用传入相同的 eventName 和 listener 组合将导致多次添加和调用 listener

#### emitter.prependListener(eventName, listener) 
将 listener 函数添加到名为 eventName 的事件的监听器数组的开头。不检查是否已添加 listener。多次调用传入相同的 eventName 和 listener 组合将导致多次添加和调用 listener

#### emitter.once(eventName, listener)
返回对 EventEmitter 的引用，以便可以链式调用
将名为 eventName 的事件添加一次性 listener 函数。下次触发 eventName 时，将移除此监听器，然后再调用

#### emitter.prependOnceListener(eventName, listener)
将名为 eventName 的事件的一次性 listener 函数添加到监听器数组的开头。下次触发 eventName 时，将移除此监听器，然后再调用。

#### emitter.setMaxListeners(n)
emitter.setMaxListeners() 方法允许修改此特定 EventEmitter 实例的限制。该值可以设置为 Infinity（或 0）以指示无限数量的监听器。

## events
#### events.defaultMaxListeners
默认情况下，最多可为任何单个事件注册 10 个监听器

#### events.getEventListeners(emitterOrTarget, eventName)
返回名为 eventName 的事件的监听器数组的副本

#### events.getMaxListeners(emitterOrTarget)
返回当前设置的最大监听器数量

#### events.errorMonitor
EventEmitter.errorMonitor 是一个特殊的符号（Symbol），用于监听 error 事件。与常规的 error 事件监听器不同，使用 errorMonitor 监听器不会影响默认的错误处理行为
注意：
errorMonitor 监听器不会阻止默认行为 ：使用 errorMonitor 监听器不会阻止默认的错误处理行为。如果没有常规的 error 事件监听器，Node.js 仍会抛出错误并终止进程。
优先级 ：errorMonitor 监听器会在常规的 error 事件监听器之前被调用
```js
const { EventEmitter, errorMonitor } = require('events');

const myEmitter = new EventEmitter();

// 监听 errorMonitor 事件
myEmitter.on(errorMonitor, (err) => {
  console.log('Error monitored:', err.message);
});

// 监听 error 事件
myEmitter.on('error', (err) => {
  console.log('Error handled:', err.message);
});

// 触发 error 事件
myEmitter.emit('error', new Error('Something went wrong'));

```

### events.once(emitter, name[, options])
用于创建一个一次性事件监听器，并返回一个 Promise。这个方法在处理一次性异步事件时非常有用。例如，你可以使用它等待某个特定事件的触发，而不需要手动管理事件监听器
```txt
emitter <EventEmitter>

name <string> | <symbol>

options <Object>

signal <AbortSignal> 可用于取消等待事件。

返回：<Promise>
```
```js
const { once, EventEmitter } = require('node:events');

async function run() {
  const ee = new EventEmitter();

  process.nextTick(() => {
    ee.emit('myevent', 42);
  });

  const [value] = await once(ee, 'myevent');
  console.log(value);

  const err = new Error('kaboom');
  process.nextTick(() => {
    ee.emit('error', err);
  });

  try {
    await once(ee, 'myevent');
  } catch (err) {
    console.error('error happened', err);
  }
}

run();
```
使用 AbortSignal 可用于取消等待事件
```js
const { EventEmitter, once } = require('node:events');

const ee = new EventEmitter();
const ac = new AbortController();

async function foo(emitter, event, signal) {
  try {
    await once(emitter, event, { signal });
    console.log('event emitted!');
  } catch (error) {
    if (error.name === 'AbortError') {
      console.error('Waiting for the event was canceled!');
    } else {
      console.error('There was an error', error.message);
    }
  }
}

foo(ee, 'foo', ac.signal);
ac.abort(); // Abort waiting for the event
ee.emit('foo'); // Prints: Waiting for the event was canceled!
```
### events.captureRejections
更改所有新的 EventEmitter 对象的默认 captureRejections 选项

### events.on(emitter, eventName[, options])
```txt
emitter <EventEmitter>

eventName <string> | <symbol> 正在监听的事件的名称

options <Object>

signal <AbortSignal> 可用于取消等待事件。

close - <string[]> 将结束迭代的事件的名称。

highWaterMark - <integer> 默认值：Number.MAX_SAFE_INTEGER 高水位线。每当缓冲的事件大小高于它时，触发器就会暂停。仅在实现 pause() 和 resume() 方法的触发器上受支持。

lowWaterMark - <integer> 默认值：1 低水位线。每当缓冲的事件大小低于它时，触发器就会恢复。仅在实现 pause() 和 resume() 方法的触发器上受支持。

返回：<AsyncIterator> 迭代 emitter 触发的 eventName 事件
```
```js
const { on, EventEmitter } = require('node:events');

(async () => {
  const ee = new EventEmitter();

  // Emit later on
  process.nextTick(() => {
    ee.emit('foo', 'bar');
    ee.emit('foo', 42);
  });

  for await (const event of on(ee, 'foo')) {
    // The execution of this inner block is synchronous and it
    // processes one event at a time (even with await). Do not use
    // if concurrent execution is required.
    console.log(event); // prints ['bar'] [42]
  }
  // Unreachable here
})();
```

### events.setMaxListeners(n[, ...eventTargets])
```js
const {
  setMaxListeners,
  EventEmitter,
} = require('node:events');

const target = new EventTarget();
const emitter = new EventEmitter();

setMaxListeners(5, target, emitter);
```

## EventTarget

### Node.js EventTarget 与 DOM EventTarget
1. 尽管 DOM EventTarget 实例可能是分层的，但 Node.js 中没有分层和事件传播的概念。也就是说，调度到 EventTarget 的事件不会通过嵌套目标对象的层次结构传播，这些目标对象可能每个都有自己的事件句柄集。
2. 在 Node.js EventTarget 中，如果事件监听器是一个异步函数或返回一个 Promise，而返回的 Promise 拒绝，则拒绝被自动捕获并以与同步抛出的监听器相同的方式处理（详见 EventTarget 错误处理）。

### NodeEventTarget 与 EventEmitter
1. NodeEventTarget 对象实现了 EventEmitter API 的修改子集，允许它在某些情况下模拟 EventEmitter。NodeEventTarget 不是 EventEmitter 的实例，在大多数情况下不能用来代替 EventEmitter。

2. 与 EventEmitter 不同，任何给定的 listener 最多可以在每个事件 type 中注册一次。尝试多次注册 listener 将被忽略。

3. NodeEventTarget 不模拟完整的 EventEmitter API。特别是 prependListener()、prependOnceListener()、rawListeners() 和 errorMonitor API 未被模拟。'newListener' 和 'removeListener' 事件也不会触发。

4. NodeEventTarget 没有为类型为 'error' 的事件实现任何特殊的默认行为。

5. NodeEventTarget 支持 EventListener 对象以及作为所有事件类型句柄的函数。


### EventTarget 错误处理
当注册的事件监听器抛出错误（或返回拒绝的 Promise）时，默认情况下，错误将被视为 process.nextTick() 上的未捕获异常。这意味着 EventTarget 中未捕获的异常将默认终止 Node.js 进程。

在事件监听器中抛出不会阻止调用其他已注册的处理程序。

EventTarget 没有为 'error' 类型的事件（如 EventEmitter）实现任何特殊的默认处理。

当前错误在到达 process.on('uncaughtException') 之前首先转发到 process.on('error') 事件。此行为已弃用，并将在未来版本中更改，以使 EventTarget 与其他 Node.js API 保持一致。任何依赖 process.on('error') 事件的代码都应与新行为保持一致。



## Class:EventTarget

### eventTarget.addEventListener(type, listener[, options])
为 type 事件添加新的句柄。对于每个 type 和每个 capture 选项值，任何给定的 listener 仅添加一次。

如果 once 选项为 true，则在下一次调度 type 事件后移除 listener
```txt
type <string>

listener <Function> | <EventListener>

options <Object>

once <boolean> 当为 true 时，监听器在第一次调用时自动移除。默认值：false。

passive <boolean> 当为 true 时，提示监听器不会调用 Event 对象的 preventDefault() 方法。默认值：false。

capture <boolean> Node.js 不直接使用。为 API 完整性而添加。默认值：false。

signal <AbortSignal> 当调用给定的 AbortSignal 对象的 abort() 方法时，则监听器将被移除。
```
```js
function handler(event) {}

const target = new EventTarget();
target.addEventListener('foo', handler, { capture: true });  // first
target.addEventListener('foo', handler, { capture: false }); // second

// Removes the second instance of handler
target.removeEventListener('foo', handler);

// Removes the first instance of handler
target.removeEventListener('foo', handler, { capture: true });
```

### eventTarget.dispatchEvent(event)
将 event 调度到 event.type 的句柄列表。

返回：`boolean` 如果任一事件的 cancelable 属性值为 false 或者未调用其 preventDefault() 方法，则为 true，否则为 false。

### eventTarget.removeEventListener(type, listener[, options])
从事件 type 的句柄列表中删除 listener。
```txt
type <string>

listener <Function> | <EventListener>

options <Object>

capture <boolean>
```

## Class:NodeEventTarget   
继承：`EventTarget`

NodeEventTarget 是 EventTarget 的 Node.js 特定扩展，它模拟了 EventEmitter API 的子集。

### nodeEventTarget.addListener(type, listener)
EventTarget 类的 Node.js 特定扩展，可模拟等效的 EventEmitter API。addListener() 和 addEventListener() 之间的唯一区别是 addListener() 将返回对 EventTarget 的引用
### nodeEventTarget.emit(type, arg)
EventTarget 类的 Node.js 特定扩展，将 arg 分派到 type 的处理程序列表。

返回：<boolean> 如果为 type 注册的事件监听器存在，则为 true，否则为 false。

### nodeEventTarget.eventNames()
node.js 特定于 EventTarget 类的扩展，它返回事件 type 名称的数组，事件监听器注册了这些名称。

### nodeEventTarget.listenerCount(type)
EventTarget 类的 Node.js 特定扩展，返回为 type 注册的事件监听器的数量。

### nodeEventTarget.getMaxListeners()
返回最大事件监听器数量的 EventTarget 类的 Node.js 特定扩展。

### nodeEventTarget.off(type, listener[, options]) 
eventTarget.removeEventListener() 的 Node.js 特定别名。

### nodeEventTarget.on(type, listener)
eventTarget.addEventListener() 的 Node.js 特定别名。

### nodeEventTarget.once(type, listener)
EventTarget 类的 Node.js 特定扩展，它为给定的事件 type 添加了 once 监听器。这相当于调用 on 并将 once 选项设置为 true。

### nodeEventTarget.removeAllListeners([type])
EventTarget 类的 Node.js 特定扩展。如果指定了 type，则删除 type 的所有注册监听器，否则删除所有注册的监听器。

### nodeEventTarget.removeListener(type, listener[, options])#
Node.js 特定于 EventTarget 类的扩展，用于删除给定 type 的 listener。removeListener() 和 removeEventListener() 之间的唯一区别是 removeListener() 将返回对 EventTarget 的引用



