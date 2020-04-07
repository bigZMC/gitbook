## 函数参数的默认值

### 基本用法

ES6 之前，不能直接为函数的参数指定默认值，只能采用变通的方法。 

```javascript
function log(x, y) {
  y = y || 'World';
  console.log(x, y);
}

log('Hello') // Hello World
log('Hello', 'China') // Hello China
log('Hello', '') // Hello World
```

这种写法的缺点在于，**如果参数`y`赋值了，但是对应的布尔值为`false`，则该赋值不起作用**。就像上面代码的最后一行，参数`y`等于空字符，结果被改为默认值。 

为了避免这个问题，通常需要先判断一下参数`y`是否被赋值，如果没有，再等于默认值。 

```javascript
if (typeof y === 'undefined') {
  y = 'World';
}
```

ES6 允许为函数的参数设置默认值，即直接写在参数定义的后面。**只有在`y === undefined`的情况下，才会使用默认值。** 

```javascript
function log(x, y = 'World') {
  console.log(x, y);
}

log('Hello') // Hello World
log('Hello', 'China') // Hello China
log('Hello', '') // Hello
```

**参数变量是默认声明的，所以不能用`let`或`const`再次声明。** 

```javascript
function foo(x = 5) {
  let x = 1; // error
}

function foo(x = 5) {
  const x = 2; // error
}
```

使用参数默认值时，函数不能有同名参数。 

```javascript
// 不报错，这种情况下，第二个x会覆盖第一个x的值
function foo(x, x, y) {
  console.log(x, y)
}

foo(1, 2, 3) // 2, 3
foo(1, 2) // 2, undefined
foo(1) // undefined, undefined


// 报错
function foo(x, x, y = 1) {
  // ...
}
// SyntaxError: Duplicate parameter name not allowed in this context
```

另外，一个容易忽略的地方是，参数默认值不是传值的，而是每次都重新计算默认值表达式的值。也就是说，**参数默认值是惰性求值的**。 

```javascript
let x = 99;
function foo(p = x + 1) {
  console.log(p);
}

foo() // 100

x = 100;
foo() // 101
```

上面代码中，参数`p`的默认值是`x + 1`。这时，每次调用函数`foo`，都会重新计算`x + 1`，而不是默认`p`等于 100。 

还需要注意以下两点：

- 参数默认值可以与解构赋值结合使用，具体可以参考[变量的解构赋值/函数参数的解构赋值](destructuring.md)章节
- **参数默认值应该放在最末尾(尾参数)**，比较容易看出来

### 函数的length属性

`length`属性是函数的期望参数个数。指定了默认值以后，函数的`length`属性，**将返回没有指定默认值的参数个数**。也就是说，指定了默认值后，`length`属性将失真。  

```javascript
(function (a) {}).length // 1
(function (a = 5) {}).length // 0
(function (a, b, c = 5) {}).length // 2
```

这是因为`length`属性的含义是，该函数预期传入的参数个数。某个参数指定默认值以后，预期传入的参数个数就不包括这个参数了。同理，后文的 **rest 参数也不会计入`length`属性**。如果设置了默认值的参数不是**尾参数**，那么`length`属性也不再计入后面的参数了。 

```javascript
(function(...args) {}).length // 0

(function (a = 0, b, c) {}).length // 0，从第一个参数就是默认参数，后面的不计数
(function (a, b = 1, c) {}).length // 1
```

### 作用域

**一旦设置了参数的默认值，函数进行声明初始化时，参数会形成一个单独的作用域**（context）。等到初始化结束，这个作用域就会消失。这种语法行为，**在不设置参数默认值时，是不会出现的**。 

```javascript
var x = 1;

// 默认值变量x指向第一个参数x，而不是全局变量x
function f(x, y = x) {
  console.log(y);
}

f(2) // 2
```

```javascript
let x = 1;

// 参数y = x形成一个单独的作用域，变量x本身没有定义，所以指向外层的全局变量x
function f(y = x) {
  let x = 2;
  console.log(y);
}

f() // 1
```

下面两种情况会报错

```javascript
function f(y = x) {
  let x = 2;
  console.log(y);
}

// 全局变量x不存在
f() // ReferenceError: x is not defined
```

```javascript
var x = 1;

function foo(x = x) {
  // ...
}

// 参数x = x形成一个单独作用域。实际执行的是let x = x，由于暂时性死区的原因，这行代码会报错”x 未定义“
foo() // ReferenceError: x is not defined
```

函数的参数是一个函数，也遵循这个规则

```javascript
let foo = 'outer';

function bar(func = () => foo) {
  let foo = 'inner';
  console.log(func());
}

bar(); // outer

// 外部没有定义全局函数foo，就会报错
function bar(func = () => foo) {
  let foo = 'inner';
  console.log(func());
}

bar() // ReferenceError: foo is not defined
```

下面是一个更复杂的例子

```javascript
var x = 1;
function foo(x, y = function() { x = 2; }) {
  var x = 3;
  y();
  console.log(x);
}

foo() // 3
x // 1
```

这里函数`foo`的参数形成了一个单独作用域。因为`foo`函数内部的`x`是用`var`声明，和参数`x`不在同一作用域，因此在函数`foo`内部调用`y`函数，并不会影响内部变量`x`,所以输出的是3

```javascript
var x = 1;
function foo(x, y = function() { x = 2; }) {
  x = 3;
  y();
  console.log(x);
}

foo() // 2
x // 1
```

如果内部变量`x`不用`var`声明，那么就和参数中的`x`相同，`y`函数就能修改到，所以输出2。由于外部的变量`x`一直没有参与运算，所以两个例子中都输出1。

------

## rest参数

ES6 引入 rest 参数（形式为`...变量名`），用于获取函数的多余参数，这样就不需要使用`arguments`对象了。**rest 参数搭配的变量是一个数组**，该变量将多余的参数放入数组中。 

```javascript
function add(...values) {
  let sum = 0;

  for (var val of values) {
    sum += val;
  }

  return sum;
}

// 可以传入任意数目的参数
add(2, 5, 3) // 10
```

下面是一个 rest 参数代替`arguments`变量的例子。 

```javascript
// arguments变量的写法
function sortNumbers() {
  return Array.prototype.slice.call(arguments).sort();
}

// rest参数的写法
const sortNumbers = (...numbers) => numbers.sort();
```

 注意，**rest 参数之后不能再有其他参数（即只能是最后一个参数）**，否则会报错。 

```javascript
// 报错
function f(a, ...b, c) {
  // ...
}
```

------

## 箭头函数

ES6 允许使用“箭头”（`=>`）定义函数。

```javascript
var f = v => v;

// 等同于
var f = function (v) {
  return v;
};
```

如果箭头函数不需要参数或需要多个参数，就使用一个圆括号代表参数部分。如果箭头函数的代码块部分多于一条语句，就要使用大括号将它们括起来，并且使用`return`语句返回。

```javascript
var f = () => 5;
// 等同于
var f = function () { return 5 };

var sum = (num1, num2) => num1 + num2;
// 等同于
var sum = function(num1, num2) {
  return num1 + num2;
};
```

由于大括号被解释为代码块，所以如果**箭头函数直接返回一个对象，必须在对象外面加上括号，否则会报错**。

```javascript
// 报错
let getTempItem = id => { id: id, name: "Temp" };

// 不报错
let getTempItem = id => ({ id: id, name: "Temp" });
let getTempItem = id => {
  return {
     id: id, name: "Temp"
  }
}
```

如果箭头函数**只有一行语句，且不需要返回值**，可以采用下面的写法，就不用写大括号了。

```javascript
let fn = () => void doesNotReturn();
// 等同于
let fn() {
  doesNotReturn();
}
```

- 如果返回一个对象，使用`()`包裹
- 如果箭头函数只有一行语句，且不需要返回值，使用`void`修饰

箭头函数还能和rest参数结合使用

```javascript
const numbers = (...nums) => nums;

numbers(1, 2, 3, 4, 5)
// [1,2,3,4,5]

const headAndTail = (head, ...tail) => [head, tail];

headAndTail(1, 2, 3, 4, 5)
// [1,[2,3,4,5]]
```

### 使用注意点

-  函数体内的`this`对象，就是**定义时所在的对象，而不是使用时所在的对象**。 
-  不可以当作构造函数，也就是说，不可以使用`new`命令，否则会抛出一个错误。 
-  不可以使用`arguments`对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替。 
-  不可以使用`yield`命令，因此箭头函数不能用作 Generator 函数。 

```javascript
function foo() {
  setTimeout(() => {
    console.log('id:', this.id);
  }, 100);
}

var id = 21;

foo.call({ id: 42 });
// id: 42
```

上面代码中，`setTimeout`的参数是一个箭头函数，这个箭头函数的定义生效是在`foo`函数生成时，而它的真正执行要等到 100 毫秒后。如果是普通函数，执行时`this`应该指向全局对象`window`，这时应该输出`21`。但是，箭头函数导致`this`总是指向函数定义生效时所在的对象（本例是`{id: 42}`），所以输出的是`42`。 