## 协程

协程(`coroutine`)是一种异步编程的解决方案，意思是多个线程互相协作，完成异步任务。运行流程如下

- 协程`A`开始执行
- 协程`A`执行到一半，进入暂定，执行权转移到协程`B`
- 一段时间后，协程`B`交还执行权
- 协程`A`恢复执行

上面流程中的协程`A`就是异步任务，因为它分为多段执行

`Generator`函数就是协程在`ES6`的实现，最大特点就是**可以交出函数的执行权，即暂停执行**。

---

## Thunk 函数

`Thunk`函数是自动执行`Generator`函数的一种方法

 ### 参数的求值策略

```javascript
let x = 1

function f(m) {
  return m * 2
}
```

- 传值调用(`call by value`)，在进入函数体前就进行计算

```javascript
f(x + 5)
// 等同于
f(6)
```

- 传名调用(`call by name`)，即直接传入表达式，只在用它时求值

```javascript
f(x + 5)
// 等同于
return (x + 5) * 2
```

**`Thunk`函数就是一个临时函数**，编译器在进行`传名调用`时，将参数放在`Thunk`函数中，再将`Thunk`函数放入函数体。

```javascript
function f(m) {
  return m * 2
}

f(x + 5)

// Thunk函数
let thunk = function() {
  return x + 5
}

function f(thunk) {
  return thunk() * 2
}
```

### JavaScript 语言的 Thunk 函数

**`JavaScript`语言是传值调用**的，它的`Thunk`函数含义有所不同，替换的不是表达式，而是多参数函数，**将其替换成一个只接受回调函数作为参数的单参数函数**。

```javascript
// 正常版本的readFile,多参数版本
fs.readFile(fileName, callback)

// Thunk版本,单参数
let Thunk = function(fileName) {
	return function(callback) {
		return fs.readFile(fileName, callback)
	}
}

let readFileThunk = Thunk(fileName)
readFileThunk(callback)
```

任意函数，只要参数有回调函数，都能写成`Thunk`函数的形式

```javascript
const Thunk = function(fn) {
  return function(...args) {
    return function(callback) {
      return fn.call(this, ...args, callback)
    }
  }
}

// 上一个例子
let readFileThunk = Thunk(fs.readFile)
readFileThunk(fileName)(callback)
```

`Thunk`函数可以用于`Generator`函数的自动流程管理，**重点理解**

```javascript
var fs = require('fs')
var thunkify = require('thunkify') // thunkify模块,生产环境的转换器
var readFileThunk = thunkify(fs.readFile)

let gen = function* () {
  let r1 = yield thunkify('/file/fstab')
  console.log(r1.toString)
  let r2 = yield thunkify('/file/shells')
  console.log(r2.toString) 
}
```

上面代码中，**`yield`命令用于将程序的执行权移出`Generator`函数，通过`Thunk`函数，将执行权交还给`Generator`函数。**下面展示一下手动执行，需要做的就是将回调函数，传入`next`方法的`value`属性。

```javascript
let g = gen()

let r1 = g.next()
r1.value(function(err, data) {
  if (err) throw err
  let r2 = g.next(data)
  r2.value(function(err, data) {
    if (err) throw err
    g.next(data)
  })
})
```

`Thunk`函数还可以做到自动执行`Generator`函数

```javascript
function run(fn) {
  let gen = fn()
  
  function next(err, data) {
    let result = gen.next(data)
    if(result.done) return
    result.value(next)
  }
  
  next()
}

function* g() {}

run(g)
```

`run`函数就是`Generator`函数的自动执行器。内部的`next`函数就是`Thunk`的回调函数。在之后完每一步后，判断是否完成，没有完成就将`next`函数再传入`Thunk`函数(`result.value`属性)。

---

## co 模块

`co`模块是一个用于`Generator`函数自动执行的小工具。

```javascript
let co = require('co')

let gen = function* () {
  let f1 = yield readFile('/etc/fstab')
  let f2 = yield readFile('/etc/shells')
  console.log(f1.toString())
  console.log(f2.toString())
}

co(gen)
```

只需要将`Generator`函数传入`co`，就会自动执行。返回一个`Promise`对象，可以用`then`添加回调函数

```javascript
co(gen).then(() => {
  console.log('Generator 函数执行完成')
})
```

### 基于 Promise 对象的自动执行

```javascript
let fs = require('fs')

// 将fs.readFile包装成Promise对象
let readFile = function(fileName) {
  return new Promise((resolve, reject) => {
    fs.readFile(fileName, (error, data) => {
      if(error) return reject(error)
      resolve(data)
    })
  })
}

let gen = function* () {
  let f1 = yield readFile('/etc/fstab')
  let f2 = yield readFile('/etc/shells')
  console.log(f1.toString())
  console.log(f2.toString())
}

// 1.手动执行Generator函数
let g = gen()

g.next().value.then(data => {
  g.next(data).value.then(data => {
    g.next(data)
  })
})

// 2.自动执行
function run(gen) {
  let g = gen()
  
  function next(data) {
    let result = g.next(data)
    if(result.done) return result.value
    result.value.then(data => {
      next(data)
    })
  }
  
  next()
}
```

### 处理并发的异常操作

`co`支持并发的异步操作，允许某些操作同步进行，将这些操作放在数组或对象中，放在`yield`语句后

```javascript
// 数组的写法
co(function* (){
  let res = yield [
    Promise.resolve(1),
    Promise.resolve(2)
  ]
  console.log(res)
}).catch(onerror)

// 对象的写法
co(function* (){
  let res = yield {
    1: Promise.resolve(1),
    2: Promise.resolve(2)
	}
  console.log(res)
}).catch(onerror)

// 实例
co(function* (){
  let values = [n1, n2, n3]
  yield values.map(somethingAsync)
})

function* somethingAsync(x) {
  // do sth async
  return y
}
```



