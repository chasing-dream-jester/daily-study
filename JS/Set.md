## Set 数据结构
```ts
interface Set<T> {
    /**
     * Appends a new element with a specified value to the end of the Set.
     */
    add(value: T): this;

    clear(): void;
    /**
     * Removes a specified value from the Set.
     * @returns Returns true if an element in the Set existed and has been removed, or false if the element does not exist.
     */
    delete(value: T): boolean;
    /**
     * Executes a provided function once per each value in the Set object, in insertion order.
     */
    forEach(callbackfn: (value: T, value2: T, set: Set<T>) => void, thisArg?: any): void;
    /**
     * @returns a boolean indicating whether an element with the specified value exists in the Set or not.
     */
    has(value: T): boolean;
    /**
     * @returns the number of (unique) elements in Set.
     */
    readonly size: number;
}
```
### Set常用方法
#### add添加元素
`add(value: T): this;`：向集合中添加一个元素。 
```js
let mySet=new Set();
mySet.add(1);
```
#### delete删除元素
`delete(value: T): boolean;` 向集合中删除一个元素,如果删除成功，返回true，否则返回false。 
```js
mySet.delete(2);
```
#### delete删除元素
`clear(): void;` 向集合中删除所有元素。 
```js
mySet.clear();
```
#### has检查元素
`has(value: T): boolean;`检查集合中是否包含某个元素，如果有返回true，否则返回false
```js
mySet.has(1)
```

#### forEach遍历集合
`forEach(callbackfn: (value: T, value2: T, set: Set<T>) => void, thisArg?: any): void;`遍历集合

```js
mySet.forEach(element => {
  console.log(element);
});
```
