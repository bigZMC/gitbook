## Object.create()

`Object.create()`方法创建一个新对象，使用现有的对象来提供新创建的对象的`__proto__`。

```javascript
// 语法
Object.create(proto[, propertiesObject])
```

- `proto` 新创建对象的原型对象、
- `propertiesObject` 添加到新创建对象的属性，格式为`key: { descriptor }`，这些描述符和`Object.defineProperties()`的第二个参数相对应

注意：如果`propertiesObject`参数是`null`或非原始包装对象，则报错

```javascript
function MyClass() {
	// ...
}

function SubClass() {
  // ...
}

// 子类续承父类
SubClass.prototype = Object.create(MyClass.prototpe)
```

使用`propertiesObject`参数

```javascript
let o = Object.create({}, { p: { value: 42 } })

// 相当于
// 默认是不可写,不可枚举,不可配置的属性
let o = Object.create({}, {
  p: {
    value: 42, 
    writable: false,
    enumerable: false,
    configurable: false 
  } 
})
```

------

## Object.defineProperty()

`Object.defineProperty()`方法在对象上定义一个新属性，或修改对象的现有属性

```javascript
// 语法
Object.defineProperty(obj, prop, descriptor)
```

- `obj` 定义属性的对象
- `prop` 要定义或修改的属性的名称或`Symbol`
- `descriptor` 属性描述符

属性描述符一共有以下几种配置

- `configurable` 属性描述符能否被改变，默认为`false`。一旦被定义为`false`，再次修改属性除`value`和`writable`其余描述符时，都会抛出`Uncaught TypeError: Cannot redefine property`的错误。
- `enumerable` 该属性能否被枚举
- `value` 该属性对应的值
- `writable` 能否修改`value`的值，默认为`false`
- `get` 属性的`getter`函数，默认为`undefined`，在访问属性的时候调用此函数
- `set` 属性的`setter`函数，默认为`undefined`，在属性值被修改时，会被调用

```javascript
let obj = {}
let descriptor = Object.create(null) // 没有继承的属性
descriptor.value = 'static'
Object.defineProperty(obj, 'key', descriptor)

// 显示
Object.defineProperty(obj, 'key', {
  enumerable: false,
  configurable: false,
  writable: false,
  value: 'static'
})
```

下面还有一些例子

```javascript
let o = {} // 创建一个新对象

// 在对象中添加一个属性与数据描述符的示例
Object.defineProperty(o, 'a', {
  value : 37,
  writable : true,
  enumerable : true,
  configurable : true
})

// 数据描述符和存取描述符不能混合使用
Object.defineProperty(o, 'conflict', {
  value: 0x9f91102,
  get() { return 0xdeadbeef; } 
})
// 抛出错误 TypeError: value appears only in data descriptors, get appears only in accessor descriptors
```

------

## Object.defineProperties()

`Object.defineProperties()`方法直接在一个对象上定义新的属性或修改现有属性，并返回该对象。 

```javascript
// 语法
Object.defineProperties(obj, props)
```

- `obj` 定义属性的对象
- `props` 定义其可枚举属性或修改的属性描述符的对象。 

```js
let obj = {} 

Object.defineProperties(obj, {
  'property1': {
    value: true,
    writable: true
  },
  'property2': {
    value: 'Hello',
    writable: false
  }
})
```

------

## Object.freeze()和Object.isFrozen()

`Object.freeze()`方法可以**冻结**一个对象，对象再也不能被修改，返回和传入参数相同的对象

-  不能向这个对象添加新的属性 
-  不能删除已有属性 
-  不能修改该对象已有属性的可枚举性、可配置性、可写性，以及不能修改已有属性的值 
-  该对象的原型也不能被修改 

```javascript
// 语法
Object.freeze(obj)
```

对象是否冻结，可以通过`Object.isFrozen()`来判断

```javascript
// 语法
Object.isFrozen(obj)
```

下面是一些例子

```javascript
let obj = {
  prop: {},
  foo: 'bar'
}

let o = Object.freeze(obj) // 冻结对象可以不必重新接收,因为返回值完全等于传入对象
o === obj // true
Object.isFrozen(o) // true

obj.foo = 'quux' // 静默地不做任何事
obj.prop.foo = 'baz' // 被修改到
```

可以看出，**`Object.freeze()`想要真正冻结对象，必须采用递归的方式依次`freeze`**

数组因为也是对象，也可以用该函数

```javascript
let a = [0];
Object.freeze(a)

a[0] = 1
a[0] // 0
```

------

## Object.seal()和Object.isSealed()

`Object.seal()`方法**封闭**一个对象，阻止添加新属性并将所有现有属性标记为不可配置，返回和传入参数相同的对象

-  对象不能添加新属性 
-  所有已有属性不可配置
-  **当前属性的值只要原来是可写的就可以改变** ，这点和`Object.freeze()`有明显区别

对象是否封闭，可以通过`Object.isSealed()`来判断

```javascript
// 语法
Object.isSealed(obj)
```

下面是一些例子

```javascript
let obj = {
  prop: {},
  foo: 'bar'
}

let o = Object.seal(obj) // 封闭对象可以不必重新接收,因为返回值完全等于传入对象
o === obj // true
Object.isSealed(o) // true

obj.foo = 'quux' // 正常修改
delete obj.foo // obj.foo没有被删除

// 通过Object.defineProperty添加属性将会报错
Object.defineProperty(obj, 'bar', {
  value: 17
}) // throws a TypeError

 // 通过Object.defineProperty修改属性值
Object.defineProperty(obj, 'foo', {
  value: 'baz'
})
obj.foo // baz
```

------

## Object.prototype.hasOweProperty()

`hasOwnProperty()` 方法会返回一个布尔值，指示对象自身属性中是否具有指定的属性。 

```javascript
// 语法
obj.hasOwnProperty(prop)
```

注意： 该方法会**忽略掉那些从原型链上继承到的属性**，但是不可枚举属性也会返回`true`

```javascript
let obj = {
  value: 123
}
Object.defineProperty(obj, 'foo', {
  value: 'bar',
  enumerable: false
})

obj.hasOwnProperty('value') // true
obj.hasOwnProperty('baz') // false
obj.hasOwnProperty('foo') // true 
```

因此我们可以使用`for...in`过滤掉不可枚举属性，遍历一个对象的所有自身属性

```javascript
let buz = {
  fog: 'stack'
}

for (let key in buz) {
  if (buz.hasOwnProperty(key)) {
    console.log(key, buz[key]);
  } else {
    console.log(key)
  }
}
```

