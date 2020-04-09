### 箭头函数不会创建自己的`this`

箭头函数没有自己的`this`，它会捕获自己在**定义时**（注意，是定义时，不是调用时）所处的**外层执行环境的`this`**，并继承这个`this`值。所以，箭头函数中**`this`的指向在它被定义的时候就已经确定了，之后永远不会改变**。 

```javascript
var id = 'Global';

function fun1() {
  // 普通函数在执行时找到调用对象，这里是全局对象，所以输出的是window.id Global
  setTimeout(function(){
    console.log(this.id);
  }, 2000);
}

function fun2() {
  // 箭头函数在定义的时候确定this指向fun2执行环境的this
  setTimeout(() => {
    console.log(this.id);
  }, 2000)
}

fun1.call({id: 'Obj'});     // 'Global'

fun2.call({id: 'Obj'});     // 'Obj'
```

再看另外一个例子

```javascript
var id = 'GLOBAL';
var obj = {
  id: 'OBJ',
  a: function(){
    console.log(this.id);
  },
  b: () => {
    console.log(this.id);
  }
};

obj.a();    // 'OBJ'
obj.b();    // 'GLOBAL'
```

对象`obj`的方法`a`使用普通函数定义的，**普通函数作为对象的方法调用时，this指向它所属的对象**。所以，`this.id`就是`obj.id`，所以输出`OBJ`。但是方法`b`是使用箭头函数定义的，箭头函数中的`this`实际是继承的它定义时所处的全局执行环境中的`this`，所以指向全局对象，所以输出`GLOBAL`。（**这里要注意，定义对象的大括号{}是无法形成一个单独的执行环境的，它依旧是处于全局执行环境中**） 

------

### 箭头函数继承而来的`thi`指向永远不变

箭头函数中的`this`就**永远指向**它定义时所处的执行环境中的`this`，在上面的例子中，即便这个函数是作为对象`obj`的方法调用，`this`依旧指向全局对象。 

------

### .call()/.apply()/.bind()无法改变箭头函数中this的指向

`call()/apply()/bind()`方法可以用来动态修改函数执行时`this`的指向，但由于箭头函数的this定义时就已经确定且永远不会改变。所以使用这些方法永远也改变不了箭头函数`this`的指向 

```javascript
var id = 'Global';
// 箭头函数定义在全局作用域
let fun1 = () => {
  console.log(this.id)
};

fun1();     // 'Global'
// this的指向不会改变，永远指向Window对象
fun1.call({id: 'Obj'});     // 'Global'
fun1.apply({id: 'Obj'});    // 'Global'
fun1.bind({id: 'Obj'})();   // 'Global'
```

------

### 箭头函数不能作为构造函数使用

构造函数`new`的过程

-  `js`内部首先会生成一个对象 
-  再把函数中的`this`指向该对象 
-  执行构造函数中的语句 
-  返回该对象实例 

因为箭头函数没有自己的`this`，它的`this`其实是继承了外层执行环境中的`this`，且`this`指向永远不会随在哪里调用、被谁调用而改变，所以箭头函数不能作为构造函数使用，或者说构造函数不能定义成箭头函数，否则用`new`调用时会报错 

```javascript
let Fun = (name, age) => {
  this.name = name;
  this.age = age;
};

// 报错
let p = new Fun('zmc', 24);
```

------

### 箭头函数没有`arguments`

箭头函数没有自己的`arguments`对象。在箭头函数中访问`arguments`实际上获得的是外层局部（函数）执行环境中的值。 

```javascript
// 例子一
let fun = (val) => {
  console.log(val);   // 111
  // 下面一行会报错
  // Uncaught ReferenceError: arguments is not defined
  // 因为外层全局环境没有arguments对象
  console.log(arguments); 
};
fun(111);

// 例子二
function outer(val1, val2) {
  let argOut = arguments;
  console.log(argOut);    // [111, 222]
  let fun = () => {
    let argIn = arguments;
    console.log(argIn);     // [111, 222]，沿用outer的arguments对象
    console.log(argOut === argIn);  // true
  };
  fun();
}
outer(111, 222);
```

 **可以在箭头函数中使用`rest`参数代替`arguments`对象，来访问箭头函数的参数列表** 

------

### 箭头函数没有原型`prototype`

```javascript
let sayHi = () => {
  console.log('Hello World !')
};
console.log(sayHi.prototype); // undefined
```

