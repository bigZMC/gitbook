## 属性的简介表达式

ES6 允许在大括号里面，直接写入变量和函数，作为对象的属性和方法。这样的书写更加简洁。

```javascript
const foo = 'bar';
const baz = {foo};
// 等同于
const baz = {foo: foo}; // {foo: "bar"}

function f(x, y) {
  return {x, y};
}
// 等同于
function f(x, y) {
  return {x: x, y: y};
}

f(1, 2) // Object {x: 1, y: 2}
```

除了属性简写，方法也可以简写。

```javascript
const o = {
  method() {
    return "Hello!";
  }
};

// 等同于
const o = {
  method: function() {
    return "Hello!";
  }
};
```

这种写法用于函数的返回值，将会非常方便。

```javascript
function getPoint() {
  const x = 1;
  const y = 10;
  return {x, y};
}

getPoint()
// {x:1, y:10}
```

`CommonJS`模块输出一组变量，就非常合适使用简洁写法。

```javascript
let ms = {};

function getItem (key) {
  return key in ms ? ms[key] : null;
}

function setItem (key, value) {
  ms[key] = value;
}

function clear () {
  ms = {};
}

module.exports = { getItem, setItem, clear };
// 等同于
module.exports = {
  getItem: getItem,
  setItem: setItem,
  clear: clear
};
```

注意，简写的对象方法不能用作构造函数，会报错。**目前只有`function`的写法才可以作为构造函数**，其余的箭头函数或简写方法都会抛出异常，可以理解为**这些抽象操作是作为`ECMAScript`语言的辅助操作**。可以参考[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Method_definitions#Method_definitions_are_not_constructable)或[这篇文章](https://www.stefanjudis.com/today-i-learned/not-every-javascript-function-is-constructable/)

```javascript
const example = {
  Fn: function() { console.log(this); },
  Arrow: () => { console.log(this); },
  Shorthand() { console.log(this); }
};

example.Fn() // {Fn: ƒ, Arrow: ƒ, Shorthand: ƒ} 指向example对象
example.Arrow() // window
example.Shorthand() // {Fn: ƒ, Arrow: ƒ, Shorthand: ƒ} 指向example对象

new example.Fn();        // Fn {}
new example.Arrow();     // Uncaught TypeError: example.Arrow is not a constructor
new example.Shorthand(); // Uncaught TypeError: example.Shorthand is not a constructor
```

------

## 属性名表达式

ES5定义对象的属性有两种方法

```javascript
// 方法一
obj.foo = true;

// 方法二
obj['a' + 'bc'] = 123;
```

但是，如果是按照字面量方式定义对象（使用大括号），ES5就只能用方法一

```javascript
var obj = {
  foo: true,
  abc: 123
};
```

ES6允许字面量定义对象时，用方法二（表达式）作为对象属性名

```javascript
let propKey = 'foo';

let obj = {
  [propKey]: true,
  ['a' + 'bc']: 123
};
// { foo: true, abc: 123 }

let lastWord = 'last word';

const a = {
  'first word': 'hello',
  [lastWord]: 'world'
};

a['first word'] // "hello"
a[lastWord] // "world"
a['last word'] // "world"
```

表达式还可以用于定义方法名。

```javascript
let obj = {
  ['h' + 'ello']() {
    return 'hi';
  }
};

obj.hello() // hi
```

注意，**属性名表达式与简洁表示法，不能同时使用**，会报错。

```javascript
// 报错,这里简洁表示法将foo变成了bar,但是只能看作是字符串
const foo = 'bar';
const bar = 'abc';
const baz = { [foo] }; // 相当于 { 'bar' }

// 正确
const foo = 'bar';
const baz = { [foo]: 'abc'};
```

 注意，**属性名表达式如果是一个对象，默认情况下会自动将对象转为字符串**`[object Object]`，这一点要特别小心，和ES5的表现形式一样，只不过ES5不能通过字面量方式声明，只能通过`obj[key]`的形式。

```javascript
const keyA = {a: 1};
const keyB = {b: 2};

const myObject = {
  [keyA]: 'valueA',
  [keyB]: 'valueB'
};

myObject // Object {[object Object]: "valueB"}
// 由于keyA和keyB都是[object Object],所以上面的被覆盖
```

------

## 属性的可枚举性

对象的每个属性都有一个描述对象（Descriptor），用来控制该属性的行为。`Object.getOwnPropertyDescriptor`方法可以获取该属性的描述对象。

```javascript
let obj = { foo: 123 };
Object.getOwnPropertyDescriptor(obj, 'foo')
//  {
//    value: 123,
//    writable: true,
//    enumerable: true,
//    configurable: true
//  }
```

描述对象的`enumerable`属性，称为“可枚举性”，如果该属性为`false`，就表示某些操作会忽略当前属性。 

目前，有四个操作会忽略`enumerable`为`false`的属性。

- `for...in`循环：只遍历对象自身的和继承的可枚举的属性。
- `Object.keys()`：返回对象自身的所有可枚举的属性的键名。
- `JSON.stringify()`：只串行化对象自身的可枚举的属性。
- `Object.assign()`： 忽略`enumerable`为`false`的属性，只拷贝对象自身的可枚举的属性。

引入“可枚举”（`enumerable`）这个概念的最初目的，就是**让某些属性可以规避掉`for...in`操作，不然所有内部属性和方法都会被遍历到**。比如，对象原型的`toString`方法，以及数组的`length`属性，就通过“可枚举性”，从而避免被`for...in`遍历到 

```javascript
Object.getOwnPropertyDescriptor(Object.prototype, 'toString').enumerable
// false

Object.getOwnPropertyDescriptor([], 'length').enumerable
// false
```

另外，ES6 规定，所有 Class 的原型的方法都是不可枚举的。

```javascript
Object.getOwnPropertyDescriptor(class {foo() {}}.prototype, 'foo').enumerable
// false
```

总的来说，操作中引入继承的属性会让问题复杂化，大多数时候，我们只关心对象自身的属性。所以，**尽量不要用`for...in`循环，而用`Object.keys()`代替**。 

------

## 属性的遍历

1. ### for...in

`for...in`循环遍历对象自身的和继承的可枚举属性（不含 Symbol 属性）。 

2. ###  Object.keys(obj)

`Object.keys`返回一个数组，包括对象自身的（不含继承的）所有可枚举属性（不含 Symbol 属性）的键名。 

3. ###  Object.getOwnPropertyNames(obj)

`Object.getOwnPropertyNames`返回一个数组，包含对象自身的（不含继承的）所有属性（不含 Symbol 属性，但是包括不可枚举属性）的键名。 

4. ###  Object.getOwnPropertySymbols(obj)

`Object.getOwnPropertySymbols`返回一个数组，包含对象自身的（不含继承的）所有 Symbol 属性的键名。 

5. ### Reflect.ownKeys(obj) 

`Reflect.ownKeys`返回一个数组，包含对象自身的（不含继承的）所有键名，不管键名是 Symbol 或字符串，也不管是否可枚举。 

以上的 5 种方法遍历对象的键名，都遵守同样的属性遍历的次序规则。

- 首先遍历所有数值键，按照数值升序排列。
- 其次遍历所有字符串键，按照加入时间升序排列。
- 最后遍历所有 Symbol 键，按照加入时间升序排列。

```javascript
Reflect.ownKeys({ [Symbol()]:0, b:0, 10:0, 2:0, a:0 })
// ['2', '10', 'b', 'a', Symbol()]
// 先遍历数值键10, 2 按照升序排列
// 其次遍历字符串键'b', 'a' 按照加入时间排列
// 最后遍历所有Symbol键 按照加入时间排列
```

------

## super关键字

`this`关键字总是指向函数所在的当前对象，ES6新增`super`关键字，指向当前对象的原型对象

```javascript
const proto = {
  foo: 'hello'
};

const obj = {
  foo: 'world',
  find() {
    return super.foo;
  }
};

Object.setPrototypeOf(obj, proto); // 设置指定的对象的原型
obj.find() // "hello"
```

注意，**`super`关键字表示原型对象时，只能用在对象的方法之中**，用在其他地方都会报错。 

```javascript
// 报错,super在属性中
const obj = {
  foo: super.foo
}

// 报错,super用在函数中,然后赋值给foo属性
const obj = {
  foo: () => super.foo
}

// 报错,和第二种写法一样的错误
const obj = {
  foo: function () {
    return super.foo
  }
}
```

**目前，只有对象方法的简写法可以让`JavaScript`引擎确认，定义的是对象的方法。** 

```javascript
const proto = {
  x: 'hello',
  foo() {
    console.log(this.x);
  },
};

const obj = {
  x: 'world',
  foo() {
    super.foo();
  }
}

Object.setPrototypeOf(obj, proto);

obj.foo() // "world"
```

还有一种情况值得注意，如上面的代码所示，`super.foo`指向原型对象`proto`的`foo`方法，但是绑定的`this`还是当前对象`obj`，因此依然输出`world`

------

## 对象的扩展运算符

### 解构赋值

对象的解构赋值用于从一个对象取值，相当于**将目标对象自身的所有可遍历的（enumerable）、但尚未被读取的属性，分配到指定的对象上面**。所有的键和它们的值，都会拷贝到新对象上面。 

```javascript
let { x, y, ...z } = { x: 1, y: 2, a: 3, b: 4 };
x // 1
y // 2
z // { a: 3, b: 4 }
```

由于**解构赋值要求等号右边是一个对象**，所以如果等号右边是`undefined`或`null`，就会报错，因为它们无法转为对象。

```javascript
let { ...z } = null; // 运行时错误
let { ...z } = undefined; // 运行时错误
```

注意，**解构赋值的拷贝是浅拷贝**，在解构赋值那一章已经提到过，无论是数组解构还是对象解构，都要注意这一点

```javascript
let obj = { a: { b: 1 } };
let { ...x } = obj;
obj.a.b = 2;
x.a.b // 2
```

另外，扩展运算符的解构赋值，不能复制继承自原型对象的属性**。 

```javascript
const o = Object.create({ x: 1, y: 2 }); // Object.create()添加的属性在原型下
// o.__proto__ { x: 1, y: 2 }
o.z = 3;

let { x, ...newObj } = o;
let { y, z } = newObj;
x // 1
y // undefined
z // 3
```

上面代码中，变量`x`是**单纯的解构赋值，所以可以读取对象`o`继承的属性。**变量`y`和`z`是扩展运算符的解构赋值，只能读取对象`o`自身的属性，所以变量`z`可以赋值成功，变量`y`取不到值。 

### 扩展运算符

对象的扩展运算符（`...`）用于**取出参数对象的所有可遍历属性，拷贝到当前对象之中**。 

```javascript
let z = { a: 3, b: 4 };
let n = { ...z };
n // { a: 3, b: 4 }
```

由于数组是特殊的对象，所以对象的扩展运算符也可以用于数组。**注意扩展运算符除此之外，只能在函数中使用。**

```javascript
let foo = { ...['a', 'b', 'c'] };
foo
// {0: "a", 1: "b", 2: "c"}
```

如果扩展运算符后面是一个空对象，则没有任何效果。这里和数组的扩展一致，`...[]`也不会产生任何效果。

```javascript
{...{}, a: 1}
// { a: 1 }
```

如果扩展运算符后面不是对象，则会自动将其转为对象。

```javascript
// 等同于 {...Object(1)}
{...1} // {}
```

上面代码中，扩展运算符后面是整数`1`，会自动转为数值的包装对象`Number{1}`。**由于该对象没有自身属性，所以返回一个空对象**。

```javascript
// 等同于 {...Object(true)}
{...true} // {}

// 等同于 {...Object(undefined)}
{...undefined} // {}

// 等同于 {...Object(null)}
{...null} // {}
```

但是，**如果扩展运算符后面是字符串，它会自动转成一个类似数组的对象**，因此返回的不是空对象。

```javascript
{...'hello'}
// {0: "h", 1: "e", 2: "l", 3: "l", 4: "o"}
```

对象的扩展运算符等同于使用`Object.assign()`方法。

```javascript
let aClone = { ...a };
// 等同于
let aClone = Object.assign({}, a);
```

扩展运算符可以用于合并两个对象。

```javascript
let ab = { ...a, ...b };
// 等同于
let ab = Object.assign({}, a, b);
```

如果用户自定义的属性，放在扩展运算符后面，则扩展运算符内部的同名属性会被覆盖掉。

```javascript
let aWithOverrides = { ...a, x: 1, y: 2 };
// 等同于
let aWithOverrides = { ...a, ...{ x: 1, y: 2 } };
// 等同于
let x = 1, y = 2, aWithOverrides = { ...a, x, y };
// 等同于
let aWithOverrides = Object.assign({}, a, { x: 1, y: 2 });
```

与数组的扩展运算符一样，对象的扩展运算符后面可以跟表达式。

```javascript
const obj = {
  ...(x > 1 ? {a: 1} : {}),
  b: 2,
};
```

扩展运算符的参数对象之中，如果有取值函数`get`，这个函数是会执行的。

```javascript
let a = {
  get x() {
    throw new Error('not throw yet');
  }
}

let aWithXGetter = { ...a }; // 报错
```

上面例子中，**取值函数`get`在扩展`a`对象时会自动执行**，导致报错。

------

## 链判断运算符

如果读取对象内部的某个属性，往往需要判断一下该对象是否存在。比如，要读取`message.body.user.firstName`，安全的写法是写成下面这样。

```javascript
const firstName = (message
  && message.body
  && message.body.user
  && message.body.user.firstName) || 'default';
```

或者使用三元运算符`?:`，判断一个对象是否存在。

```javascript
const fooInput = myForm.querySelector('input[name=foo]')
const fooValue = fooInput ? fooInput.value : undefined
```

这样的层层判断非常麻烦，因此 [ES2020](https://github.com/tc39/proposal-optional-chaining) 引入了“链判断运算符”（optional chaining operator）`?.`，简化上面的写法。

```javascript
const firstName = message?.body?.user?.firstName || 'default';
const fooValue = myForm.querySelector('input[name=foo]')?.value
```

上面代码使用了`?.`运算符，**直接在链式调用的时候判断，左侧的对象是否为`null`或`undefined`。如果是的，就不再往下运算，而是返回`undefined`**。 

链判断运算符有三种用法。

- `obj?.prop` // 对象属性
- `obj?.[expr]` // 同上
- `func?.(...args)` // 函数或对象方法的调用

下面是判断对象方法是否存在，如果存在就立即执行的例子。

```javascript
iterator.return?.() // 第三种用法,iterator对象如果有return函数则立即执行
```

下面是这个运算符常见的使用形式，以及不使用该运算符时的等价形式。

```javascript
a?.b
// 等同于
a == null ? undefined : a.b

a?.[x]
// 等同于
a == null ? undefined : a[x]

a?.b()
// 等同于
a == null ? undefined : a.b()
// 如果a.b不是函数,会报错

a?.()
// 等同于
a == null ? undefined : a()
// 如果a不是函数,也会报错
```

链判断运算符还有以下几个注意点

- 短路机制

```javascript
a?.[++x]
// 等同于
a == null ? undefined : a[++x]
```

如果`a`是`undefined`或`null`，那么`x`不会进行递增运算

- `delete`运算符

```javascript
delete a?.b
// 等同于
a == null ? undefined : delete a.b
```

 如果`a`是`undefined`或`null`，会直接返回`undefined`，而不会进行`delete`运算 

- 括号的影响

 如果属性链有圆括号，**链判断运算符对圆括号外部没有影响，只对圆括号内部有影响**。 

```javascript
(a?.b).c
// 等价于
(a == null ? undefined : a.b).c
```

 不管`a`对象是否存在，圆括号后面的`.c`总是会执行 

- 报错场合

以下写法是禁止的，会报错。

```javascript
// 构造函数
new a?.()
new a?.b()

// 链判断运算符的右侧有模板字符串
a?.`{b}`
a?.b`{c}`

// 链判断运算符的左侧是 super
super?.()
super?.foo

// 链运算符用于赋值运算符左侧
a?.b = c
```

- 右侧不得为十进制数值

为了保证兼容以前的代码，允许`foo?.3:0`被解析成`foo ? .3 : 0`，因此规定如果`?.`后面紧跟一个十进制数字，**那么`?.`不再被看成是一个完整的运算符，而会按照三元运算符进行处理**，也就是说，那个小数点会归属于后面的十进制数字，形成一个小数。 

------

## Null判断运算符

读取对象属性的时候，如果某个属性的值是`null`或`undefined`，有时候需要为它们指定默认值。常见做法是通过`||`运算符指定默认值。

```javascript
const headerText = response.settings.headerText || 'Hello, world!';
const animationDuration = response.settings.animationDuration || 300;
const showSplashScreen = response.settings.showSplashScreen || true;
```

上面的三行代码都通过`||`运算符指定默认值，但是这样写是错的。开发者的原意是，只要属性的值为`null`或`undefined`，默认值就会生效，**但是属性的值如果为空字符串或`false`或`0`，默认值也会生效。** 

 [ES2020](https://github.com/tc39/proposal-nullish-coalescing) 引入了一个新的 Null 判断运算符`??`。它的行为类似`||`，但是只有运算符左侧的值为`null`或`undefined`时，才会返回右侧的值。 

这个运算符的一个目的，就是跟链判断运算符`?.`配合使用，为`null`或`undefined`的值设置默认值。

```javascript
const animationDuration = response.settings?.animationDuration ?? 300;
```

这个运算符很适合判断函数参数是否赋值。

```javascript
function Component(props) {
  const enable = props.enabled ?? true;
}
```

上面代码判断`props`参数的`enabled`属性是否赋值，等同于下面的写法。

```javascript
function Component(props) {
  const {
    enabled: enable = true,
  } = props;
  // 通过解构赋值,并设置默认值,达到同样的效果
}
```

 **如果多个逻辑运算符一起使用，必须用括号表明优先级，否则会报错。** 

```javascript
// 报错
lhs && middle ?? rhs
lhs ?? middle && rhs
lhs || middle ?? rhs
lhs ?? middle || rhs

// 加入表明优先级的括号
(lhs && middle) ?? rhs;
lhs && (middle ?? rhs);

(lhs ?? middle) && rhs;
lhs ?? (middle && rhs);

(lhs || middle) ?? rhs;
lhs || (middle ?? rhs);

(lhs ?? middle) || rhs;
lhs ?? (middle || rhs);
```