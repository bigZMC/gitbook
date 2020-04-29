## 概述

`ES5`的对象属性名都是字符串，这容易造成**属性名的冲突**。`ES6`引入了一种新的原始数据类型`Symbol`，表示独一无二的值。它是`JavaScript`语言的第七种数据类型，前六种是：`undefined`、`null`、布尔值（`Boolean`）、字符串（`String`）、数值（`Number`）、对象（`Object`）。 

 `Symbol` 值通过`Symbol()`函数生成。`ES6`中，对象的属性名可以有两种类型

-  字符串
- `Symbol`类型，可以保证不与其他属性名产生冲突

```javascript
let s = Symbol()

typeof s // symbol
```

注意：`Symbol()`函数前不能使用`new`命令，会报错。**因为`Symbol`是一个原始类型，不是一个对象，也不能添加属性**。基本上，**可以理解为类似于字符串的数据类型**。

`Symbol()`函数可以接受**一个字符串作为参数，表示对`Symbol`实例的描述，没有实际意义，主要是为了容易区分**，可以在控制台显示，或者转为字符串。

- `Symbol()`函数的参数是一个字符串，也**只能是字符串**，如果是一个对象，则会调用`toString()`方法转成字符串
- 相同参数的`Symbol`返回值，是不相等的

```javascript
let s1 = Symbol('foo')
let s2 = Symbol('bar')
let s3 = Symbol()
let s4 = Symbol()

s1 // Symbol(foo)
s2 // Symbol(bar)
s3 // Symbol()
s4 // Symbol(),无法和s3的值进行区分

s1.toString() // 'Symbol(foo)'
s2.toString() // 'Symbol(bar)'
s3.toString() // 'Symbol()'
s4.toString() // 'Symbol()'

s3 === s4 // false
s3.toString() === s4.toString() // true

let s5 = Symbol('bar')
s2 === s5 // false,相同参数的Symbol值不相同
```

`Symbol`值不能与其他类型的值进行运算，会报错 

```javascript
let sym = Symbol('My symbol')

"your symbol is " + sym
// TypeError: can't convert symbol to string
`your symbol is ${sym}`
// TypeError: can't convert symbol to string
```

但是，`Symbol`值可以显式转为字符串

```javascript
let sym = Symbol('My symbol')

String(sym) // 'Symbol(My symbol)'
sym.toString() // 'Symbol(My symbol)'
```

最后，`Symbol`值还可以转为布尔值，全部转为`true`，但是不能转为数值

```javascript
let sym = Symbol()
Boolean(sym) // true
!sym // false

Boolean(Symbol(false)) // true,无论Symbol描述如何,都会被转为true

Number(sym) // TypeError
sym + 2 // TypeError
```

------

## Symbol.prototype.description

创建`Symbol`的时候，可以添加一个描述，我们只能通过显式转为字符串才能获取到，因此`ES2019`提供了一个实例属性`description`，**直接返回`Symbol`的描述**

```javascript
let sym = Symbol('foo')

sym // Symbol(foo)
String(sym) // 'Symbol(foo)'
sym.toString() // 'Symbol(foo)'

sym.description // 'foo'
```

------

## Symbol作为属性名

由于每一个`Symbol`值都是不相等的，所以可以用于对象的属性名，保证不会出现同名属性。

```javascript
let mySymbol = Symbol()

// 第一种写法
let a = {}
a[mySymbol] = 'hello'

// 第二种写法
let a = {
  [mySymbol]: 'hello'
}

// 第三种写法
let a = {}
Object.defineProperty(a, mySymbol, { value: 'hello' })

a[mySymbol] // 'hello'
```

注意，`Symbol`值作为对象属性名时，不能用点运算符，因为**点运算符后面总是字符串**，只能使用方括号或者`Object.defineProperty`赋值

```javascript
const mySymbol = Symbol()
let obj = {}

obj.mySymbol = 'hello'
obj[mySymbol] // undefined
obj['mySymbol'] // 'hello',使用点运算符赋值,mySymbol被看作字符串
```

下面有两个例子，将`Symbol`类型用于定义一组常量，保证了常量的值不相等

```javascript
const COLOR_RED = Symbol()
const COLOR_GREEN = Symbol()

// 将Symbol赋值给常量,在switch中进行比较
// 注意,不是Symbol类型比较,而是通过常量名进行比较,常量值变成了唯一的符号,没有实际意义
function getComplement(color) {
  switch (color) {
    case COLOR_RED:
      return COLOR_RED
    case COLOR_GREEN:
      return COLOR_GREEN
    default:
      throw new Error('undefined color')
  }
}
```
下例中的`Triangle`字符也是一个没有意义的值，只需要保证不会和其他`shapeType`属性不冲突，所以可以换成`Symbol`值
```javascript
const shapeType = {
  // triangle: 'Triangle'
  triangle: Symbol('Triangle')
}

function getArea(shape, options) {
  let area = 0
  switch (shape) {
    case shapeType.triangle:
      area = 0.5 * options.width * options.height;
      break
  }
  return area
}

getArea(shapeType.triangle, { width: 100, height: 100 })
```

------

## 属性名的遍历

`Symbol`作为属性名时，常规的遍历方法会忽略这个属性。包括`for...in`，`for...of`，`Object.keys()`，`Object.getOwnPropertyNames`，还是对象转字符串`JSON.stringify()`，以上的这些方法都取不到`Symbol`属性名

但是有如下两种方法可以实现对`Symbol`属性值的获取或遍历

- `Object.getOwnPropertySymbols()`，返回一个数组，成员是当前对象的所有`Symbol`属性值

```javascript
const obj = {
  [Symbol('a')]: 'hello',
  [Symbol('b')]: 'world'
}

Object.getOwnPropertySymbols(obj) // [Symbol(a), Symbol(b)]
```

- `Reflect.ownKeys()`，返回所有键名，包括常规键名和`Symbol`键名(依然遵循数字，字符，`Symbol`的排序规则)

```javascript
let obj = {
  [Symbol('my_key')]: 1,
  enum: 2,
  nonEnum: 3
}

Reflect.ownKeys(obj) // ['enum', 'nonEnum', Symbol(my_key)]
```

可以利用`Symbol`键名不会被常规方法遍历，为对象定义一些非私有的，但又希望只用于内部的方法

```javascript
const size = Symbol('size')

class Collection {
  constructor() {
    this[size] = 0
  }

  add(item) {
    this[this[size]] = item
    this[size]++
  }

  static sizeOf(instance) {
    return instance[size]
  }
}

let x = new Collection()
Collection.sizeOf(x) // 0,使用静态函数,获取的就是构造函数中的0

x.add('foo') // this[0] = 'foo'
Collection.sizeOf(x) // 1,调用add函数,size的值加一

Object.keys(x) // ['0']
Object.getOwnPropertyNames(x) // ['0']
Object.getOwnPropertySymbols(x) // [Symbol(size)]
```

------

## Symbol.for()，Symbol.keyFor()

如果想要使用同一个`Symbol`值，而没有使用变量保存起来，可以使用`Symbol.for(keyName)`**搜索是否有以`keyName`作为描述符的`Symbol`值，如果有就返回该值，如果没有就创建一个新的`Symbol`值**。

```javascript
let s1 = Symbol.for('foo')
let s2 = Symbol('foo')
let s3 = Symbol.for('foo')

s1 === s2 // false
s1 === s3 // true
s2 === s3 // false
```

`Symbol.for()`与`Symbol()`这两种写法，都会生成新的`Symbol`。只不过，`Symbol.for()`的`key`会被登记在全局环境中供搜索。由于`Symbol()`并没有进行登记，所以上例中`s2`和`s3`也不相等，可以简单理解为**只有使用`Symbol.for()`返回的值才能进行比较**。

`Symbol.keyFor()`方法返回一个**已登记的`Symbol`类型值的`key`**。已登记的意思是`Symbol()`返回的值并不能用改方法返回`key`值。

```javascript
let s1 = Symbol.for('foo')
Symbol.keyFor(s1) // 'foo'

let s2 = Symbol('foo')
Symbol.keyFor(s2) // undefined,s2属于未登记的Symbol值

s1.description // 'foo'
s2.description // 'foo',都可以通过description属性取值
```

注意，`Symbol.for()`**所登记的名字，都在全局环境中**，可以用在不同的`iframe`中取到同一个值

```javascript
function foo() {
  return Symbol.for('bar')
}

const x = foo()
const y = Symbol.for('bar')
x === y // true
```

以下是一个单例模式使用`Symbol`的例子

```javascript
// singleton.js
const FOO_KEY = Symbol.for('foo')

function A() {
  this.foo = 'hello'
}

if (!global[FOO_KEY]) {
  global[FOO_KEY] = new A() // 返回单例
}

module.exports = global[FOO_KEY]

// app.js
const singleton = require('./singleton.js')
singleton.foo // 'hello'

// 这里是用Symbol.for()生成的常量FOO_KEY
// 虽然不会被无意覆盖,但是依然可以被改写
global[Symbol.for('foo')] = { foo: 'world' }
singleton.foo // 'world'
```

我们还可以使用`Symbol()`创建`FOO_KEY`，其他脚本将无法引用这个值，也就没法修改。但是，如果多次执行这个脚本，每次得到的`FOO_KEY`是不一样的。