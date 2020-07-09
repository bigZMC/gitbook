## 基本概念

`Generator`函数是`ES6`提供的一种异步编程解决方案。可以将其理解为一个状态机，封装了多个内部状态。

```javascript
function* helloWorldGenerator() {
  yield 'hello'
  yield 'world'
  return 'ending'
  yield 'another ending'
}

let hw = helloWorldGenerator()

hw.next()
// { value: 'hello', done: false }
hw.next()
// { value: 'world', done: false }
hw.next()
// { value: 'ending', done: true }
hw.next()
// { value: undefined, done: true }
hw.next()
// 注意,return返回之后,后面的语句不会再生效
// { value: undefined, done: true }
```

可以看到，`Generator`函数返回一个遍历器对象，并不会立即执行。这个返回的遍历器对象`hw`必须调用`next`方法，**每次调用`next`方法，指针就从函数头部或者上一次停下来的地方开始执行**，使得指针移向下一个状态，直到遇到下一个`yield`或者`return`(如果没有`return`，执行到函数结束)。

- 第一次执行，调用`next`方法，在`yield`表达式暂停，并返回`yield`后面的值
- 多次调用，直到遇到`return`，或者函数结束
  - 遇到`return`返回`{ value: 'return 后面的值', done: true }`
  - 函数结束返回`{ value: undefined, done: true }`
  - 注意，**`return`返回之后，后面的语句不再生效**，调用`next`方法返回值始终是`{ value: undefined, done: true }`

---

## yield 表达式

`yield`表达式就是暂停标志，在调用`next`方法的时候可以暂停执行。

注意，**`yield`表达式是惰性求值的**

```javascript
function* gen() {
  yield 123 + 456
  console.log('表达式完成后执行')
}

const g = gen()

g.next()
// { value: 579, done: false }
g.next()
// '表达式完成后执行'
// { value: undefined, done: true }
```

当`Generator`函数不用`yield`表达式时，就变成了一个单纯的暂缓执行函数

```javascript
function* f() {
  console.log('执行')
}

setTimeout(() => {
  f().next() // 注意,这里必须用next方法调用
}, 1000)
```

需要注意，和`await`必须在`async`修饰的函数中一样，**`yield`表达式也必须在`Generator`函数中**，否则会报错

---

## 与 Iterator 接口的关系

任意一个对象的`Symbol.iterator`方法，调用后返回该对象的遍历器对象。由于`Generator`函数本身就是遍历器生成函数，因此可以将`Generator`赋值给对象的`Symbol.iterator`属性

```javascript
let myIterable = {}

myIterable[Symbol.iterator] = function* () {
  yield 1
  yield 2
  yield 3
}

[...myIterable] // [1, 2, 3]
```

`Generator`函数执行后，**返回一个遍历器对象，该对象本身也具有`Symbol.iterator`属性，执行后返回自身**

```javascript
function* gen() {}

let g = gen() // 返回一个遍历器对象

g[Symbol.iterator] === g // 返回的遍历器对象就是其Symbol.iterator属性
```

---

## next 方法的参数

`yield`表达式本身没有返回值，或者说总是返回`undefined`。`next`方法可以带一个参数，**该参数会被当作上一个`yield`表达式的返回值**

```javascript
function* f() {
  for(let i = 0;true;i++) {
    let result = yield i // 在14行next方法带参数true后,给result赋值
    if(result) {
      i = -1
    }
  }
}

let g = f()

g.next() // { value: 0, done: false }
g.next() // { value: 1, done: false }
g.next(true) // { value: 0, done: false }
```

下面还有一个复杂点的例子，解释了`参数被当作上一个yield表达式的返回值`

```javascript
function* foo(x) {
  let y = 2 * (yield (x + 1))
  let z = yield (y / 3)
  return x + y + z
}

let a = foo(5)
a.next() // { value:6, done: false }
a.next() // { value:NaN, done: false }
a.next() // { value:NaN, done: true }

let b = foo(5)
b.next() // { value:6, done: false }
b.next(12) // { value:8 done: false }
b.next(13) // { value:42, done: true }
```

上面代码中，`next`方法没有提供参数，第二次运行，导致`y`的值等于`2 * undefined`，然后在第3行的`yield`表达式后，返回`undefined / 3`即`NaN`；第三次同理，不带参数，`z`的值就是`undefined`，返回结果`5 + undefined + NaN`

如果`next`提供了参数，那么第二次运行的时候，`y`的值就是`2 * 12`，返回的值就是`24 / 3`；第三次直接给`z`赋值13，返回值就是`5 + 24 + 13`，所以就是`42`。

---

## for...of 循环

`for...of`循环可以自动遍历`Generator`函数，因为其返回的是一个遍历器对象

```javascript
function* foo() {
  yield 1
  yield 2
  yield 3
  yield 4
  yield 5
  return 6
}

for (let v of foo()) {
  console.log(v)
}
// 1 2 3 4 5
```

注意，**一旦`next`方法返回对象的`done`属性是`true`，`for...of`循环就会终止**，因此上文中`return`返回的`value: 6`不会出现在`for...of`循环中

下面是一个利用`Generator`函数和`for...of`循环实现斐波那契数列的例子

```javascript
function* fibonacci() {
  let [prev, curr] = [0, 1]
  for(;;) {
    yield curr
    [prev, curr] = [curr, prev + curr]
  }
}

for(let n of fibonacci()) {
  if(n > 1000) break;
  console.log(n)
}
```

我们还能`Generator`实现对`Object`的遍历

```javascript
function* objectEntries(obj) {
  let propKeys = Reflect.ownKeys(obj)
  
  for(let propKey of propKeys) {
    yield [propKey, obj[propKey]]
  }
}

let jane = { first: 'Jane', last: 'Doe' }

for(let [key, value] of objectEntries[jane]) {
  console.log(`${key}: ${value}`)
}
// first: Jane
// last: Doe
```

注意，在`Iterator`章节中提到，内部调用遍历器对象的除了`for...of`，还有扩展运算符，解构赋值和`Array.from`

```javascript
function* numbers() {
  yield 1
  yield 2
  return 3
  yield 4
}

[...numbers()] // [1, 2]

Array.from(numbers()) // [1, 2]

let [x, y] = numbers()
x // 1
y // 2

for(let n of numbers()) {
  console.log(n)
}
// 1
// 2
```

---

## Generator.prototype.throw()

`Generator`函数返回的遍历器对象，都有一个`throw`方法，可以在函数体外抛出错误，在`Generator`函数体内捕获。需要注意的是，**一旦被外部捕获，或者抛出异常没有被捕获，就会被认为`Generator`函数执行完成**，之后调用的`next`方法都会返回`{ value: undefined, done: true }`。

```javascript
let g = function* () {
  try {
    yield 'a'
  } catch (e) {
    console.log('内部捕获', e)
  }
  yield 'b'
}
  
let i = g()
i.next() // { value: 'a', done: false }
  
try {
  i.throw('a')
  i.throw('b')
} catch (e) {
  console.log('外部捕获', e)
}
// 内部捕获 a
// { value: 'b', done: false } 被内部捕获,throw会执行下一条yield语句
// 外部捕获 b 
i.next() // { value: undefined, done: true }
```

可以看到，遍历器对象的`throw`方法可以带一个参数，该参数会被`catch`语句接收。`i`连续抛出两个错误，第一个错误被`Generator`函数体内的`catch`捕获，**第二次抛出异常时，由于内部的`catch`已经执行过，不会再捕获，所以这个错误被跑到函数体外**，被外部的`catch`语句捕获。

如果`Generator`函数内部没有部署`try...catch`代码块，那么**`throw`方法**(注意不是`throw`命令，`throw`命令只能在外部捕获，和`Generator`函数无关)抛出的错误会被外部`try...catch`代码块捕获。如果外面也没有部署，程序会报错，中止执行。

```javascript
let gen = function* () {
  yield console.log('hello')
  yield console.log('world')
}

let g = gen()
g.next()
// 'hello'
g.throw()
// Uncaught undefined 内外部都没有try...catch代码块,无法捕获直接报错
g.next()
// { value: undefined, done: true }
```

注意，**`throw`方法抛出的错误要被内部捕获，必须至少执行一次`next`方法**

```javascript
function* gen() {
  try {
    yield 1
  } catch (e) {
    console.log('内部捕获')
  }
}

let g = gen()
g.throw(1)
// Uncaught 1
```

上面代码中，`g`一次`next`方法都没有执行过，直接执行`g.throw(1)`，不会被内部捕获，而是在外部抛出。这是因此**第一次执行`next`方法，等同于启动执行`Generator`函数的内部代码**，否则`Generator`函数还没有开始执行，`throw`方法只能在外部抛出错误。

还有一点需要了解，**就是`throw`方法被捕获后，会自动执行一次`next`方法**，我们来改造一下第一个例子

```javascript
let g = function* () {
  try {
    yield 'a'
  } catch (e) {
    console.log('内部捕获', e)
  }
  yield 'b'
  yield 'c'
}
  
let i = g()
i.next() // { value: 'a', done: false }
  
try {
  i.throw('a')
} catch (e) {
  console.log('外部捕获', e)
}
// 内部捕获 a 
// { value: 'b', done: false }

i.next() // { value: 'c', done: false }
```

可以看出，第一个例子中，调用了两次`i.throw()`，第二次被外部捕获异常后，再调用`i.next()`就会被认为`Generator`函数执行完成，返回`{ value: undefined, done: true }`。

上面这个例子只调用了一次`i.throw()`，因此不仅可以继续执行`next`方法返回正常值，并且**`throw`方法也在执行的时候，同时执行了下一条`yield`语句**。

---

## Generator.prototype.return()

`return`方法用于返回给定的值，并终结遍历`Generator`函数

```javascript
function* gen() {
  yield 1
  yield 2
  yield 3
}

let g = gen()

g.next() // { value: 1, done: false }
g.return('foo') // { value: 'foo', done: true }
g.next() // { value: undefined, done: true }
```

如果`return`方法参数为空，那么`value`的值就是`undefined`

如果`Generator`函数**内部有`try..finally`代码块，且正在执行`try`代码块**，那么`return`会导致立即进入`finally`代码块，执行完后才会结束

```javascript
function* numbers () {
  yield 1
  try {
    yield 2
    yield 3
  } finally {
    yield 4
    yield 5
  }
  yield 6
}
var g = numbers()
g.next() // { value: 1, done: false }
g.next() // { value: 2, done: false } 执行到了try代码块
g.return(7) // { value: 4, done: false } 先执行完finally代码块语句
g.next() // { value: 5, done: false }
g.next() // { value: 7, done: true }
```

---

## yield* 表达式

`yield*`表达式，是用来在一个`Generator`函数中执行另一个`Generator`函数的语法糖，解决了嵌套的问题

```javascript
function* foo() {
  yield 'a'
  yield 'b'
}

// 普通写法
function* bar() {
  yield 'x'
  // 需要手动遍历
  for(let v of foo()) {
    yield v
  }
  yield 'y'
}

// yield* 表达式
function* bar() {
  yield 'x'
  yield* foo()
  yield 'y'
}

[...bar()] // ['x', 'a', 'b', 'y'] 
```

`yield*`表达式还有`[展开遍历器对象]`的作用，也可以看作是`[递归]`

```javascript
let delegatedIterator = (function* () {
  yield 'Hello!'
  yield 'Bye!'
}())

let delegatingIterator = (function* () {
  yield 'Greetings!'
  yield* delegatedIterator // 如果用yield表达式,返回的是一个遍历器对象,而不是具体的值
  yield 'Ok, bye.'
}())

for(let value of delegatingIterator) {
  console.log(value)
}
// 'Greetings!''
// 'Hello!'
// 'Bye!'
// 'Ok, bye.'
```

任何数据结构，只要具有`Iterator`接口，都能被`yield* `遍历

```javascript
let read  = (function* () {
  yield 'hello'
  yield* 'hello' // 字符串具有Iterator接口
})()

read.next().value // 'hello'
read.next().value // 'h'
```

可以把`yield*`表达式看作是`for...of`的简写，不过有`return`语句时，需要用另外一个变量来接收`let value = yield* iterator`

```javascript
function* foo() {
  yield 2
  yield 3
  return 'foo'
}

function* bar() {
  yield 1
  var value = yield* foo()
  console.log('value: ' + value)
  yield 4
}

var it = bar();

it.next()
// {value: 1, done: false}
it.next()
// {value: 2, done: false}
it.next()
// {value: 3, done: false}
it.next();
// 'value: foo'
// {value: 4, done: false}
it.next()
// {value: undefined, done: true}
```

`yield*`命令可以方便的取出嵌套数组的所有成员

```javascript
function* iterTree(tree) {
  if(Array.isArray(tree)) {
    for(let i = 0;i < tree.length;i++) { // 上面讲过yield必须配合*函数,不要使用forEach
      yield* iterTree(tree[i])
    } else {
      yield tree
    }
  }
}

const tree = [ 'a', ['b', 'c'], ['d', 'e'] ]

[...iterTree(tree)] // ['a', 'b', 'c', 'd', 'e']
```

---

## Generator函数的this

`Generator`函数，返回的是一个遍历器，这个遍历器就是`Generator`函数的实例，而不是`this`对象

```javascript
function* gen() {
	this.a = 1  
}

gen.prototype.hello = function() {
  return 'hi!'
}

let obj =  gen()

obj instanceof gen // true
obj.hello() // 'hi!'
obj.a // undefined
```

`Generator`函数也不能和`new`命令一起用，会报错

可以采用一个变通的方法，绑定`this`的指向，让对象成为`Generator`函数的实例对象

```javascript
function *F() {
  this.a = 1
  yield this.b = 2
  yield this.c = 3
}

let obj = {}
let f = F.call(obj)

f.next()  // Object { value: 2, done: false }
f.next()  // Object { value: 3, done: false }
f.next()  // Object { value: undefined, done: true }

obj.a // 1
obj.b // 2
obj.c // 3
```

通过将`this`指向`obj`对象，并执行`next`方法对其赋值，这样就将所有内部属性绑定在`obj`上了

还可以将`obj`替换为`F.prototype`，这样返回的`f`对象就可以直接在`F`的原型链上找到属性了

```javascript
function *F() {
  this.a = 1
  yield this.b = 2
  yield this.c = 3
}

let f = F.call(F.prototype)

f.next()  // Object { value: 2, done: false }
f.next()  // Object { value: 3, done: false }
f.next()  // Object { value: undefined, done: true }

f.a // 1
f.b // 2
f.c // 3
```

---

## 应用

### 异步操作的同步化表达

```javascript
function* main() {
  let result = yield request('request-url')
  console.log(JSON.parse(result).value)
}

function request(url) {
  ajaxRequest(url, function(response) {
  	it.next(response) // 请求成功后将response当作参数进行下一步操作    
	})
}

let it = main()
it.next() // 立即执行request请求
```

### 控制流管理

避免`ajax`请求的回调地狱，也在`Promise`基础上更间接，直观

```javascript
function* longRunningTask(value1) {
  try {
    let value2 = yield step1(value1)
    let value3 = yield step2(value2)
    let value4 = yield step3(value3)
  } catch (e) {}
}

function scheduler(task) {
  let taskObj = task.next(task.value) // 直接调用next方法获取返回值
  if(!taskObj.done) {
    task.value = taskObj.value
    scheduler(task)
  }
}

scheduler(longRunningTask(initialValue)) // 传入参数,遍历器对象
```

### 部署 Iterator 接口

上面提到过，`Generator`函数可以为任意对象部署`Iterator`接口

```javascript
function* iterEntries(obj) {
  let keys = Object.keys(obj)
  for(let i = 0;i < keys.length;i++) {
    let key = keys[i]
    yield [key, obj[key]]
  }
}

let person = {
  name: '张三',
  age: 20
}

for(let [key, value] of iterEntries(person)) {
	console.log(`${key}: ${value}`) 
}
// name: 张三
// age: 20
```

### 作为数据结构

因为`Generator`可以返回一系列的值，可以对任意表达式，提供类似数组的接口