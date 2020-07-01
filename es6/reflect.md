## 概述

`Reflect`对象是`ES6`为了操作对象而提供的新`API`，设计目的如下

- 将`Object`对象的一些明显属于语言内部的方法，放在`Reflect`对象上
- 修改某些`Object`方法的返回结果，让其变得更合理。比如，`Object`对象在定义属性时，如果不成会抛出一个错误，而用`Reflect`则会返回`false`

```javascript
// Object
try {
  Object.defineProperty(target, property, attributes)
} catch(e) {
  // do sth
}

// Reflect
if(Reflect.defineProperty(target, property, attributes)) {
  
}
```

- 让`Object`操作都变成函数行为。目前有些`Object`操作是命令式的，比如`name in obj`和`delete obj[name]`，现在可以使用`Reflect.has(obj, name)`和`Reflect.deleteProperty(obj, name)`来代替
- `Reflect`对象的方法和`Proxy`对象的方法一一对应，可以在`Proxy`对象拦截器中调用对应的`Reflect`方法

```javascript
var loggedObj = new Proxy(obj, {
  get(target, name) {
    console.log('get', target, name)
    return Reflect.get(target, name)
  },
  deleteProperty(target, name) {
    console.log('delete' + name)
    return Reflect.deleteProperty(target, name)
  },
  has(target, name) {
    console.log('has' + name)
    return Reflect.has(target, name)
  }
})
```

上面代码中，每个`Proxy`对象的拦截操作，内部都调用对应的`Reflect`方法，保证原生行为能正常执行

---

## 静态方法

**注意，所有静态方法的第一个参数都必须是对象，否则会报错**

---

### Reflect.get(target, name, receiver)

`Reflect.get`方法查找并返回`target`对象的`name`属性，如果没有该属性，则返回`undefined`

```javascript
var myObject = {
  foo: 1,
  bar: 2,
  get baz() {
    return this.foo + this.bar
  }
}

Reflect.get(myObject, 'foo') // 1
Reflect.get(myObject, 'bar') // 2
Reflect.get(myObject, 'baz') // 3
```

如果`name`属性部署了读取函数(`getter`)，则读取函数的`this`绑定`receiver`(如果有的话)

```javascript
var myReceiverObject = {
  foo: 4,
  bar: 4
}

Reflect.get(myObject, 'bar', myReceiverObject) // 2
// myObject的bar属性没有getter函数,this绑定的就是target对象myObject
Reflect.get(myObject, 'baz', myReceiverObject) // 8
// 这里this绑定的是myReceiverObject,相当于myReceiverObject.foo + myReceiverObject.bar
```

---

### Reflect.set(target, name, value, receiver)

`Reflect.set`方法设置`target`对象的`name`属性等于`value`

```javascript
var myObject = {
  foo: 1,
  set bar(value) {
    return this.foo = value
  }
}

myObject.foo // 1

Reflect.set(myObject, 'foo', 2)
myObject.foo // 2

Reflect.set(myObject, 'bar', 4) // 触发bar属性的setter函数
myObject.foo // 4
```

如果`name`属性设置了赋值函数，则赋值函数的`this`绑定`receiver`(如果有的话)

```javascript
var myObject = {
  foo: 1,
  set bar(value) {
    return this.foo = value
  }
}

var myReceiverObject = {
  foo: 0,
}

Reflect.set(myObject, 'foo', 4, myReceiverObject) // 'foo'属性没有赋值函数setter
myObject.foo // 1
myReceiverObject.foo // 4,传入receiver,那么就会修改receiver的对象属性

Reflect.set(myObject, 'bar', 8, myReceiverObject)
// this绑定到myReceiverObject,相当于myReceiverObject.foo = 8
myObject.foo // 1
myReceiverObject.foo // 8
```

还需要注意一点，**一旦传入`receiver`，就会将属性赋值到`receiver`上面**，这一点在上面例子的`14`行代码中的到印证。下面还有个例子，也是因为这个原因导致的。

注意，如果`Proxy`和`Reflect`联合使用，前者拦截赋值操作，后者完成赋值的默认行为，**并且传入`receiver`**(如果不传入则不会触发`Proxy.defineProperty`)，那么`Reflect.set`会触发`Proxy.defineProperty`

```javascript
let p = {
  a: 'a'
}

let handler = {
  set(target, key, value, receiver) {
    console.log('set')
    Reflect.set(target, key, value, receiver)
  },
  defineProperty(target, value, attribute) {
    console.log('defineProperty')
    Reflect.defineProperty(target, value, attribute)
  }
}

let obj = new Proxy(p, handler)
obj.a = 'A'
// 'set'
// 'defineProperty'
```

上面已经说到，**`Reflect.set`方法传入`receiver`对象，就会修改`receiver`对象的属性**。这里的`receiver`就是`Proxy`对象`obj`(`target`是原始对象`p`)，因此第`8`行代码的`Reflect.set`方法修改的就是`obj`。因此再次进入`defineProperty`的拦截中。

---

### Reflect.has(obj, name)

`Reflect.has`方法对应`in`运算符

```javascript
var myObject = {
	foo: 1
}

// 旧写法
'foo' in myObject // true

// Reflect
Reflect.has(myObject, 'foo') // true
```

---

### Reflect.deleteProperty(obj, name)

`Reflect.deleteProperty`方法对应`delete`操作符

```javascript
const myObj = { 
  foo: 'bar' 
}

// 旧写法
delete myObj.foo

// Reflect
Reflect.deleteProperty(myObj, 'foo')
```

返回布尔值，如果删除成功，或者被删除的属性不存在，返回`true`。删除失败，或被删除的属性依然存在，返回`false`

---

### Reflect.defineProperty(target, propKey, descriptor)

`Reflect.defineProperty`方法基本等同于`Object.defineProperty`，可以与`Proxy`对象的`defineProperty`方法配合使用

```javascript
const p = new Proxy({}, {
  defineProperty(target, propKey, descriptor) {
    console.log(descriptor)
    return Reflect.defineProperty(target, propKey, descriptor)
  }
})

p.foo = 'bar'
// {value: 'bar', writable: true, enumerable: true, configurable: true}

p.foo // 'bar'
```

---

### Reflect.ownKeys(target)

`Reflect.ownKeys`方法用于返回对象的所有属性，基本等同于`Object.getOwnPropertyNames`与`Object.getOwnPropertySymbols`之和

```javascript
var myObject = {
  foo: 1,
  bar: 2,
  [Symbol.for('baz')]: 3,
  [Symbol.for('bing')]: 4,
}

// 旧写法
Object.getOwnPropertyNames(myObject)
// ['foo', 'bar']

Object.getOwnPropertySymbols(myObject)
//[Symbol(baz), Symbol(bing)]

// 新写法
Reflect.ownKeys(myObject)
// ['foo', 'bar', Symbol(baz), Symbol(bing)]
```

---

## 使用Proxy实现观察者模式

```javascript
const queuedObservers = new Set() // 使用Set保存观察者函数

const observe = fn => queuedObservers.add(fn) // 添加一个观察者函数
const observable = obj => new Proxy(obj, {set}) // 返回元对象的代理

// 拦截代理对象set方法,通过Reflect.set返回结果并批量执行观察者函数
function set(target, key, value, receiver) {
  const result = Reflect.set(target, key, value, receiver)
  queuedObservers.forEach(observer => observer())
  return result
}

// proxy对象
const person = observable({
  name: '张三',
  age: 20
})

function print() {
  console.log(`${person.name}, ${person.age}`)
}

observe(print)
person.name = '李四'
// '李四', 20
```

