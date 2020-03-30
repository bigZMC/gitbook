## let命令

### 基本用法

let声明的变量只在其块级作用域中生效

```javascript
{
  let a = 1
  var b = 2
}

a // ReferenceError: a is not defined
b // 2
```

let适用于for循环的计数器，解决了用var声明，变量i属于全局的问题(这个问题还可以通过闭包形式解决)

```javascript
for (let i = 0; i < 10; i++) {
  // ...
}

i // ReferenceError: i is not defined
```

值得注意的是for循环有一个特别之处，就是设置循环变量的部门是一个父作用域，而循环内部是单独的子作用域

```javascript
for (let i = 0; i < 3; i++) {
  let i = 'abc';
  console.log(i);
}
// abc
// abc
// abc
```

### 不存在变量提升（暂时性死区）

```javascript
// var 的情况
console.log(foo); // 输出undefined
var foo = 2;

// let 的情况
console.log(bar); // 报错ReferenceError
let bar = 2;
```

因为不存在变量提升，所以在同一作用域中，存在let和const命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。

```javascript
var tmp = 123;

if (true) {
  tmp = 'abc'; // ReferenceError
  let tmp;
}
```

 **暂时性死区的本质就是，只要一进入当前作用域，所要使用的变量就已经存在了，但是不可获取，只有等到声明变量的那一行代码出现，才可以获取和使用该变量。** 

```javascript
if (true) {
  // TDZ开始
  tmp = 'abc'; // ReferenceError
  console.log(tmp); // ReferenceError

  let tmp; // TDZ结束
  console.log(tmp); // undefined

  tmp = 123;
  console.log(tmp); // 123
}
```

### 不允许重复声明

let不允许在相同作用域内，重复声明同一个变量。即使声明命令不同

```javascript
// 报错
function func() {
  let a = 10;
  var a = 1;
}

// 报错
function func() {
  let a = 10;
  let a = 1;
}
```

------

## 块级作用域

块级作用域为了兼容ES5，如果在其中声明了函数，将遵循如下规则

-  允许在块级作用域内声明函数。 
-  函数声明类似于var，即会提升到**全局作用域**或**函数作用域**的头部。 
-  同时，函数声明还会提升到所在的块级作用域的头部。 

```javascript
function f() { console.log('I am outside!'); }

(function () {
  if (false) {
    // 重复声明一次函数f
    function f() { console.log('I am inside!'); }
  }

  f();
}());

// 浏览器的 ES6 环境
function f() { console.log('I am outside!'); }

(function () {
  var f = undefined;
  if (false) {
    function f() { console.log('I am inside!'); }
  }

  f();
}());
// Uncaught TypeError: f is not a function
```

还有一点是，ES6**块级作用域必须有大括号** ，如果没有大括号，JavaScript 引擎就认为不存在块级作用域。 

------

## const命令

### 基本用法

const声明一个只读的变量，一旦声明，常量的值就不能改变。

因为值不能改变，所以**必须在声明时赋值**。只声明不赋值，就会报错。

const的作用域和let命令相同，也是变量不提升，造成暂时性死区。

### 本质

const实际指向的是内存地址，所以**对于简单类型的数据（数值、字符串、布尔值），一旦赋值就不能改变**，因为值就保存在变量指向的那个内存地址。**对于复合类型的数据（对象、数组），只能保证指针固定，但是值是否可变，就不能控制了。**

```javascript
// Object
const foo = {};

// 为 foo 添加一个属性，可以成功
foo.prop = 123;
foo.prop // 123

// 将 foo 指向另一个对象，就会报错
foo = {}; // TypeError: "foo" is read-only

// Array
const a = [];
a.push('Hello'); // 可执行
a.length = 0;    // 可执行
a = ['Dave'];    // 报错
```

### 声明变量的方式

```javascript
// ES5
var function

// ES6
let const import class
```

**ES5声明的全局变量将和顶层对象的属性挂钩**，而ES6的四个声明命令将不属于顶层对象的属性。

注： 顶层对象，在浏览器环境指的是`window`对象，在 Node 指的是`global`对象。 

```javascript
var a = 1;
// 如果在 Node 的 REPL 环境，可以写成 global.a
// 或者采用通用方法，写成 this.a
window.a // 1

let b = 1;
window.b // undefined
```

