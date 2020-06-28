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

