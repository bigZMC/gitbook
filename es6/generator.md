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

