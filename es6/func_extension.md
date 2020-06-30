## 函数参数的默认值

### 基本用法

`ES6`之前，不能直接为函数的参数指定默认值，只能采用变通的方法。 

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

`ES6`允许为函数的参数设置默认值，即直接写在参数定义的后面。**只有在`y === undefined`的情况下，才会使用默认值。** 

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

`ES6`引入`rest`参数（形式为`...变量名`），用于获取函数的多余参数，这样就不需要使用`arguments`对象了。**rest 参数搭配的变量是一个数组**，该变量将多余的参数放入数组中。 

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

`ES6`允许使用“箭头”（`=>`）定义函数。

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

上面代码中，`setTimeout`的参数是一个箭头函数，捕获自己在**定义时**（注意，是定义时，不是调用时）所处的**外层执行环境的this**。这个箭头函数的定义生效是在`foo`函数生成时，而它的真正执行要等到 100 毫秒后。如果是普通函数，执行时`this`应该指向全局对象`window`，这时应该输出`21`。但是，箭头函数导致`this`总是指向函数定义生效时所在的对象（本例是`{id: 42}`），所以输出的是`42`。 

**箭头函数可以让`setTimeout`里面的`this`，绑定定义时所在的作用域，而不是指向运行时所在的作用域。** 

```javascript
function Timer() {
  this.s1 = 0;
  this.s2 = 0;
  // 箭头函数
  setInterval(() => this.s1++, 1000);
  // 普通函数
  setInterval(function () {
    this.s2++;
  }, 1000);
}

var timer = new Timer();
var s2 = 0;

setTimeout(() => console.log('s1: ', timer.s1), 3100);
setTimeout(() => console.log('s2: ', timer.s2, s2), 3100);
// s1: 3
// s2: 0 3
```

`Timer`函数内部设置了两个定时器，分别使用了箭头函数和普通函数。前者的`this`绑定定义时所在的作用域（即`Timer`函数），后者的`this`指向运行时所在的作用域（即全局对象）。所以3100毫秒后，`timer1.s1`被更新了3次，而`timer1.s2`一次都没有更新，被更新的是`window.s2`。

箭头函数可以让`this`指向固定化，这种特性很有利于封装回调函数。 

```javascript
var handler = {
  id: '123456',

  init: function() {
    // 这里this指向handler对象，如果是普通函数，this会指向document
    document.addEventListener('click', event => this.doSomething(event.type), false);
  },

  doSomething: function(type) {
    console.log('Handling ' + type  + ' for ' + this.id);
  }
}
```

`this`指向的固定化，并不是因为箭头函数内部有绑定`this`的机制，实际原因是**箭头函数根本没有自己的`this`，导致内部的`this`就是外层代码块（必须是构成单独作用域）的`this`。**正是因为它没有`this`，所以也就不能用作构造函数。 

所以，箭头函数转成 ES5 的代码如下。

```javascript
// ES6
function foo() {
  setTimeout(() => {
    console.log('id:', this.id);
  }, 100);
}

// ES5
function foo() {
  var _this = this;

  setTimeout(function () {
    console.log('id:', _this.id);
  }, 100);
}
```

### 不适用场合

第一个场合是定义对象的方法，且该方法内部包括`this`。

```javascript
const cat = {
  lives: 9,
  jumps: () => {
    this.lives--;
  }
}
```

上面代码中，`cat.jumps()`方法是一个箭头函数，这是错误的。调用`cat.jumps()`时，如果是普通函数，该方法内部的`this`指向`cat`；如果写成上面那样的箭头函数，使得`this`指向全局对象，因此不会得到预期结果。这是因为**对象不构成单独的作用域，导致`jumps`箭头函数定义时的作用域就是全局作用域。** 

第二个场合是**需要动态`this`**的时候，也不应使用箭头函数。

```javascript
var button = document.getElementById('press');
button.addEventListener('click', () => {
  // 这里this指向全局对象
  this.classList.toggle('on');
});
```

### 嵌套的箭头函数

```javascript
function mul(a) {
  return function(b) {
    return function(c) {
      return a * b * c
    }
  }
}

mul(2)(3)(4) // 24

// 箭头函数嵌套
const mul = (a) => (b) => (c) => a * b * c
```

下面是一个部署管道机制（`pipeline`）的例子，即前一个函数的输出是后一个函数的输入。

```javascript
const pipeline = (...funcs) =>
  val => funcs.reduce((a, b) => b(a), val);

const plus1 = a => a + 1;
const mult2 = a => a * 2;
const addThenMult = pipeline(plus1, mult2);

addThenMult(5)
// 12
// 第一次执行plus1(5)，再执行mult2(plus1(5))
```

------

## 尾调用优化

### 什么是尾调用

尾调用（Tail Call）是函数式编程的一个重要概念，本身非常简单，一句话就能说清楚，就是指**某个函数的最后一步是调用另一个函数**。

```javascript
function f(x){
  return g(x);
}
```

上面代码中，函数`f`的最后一步是调用函数`g`，这就叫尾调用。

以下三种情况，都不属于尾调用。

```javascript
// 情况一，调用函数g之后，还有赋值操作
function f(x){
  let y = g(x);
  return y;
}

// 情况二，和情况一相同，调用后还有操作，即使写在一行内也不算尾调用
function f(x){
  return g(x) + 1;
}

// 情况三，相当于return了undefined，所以也不算是尾调用
function f(x){
  g(x);
  // return undefined
}
```

尾调用不一定出现在函数尾部，**只要是最后一步操作即可**。

```javascript
function f(x) {
  if (x > 0) {
    return m(x)
  }
  return n(x);
}
```

### 尾调用优化

函数调用会在内存形成一个“调用记录”，又称“调用帧”（call frame），保存调用位置和内部变量等信息。如果在函数`A`的内部调用函数`B`，那么在`A`的调用帧上方，还会形成一个`B`的调用帧。等到`B`运行结束，将结果返回到`A`，`B`的调用帧才会消失。如果函数`B`内部还调用函数`C`，那就还有一个`C`的调用帧，以此类推。所有的调用帧，就形成一个**“调用栈”（call stack）**。 

**尾调用由于是函数的最后一步操作，所以不需要保留外层函数的调用帧**，因为调用位置、内部变量等信息都不会再用到了，只要直接用内层函数的调用帧，取代外层函数的调用帧就可以了。

```javascript
function f() {
  let m = 1;
  let n = 2;
  return g(m + n);
}
f();

// 等同于
function f() {
  return g(3);
}
f();

// 等同于
g(3);
```

这就叫做“尾调用优化”（Tail call optimization），即只保留内层函数的调用帧。如果所有函数都是尾调用，那么完全可以做到每次执行时，调用帧只有一项，这将大大节省内存。这就是“尾调用优化”的意义。 

注意，只有不再用到外层函数的内部变量，内层函数的调用帧才会取代外层函数的调用帧，否则就无法进行“尾调用优化”。

```javascript
function addOne(a){
  var one = 1;
  function inner(b){
    return b + one;
  }
  // 内层函数inner用到了外层函数addOne的内部变量one
  return inner(a);
}
```

上面的函数不会进行尾调用优化，因为内层函数`inner`用到了外层函数`addOne`的内部变量`one`。

### 尾递归

函数调用自身，称为递归。如果尾调用自身，就称为尾递归。

递归非常耗费内存，因为需要同时保存成千上百个调用帧，很容易发生“栈溢出”错误（stack overflow）。但对于尾递归来说，由于只存在一个调用帧，所以永远不会发生“栈溢出”错误。

```javascript
function factorial(n) {
  if (n === 1) return 1;
  return n * factorial(n - 1);
}

factorial(5) // 120
```

上面代码是一个阶乘函数，计算`n`的阶乘，最多需要保存`n`个调用记录，复杂度 O(n) 。 

如果改写成尾递归，只保留一个调用记录，复杂度 O(1) 。

```javascript
function factorial(n, total) {
  if (n === 1) return total;
  return factorial(n - 1, n * total);
}

factorial(5, 1) // 120
```

还有一个比较著名的例子，就是计算 Fibonacci 数列，也能充分说明尾递归优化的重要性。

非尾递归的 Fibonacci 数列实现如下。

```javascript
function Fibonacci (n) {
  if ( n <= 1 ) {return 1};

  return Fibonacci(n - 1) + Fibonacci(n - 2);
}

Fibonacci(10) // 89
Fibonacci(100) // 超时
Fibonacci(500) // 超时
```

尾递归优化过的 Fibonacci 数列实现如下。

```javascript
function Fibonacci2 (n , ac1 = 1 , ac2 = 1) {
  if( n <= 1 ) {return ac2};

  return Fibonacci2(n - 1, ac2, ac1 + ac2);
}

Fibonacci2(100) // 573147844013817200000
Fibonacci2(1000) // 7.0330367711422765e+208
Fibonacci2(10000) // Infinity
```

由此可见，“尾调用优化”对递归操作意义重大，所以一些函数式编程语言将其写入了语言规格。`ES6`亦是如此，第一次明确规定，所有`ECMAScript`的实现，都必须部署“尾调用优化”。这就是说，`ES6`中只要使用尾递归，就不会发生栈溢出（或者层层递归造成的超时），相对节省内存。

### 递归函数的改写

尾递归的实现，往往需要改写递归函数，确保最后一步只调用自身。做到这一点的方法，就是**把所有用到的内部变量改写成函数的参数**。比如上面的例子，阶乘函数 factorial 需要用到一个中间变量`total`，那就把这个中间变量改写成函数的参数。这样做的缺点就是不太直观，第一眼很难看出来，为什么计算`5`的阶乘，需要传入两个参数`5`和`1`？ 

两个方法可以解决这个问题。方法一是在尾递归函数之外，**再提供一个正常形式的函数**。

```javascript
function tailFactorial(n, total) {
  if (n === 1) return total;
  return tailFactorial(n - 1, n * total);
}

function factorial(n) {
  return tailFactorial(n, 1);
}

factorial(5) // 120
```

上面代码通过一个正常形式的阶乘函数`factorial`，调用尾递归函数`tailFactorial`，看起来就正常多了。

第二种方法就简单多了，就是采用 ES6 的函数默认值。

```javascript
function factorial(n, total = 1) {
  if (n === 1) return total;
  return factorial(n - 1, n * total);
}

factorial(5) // 120
```

### 尾递归优化的实现

尾递归优化只在严格模式下生效，那么正常模式下，就要自己实现尾递归优化。它的原理非常简单。尾递归之所以需要优化，**原因是调用栈太多，造成溢出，那么只要减少调用栈，就不会溢出**，可以采用“循环”换掉“递归”。 

```javascript
function tco(f) {
  var value;
  var active = false;
  var accumulated = [];

  return function accumulator() {
    accumulated.push(arguments);
    if (!active) {
      active = true;
      while (accumulated.length) {
        value = f.apply(this, accumulated.shift());
      }
      active = false;
      return value;
    }
  };
}

var sum = tco(function(x, y) {
  if (y > 0) {
    return sum(x + 1, y - 1)
  }
  else {
    return x
  }
});

sum(1, 100000)
// 100001
```

`tco`函数是尾递归优化的实现，它的奥妙就在于状态变量`active`。默认情况下，这个变量是不激活的。一旦进入尾递归优化的过程，这个变量就激活了。然后，每一轮递归`sum`返回的都是`undefined`，所以就避免了递归执行；而`accumulated`数组存放每一轮`sum`执行的参数，总是有值的，这就保证了`accumulator`函数内部的`while`循环总是会执行。这样就很巧妙地将“递归”改成了“循环”，而后一轮的参数会取代前一轮的参数，保证了调用栈只有一层。 

