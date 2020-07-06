## Set

### 基本用法

`Set`是`ES6`提供的新的**数据结构**，类似于数组，但是成员的值都是唯一的，没有重复的值。

`Set`本身是一个构造函数，用于生成`Set`数据结构

```javascript
const s = new Set()

[2, 3, 5, 4, 5, 2, 2].forEach(x => s.add(x))

for(let i of s) {
  console.log(i)
}

// 2 3 5 4
```

`Set`提供的实例具有`add()`方法，用于向`Set`结构加入成员

`Set`函数可以接受一个数组（或者具有`iterable`接口的其他数据结构）作为参数，用来初始化

```javascript
const set = new Set([1, 2, 3, 4, 5])
[...set]
// [1, 2, 3, 4]

const items = new Set([1, 2, 3, 4, 5, 5, 5, 5])
items.size // 获取set对象的长度,Set对象不具有length属性
// 5
```

可以运用`Set`成员不重复的特性，进行去重操作

```javascript
[...new Set(array)] // 去除数组的重复成员
Array.from(new Set(array)) // 和扩展运算符效果相同
[...new Set('ababbc')].join('') //去除字符串的重复字符
```

注意，`Set`添加值的时候，不会进行类型转换，所有`5`和`'5'`是两个不同的值。内部判断算法类似于全等运算符(`===`)，区别在于，全等运算符判断`NaN`时，认为不相等

```javascript
NaN === NaN // false
```

但是在`Set`内部判断中，这两个值相等

```javascript
let set = new Set()
let a = NaN
let b = NaN
set.add(a)
set.add(b)
set // Set(1) {NaN}
set.size // 1
```

另外，两个对象总不相等

```javascript
let set = new Set()

set.add({})
set.size // 1

set.add({})
set.size // 2
```

### Set实例的属性和方法

**属性**

- `Set.prototype.constructor`：构造函数，默认就是`Set`函数
- `Set.prototype.size`：返回`Set`实例的成员总数

**操作方法**

- `Set.prototype.add(value)`：添加某个值，返回`Set`结构本身
- `Set.prototype.delete(value)`：删除某个值，返回布尔值，表示删除是否成功
- `Set.prototype.has(value)`：判断一个值是否为`Set`的成员，返回布尔值
- `Set.prototype.clear()`：清空所有成员，没有返回值

```javascript
let s = new Set()

s.add(1).add(2).add(2) // add()返回Set结构本身,因此可以链式调用

s.has(1) // true
s.has(2) // true
s.has(3) // false

s.delete(2) // true
s.has(2) // false

s.delete(3) // false 删除不存在的成员,返回false

s.clear()
s // Set(0) {}
```

`Object`和`Set`结构在判断是否包含某一个键时，写法如下

```javascript
// object
const property = {
  'width': 1,
  'height': 1
}

if(property[key]) {
  // do sth
}

// set
const property = new Set()

property.add('width')
property.add('height')

if(property.has(key)) {
  // do sth
}
```

**遍历操作**

- `Set.property.keys()`：返回键名的遍历器
- `Set.property.values()`：返回键值的遍历器
- `Set.property.entries()`：返回键值对的遍历器
- `Set.property.forEach()`：使用回调函数遍历每个成员

需要注意，**`Set`的遍历顺序就是插入顺序**。这个特性在有时候能确保遍历顺序的正确性

1. `keys()`，`values()`，`entries()`

这三个方法返回的都是遍历器对象。**由于`Set`结构没有键名，只有键值（或者说键名和键值是同一个值）**，所以`keys()`和`values()`方法的行为完全一致

```javascript
let set = new Set(['red', 'green', 'blue'])

for(let item of set.keys()) {
  console.log(item)
}
// red
// green
// blue

// keys()和values()行为一致
for(let item of set.values()) {
  console.log(item)
}
// red
// green
// blue

// entries()返回的遍历器，同时包括了键名和键值
for(let item of set.entries()) {
  console.log(item)
}
// ['red', 'red']
// ['green', 'green']
// ['blue', 'blue']
```

`Set`结构的实例默认可遍历，它的默认遍历器生成函数就是`values()`方法。因此在遍历时，**可以省略`values()`方法，直接用`for...of`进行遍历**。

```javascript
Set.prototype[Symbol.iterator] === Set.prototype.values // true

for(let x of set) {
  console.log(x)
}
// red
// green
// blue
```

2. `forEach()`

   `forEach()`的参数与数组的`forEach()`一致，依次为键值、键名、集合本身 。上面提到过，**`Set`结构的键名和键值是同一个值**，因此第一个和第二个参数永远相同。另外，和数组的`forEach()`一致，也有第二个参数，表示绑定处理函数内部的`this`对象。

```javascript
let set = new Set([1, 4, 9])

set.forEach((value, key) => console.log(`${key} => ${value}`))
```

3. 遍历的应用

扩展运算符(`...`)内部使用`for...of`循环，所以也可以用在`Set`结构。并且，数组的`map`和`filter`方法也可以间接用于`Set`

```javascript
let set = new Set([1, 2, 3])
set = new Set([...set].map(x => x * 2))
// Set(3) {2, 4, 6}

let set = new Set([1, 2, 3, 4, 5])
set = new Set([...set].filter(x => (x % 2) === 0))
// Set(2) {2, 4}
```

`Set`还能实现并集，交集和差集

```javascript
let a = new Set([1, 2, 3])
let b = new Set([4, 3, 2])

// 并集
let union = new Set([...a, ...b])
// Set {1, 2, 3, 4}

// 交集
let intersect = new Set([...a].filter(x => b.has(x)))
// Set {2, 3}

// 差集
let difference = new Set([...a].filter(x => !b.has(x)))
// Set {1}
```

如果想在遍历操作中，同步修改原来的`Set`结构，目前没有直接的方法，只有两个变通方法

1. 利用原`Set`结构映射出一个新的结构，赋值给原有的实例
2. 利用`Array.from`方法

```javascript
let set = new Set([1, 2, 3])
set = new Set([...set].map(x => x * 2))

let set = new Set([1, 2, 3])
set = new Set(Array.from(set, x => x * 2))
```

这两种方法本质都是在遍历后赋值给原有实例进行覆盖操作

---

## WeakSet

`WeakSet`结构和`Set`类型，也是不重复的值的集合。但是它与`Set`有两个区别

1. **`WeakSet`的成员只能是对象**，而不能是其他类型的值

```javascript
const ws = new WeakSet()
ws.add(1)
// TypeError: Invalid value used in weak set

ws.add(Symbol())
// TypeError: Invalid value used in weak set
```

2. `WeakSet`中的对象都是`弱引用`，**即垃圾回收机制不考虑`WeakSet`对该对象的引用**，也就是说，如果其他对象都不再引用该对象，那么垃圾回收机制会自动回收该对象所占用的内存，不考虑对象还在存于`WeakSet`中。因此，**`WeakSet`适合临时存放一组对象，以及存放跟对象绑定的信息。**

由于上述特点，`WeakSet`内部有多少个成员，取决于垃圾回收机制有没有运行，运行前后很可能成员个数是不一致的，而垃圾回收机制何时运行是不可预测的，因此**`WeakSet`不可遍历**。

```javascript
// 语法
const ws = new WeatSet()

// 和Set一致,可以接受一个数组或者伪数组的对象作为参数
// 或者具有iterable接口的其他数据结构
const a = [[1, 2], [3, 4]]
const ws = new WeakSet(a)
ws // WeakSet {[1, 2], [3, 4]}
```

注意，这里`a`是一个数组，它有两个成员，也都是数组。**将`a`作为`WeakSet`构造函数的参数，`a`的成员会成为`WeatSet`的成员，而不是`a`本身作为`WeakSet`成员**。因为`WeakSet`的成员只能是对象，所以`a`的成员也必须是对象。

```javascript
const b = [3, 4]
const ws = new WeatSet(b)
// TypeError: Invalid value used in weak set
```

**操作方法**

- `WeakSet.prototype.add(value)`：向`WeakSet`实例添加一个新成员
- `WeakSet.prototype.delete(value)`：清除`WeakSet`实例的指定成员
- `WeakSet.prototype.has(value)`：返回一个布尔值，表示某个值是否在`WeakSet`实例之中

```javascript
const ws = new WeatSet()
const obj = {}
const foo = {}

ws.add(window)
ws.add(obj)

ws.has(window) // true
ws.has(foo) // false

ws.delete(window)
ws.has(window) // false
```

注意，`WeakSet`不能确定成员个数，因此**不能进行遍历，也没有`size`属性**

```javascript
ws.size // undefined
ws.forEach // undefined
```

`WeakSet`有一个用处，**存储`DOM`节点，而不用担心这些节点从文档移除时，会引发内存泄漏。**

---

## Map

### 基本用法

`Object`本质是键值对的集合，但是键只能是字符串或`Symbol`，在使用上有很大的限制

```javascript
const data = {}
const element = document.getElementById('myDiv')

data[element] = 'metadata'
data['[object HTMLDivElement]'] 
// 'metadata',对象element被自动转换成字符串'[object HTMLDivElement]'
```

为了解决这个问题，**`Map`作为键值对的集合，但是`键`的范围不限于字符串，各种类型的值(包括对象)都可以当作键**。

```javascript
const m = new Map()
const o = {p: 'Hello World'}

m.set(o, 'content')
m.get(o) // content

m.has(o) // true
m.delete(o) // true
m.has(o) // false
```

同样，`Map`也可以**接受一个数组作为参数，该数组的成员是一个个表示键值对的数组**

```javascript
const map = new Map([
  ['name', '张三'],
  ['title', 'Author']
])

map.size // 2
map.has('name') // true
map.get('name') // '张三'
map.has('title') // true
map.get('title') // 'Author'
```

当`Map`构造函数接受数组作为参数时，在`Map`构造函数内部，执行的是下面的算法

```javascript
const items = [
  ['name', '张三'],
  ['title', 'Author']
]

const map = new Map()

items.forEach(([key, value]) => map.set(key, value))
```

实际上，**任何具有`Iterator`接口，且每个成员都是一个双元素的数据结构**，都可以当作`Map`构造函数的参数。

```javascript
// 使用set作为参数
const set = new Set([
  ['foo', 1],
  ['bar', 2]
])
const m1 = new Map(set)
m1.get('foo') // 1

// 使用map作为参数
const m2 = new Map([
  ['baz', 3]
])
const m3 = new Map(m2)
m3.get('baz') // 3
```

如果对同一个键多次赋值，后面的值会覆盖前面的值

```javascript
const map = new Map()

map.set(1, 'aaa').set(1, 'bbb')

map.get(1) // 'bbb'
```

如果读取的键是未知的，则返回`undefined`

```javascript
new Map().get('abc') // undefined
```

需要注意的是，**只有对同一个对象的引用，`Map`结构才将其视为同一个键。**

```javascript
const map = new Map()

map.set(['a'], 555)
map.get(['a']) // undefined,赋值和或者的['a']是不同的对象,内存地址不同

// 那么,同样值的两个实例被看做两个不同的键
const map = new Map()

const k1 = ['a']
const k2 = ['a']

map.set(k1, 111).set(k2, 222)

map.get(k1) // 111
map.get(k2) // 222
```

如果`Map`的键是一个简单类型的值，则只要两个值严格相等(`===`)，`Map`则将其视为一个键。和`Set`相同，`NaN`也被认为是同一个键。

```javascript
let map = new Map()

// 0和-0被认为是同一个键
map.set(-0, 123)
map.get(0) // 123

map.set(true, 1)
map.set('true', 2)
map.get(true) // 1

// undefined和null也是两个不同的键
map.set(undefined, 3)
map.set(null, 4)
map.get(undefined) // 3

map.set(NaN, 5).set(NaN, 6)
map.get(NaN) // 6
```

### Map实例的属性和方法

**属性**

- `Map.prototype.size`：返回`Map`实例的成员总数

**操作方法**

- `Map.prototype.set(key, value)`：设置键名`key`对应的键值`value`，返回整个`Map`结构，如果`key`已经有值，则键值会被更新。因为**返回的是当前`Map`实例，所以可以使用链式写法，这一点和`Set`一致**。
- `Map.prototype.get(key)`：获取`key`对应的键值，如果找不到返回`undefined`
- `Map.prototype.has(key)`：返回一个布尔值，表示某个键是否在`Map`对象中
- `Map.prototype.delete(key)`：返回布尔值，表示是否删除成功某个键
- `Map.prototype.clear()`：清除所有成员

```javascript
const map = new Map()

map.set('foo', true)
map.set('bar', false)

map.size // 2

map.get('foo') // true

map.has('foo') // true
map.delete('foo') // true
map.has('foo') // false

map.clear()
map.size // 0
```

**遍历方法**

- `Map.prototype.keys()`：返回键名的遍历器
- `Map.prototype.values()`：返回键值的遍历器
- `Map.prototype.entries()`：返回所有成员的遍历器
- `Map.prototype.forEach()`：遍历`Map`的所有成员

和`Set`相同，**遍历顺序就是插入顺序**

```javascript
const map = new Map([
  ['F', 'no'],
  ['T', 'yes']
])

for(let key of map.keys()){
  console.log(key)
}
// 'F'
// 'T'

for(let value of map.values()){
  console.log(value)
}
// 'no'
// 'yes'

for(let item of map.entries()){
  console.log(item[0], item[1])
}
// 'F' 'no'
// 'T' 'yes'

for(let [key, value] of map.entries()){
  console.log(key, value)
}
// 'F' 'no'
// 'T' 'yes'
```

由于`Map`结构的默认遍历器接口，就是`entries()`方法，因此直接使用`for...of`等同于调用`entries()`方法

```javascript
Map.prototype[Symbol.iterator] === Map.prototype.entries

for(let [key, value] of map){
  console.log(key, value)
}
// 'F' 'no'
// 'T' 'yes'
```

`Map`还有一个`forEach`方法，和数组的`forEach`方法类似，同样可以接收第二参数，作为执行函数的`this`绑定

```javascript
map.forEach((value, key, map) => {
  // do sth
}, this)
```

**与其他数据结构的互相转换**

1. `Map`转为数组

最简单的办法就是使用扩展运算符(`...`)

```javascript
const map = new Map([
  [1, 'one'],
  [2, 'two'],
  [3, 'three'],
])

[...map.keys()]
// [1, 2, 3]

[...map.values()]
// ['one', 'two', 'three']

[...map.entries()]
[...map]
// [[1, 'one'], [2, 'two'], [3, 'three']]
```

再结合数组的`map`和`filter`方法，可以实现对`Map`的遍历和过滤

```javascript
const map1 = new Map([...map].filter(([key, value]) => key < 3))
// [[1, 'one'], [2, 'two']]

const map2 = new Map([...map].filter(([key, value]) => [key * 2, '_' + value]))
// [[2, '_one'], [4, '_two'], [6, '_three']]
```

2. 数组转为`Map`

将数组作为参数传入`Map`构造函数即可，数组的成员必须是一个个表示键值对的数组

```javascript
new Map([
  [true, 7],
  [{foo: 3}, ['abc']]
])
// Map {
//   true => 7,
//   Object {foo: 3} => ['abc']
// }
```

3. `Map`转为对象

如果`Map`的键都是字符串或者`Symbol`，可以无损的转为对象

```javascript
function strMapToObj(strMap) {
  let obj = Object.create(null)
  for(let [key, value] of strMap){
    obj[key] = value
  }
  return obj
}

const myMap = new Map()
	.set('yes', true)
	.set('no', false)
strMapToObj(myMap)
// { yes: true, no: false }
```

如果有非字符串或非`Symbol`的键名，**这个键名会被转为字符串，再作为对象的键名**

4. 对象转为`Map`

对象通过`Object.entries()`返回值作为`Map`构造函数的参数

```javascript
let obj = {'a': 1, 'b': 2}
let map = new Map(Object.entries(obj))
```

也可以自行实现

```javascript
function objToStrMap(obj) {
	let strMap = new Map()
  for(let key of Object.keys(obj)){
    strMap.set(key, obj[key])
  }
  return strMap
}
```

---

## WeakMap

`WeakMap`结构和`Map`结构类似，也是用于生成键值对的集合。它们之间有两点差别

1. `WeakMap`只接受对象作为键名(`null`除外)

```javascript
const wm = new WeakMap()
wm.set(1, 2)
// TypeError: 1 is not an object

wm.set(Symbol(), 2)
// TypeError: Invalid value used as weak map key

wm.set(null, 2)
// TypeError: Invalid value used as weak map key
```

2. `WeakMap`的**键名所指向的对象**，不计入垃圾回收机制

`WeakMap`的**键名所引用的对象都是弱引用**，这一点和`WeakSet`的值是弱引用相同，垃圾回收机制不将该引用考虑在内。只要所引用的对象的其他引用被清除，垃圾回收机制就会释放该对象所占用的内存。

因为`弱引用`的这个特点，`WeakMap`的使用场景是，它的键所对应的对象，可能会在将来消失。`WeakMap`结构有助于防止内存泄漏。

注意，**`WeakMap`弱引用的只有键名，而不是键值**。

```javascript
const wm = new WeakMap()
let key = {}
let obj = {foo: 1}

wm.set(key, obj)
obj = null
wm.get(key)
// Object {foo: 1}
// 键值obj是正常引用,即使外部消除了obj的引用,WeakMap内部引用依然存在
```

**操作方法**

和`WeakSet`相同，由于弱引用的关系，键名是否存在不可预测。因此，没有遍历操作，也没有`size`属性。可用的方法只有`get()`，`set()`，`has()`，`delete()`，使用方法和`Map`相同

```javascript
const wm = new WeakMap()

wm.size // undefined
wm.forEach // undefined
wm.clear // undefined
```

**用途**

1. 将`DOM`节点作为键名，一旦`DOM`节点被删除，对应的键就会被删除，不存在内存泄漏

```javascript
const wm = new WeakMap()

wm.set(
	document.getElementById('logo'),
  {timesClicked: 0}
)

document.getElementById('logo').addEventListener('click', function(){
  let logoData = wm.get(document.getElementById('logo'))
  logoData.timesClicked++
}, false)
```

2. 部署私有属性

```javascript
const _counter = new WeakMap()
const _action = new WeakMap()

class Countdown {
  constructor(counter, action) {
    _counter.set(this, counter)
    _action.set(this, action)
  },
  dec() {
    let counter = _counter.get(this)
    if(counter < 1) return
    _counter.set(this, --counter)
    if(counter === 0) {
      _aciton.get(this)()
    }
  }
}

const c = new Countdown(2, () => console.log('DONE'))

c.dec()
c.dec() // 'DONE'
```

`_counter`和`_action`是实例的弱引用，如果删除实例，键也会随之消失，不会造成内存泄漏