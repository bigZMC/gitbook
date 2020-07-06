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

`Promise.all()`方法用于将多个`Promise`实例，包装成一个新的`Promise`实例。接受一个**数组(或者不是数组，但是必须具有`Iterator`接口)**作为参数，**数组中的元素均是`Promise`实例**，如果不是，先调用`Promise.resolve()`方法转成`Promise`实例。

```javascript
const p = Promise.all([p1, p2, p3])
```

- 如果`p1`，`p2`，`p3`的状态都变成`fulfilled`，`p`的状态才变成`fulfilled`，返回值组成一个数组传递给`then`函数
- 如果`p1`，`p2`，`p3`之中有一个被`rejected`，`p`的状态就变成`rejected`，并将第一个被`rejected`的实例返回值，传给`p`的`catch`函数

注意，作为参数的`Promise`实例，**自己定义了`catch`方法，一旦被`rejected`，并不会触发`Promise.all`方法的`catch`**

```javascript
const p1 = new Promise(function(resolve, reject) {
  resolve('hello')
}).then(result => result)
	.catch(e => e)

const p2 = new Promise(function(resolve, reject) {
  throw new Error('test')
}).then(result => result)
	.catch(e => e) // 这里需要注意,如果catch没有返回值,Promise.all()结果变成['hello', undefined]

Promise.all([p1, p2])
  .then(result => console.log(result))
	.catch(e => console.log(e))
// ['hello', Error: test]
```

上面代码中，`p2`会`rejected`，但是被自身的`catch`方法捕获，由于`catch`方法返回值也是一个`Promise`实例，导致该实例执行完`catch`方法后，也会变成`resolved`。因此`Promise.all`方法会调用`then`方法。

如果`p2`没有自己的`catch`方法，就会调用`Promise.all`的`catch`方法

```javascript
const p1 = new Promise(function(resolve, reject) {
  resolve('hello')
}).then(result => result)
	.catch(e => e)

const p2 = new Promise(function(resolve, reject) {
  throw new Error('test')
}).then(result => result)

Promise.all([p1, p2])
  .then(result => console.log(result))
	.catch(e => console.log(e))
// Error: test
```

---

## Promise.race()

`Promise.race()`方法用于将多个`Promise`实例，包装成一个新的`Promise`实例。和`Promise.all`方法相反，**只要参数数组中的某一个实例改变状态，那么`Promise.race`的状态以及返回值就跟这个实例相同。**

下面是一个例子，指定时间没有获得结果就返回错误信息

```javascript
const p = Promise.race([
  fetch('/http-request-url'),
  new Promise((resolve, reject) => {
    setTimeout(() => reject(new Error('request timeout')), 5000)
  })
])

p.then(console.log)
 .catch(console.error)
```

---

## Promise.allSettled()

`Promise.allSettled`方法参数和以上两种方法一致，同样返回一个`Promise`实例。**只有等所有这些参数实例都返回结果，才会结束。**

`Promise.allSettled`一旦结束，**状态总是`fulfilled`**，不管参数中的返回值是成功还是失败。它的`then`函数也是一个数组，**每个成员都是一个对象，包含`status`，`value`或`reason`属性**。`status`属性只有两个值，`fulfilled`和`rejected`，分别又对应了`value`和`reason`属性。

```javascript
const resolved = Promise.resolve(42)
const rejected = Promise.reject(-1)

const allSettledPromise = Promise.allSettled([resolved, rejected])

allSettledPromise.then(result => console.log(result))
// [
//    { status: 'fulfilled', value: 42 },
//    { status: 'rejected', reason: -1 }
// ]
```

下面是一个实例，过滤成功和失败的请求

```javascript
const promises = [ fetch('index.html'), fetch('does-not-exist-url') ]
const results = Promise.allSettled(promises)

// 过滤成功请求
const successfulPromises = results.filter(p => p.status === 'fulfilled')

// 过滤失败请求,并输出原因
const errors = results.filter(p => p.status === 'rejected')
										.map(p => p.reason)
```

有时候，我们**不关心异步结果，只关心是否都完成**，这个时候用`Promise.allSettled`就很合适。如果是`Promise.all`，会出现某一个参数实例请求失败，其他请求还没结果的情况。除非其中的所有参数实例都定了catch函数捕获异常，确保不会触发`Promise.all`的`catch`方法。

---

## Promise.resolve()

`Promise.resolve()`方法是为了将现有对象转为`Promise`对象

```javascript
Promise.resolve('foo')

// 等价于
new Promise(resolve => resolve('foo'))
```

参数分为以下四种情况

1. 参数是一个`Promise`实例

不做任何操作，原封不动地返回这个实例

2. 参数是一个`thenable`对象

`thenable`对象表示对象具有`then`方法

```javascript
let thenable = {
  then: function(resolve, reject) {
    resolve(42)
  }
}
```

`Promise.resolve`方法会将对象转为`Promise`对象，并立即执行`thenable`对象的`then`方法

```javascript
let p = Promise.resolve(thenale)
p.then(function(value) {
  console.log(value)
})
// 42
```

`thenable`被转为`Promise`对象，执行`then`方法，`p`的状态被改为`fulfilled`，从而立即执行`p`的`then`函数

3. 参数不是具有`thenable`方法的对象，或者根本就不是对象

返回一个`Promise`对象，状态为`resolved`

```javascript
const p = Promise.resolve('hello') // 不是对象,状态是resolve,执行对象p的then方法

p.then(result => console.log(result))
// 'hello'
```

4. 不带任何参数

直接返回一个状态为`resolved`的`Promise`对象

```javascript
const p = Promise.resolve()

p.then(function () {
  // do sth
})
```

重点注意，**立即`resolve()`的`Promise`对象，在本轮`"事件循环"`结束时执行**，而`setTimeout(fn, 0)`是在下一轮`"事件循环"`开始时执行

```javascript
setTimeout(() => console.log(1), 0) // 下轮事件循环开始执行,在这里最后执行

Promise.resolve().then(() => console.log(2)) // 本轮事件循环后执行

console.log(3) // 立即执行
// 3
// 2
// 1
```

---

## Promise.reject()

`Promise.reject()`和`Promise.resolve()`用法相同，状态相反

```javascript
const p = Promise.reject('Error') 
// 等价于
const p = new Promise((resolve, reject) => reject('Error'))

p.catch(reason => console.log(reason))
// 'Error'
```

`Promise.reject()`的参数，会直接作为`reject`参数，和`Promise.resolve`参数不一致

```javascript
const thenable = {
  then(resolve, reject) {
    reject('Error')
  }
}

Promise.reject(thenable)
.catch(e => {
  console.log(e === thenable)
})
// true, Promise.reject的参数就是其catch方法的参数
```

---

## 应用

### 图片加载

```javascript
function loadImageAsync(url) {
  return new Promise((resolve, reject) => {
    const image = new Image()
    
    image.onload = function() {
      resolve(iamge)
    }
    
    image.onerror = function() {
      reject(new Error(`Could not load iamge at ${url}`))
    }
    
    image.src = url
  })
}
```

