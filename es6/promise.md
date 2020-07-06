## Promise的含义

所谓`Promise`，简单来说就是**一个容器，保存着某个未来才会结束的事件**(通常是一个异步操作)的结果。有以下两个特点

- 对象的状态不受外界影响。状态只有`pending`，`fulfilled`(已成功)，`rejected`
- 一旦状态改变，就不会在变，任何时候都可以得到这个结果。

`Promise`也有一些缺点

- 无法取消，一旦新建就会立即执行
- 如果不设置回调函数，内部抛出的错误，不会反应到外部
- 处于`pending`状态，无法得知进展到哪一步了

---

## 基本用法

`Promise`对象是一个构造函数，用来生成`Promise`实例

```javascript
const promise = new Promise(function(resolve, reject) {
  // do sth
  
  if(/* 异步操作成功 */) {
  	resolve(value)
	} else {
  	reject(error)
  }
})
```

`Promise`构造函数中的函数有两个参数，`resolve`和`reject`，由`JavaScript`引擎提供，不用自己部署。

`resolve`函数的作用是，将`Promise`对象状态从`pending`变为`fulfilled`；`reject`函数则是将状态从`pending`变为`rejected`。两个函数分别**将异步操作的结果和错误信息，作为参数传递出去**。

```javascript
function timeout(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(resolve, ms, 'done')
  })
}

timeout(100).then(value => {
  console.log(value)
})
// 100ms后输出 'done'
```

`Promise`**新建后就会立即执行**

```javascript
let promise = new Promise((resolve, reject) => {
  console.log('Promise')
  resolve()
})

promise.then(function() {
  console.log('resolved.')
})

console.log('Hi!')

// 'Promise',新建立即执行
// 'Hi!'
// 'resolved.',回调异步执行
```

注意，`resolve`函数的参数除了正常的值以外，还可以是另一个`Promise`实例，即一个异步操作的结果返回另一个异步操作

```javascript
const p1 = new Promise(function(resolve, reject) {
  // do sth
})

const p2 = new Promise(function(resolve, reject) {
  // do sth
  resolve(p1)
})
```

上面例子中，**`p1`的状态决定了`p2`的状态**。如果`p1`的状态是`pending`，那么`p2`的回调函数就会等待`p1`的状态改变。如果`p1`的状态是`resolved`或者`rejected`，那么`p2`的回调函数就会立即执行

```javascript
const p1 = new Promise(function(resolve, reject) {
  setTimeout(() => reject(new Error('fail')), 3000)
})

const p2 = new Promise(function(resolve, reject) {
  setTimeout(() => resolve(p1), 1000)
})

p2.then(result => console.log(result))
	.catch(error => console.log(error))
// Error: fail
```

`p2`在1秒钟之后改变，`resolve`方法返回的是`p1`，**由于`p2`返回的是另一个`Promise`，导致`p2`自己的状态无效了**，由`p1`的状态决定`p2`的状态。之后`p1`的状态变为`rejected`，触发了`catch`指定的回调函数。

最后需要注意，`resolve`或`reject`并不会终结`Promise`的参数函数的执行，只有`return`会影响

```javascript
new Promise((resolve, reject) => {
  resolve(1)
  console.log(2)
}).then(r => {
  console.log(r)
})
// 2
// 1,放在事件循环末尾执行,晚于同步执行函数

new Promise((resolve, reject) => {
  return resolve(1)
  // 后面的语句不会执行
  console.log(2)
})
```

---

## Promise.prototype.then()

`then`方法的作用是，为`Promise`实例添加状态改变时的回调函数。`then`方法的第一个参数是`resolved`状态的回调函数，第二个参数可选，是`rejected`状态的回调函数。

由于`then`方法返回的是一个**新的`Promise`实例**，因此可以采用链式写法

```javascript
getJSON('/posts.json').then(function(json) {
  // 返回一个Promise对象
  return getJSON(json.commentURL)
}).then(function(post) {
  // 返回一个结果,也会被包裹成为Promise对象
  return post
}).then(function(comments) {
  console.log('resolved: ', comments)
}, function(err) {
  console.log('rejected: ', err)
})
```

---

## Promise.prototype.catch()

`catch`方法是`then(null, rejection)`或`then(undefined, rejection)`的别名，用于指定发生错误时的回调函数。另外，在**`then`方法指定的回调函数，如果运行时抛出错误，也会被`catch`捕获**。

```javascript
// promise抛出一个错误,被catch捕获
const promise = new Promise(function(resolve, reject) {
  throw new Error('test')
})

promise.catch(function(error) {
  console.log(error)
})
// Error: test

// then方法抛出一个错误,也会被catch捕获
const promise =  new Promise(function(resolve, reject) {
  resolve(1)
})

promise.then(function(result) {
  console.log(result)
  throw new Error('test')
}).catch(function(error) {
  console.log(error)
})
// 1
// Error: test
```

`reject`方法的作用，等同于抛出错误

```javascript
// 通过try/catch捕获异常,并调用reject方法返回error信息
const promise = new Promise(function(resolve, reject) {
  try {
    throw new Error('test')
  } catch(e) {
    reject(e)
  }
})

// 直接返回异常信息
const promise = new Promise(function(resolve, reject) {
  reject(new Error('test'))
})
```

如果`Promise`状态已经变成`resolved`，再抛出错误是无效的，这也是`Promise`的第二个特点，**状态一旦改变，就不会再变了**。

```javascript
const promise = new Promise(function(resolve, reject) {
  resolve('ok')
  throw new Error('test') // 不会被捕获
})

promise.then(function(value) {
  console.log(value)
}).catch(function(error) {
  console.log(error)
})
// 'ok'
```

`Promise`对象的错误具有`"冒泡"`性质，会一直向后传递，直到被捕获为止。

```javascript
getJSON('/post.json').then(function(post) {
  return getJSON(post.commentURL)
}).then(function(comments) {
  // do sth
}).catch(function(error) {
  // 处理前面三个Promise产生的错误
})
```

上面代码中，一共有三个`Promise`对象，分别是由`getJSON()`产生和另外两个`then`产生。它们之中任何一个抛出的错误，都会被`catch`捕获。且`catch`方法返回的也是一个`Promise`对象，因此可以接着调用`then`方法。

```javascript
const someAsyncThing = function() {
  return new Promise(function(resolve, reject) {
    // x没有声明,报错
    resolve(x + 2)
  })
}

someAsyncThing().catch(function(error) {
  console.log('Error', error)
}).then(function() {
  console.log('carry on')
})
// Error [ReferenceError: x is not defined]
// carry on
```

如果没有报错，则会跳过`catch`方法

```javascript
Promise.resolve().catch(function(error) {
  console.log('Error', error) // 没有报错,跳过
}).then(function() {
  console.log('carry on')
})
// carry on
```

需要注意的是，**如果没有用`catch`捕获异常，`Promise`对象抛出的错误信息不会传递到外层，即不会有任何反应**，这也是`Promise`的缺点之一

```javascript
const someAsyncThing = function() {
  return new Promise(function(resolve, reject) {
    resolve(x + 2) // x没有声明,报错
  })
}

someAsyncThing().then(function() {
  console.log('carry on')
})

setTimeout(() => { console.log(123) }, 2000)
// Uncaught (in promise) ReferenceError: x is not defined
// 123
```

代码运行到第3行会抛出错误`ReferenceError: x is not defined`，但是不会退出进程，终止脚本，2秒后正常输出`123`。也就是说，**`Promise`内部的错误不会影响外部的代码**

下面是一个类似的例子

```javascript
const promise = new Promise(function(resolve, reject) {
  resolve('ok')
  setTimeout(function() { throw new Error('test') }, 0)
})

promise.then(function(value) { console.log(value) })
// 'ok'
// Uncaught Error: test
```

在代码第3行，通过`setTimeout(func, 0)`指定在下一轮`"事件循环"`再抛出错误，到那个时候，`Promise`运行已经结束了，所以这个错误会在`Promise`外被抛出，会冒泡到最外层，成为未捕获异常。

最后，`catch`中还可以继续抛出异常，交给下一个`catch`捕获

```javascript
const someAsyncThing = function() {
  return new Promise(function(resolve, reject) {
    resolve(x + 2) // x没有声明,报错
  })
}

someAsyncThing().then(function() {
  return result
}).catch(function(error) {
  console.log('Error', error)
  y + 2 // y没有声明,报错
}).catch(function(error) {
  console.log('carry on', error)
})
// Error [ReferenceError: x is not defined]
// carry on [ReferenceError: y is not defined]
```

---

## Promise.prototype.finally()

`finally`方法用于指定不管状态如何，都会执行的操作。`finally`不接受任何参数，也就**无从得知`Promise`的具体状态，也就是说，该方法中的操作，不依赖于`Promise`的执行结果**。

本质上，`finally`是`then`方法的特例，不管`Promise`的状态，都会执行回调函数`cb`

```javascript
Promise.prototype.finally = function(cb) {
  let P = this.constructor
  // 返回then方法,同样可以链式调用
  return this.then(
  	value => P.resolve(cb()).then(() => value),
    reason => P.resolve(cb()).then(() => { throw reason })
  )
}
```

从上面的代码中可以看到，`finally`总能返回原来的值

```javascript
// resolve 的值是 undefined
Promise.resolve(2).then(() => {}, () => {})

// resolve 的值是 2
Promise.resolve(2).finally(() => {})
```

---

## Promise.all()

