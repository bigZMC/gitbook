## 含义

`async`函数就是`Generator`函数的语法糖，不过是把星号(`*`)替换成了`async`，`yield`替换成了`await`，`async`函数对`Generator`函数做出了四点改进

- 内置执行器，`Generator`函数的执行必须靠执行器，所以才有了`co`模块，而`async`函数自带执行器

- 更好的语义，`async`和`await`，比起`*`和`yield`，语义更清楚

- 更广的适用性，`co`模块约定，`yield`命令后面只能是`Thunk`函数或者`Promise`对象，而`await`命令后，可以是`Promise`对象和原始类型的值

- 返回值是`Promise`，比起`Generator`返回`Iterator`对象更方便

---

## 基本用法

**`async`函数返回一个`Promise`对象**，用`then`方法添加回调函数

```javascript
function timeout(ms) {
  return new Promise(resolve => {
    setTimeout(resolve, ms)
  })
}

async function asyncPrint(value, ms) {
  await timeout(ms)
  console.log(value)
}

aysnc('hello world', 100)
```

由于`async`函数返回的是`Promise`对象，可以作为`await`命令的参数，因此上面例子可以进一步改写

```javascript
async function timeout(ms) {
  await new Promise(resolve => {
    setTimeout(resolve, ms)
  })
}

async function asyncPrint(value, ms) {
  await timeout(ms)
  console.log(value)
}

aysnc('hello world', 100)
```

---

## 语法

### await 命令

一般来说，`await`命令后面是一个`Promise`对象，返回该对象的结果。如果不是，就直接返回对象的值

```javascript
async function f() {
  // 等同于 return 123
  return await 123
}

f().then(v => console.log(v)) // 123
```

如果`await`命令后面是一个`thenable`对象(定义了`then`方法的对象)，那么`await`会将其等同于`Promise`对象

```javascript
class Sleep {
  constructor(timeout) {
    this.timeout = timeout
  }
  then(resolve, reject) {
    const startTime = Date.now()
    setTimeout(() => {
      return resolve(Date.now() - startTime)
    }, this.timeout)
  }
}

(async () => {
  const sleepTime = await new Sleep(1000)
  console.log(sleepTime)
})()
// 1000
```

`await`命令后面的`Promise`对象如果变为`reject`状态，则会被`catch`方法拦截，并且会中断执行整个`async`函数

```javascript
async function f() {
  await Promise.reject('Error')
  await Promise.resolve('hello')
}

f().then(v => console.log(v))
	 .catch(e => console.log(e))
// 'Error'
// 第二句resolve不会执行
```

如果想要正常进行整个`async`函数，可以将`await`放在`try...catch`代码块中

```javascript
async function f() {
  try {
    await Promise.reject('Error')
  } catch(e) {
    console.log('内部:', e)
  }
  return await Promise.resolve('hello') // 需要return结果,只有async才能直接返回一个promise对象
}

f().then(v => console.log(v))
	 .catch(e => console.log('外部:', e))
// 内部: Erro
// 'hello'
```

下面是一个实例，通过`try...catch`结构，实现多次重复尝试

```javascript
const NUM_RETRIES = 3

async function test() {
  let i
  for(i = 0;i < NUM_RETRIES;++i) {
    try {
      await Promise.reject('Error')
      break
    } catch(e) {}
  }
  console.log(i)
}

test()

// 注意,如果在第7行捕获了reject的错误
await Promise.reject('Error').catch(e => {})
// catch返回的是一个处理了的Promise对象,就会来到break,此时打印i的结果就是0
```

### 同步函数的处理

如果多个函数间是独立的异步操作，互不依赖，可以同时触发

```javascript
// 写法一,通过Promise.all和解构赋值
let [foo, bar] = await Promise.all([getFoo(), getBar()])

// 写法二,先执行,后赋值
let fooPromise = getFoo()
let barPromise = getBar()
let foo = await fooPromise
let bar = await barPromise
```

### 保留运行堆栈

```javascript
const a = () => {
  b().then(() => c())
}
```

上面代码中，`a`内部运行了一个异步任务`b`。当`b`运行时，`a`不会中断，而是继续执行。等`b`运行结束，`a`可能已经运行结束，`b`所在上下文就消失了，如果`b`或`c`报错，错误堆栈就不会包括`a`

```javascript
const a = async () => {
  await b()
  c()
}
```

改成`async`函数后，执行到`b`，`a`会暂停执行，上下文环境保存。

---

## 实例： 按顺序完成异步操作

```javascript
async function logInOrder(urls) {
  // 并发读取远程url
  const textPromises = urls.map(async url => {
    const response = await fetch(url)
    return response.text()
  })
  
  // 按次序输出
  for(const textPromise of textPromises) {
    console.log(await textPromise)
  }
}
```

`map`方法的参数是`async`函数，但是**`map`是并发执行的，`async`内部是继发执行，外部不受影响**。`for...of`循环内部使用了`await`，因此实现了顺序执行，且能并发执行`fetch`操作。

---

## 顶层 await

`await`命令只能出现在`async`函数内部，否则会报错。为了结果模块异步加载的问题，有提案引入了`顶层await`，允许独立使用`await`

```javascript
// awaiting.js
let output

async function main() {
  const dynamic = await import(someMission)
  const data = await fetch(url)
  output = someProcess(dynamic.default, data)
}

main()

export { output }
```

在加载`awaiting.js`模块时，调用`output`返回值，取决于执行的时间

```javascript
// usage.js
import { output } from './awaiting.js'

function outputPlusValue(value) { return output + value }

console.log(outputPlusValue(100)) // 结果取决于output的值
```

目前的方法是返回一个`Promise`对象，从这个`Promise`对象的状态来判断异步操作是否结束

```javascript
// awaiting.js
let output

export default (async function() {
  const dynamic = await import(someMission)
  const data = await fetch(url)
  output = someProcess(dynamic.default, data)
})()

export { output }
```

```javascript
// usage.js
import promise, { output } from './awaiting.js'

function outputPlusValue(value) { return output + value }

promise.then(() => {
  console.log(outputPlusValue(100))
})
```

将`awaiting.js`对象的输出，放在`promise.then`方法中，确保了异步操作完成后才去读取`output`

如果采用`顶层await`命令，可以这么写

```javascript
// awaiting.js
const dynamic = import(someMission)
const data = fetch(url)

export const output = someProcess((await dynamic).default, await data)

// usage.js
import { output } from './awaiting.js'

function outputPlusValue(value) { return output + value }

console.log(outputPlusValue(100))
```

最后一个例子，加载多个顶层`await`的模块

```javascript
// x.js
console.log('X1')
await new Promise(r => setTimeout(r, 1000))
console.log('X2')

// y.js
console.log('Y')

// z.js
import './x.js'
import './y.js'
console.log('Z')

// X1
// Y
// X2
// Z
```

`z.js`并没有等待`x.js`加载完成，再加载`y.js`，而是同步执行