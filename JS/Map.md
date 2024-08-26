## Map 数据结构
```ts
interface Map<K, V> {
    clear(): void;
    /**
     * @returns true if an element in the Map existed and has been removed, or false if the element does not exist.
     */
    delete(key: K): boolean;
    /**
     * Executes a provided function once per each key/value pair in the Map, in insertion order.
     */
    forEach(callbackfn: (value: V, key: K, map: Map<K, V>) => void, thisArg?: any): void;
    /**
     * Returns a specified element from the Map object. If the value that is associated to the provided key is an object, then you will get a reference to that object and any change made to that object will effectively modify it inside the Map.
     * @returns Returns the element associated with the specified key. If no element is associated with the specified key, undefined is returned.
     */
    get(key: K): V | undefined;
    /**
     * @returns boolean indicating whether an element with the specified key exists or not.
     */
    has(key: K): boolean;
    /**
     * Adds a new element with a specified key and value to the Map. If an element with the same key already exists, the element will be updated.
     */
    set(key: K, value: V): this;
    /**
     * @returns the number of elements in the Map.
     */
    readonly size: number;
}
```

### Map常用方法

#### set向 Map 中添加指定键和值的键值对
`set(key: K, value: V): this` 
```js
let map=new Map();
map.set('name','cupid');
```

#### get 返回指定键对应的值。
`get(key: K): V | undefined`

```js
let map=new Map();
map.set('name','cupid');
map.get('name')   //cupid
```
#### has判断 Map 中是否存在指定键
`has(key: K): boolean`

```js
let map=new Map();
map.set('name','cupid');
map.has('name')   //true
```

#### delete(key): 删除 Map 中指定键的键值对。
`delete(key: K): boolean`
```js
let map=new Map();
map.set('name','cupid');
map.delete('name')   //true
```

#### clear(): 清空 Map，删除所有的键值对。
`clear(): void`
```js
let map=new Map();
map.set('name','cupid');
map.clear(); 
```

#### size 返回 Map 中键值对的数量
`readonly size: number` 
```js
let map=new Map();
map.set('name','cupid');
const size=map.size; //1
```

#### forEach 遍历 Map 中的键值对，并对每个键值对执行指定的回调函数。
`forEach(callbackfn: (value: V, key: K, map: Map<K, V>) => void, thisArg?: any): void`  callbackfn函数会接受三个参数分别代表： 值、键和被遍历的 Map 对象本身
```js
let map=new Map();
map.set('name','cupid');
map.set('age','18');

map.forEach((item ,key )=>{
  console.log(item,key)
})

```

#### keys 返回一个包含 Map 中所有键的迭代器

```js
let myMap = new Map();
myMap.set('key1', 'value1');
myMap.set('key2', 'value2');

let keysIterator = myMap.keys();

for (let key of keysIterator) {
  console.log(key);
}
```

#### values  返回一个包含 Map 中所有值的迭代器。

```js
let myMap = new Map();
myMap.set('key1', 'value1');
myMap.set('key2', 'value2');

let valuesIterator = myMap.values();

for (let value of valuesIterator) {
  console.log(value);
}
```

#### entries(): 返回一个包含 Map 中所有键值对的迭代器

```js
let myMap = new Map();
myMap.set('key1', 'value1');
myMap.set('key2', 'value2');

let entriesIterator = myMap.entries();

for (let [key,value] of entriesIterator) {
  console.log(key,value);
}
```
