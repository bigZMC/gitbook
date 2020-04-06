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

参数变量是默认声明的，所以不能用`let`或`const`再次声明。 

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

