## 概述

`Proxy`用于修改某些操作的默认行为，等同于在语言层面做出修改，属于一种"元编程(`meta programming`)"

`Proxy`可以理解为，在**目标对象**之前架设一层"拦截"，外层对该对象的访问，都必须先通过这层拦截，可以对外界的访问进行**过滤和改写**。

```javascript
var obj = new Proxy({}, {
  get: function(target, propKey, receiver) {
    console.log(`getting ${propKey}!`)
    return Reflect.get(target, propKey, receiver)
  },
  set: function(target, propKey, value, receiver) {
    console.log(`setting ${propKey}!`)
    return Reflect.set(target, propKey, value, receiver)
  }
})

obj.count = 1
// getting count!

++obj.count
// getting count!
// setting count!
// 2
```

`Proxy`实际上**重载(`overload`)了点运算符**，即用自己的定义覆盖了语言的原始定义

### 语法

```javascript
var proxy = new Proxy(target, handler)
```

- `target`：表示所要拦截的目标对象
- `handler`：也是一个对象，用来定义拦截行为

```javascript
var proxy = new Proxy({}, {
  get: function(target, propKey) {
    return 35;
  }
});

proxy.time // 35
proxy.name // 35
proxy.title // 35
```

注意，要是的`Proxy`起作用，**必须针对`Proxy`实例进行操作**，而不是针对目标对象`target`

如果`handler`没有设置任何拦截，那就等于直接通向原对象

```javascript
var target = {}
var proxy = new Proxy(target, {})

proxy.a = 'b'
target.a // 'b'
```

还可以将`Proxy`对象，设置到`object.proxy`属性，从而可以在`object`对象上调用

```javascript
var object = { proxy: new Proxy(target, handler) }
```

`Proxy`实例还能作为其他对象的原型对象，通过`Object.create`实现

```javascript
var proxy = new Proxy({}, {
  get: function(target, propKey){
    return 35
  }
})

let obj = Object.create(proxy)
obj.time // 35
```

---

## Proxy实例方法

### get()

`get`方法用于拦截某个属性的读取操作，接受三个参数，依次为目标对象，属性名和`proxy`实例本身 (严格地说，**是操作行为所针对的对象**) ，最后一个参数可选

```javascript
let obj = { a: 1 }

let proxy = new Proxy(obj, {
  get: function(target, propKey, receiver) {
    console.log(obj === target, propKey, proxy === receiver)
    return 35
  }
})

proxy.b
// true 'b' true
// 35
```

上例可以看出，`get`的第一个参数就是`Proxy`构造函数的第一个参数`target`，第二个参数是访问的属性名，第三个参数就是返回`proxy`实例本身

下面是一个拦截读取操作的例子

```javascript
var person = {
  name: '张三'
}

var proxy = new Proxy(person, {
  get: function(target, propKey) {
    if(propKey in target) {
      return target[propKey]
    } else {
      throw new ReferenceError(`Prop name ${propKey} does not exist.`)
    }
  }
})

person.name // '张三'
person.age // throw Error 'Prop name age does not exist.'
```

`get`方法可以继承

```javascript
let proto = new Proxy({ foo: 'baz' }, {
  get(target, propKey, receiver) {
    console.log('GET ' + propKey)
    return target[propKey]
  }
})

let obj = Object.create(proto)
obj.foo // 'GET foo'
```

利用`Proxy`，可以将读取属性的操作，转变为执行某个函数，从而实现属性的链式操作

```javascript
var pipe = function(value) {
  var funcStack = []
  var oproxy = new Proxy({}, {
    get: function(pipeObject, fnName, receiver) {
      if(fnName === 'get') {
        // 如果是函数名是get,那么对之前的函数栈进行调用
        return funcStack.reduce((val, fn) => {
          return fn(val)
        }, value)
      }
      // 不是get的函数名,保存在函数栈中
      funcStack.push(window[fnName])
      // 返回proxy本身,可以进行链式操作,get方法的第三个参数
      return receiver
    }
  })
  
  return oproxy
}

var double = n => n * 2
var pow = n => n * n
var reverseInt = n => n.toString().split('').reverse().join('') | 0

pipe(3).double.pow.reverseInt.get // 63
```

### 重点理解

下面还有一个例子，说明了`get`方法的**第三个参数表示原始的读操作所在的那个对象**

```javascript
const proxy = new Proxy({}, {
  get(target, key, receiver) {
    return receiver
  }
})

const d = Object.create(proxy)
d.a === d // true
```

`d`对象本身没有`a`属性，所以读取`d.a`的时候，会去`d`的原型`proxy`对象找。因为设置了`get`方法，返回的就是`d`对象本身，表示原始的读操作所在的那个对象。

最后，如果一个属性不可配置(`configurable`)且不可写(`writable`)，则`Proxy`不能修改该属性，否则通过`Proxy`对象访问该属性会报错

```javascript
const target = Object.defineProperties({}, {
  foo: {
    value: 123,
    configurable: false,
    writable: false
  }
})

const proxy = new Proxy(target, {
  get(target, propKey) {
    return 'abc'
  }
})

proxy.foo
// TypeError: Invariant check failed
```

---

### set()

`set`方法用来拦截某个属性的赋值操作，可以接受四个参数，依次是目标对象，属性名，属性值和`Proxy`实例本身，其中最后一个参数可选

```javascript
let validator = {
  set: function(target, propKey, value) {
    if(propKey === 'age') {
      if(!Number.isInteger(value)) {
        throw new TypeError('The age is not an integer')
      }
      if(value > 200) {
        throw new RangeError('The age seems invalid')
      }
    }
    
    // 满足条件的age属性以及其他属性,直接保存
    target[propKey] = value
  }
}

let person = new Proxy({}, validator)

person.age = 100

person.age // 100
person.age = 'young' // throw TypeError 'The age is not an integer'
person.age = 300 // throw RangeError 'The age seems invalid'
```

有时候，我们会在对象上设置内部属性，属性名的第一个字符使用下划线开头。结合`get`和`set`方法，可以做到防止内部属性被外部读写

```javascript
const handler = {
  get(target, key) {
    invariant(key, 'get')
    return target[key]
  },
  set(target, key, value) {
    invariant(key, 'set')
    target[key] = value
  }
}

function invariant(key, action) {
  if(key[0] === '_') {
    // key的第一个字符是下划线,表示私有属性,读写操作都会抛出错误
    throw new Error(`Invalid attempt to ${action} private '${key}' property`)
  }
}

const proxy = new Proxy({}, handler)

proxy._prop
// Error: Invalid attempt to get private '_prop' property
proxy._prop = 'c'
// Error: Invalid attempt to set private '_prop' property
```

`set`方法的第四个参数，和`get`方法的第三个参数相同，都是**表示原始的操作行为所在的那个对象**。下面例子中，`myObj`没有`foo`属性，会去原型链上找，也就是`proxy`对象。设置`foo`属性就会触发`set`方法，返回值就是赋值行为所在的对象`myObj`。

```javascript
const handler = {
  set(obj, prop, value, receiver) {
    obj[prop] = receiver
  }
}

const proxy = new Proxy({}, handler)
proxy.foo = 'bar'
proxy.foo === proxy // true

const myObj = {}
Object.setPrototypeOf(myObj, proxy)
// 或者 const myObj = Object.create(proxy)

myObj.foo = 'baz'
myObj.foo === myObj // true
```

如果目标对象的某个属性，不可写且不可配置，那么`set`方法将不起作用(这点和`get`不同，`set`只是不起作用，`get`会报错)。

```javascript
const obj = {}
Object.defineProperty(obj, 'foo', {
  value: 'bar',
  writable: false
})

const handler = {
  set(obj, prop, value, receiver) {
    obj[prop] = 'baz'
  }
}

const proxy = new Proxy(obj, handler)
proxy.foo = 'baz'
proxy.foo // 'bar'
```

严格模式下，`set`方法不返回`true`就会报错，不返回也会报错

```javascript
'use strict'
const handler = {
  set(obj, prop, value, receiver) {
    obj[prop] = 'baz'
    return false
  }
}

const proxy = new Proxy({}, handler)
proxy.foo = 'bar'
// TypeError: 'set' on proxy: trap returned falsish for property 'foo'
```

---

### apply()

`apply`方法拦截函数的调用，`call`和`apply`操作。`apply`方法接受三个参数，依次是目标对象，目标对象的上下文对象(`this`)和目标对象的参数数组。

```javascript
var target = function() {
  return 'I am the target'
}
var handler = {
  apply(target, ctx, args) {
    return 'I am the proxy'
  }
}

var proxy = new Proxy(target, handler)
proxy()
// 'I am the proxy'
```

上例中，当`proxy`作为函数调用时，就会被`apply`拦截，返回一个字符串。

下面是另外一个例子

```javascript
var twice = {
  apply(target, ctx, args) {
    console.log(args)
    return Reflect.apply(...arguments) * 2
  }
}
function sum(left, right) {
  return left + right
}

var proxy = new Proxy(sum, twice)
proxy(1, 2) 
// [1, 2]
// 6

proxy.call(null, 5, 6)
// [5, 6]
// 22

proxy.apply(null, [7, 8])
// [7, 8]
// 30
```

---

### has()

`has`方法用来拦截`HasProperty`操作，即判断对象是否具有某个属性时，这个方法会生效。**典型的操作就是`in`运算符**(注意，这里只针对`in`运算符，对`for...in`循环不生效)。`has`方法可以接受两个参数，分别是目标对象和属性名。

```javascript
var handler = {
  has(target, key) {
    if(key[0] === '_') {
    	// 私有属性返回false
      return false
    }
    return key in target
  }
}

var target = { _prop: 'foo', prop: 'foo' }
var proxy = new Proxy(target, handler)
'_prop' in proxy // false
'_prop' in target // true
```

如果某个属性不可配置或者整个对象禁止扩展，`has`会报错

```javascript
var obj = { foo: 'foo' }
Object.defineProperties(obj, {
  bar: {
    value: 'bar',
    configurable: false
  }
})

var proxy = new Proxy(obj, {
  has(target, key) {
    return false
  }
})

'foo' in obj // true
'bar' in obj // true
'foo' in proxy // false,has改写
'bar' in proxy // TypeError: 'has' on proxy: trap returned falsish for property 'bar' which exists in the proxy target as non-configurable

Object.preventExtensions(obj)
'foo' in proxy // TypeError: 'has' on proxy: trap returned falsish for property 'foo' which exists in the proxy target as non-configurable
```

可以看到，`obj`的属性`bar`是不可配置的，使用`has`拦截就会报错。当设置`obj`对象不可扩展的时候，无论是`foo`还是`bar`属性都会抛出异常。

---

### construct()

`construct`方法用于拦截`new `命令，接受三个参数，分别是目标对象`target`，构造函数的参数数组`args`和创造的实例对象时，`new`命令作用的构造函数(下例中的`proxy`对象)

```javascript
var proxy = new Proxy(function() {}, {
  construct(target, args) {
    console.log('called: ' + args.join(', '))
    return { value: args[0] * 10 }
  }
})

(new proxy(1, 2)).value
// called: 1, 2
// 10
```

注意，**`construct`方法返回值必须是一个对象**，否则会报错

---

### deleteProperty()

`deleteProperty`方法用于拦截`delete`操作，如果这个方法抛出错误或者返回`false`，当前属性就无法被`delete`命令删除。 

```javascript
var handler = {
  deleteProperty (target, key) {
    if (key[0] === '_') {
      throw new Error(`Invalid attempt to ${action} private '${key}' property`);
    }
    delete target[key]
  }
}

var target = { _prop: 'foo' }
var proxy = new Proxy(target, handler)
delete proxy._prop
// Error: Invalid attempt to delete private '_prop' property
```

同样，不可配置的属性，不能被`deleteProperty`方法删除

---

### defineProperty()

`defineProperty()`方法拦截了`Object.defineProperty()`操作

```javascript
var handler = {
  defineProperty(target, key, descriptor) {
    return false
  }
}

var proxy = new Proxy({}, handler)
proxy.foo = 'foo' // 不生效
```

如果目标对象的某个属性不可写或者不可配置，`defineProperty`方法不能修改这个属性。如果整个对象不可扩展，`defineProperty`方法不能增加属性，否则会报错

---

### ownKeys()

`ownKeys()`方法用来拦截对象自身属性的读取操作

- `Object.getOwnPropertyNames()`
- `Object.getOwnPropertySymbols()`
- `Object.keys()`
- `for...in`循环

```javascript
let target = {
  _bar: '_bar',
  _prop: '_prop',
  prop: 'prop'
}

let handler = {
  ownKeys(target) {
    return Reflect.ownKeys(target).filter(key => key[0] !== '_')
  }
}

let proxy = new Proxy(target, handler)
for(let key of Object.keys(proxy)) {
  console.log(target[key])
}
// 'prop' 
```

注意，在使用`Object.keys()`方法时，有三类属性会被`ownKeys`方法过滤

- 目标对象不存在的属性
- 属性名为`Symbol`值
- 不可遍历的属性

```javascript
let target = {
  a: 1,
  b: 2,
  c: 3,
  [Symbol.for('secret')]: 4
}

Object.defineProperty(target, 'key', {
  enumerable: false,
  configurable: true,
  writable: true,
  value: 'static'
})

let handler = {
  ownKeys(target) {
    return ['a', 'd', Symbol.for('secret'), 'key']
  }
}

let proxy = new Proxy(target, handler)

Object.keys(proxy)
// ['a']
```

`ownKeys()`方法中，显式的返回了不存在的属性`d`，`Symbol`值，不可遍历属性`key`，都被自动过滤掉

在`for...in`循环中也有类似的规则

```javascript
const obj = { hello: 'world' }
const proxy = new Proxy(obj, {
  ownKeys() {
    return ['a', 'b']
  }
})

for(let key in proxy) {
  console.log(key) // 没有输出
}
```

上例中，`ownKeys()`方法返回`a`和`b`属性，由于`obj`没有这两个属性，因此没有任何输出

需要注意的是，`ownKeys()`方法返回的数组成员，也必须都是字符串或者`Symbol`值，其他类型的值，或返回的不是数组，都会报错

```javascript
let obj = {}

let proxy = new Proxy(obj, {
  ownKeys() {
    return [123, true, undefined, null, {}, []]
  }
})

Object.getOwnPropertyNames(p)
// Uncaught TypeError: 123 is not a valid property name
```

---

## Proxy.revocable()

`Proxy.revocable()`方法返回一个可取消的`Proxy`实例

```javascript
let {proxy, revoke} = Proxy.revocable({}, {})

proxy.foo = 123
proxy.foo // 123

revoke()
proxy.foo // TypeError: Revoked
```

`Proxy.revocable()`方法返回一个对象，包含`Proxy`实例和`revoke`函数，用于取消`Proxy`实例。

`Proxy.revocable()`的一个使用场景是，目标对象不允许直接访问，必须通过代理访问，一旦访问结束，就收回代理权，不允许再次访问。 

---

## this问题

目标对象内部的`this`关键字会指向`Proxy`代理，无法保证与目标对象的行为一致

```javascript
const _name = new WeakMap()

class Person {
  constructor(name) {
    _name.set(this, name)
  }
  get name() {
    return _name.get(this)
  }
}

const jane = new Person('Jane')
jane.name // 'Jane'

const proxy = new Proxy(jane, {})
proxy.name // undefined
```

上例中，目标对象`jane`的`name`属性，保存在外部`WeakMap`中，通过`this`来区分。由于`proxy.name`访问时，`this`指向`proxy`，因此取不到值，返回`undefined`

---

## 实例 Web服务客户端

```javascript
function createWebService(baseUrl) {
  return new Proxy({}, {
    get: function(target, propKey, receiver) {
      return () => httpRequest(baseUrl + '/' + propKey)
    }
  })
}
```