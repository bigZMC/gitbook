## Iterator的概念

`JavaScript`中有4中表示`"集合"`的数据结构，分别是`Array`，`object`，`Map`和`Set`。`Iterator`(遍历器)就是一种统一的接口机制，来处理所有不同的数据结构。

任何数据结构只要部署`Iterator`接口，就可以完成遍历操作。每次调用`next`方法，都会返回数据结构的当前成员的信息对象，包含`value`和`done`两个属性

```javascript
function makeIterator(arr) {
  var nextIndex = 0
  return {
    next() {
      return nextIndex < arr.length?
        { value: arr[nextIndex++], done: false }:
      	{ value: undefined, done: true }
    }
  }
}

let it = makeIterator(['a', 'b'])

it.next() // { value: 'a', done: false }
it.next() // { value: 'b', done: false }
it.next() // { value: undefined, done: true }
```

`makeIterator`函数，就是一个遍历器生成函数，返回一个遍历器对象。对数组`['a', 'b']`执行这个函数，**返回的就是该数组的遍历器对象**(即指针对象)`it`。

指针对象的`next`方法，用来移动指针。开始时，指针指向数组的开始位置，每次调用`next`方法，指针就会指向数组的下一个成员。

由于`Iterator`只是把接口规格加到数据结构上，所以，**遍历器与它所遍历的数据结构，实际上是分开的**。

```javascript
function idMaker() {
  var index = 0;
  return {
    next() {
      return { value: index++, done: false }
    }
  }
}

let it = idMaker() // 注意,这里没有任何数据结构

it.next().value // 0
it.next().value // 1
it.next().value // 2
// ...
```

如果用`TypeScript`来描述

```typescript
interface Iterable {
  [Symbol.iterator](): Iterator
}

interface Iterator {
  next(value?: any): IterationResult
}

interface IterationResult {
  value: any,
  done: boolean
}
```

---

## 默认Iterator接口

`ES6`规定，默认的`Iterator`接口部署在数据结构的`Symbol.iterator`属性，或者说，**一个数据结构只要具有`Symbol.iterator`属性，就可以认为是`"可遍历的"(iterable)`**。注意，`Symbol.iterator`方法返回的不是遍历器生成函数，会报错。

```javascript
const obj = {
  [Symbol.iterator]() {
    return {
      next() {
        return {
          value: 1,
          done: true
        }
      }
    }
  }
}

let it = obj[Symbol.iterator]()
// 执行[Symbol.iterator]方法,返回遍历器对象

it.next() // { value: 1, done: true }
```

```javascript
var obj = {}

obj[Symbol.iterator] = () => 1

[...obj] // TypeError: [] is not a function
```

`ES6`有些数据结构原生具备了`Iterator`接口，不用做任何处理，就能被`for...of`循环遍历。这些数据结构原生部署了`Symbol.iterator`属性，包括以下几种

- `Array`
- `Map`
- `Set`
- `String`
- 函数的`arguments`对象

```javascript
let arr = ['a', 'b', 'c']
let iter = arr[Symbol.iterator]()
// 数组原生具有遍历器接口,只需要调用Symbol.iterator,就能返回遍历器对象

iter.next() // { value: 'a', done: false }
iter.next() // { value: 'b', done: false }
iter.next() // { value: 'c', done: false }
iter.next() // { value: undefined, done: true }
```

而其他数据结构的对象，想要被`for...of`循环调用，就必须在`Symbol.iterator`属性上部署遍历器生成方法

```javascript
class RangeIterator {
  constructor(start, stop) {
    this.value = start
    this.stop = stop
  }
  
  [Symbol.iterator]() { return this }
  // 返回实例本身,调用next方法
  
  next() {
    if(this.value < this.stop) {
      return { value: this.value++, done: false }
    }
    return { value: undefined, done: true }
  }
}

function range(start, stop) {
  return new RangeIterator(start, stop)
}

for(let value of range(0, 3)) {
  console.log(value)
}
// 0
// 1
// 2
```

下面是一个通过遍历器实现指针结构的例子

```javascript
function Obj(value) {
  this.value = value
  this.next = null
}

Obj.prototype[Symbol.iterator] = function() {
  let current = this // 返回对象实例
  
  return {
    next: function() {
      if(current) { // 如果当前对象存在
        let value = current.value
        current = current.next
        return { value, done: false }
      }
      return { value: undefined, done: false }
    }
  }
}

let one = new Obj(1)
let two = new Obj(2)
let three = new Obj(3)

one.next = two // 给one赋值next,对应代码第13行
two.next = three

for(let i of one) {
  console.log(i)
}
// 1
// 2
// 3
```

对于伪数组对象(存在**数值键名**和`length`属性)，部署`Iterator`接口，可以将`Symbol.Iterator`方法直接引用数组的`Iterator`接口

```javascript
let iterable = {
  0: 'a',
  1: 'b',
  2: 'c',
  length: 3,
  [Symbol.iterator]: Array.prototype[Symbol.iterator]
}

for(let value of iterable) {
  console.log(value)
}
// 'a'
// 'b'
// 'c'
```

注意，**伪数组的键名一定要是数值类型**，普通对象包含`length`属性直接部署`Array.prototype[Symbol.iterator]`是不生效的，这也是为什么`for...of`不会遍历非数值键名的成员

```javascript
let iterable = {
  a: 'a',
  b: 'b',
  c: 'c',
  length: 3,
  [Symbol.iterator]: Array.prototype[Symbol.iterator]
}

for (let value of iterable) {
  console.log(value) // undefined, undefined, undefined
}
```

 在实际应用中，有了遍历器接口，就可以使用`for...of`或者`while`

```javascript
let $iterator = ITERABLE[Symbol.iterator]()
let $result = $iterator.next()

white(!$result.done) {
  let value = $result.value
  $result = $iterator.next()
}
```

---

## 字符串的Iterator接口

字符串是一个伪数组对象，也**原生具有`Iterator`接口**

```javascript
let str = 'hi'
typeof str[Symbol.iterator] // 'function'

let iterator = str[Symbol.iterator]()

iterator.next()  // { value: 'h', done: false }
iterator.next()  // { value: 'i', done: false }
iterator.next()  // { value: undefined, done: true }
```

我们也可以覆盖原生的`Symbol.iterator`方法

```javascript
let str = 'hi'
[...str] // ['h', 'i']

str[Symbol.iterator] = function() {
  return {
    _fisrt: true,
    next() {
      if(this._first) {
        this._first = false
        return { value: 'bye', done: false }
      }
      return { done: true }
    }
  }
}

[...str] // ['bye']
str // 'hi',字符串本身没有改变
```

---

## Iterator接口与Generator函数

`Symbol.iterator`方法几乎不用部署任何代码，只要用`yield`命令给出返回值即可

```javascript
let myIterable = {
  [Symbol.iterator]: function*() {
    yield 1
    yield* [2,3,4]
  	yield 5
  }
}

[...myIterable] // [1, 2, 3, 4, 5]

let obj = {
  *[Symbol.iterator]() {
    yield 'hello'
    yield 'world'
  }
}

for(let x of obj) {
  console.log(x)
}
// 'hello'
// 'world'
```

---

## 遍历器对象的return()方法

遍历器对象除了具有`next`方法，还可以具有`return`方法。`next`方法必须部署，`return `可选

`return`方法的使用场景是，如果`for...of`循环提前退出，就会调用`return`方法。注意，**`return`方法必须返回一个对象**，这是由`Generator`规格决定的。

```javascript
function readLinesSync(file) {
  return {
    [Symbol.iterator]() {
      return {
        next() {
          return { done: false }
        },
        return() {
          file.close()
          return { done: true }
        }
      }
    }
  }
}
```

以下两种情况会触发`return`方法

```javascript
// 情况一,break
for(let line of readLinesSync(file)) {
  console.log(line)
  break
}

// 情况二,抛出异常
for (let line of readLinesSync(file)) {
  console.log(line)
  throw new Error()
}
```

---

## for...of 循环

`for...of`循环内部调用的是数据结构的`Symbol.iterator`方法，也就是说具有`Symbol.iterator`属性的数据结构，都能被`for...of`遍历。

### 数组

可以用`for...of`来代替数组的`forEach`方法

```javascript
const arr = ['red', 'green', 'blue']

for(let value of arr) {
  console.log(value)
}
// 'red'
// 'green'
// 'blue'
```

区别于`for...in`，智能获取键名，`for...of`允许遍历获得键值，如果需要同时获取索引的话，可以借助数组的`entries`方法。注意，**`for...of`只返回具有数字索引的属性**

```javascript
const arr = ['a', 'b', 'c', 'd']
arr.foo = 'hello'

for(let i in arr) {
  console.log(i) // 0 1 2 3 'foo'
}

for(let [index, value] of arr.entries()) {
  console.log(index, value)
}
// 0 'a'
// 1 'b'
// 2 'c'
// 3 'd'
// 注意,这里没有遍历到键名为'foo'的成员,因为其不是数值键值
```

### Set和Map结构

```javascript
let engines = new Set(['Gecko', 'Trident', 'Webkit', 'Webkit'])

for (let e of engines) {
  console.log(e)
}
// Gecko
// Trident
// Webkit

let map = new Map()
map.set('edition', 6)
	 .set('committee', 'TC39')
	 .set('standard', 'ECMA-262')

for (let [name, value] of map) {
  console.log(name + ':  '+ value)
}
// edition: 6
// committee: TC39
// standard: ECMA-262
```

### 类似数组的对象

如果伪数组对象没有实现`Iterator`接口，可以通过`Array.from`方法将其转换成数组

```javascript
let arrayLike = { length: 2, 0: 'a', 1: 'b' }

for(let x of arrayLike) {
  console.log(x)
}
// 没有实现Iterator接口,报错

for (let x of Array.from(arrayLike)) {
  console.log(x)
}
// 通过Array.from()转成数组后循环
```

### 与其他遍历语法的比较

目前常用的遍历语法有以下四种

1. `for`循环

最简单，但是语法比较麻烦

```javascript
for(let index = 0;index < arr.length;index++) {}
```

2. `forEach`

无法中途跳出循环，`break`或`return`无效

```javascript
arr.forEach(value => {})
```

3. `for...in`

```javascript
for(let index in arr) {}
```

`for...in`可以遍历数组键名，但是有几个缺点

- 数组键名是数字，但是遍历出来的仍然是字符串`'0'`，`'1'`等
- 不仅会遍历数字键名，还会遍历手动添加的其他键，包括原型链上的键
- 遍历顺序不可控

4. `for...of`

```javascript
for(let index of arr) {}
```

- 可以和`break`，`return`，`continue`配合使用
- 可以遍历任何实现了`Iterator`接口的数据结构