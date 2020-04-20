## Object.is()

ES5比较两个值是否相等，有两种运算符，但是都有缺点

- `==`相等运算符，会自动转换数据类型
- `===`严格相等运算符，`NaN`不等于自身，`+0 === -0`

ES6提出"Same-value equality"（同值相等）算法，用来解决这个问题。`Object.is()`就是部署这个算法的新方法。**用来比较两个值是否严格相等 ，与严格比较运算符（`===`）的行为基本一致**。 

```javascript
Object.is('foo', 'foo')
// true
Object.is({}, {})
// false
```

 不同之处只有两个：一是`+0`不等于`-0`，二是`NaN`等于自身。 

```javascript
+0 === -0 // true
NaN === NaN // false

Object.is(+0, -0) // false
Object.is(NaN, NaN) // true
```

ES5 可以通过下面的代码，部署`Object.is`。

```javascript
Object.defineProperty(Object, 'is', {
  value: function(x, y) {
    if (x === y) {
      // 针对+0 不等于 -0的情况
      return x !== 0 || 1 / x === 1 / y;
    }
    // 针对NaN的情况, NaN !== NaN => true
    return x !== x && y !== y;
  },
  configurable: true,
  enumerable: false,
  writable: true
});
```

------

## Object.assign()

`Object.assign`方法用于对象的合并，将源对象的**所有可枚举属性**，复制到目标对象。**该方法会修改目标对象（第一个参数）**。Object.assign`方法的**第一个参数是目标对象，后面的参数都是源对象**。 并且，如果目标对象与源对象有同名属性，或多个源对象有同名属性，则**后面的属性会覆盖前面的属性**。 

```javascript
const target = { a: 1, b: 1 };

const source1 = { b: 2, c: 2 };
const source2 = { c: 3 };

Object.assign(target, source1, source2);
target // {a:1, b:2, c:3}
```

如果只有一个参数，`Object.assign`会直接返回该参数。 

```javascript
const obj = {a: 1};
Object.assign(obj) === obj // true
```

如果该参数不是对象，则会先转成对象，然后返回。由于`undefined`和`null`无法转成对象，所以如果它们作为参数，就会报错。 

```javascript
typeof Object.assign(2) // "object"
Object.assign(2) // Number {2}

Object.assign(undefined) // 报错
Object.assign(null) // 报错
```

如果非对象参数出现在源对象的位置（非首位参数），分为以下几种情况

- `undefined`和`null`无法转成对象，`Object.assign()`会直接忽略该参数

```javascript
let obj = {a: 1};
Object.assign(obj, undefined) === obj // true
Object.assign(obj, null) === obj // true
```

- 字符串会以数组的形式，拷贝入目标对象
- 其他类型的值（即数值，布尔值），不会产生任何效果

```javascript
const v1 = 'abc';
const v2 = true;
const v3 = 10;

Object.assign({}, v1, v2, v3); // { "0": "a", "1": "b", "2": "c" }
```

这是因为，只有**字符串的包装对象，会产生可枚举属性**

```javascript
Object(true) // {[[PrimitiveValue]]: true}
Object(10)  //  {[[PrimitiveValue]]: 10}
Object('abc') // {0: "a", 1: "b", 2: "c", length: 3, [[PrimitiveValue]]: "abc"}
```

可见，`Object.assign()`拷贝的属性是有限制的，只拷贝源对象的自身属性（不包括继承属性，只有`for...in`才会遍历到继承属性），也不拷贝不可枚举的属性。

```javascript
Object.assign({b: 'c'},
  Object.defineProperty({}, 'invisible', {
    enumerable: false, // 不可枚举
    value: 'hello'
  })
)
// { b: 'c' }
```


属性名为 `Symbol`值的属性，也会被`Object.assign`拷贝。

```javascript
Object.assign({ a: 'b' }, { [Symbol('c')]: 'd' })
// { a: 'b', Symbol(c): 'd' }
```

### 注意点

1. **浅拷贝**

 `Object.assign`方法实行的是浅拷贝，而不是深拷贝。 

```javascript
const obj1 = {a: {b: 1}};
const obj2 = Object.assign({}, obj1);

obj1.a.b = 2;
obj2.a.b // 2
```

2. **同名属性的替换**

 对于这种嵌套的对象，一旦遇到同名属性，`Object.assign`的**处理方法是替换，而不是添加**。 

```javascript
const target = { a: { b: 'c', d: 'e' } }
const source = { a: { b: 'hello' } }
Object.assign(target, source)
// { a: { b: 'hello' } }
```

 一些函数库提供`Object.assign`的定制版本（比如`Lodash`的`_.defaultsDeep`方法），可以得到深拷贝的合并。 

3. **可以用来处理数组**

`Object.assign`可以用来处理数组，但是会把数组视为对象。

```javascript
Object.assign([1, 2, 3], [4, 5])
// [4, 5, 3]

// 相当于
Object.assign({0: 1, 1: 2, 2: 3}, {0: 4, 1: 5})
```

4. **取值函数的处理**

`Object.assign`**只能进行值的复制**，如果要**复制的值是一个取值函数，那么将求值后再复制**。 

```javascript
const source = {
  get foo() { return 1 }
};
const target = {};

Object.assign(target, source)
// { foo: 1 }

// 相当于
Object.assign(target, { foo: 1 })
```

### 常见用途

1. **为对象添加属性**

```javascript
class Point {
  constructor(x, y) {
    // 通过Object.assign方法，将x属性和y属性添加到Point类的对象实例
    Object.assign(this, {x, y});
  }
}
```

2. **为对象添加方法**

```javascript
// 直接将两个函数放在大括号中，再使用assign方法添加到SomeClass.prototype之中
Object.assign(SomeClass.prototype, {
  someMethod(arg1, arg2) {
    ···
  },
  anotherMethod() {
    ···
  }
});

// 等同于下面的写法
SomeClass.prototype.someMethod = function (arg1, arg2) {
  ···
};
SomeClass.prototype.anotherMethod = function () {
  ···
};
```

3. **克隆对象**

```javascript
function clone(origin) {
  return Object.assign({}, origin);
}
```

不过，只能克隆原始对象自身的值，不能克隆它继承的值。如果想要保持继承链，可以采用下面的代码。 

```javascript
function clone(origin) {
  let originProto = Object.getPrototypeOf(origin);
  return Object.assign(Object.create(originProto), origin);
}
```

4. **合并多个对象**

```javascript
const merge =
  (target, ...sources) => Object.assign(target, ...sources);
```

如果希望合并后**返回一个新对象**，可以改写上面函数，**对一个空对象合并**

```javascript
const merge =
  (...sources) => Object.assign({}, ...sources);
```

5. **为属性指定默认值**

```javascript
const DEFAULTS = {
  logLevel: 0,
  outputFormat: 'html'
};

function processContent(options) {
  options = Object.assign({}, DEFAULTS, options);
  console.log(options);
  // ...
}
```

注意，**由于存在浅拷贝的问题**，`DEFAULTS`对象和`options`对象的所有属性的值，**最好都是简单类型，不要指向另一个对象**。否则，`DEFAULTS`对象的该属性很可能不起作用。 

```javascript
const DEFAULTS = {
  url: {
    host: 'example.com',
    port: 7070
  },
};

processContent({ url: {port: 8000} })
// {
//   url: {port: 8000}
// }
```

------

## Object.getOwnPropertyDescriptors()

ES5 的`Object.getOwnPropertyDescriptor()`方法会返回某个对象属性的描述对象（`descriptor`）。 

```javascript
Object.getOwnPropertyDescriptor(obj, key)
```

ES2017 引入了`Object.getOwnPropertyDescriptors()`方法，**返回指定对象所有自身属性（非继承属性）的描述对象**。 

```javascript
const obj = {
  foo: 123,
  get bar() { return 'abc' }
}

Object.getOwnPropertyDescriptors(obj)
// { foo:
//    { value: 123,
//      writable: true,
//      enumerable: true,
//      configurable: true },
//   bar:
//    { get: [Function: get bar],
//      set: undefined,
//      enumerable: true,
//      configurable: true } }

Object.getOwnPropertyDescriptor(obj, 'foo')
//  { value: 123,
//    writable: true,
//    enumerable: true,
//    configurable: true }
Object.getOwnPropertyDescriptor(obj, 'bar')
//  { get: [Function: get bar],
//    set: undefined,
//    enumerable: true,
//    configurable: true }
```

该方法的实现非常容易。

```javascript
function getOwnPropertyDescriptors(obj) {
  const result = {};
  // 包含对象自身的（不含继承的）所有键名，不管键名是 Symbol 或字符串，也不管是否可枚举。
  for (let key of Reflect.ownKeys(obj)) {
    result[key] = Object.getOwnPropertyDescriptor(obj, key);
  }
  return result;
}
```

该方法的引入目的，**主要是为了解决`Object.assign()`无法正确拷贝`get`属性和`set`属性的问题**。因为**`Object.assign`方法总是拷贝一个属性的值，而不会拷贝它背后的赋值方法或取值方法**。 

```javascript
const source = {
  set foo(value) {
    console.log(value);
  }
};

const target1 = {};
Object.assign(target1, source);
Object.getOwnPropertyDescriptor(target1, 'foo')
// { value: undefined,
//   writable: true,
//   enumerable: true,
//   configurable: true }
```

这时，`Object.getOwnPropertyDescriptors()`方法配合`Object.defineProperties()`方法，就可以实现正确拷贝。 

```javascript
const source = {
  set foo(value) {
    console.log(value);
  }
};

const target2 = {};
Object.defineProperties(target2, Object.getOwnPropertyDescriptors(source));
Object.getOwnPropertyDescriptor(target2, 'foo')
// { get: undefined,
//   set: [Function: set foo],
//   enumerable: true,
//   configurable: true }
```

上面代码中，两个对象合并的逻辑可以写成一个函数。

```javascript
const shallowMerge = (target, source) => Object.defineProperties(
  target,
  Object.getOwnPropertyDescriptors(source)
);
```

`Object.getOwnPropertyDescriptors()`方法的另一个用处，是配合`Object.create()`方法，将对象属性克隆到一个新对象。 

```javascript
const shallowClone = (obj) => Object.create(
  Object.getPrototypeOf(obj),
  Object.getOwnPropertyDescriptors(obj)
);
```

`Object.getOwnPropertyDescriptors()`也可以用来实现 Mixin（混入）模式。

```javascript
let mix = (object) => ({
  with: (...mixins) => mixins.reduce(
    (obj, mixin) => Object.create(
      obj, Object.getOwnPropertyDescriptors(mixin)
    ), object)
});

// multiple mixins example
let a = {a: 'a'};
let b = {b: 'b'};
let c = {c: 'c'};
let d = mix(c).with(a, b);

d.c // "c"
d.b // "b"
d.a // "a"
```

------

## Object.setPrototypeOf()

`Object.setPrototypeOf`方法用来设置一个对象的原型对象（`prototype`），**返回参数对象本身**。它是 ES6 正式推荐的设置原型对象的方法。 

```javascript
// 语法
Object.setPrototypeOf(object, prototype)

// 用法
const o = Object.setPrototypeOf({}, null);

// 相当于
function setPrototypeOf(obj, proto) {
  obj.__proto__ = proto
  return obj
}
```

下面是一个例子。

```javascript
let proto = {};
let obj = { x: 10 };
Object.setPrototypeOf(obj, proto);

proto.y = 20;
proto.z = 40;

obj.x // 10
obj.y // 20
obj.z // 40, y,z均是原型链上的属性
obj // { x: 10 }
obj.__proto__ // { y: 20, z: 40 }
```

如果第一个参数不是对象，会自动转为对象。但是由于**返回的还是第一个参数，所以这个操作不会产生任何效果**。

```javascript
Object.setPrototypeOf(1, {}) === 1 // true
Object.setPrototypeOf('foo', {}) === 'foo' // true
Object.setPrototypeOf(true, {}) === true // true
```

由于`undefined`和`null`无法转为对象，所以**如果第一个参数是`undefined`或`null`，就会报错**。

```javascript
Object.setPrototypeOf(undefined, {})
// TypeError: Object.setPrototypeOf called on null or undefined

Object.setPrototypeOf(null, {})
// TypeError: Object.setPrototypeOf called on null or undefined
```

------

## Object.getPrototypeOf()

该方法与`Object.setPrototypeOf`方法配套，用于读取一个对象的原型对象。 

```javascript
// 语法
Object.getPrototypeOf(obj);

function Rectangle() {
  // ...
}

const rec = new Rectangle();

Object.getPrototypeOf(rec) === Rectangle.prototype // true
Object.getPrototypeOf(rec) === rec.__proto__ // true

Object.setPrototypeOf(rec, Object.prototype);
Object.getPrototypeOf(rec) === Rectangle.prototype // false
```

如果参数不是对象，会被自动转为对象。

```javascript
// 等同于 Object.getPrototypeOf(Number(1))
Object.getPrototypeOf(1)
// Number {[[PrimitiveValue]]: 0}

// 等同于 Object.getPrototypeOf(String('foo'))
Object.getPrototypeOf('foo')
// String {length: 0, [[PrimitiveValue]]: ""}

// 等同于 Object.getPrototypeOf(Boolean(true))
Object.getPrototypeOf(true)
// Boolean {[[PrimitiveValue]]: false}

Object.getPrototypeOf(1) === Number.prototype // true
Object.getPrototypeOf('foo') === String.prototype // true
Object.getPrototypeOf(true) === Boolean.prototype // true
```

如果参数是`undefined`或`null`，它们无法转为对象，所以会报错。

```javascript
Object.getPrototypeOf(null)
// TypeError: Cannot convert undefined or null to object

Object.getPrototypeOf(undefined)
// TypeError: Cannot convert undefined or null to object
```

------

## Object.keys()

`Object.keys`返回一个数组，成员是**参数对象自身的（不含继承的）所有可遍历（enumerable）属性（不含 Symbol 属性）的键名**。 

```javascript
var obj = { foo: 'bar', baz: 42, [Symbol('bar')]: 'foo' };
Object.keys(obj) // ["foo", "baz"]
```

## Object.values()

`Object.values`返回一个数组，成员是**参数对象自身的（不含继承的）所有可遍历（enumerable）属性（不含 Symbol 属性）的键值**。 

```javascript
var obj = { foo: 'bar', baz: 42, [Symbol('bar')]: 'foo' };
Object.keys(obj) // ["foo", "baz"]
```

下面还有个补充例子，说明了只返回可遍历属性

```javascript
const obj = Object.create({}, {p: {value: 42}});
Object.values(obj) // []

const obj = Object.create({}, {p:
  {
    value: 42,
    enumerable: true
  }
});
Object.values(obj) // [42]
```

如果`Object.values`方法的参数是一个字符串，会返回各个字符组成的一个数组。

```javascript
Object.values('foo')
// ['f', 'o', 'o']

Object.keys('foo')
// ['0', '1', '2']
```

字符串会**先转成一个类似数组的对象。字符串的每个字符，就是该对象的一个属性**。因此，`Object.values`返回每个属性的键值，就是各个字符组成的一个数组。 

如果参数不是对象，`Object.values`会先将其转为对象。**由于数值和布尔值的包装对象，都不会为实例添加非继承的属性。所以，`Object.values`和`Object.keys`会返回空数组**。和之前例子相同，由于`undefined`和`null`不能转成对象，所以都会报错。

```javascript
Object.values(42) // []
Object.values(true) // []

Object.keys(42) // []
Object.keys(true) // []

Object.keys(null) // Uncaught TypeError: Cannot convert undefined or null to object
Object.values(undefined) // Uncaught TypeError: Cannot convert undefined or null to object
```

## Object.entries()

`Object.entries`返回一个数组，成员是**参数对象自身的（不含继承的）所有可遍历（enumerable）属性（不含 Symbol 属性）的键值对数组**。 

```javascript
var obj = { foo: 'bar', baz: 42, [Symbol('bar')]: 'foo' };
Object.entries(obj) // [ ["foo", "bar"], ["baz", 42] ]
```

`Object.entries`的基本用途是遍历对象的属性。

```javascript
let obj = { one: 1, two: 2 };
for (let [k, v] of Object.entries(obj)) {
  console.log(
    `${JSON.stringify(k)}: ${JSON.stringify(v)}`
  );
}
// "one": 1
// "two": 2
```

`Object.entries`方法的另一个用处是，将对象转为真正的`Map`结构。

```javascript
const obj = { foo: 'bar', baz: 42 };
const map = new Map(Object.entries(obj));
map // Map { foo: "bar", baz: 42 }
```

总结一下，上述三个方法的使用都非常相似，都是对对象的遍历，只不过返回的数据不同，返回的数组的成员排序，和之前在`属性的遍历`中介绍的规则相同

- 首先遍历所有数值键，按照数值升序排列。
- 其次遍历所有字符串键，按照加入时间升序排列。
- 最后遍历所有 Symbol 键，按照加入时间升序排列，只不过这三个方法都不会遍历Symbol键。

```javascript
const obj = { 100: 'a', x : 'y', 2: 'b', 7: 'c' };
Object.values(obj)
// ["b", "c", "a", "y"]
```

------

## Object.fromEntries()

`Object.fromEntries()`方法是`Object.entries()`的逆操作，用于将一个键值对数组转为对象。 

```javascript
Object.fromEntries([
  ['foo', 'bar'],
  ['baz', 42]
])
// { foo: "bar", baz: 42 }
```

该方法的主要目的，是将键值对的数据结构还原为对象，因此特别适合将 Map 结构转为对象。

```javascript
// 例一
const entries = new Map([
  ['foo', 'bar'],
  ['baz', 42]
]);

Object.fromEntries(entries)
// { foo: "bar", baz: 42 }

// 例二
const map = new Map().set('foo', true).set('bar', false);
Object.fromEntries(map)
// { foo: true, bar: false }
```

该方法的一个用处是**配合`URLSearchParams`对象，将查询字符串转为对象**。

```javascript
Object.fromEntries(new URLSearchParams('foo=bar&baz=qux'))
// { foo: "bar", baz: "qux" }

const params = new URLSearchParams('foo=bar&baz=qux')
for(let param of params) {
  console.log(param)
}
// ['foo', 'bar']
// ['baz', 'qux']
Object.fromEntries(params)
// { foo: "bar", baz: "qux" }
```