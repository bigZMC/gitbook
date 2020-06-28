允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这被称为解构。

## 数组的解构赋值

### 基本用法

```javascript
let [foo, [[bar], baz]] = [1, [[2], 3]];
foo // 1
bar // 2
baz // 3

let [ , , third] = ["foo", "bar", "baz"];
third // "baz"

let [x, , y] = [1, 2, 3];
x // 1
y // 3

let [head, ...tail] = [1, 2, 3, 4];
head // 1
tail // [2, 3, 4]

let [x, y, ...z] = ['a'];
x // "a"
y // undefined
z // []

// 解构不成功，变量的值就等于undefined
let [foo] = [];
let [bar, foo] = [1];

foo // undefined
```

另外一种情况是不完全解构，即等号左边的模式，只匹配一部分的等号右边的数组。

```javascript
let [x, y] = [1, 2, 3];
x // 1
y // 2

let [a, [b], d] = [1, [2, 3], 4];
a // 1
b // 2
d // 4
```

如果等号右边不是可遍历的解构，那么将会报错。

```javascript
// 报错
let [foo] = 1;
let [foo] = false;
let [foo] = NaN;
let [foo] = undefined;
let [foo] = null;
let [foo] = {};
```

反之，**只要某种数据解构具有`Iterator`接口，都可以采用数组形式的解构赋值**。

### 默认值

解构赋值允许指定默认值

```javascript
let [foo = true] = [];
foo // true

let [x, y = 'b'] = ['a']; // x='a', y='b'
let [x, y = 'b'] = ['a', undefined]; // x='a', y='b'
```

 注意，`ES6`内部使用严格相等运算符（`===`），判断一个位置是否有值。  所以，只有当一个数组成员严格等于`undefined`，默认值才会生效。 

```javascript
let [x = 1] = [undefined];
x // 1

let [x = 1] = [null];
x // null
```

注意：默认值是一个表达式的话，那么这个表达式是**惰性求值**的，即只有在用到的时候，才会求值

```javascript
function f() {
  console.log('aaa');
}

// 因为x能取到值，所以函数f根本不会执行
let [x = f()] = [1];

// 因为x取不到值，所以会执行f函数
let [x = f()] = [];
// aaa
```

还有一个需要注意的是，**执行顺序**

```javascript
let [x = 1, y = x] = [];     // x=1; y=1

// 这里会先计算x为2，再给y赋值x的值，所以才会出现x = 2，y = 2
let [x = 1, y = x] = [2];    // x=2; y=2
```

------

## 对象的解构赋值

### 基本用法

```javascript
let { foo, bar } = { foo: 'aaa', bar: 'bbb' };
foo // 'aaa'
bar // 'bbb'
```

对象的解构和数组有一个重要的不同。数组的元素是次序排列的；而对象的属性没有次序，变量必须和属性同名，才能取到正确的值。

```javascript
let { bar, foo } = { foo: 'aaa', bar: 'bbb' };
foo // "aaa"
bar // "bbb"

let { baz } = { foo: 'aaa', bar: 'bbb' };
baz // undefined
```

变量名和属性名不一致，写法如下

```javascript
let { foo: baz } = { foo: 'aaa', bar: 'bbb' };
baz // "aaa"

let obj = { first: 'hello', last: 'world' };
let { first: f, last: l } = obj;
f // 'hello'
l // 'world'
```

所以，对象的解构赋值是下面形式的简写

```javascript
let { foo: foo, bar: bar } = { foo: 'aaa', bar: 'bbb' };
```

也就是说，对象的解构赋值的内部机制，是**先找到同名的属性，然后再赋给对应的变量**。

和数组的解构一样，对象解构也能嵌套使用，也可以和数组的解构混合使用。

```javascript
let obj = {
  p: [
    'Hello',
    { y: 'World' }
  ]
};

let { p: [x, { y }] } = obj;
x // "Hello"，x的获取是通过数组的解构
y // "World"
```

嵌套赋值的示例如下

```javascript
let obj = {};
let arr = [];

({ foo: obj.prop, bar: arr[0] } = { foo: 123, bar: true });

obj // {prop:123}，obj声明的时候是{}，解构的时候根据等式右边的变量为其增加了prop属性
arr // [true]，和对象解构同理，声明的时候是空数组
```

如果解构模式是嵌套的对象，而且子对象所在的父属性不存在，那么将会报错。因为**内部机制是先找到同名属性**，因为找不到的时候是`undefined`，再向下找子属性就会报错。

```javascript
// 报错，此时foo为undefined，foo.bar就会报错
let {foo: {bar}} = {baz: 'baz'};
```

### 默认值

对象的解构也可以指定默认值。

```javascript
var {x = 3} = {};
// 等同于 var {x: x = 3} = {}; 和第三例相同
x // 3

var {x, y = 5} = {x: 1};
x // 1
y // 5

var {x: y = 3} = {};
y // 3

var {x: y = 3} = {x: 5};
y // 5

var { message: msg = 'Something went wrong' } = {};
msg // "Something went wrong"
```

和数组设置默认值一样，生效条件是，对象的属性值严格等于`undefined`

```javascript
var {x = 3} = {x: undefined};
x // 3

var {x = 3} = {x: null};
x // null
```

### 注意点

- 如果将一个已经声明的变量用于解构赋值，必须非常小心


```javascript
// 错误的写法
let x;
{x} = {x: 1};
// SyntaxError: syntax error
```

上面的错误是由于`{x}`被理解成一个代码块，从而发生语法错误。只有不将大括号写在行首，避免JavaScript将其理解为代码块，才能解决该问题。

```javascript
// 正确的写法
let x;
({x} = {x: 1});
```

- 由于数组本质是特殊的对象，因为可以对数组进行对象属性的解构

```javascript
let arr = [1, 2, 3];
let {0 : first, [arr.length - 1] : last} = arr;

// 等同于 let {0 : first, [2] : last} = {0: 1, 1: 2, 2: 3}
// [2]的写法参照对象扩展-属性名表达式
first // 1
last // 3
```

------

## 字符串的解构赋值

当字符串使用解构赋值时，字符串被转换成了一个类似数组的对象

```javascript
const [a, b, c, d, e] = 'hello';
a // "h"
b // "e"
c // "l"
d // "l"
e // "o"
```

类似数组的对象都有`length`属性，因此还可以对该属性解构赋值

```javascript
let {length : len} = 'hello';
len // 5
```

------

## 函数参数的解构赋值

函数的参数也可以使用解构赋值

```javascript
function add([x, y]){
  return x + y;
}

add([1, 2]); // 3
```

函数`add`的参数表面上是一个数组，但是传入参数时，数组参数被结构成了变量 `x`和`y`。  对于函数内部的代码来说，它们能感受到的参数就是`x`和`y`。 

```javascript
[[1, 2], [3, 4]].map(([a, b]) => a + b);
// [ 3, 7 ]
```

函数参数的解构也可以使用默认值。

```javascript
function move({x = 0, y = 0} = {}) {
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, 0]
move({}); // [0, 0]
move(); // [0, 0]
```

下面的写法会得到不一样的结果

```javascript
function move({x, y} = { x: 0, y: 0 }) {
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, undefined]
move({}); // [undefined, undefined]
move(); // [0, 0]
```

 上面代码是为函数`move`的参数指定默认值，而不是为变量`x`和`y`指定默认值，所以会得到与前一种写法不同的结果。 

------

## 用途

- 交换变量的值

```javascript
let x = 1;
let y = 2;

[x, y] = [y, x];

x // 2
y // 1
```

- 从函数返回多个值

函数只能返回一个值，如果要返回多个值，只能将它们放在数组或对象里返回。有了解构赋值，取出这些值就非常方便。 

```javascript
// 返回一个数组
function example() {
  return [1, 2, 3];
}
let [a, b, c] = example();

// 返回一个对象
function example() {
  return {
    foo: 1,
    bar: 2
  };
}
let { foo, bar } = example();
```

- 函数参数的定义

```javascript
// 参数是一组有次序的值
function f([x, y, z]) { ... }
f([1, 2, 3]);

// 参数是一组无次序的值
function f({x, y, z}) { ... }
f({z: 3, y: 2, x: 1});
```

- 函数参数的默认值

```javascript
jQuery.ajax = function (url, {
  async = true,
  beforeSend = function () {},
  cache = true,
  complete = function () {},
  crossDomain = false,
  global = true,
  // ... more config
} = {}) {
  // ... do stuff
};
```

- 遍历Map结构

```javascript
const map = new Map()
map.set('first', 'hello')
map.set('second', 'world')

for (let item of map) {
  console.log(item)
}
// ['first', 'hello']
// ['second', 'world']

for (let [key, value] of map) {
  console.log(key + ' is ' + value)
}
// first is hello
// second is world

// 获取键名
for (let [key] of map) {
  // ...
}

// 获取键值
for (let [,value] of map) {
  // ...
}
```