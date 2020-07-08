## 防止原型链函数污染环境

一般在原型链上直接添加函数，会污染对象全局的函数变量，我们一般通过和`this`的绑定，达到只对某个实例进行绑定函数的效果

```javascript
// 污染了Function原型链
Function.prototype.checkEmail = function() {}

// 定义addMethods方法,和this绑定
Function.prototype.addMethods = function(fnName, fn) {
  this[fnName] = fn
  return this
}

let obj = new Function()
obj.addMethods('test', function() {
  console.log('在obj实例上添加test函数')
})
obj.test() // '在obj实例上添加test函数'

let res = new Function()
res.test // undefined
```

如果，还想进一步封装，那么可以对实例对象的原型链再进行操作

```javascript
Function.prototype.addMethods = function(fnName, fn) {
  this.prototype[fnName] = fn
  return this
}

let Methods = new Function()
Methods.addMethods('test', function() {
  console.log('在Methods实例上添加test函数')
})

let obj = new Methods() // 这里必须使用new来实例化对象
obj.test() // '在Methods实例上添加test函数'
```

---

  ## 封装



