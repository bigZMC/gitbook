## 扩展运算符

扩展运算符（spread）是三个点（`...`）。它好比 **rest 参数的逆运算**，**将一个数组转为用逗号分隔的参数序列**。 

```javascript
console.log(...[1, 2, 3])
// 1 2 3

console.log(1, ...[2, 3, 4], 5)
// 1 2 3 4 5

[...document.querySelectorAll('div')]
// [<div>, <div>, <div>]
```

该运算符主要用于函数调用

```javascript
function push(array, ...items) {
  array.push(...items);
}

function add(x, y) {
  return x + y;
}

const numbers = [4, 38];
add(...numbers) // 42
```


扩展运算符与正常的函数参数可以结合使用，非常灵活。

```javascript
function f(v, w, x, y, z) {
	console.log(v, w, x, y, z)
}
const args = [0, 1];
f(-1, ...args, 2, ...[3]); // -1, 0, 1, 2, 3
```

扩展运算符后面还可以放置表达式。

```javascript
const arr = [
  ...(x > 0 ? ['a'] : []),
  'b',
];
```

如果扩展运算符后面是一个空数组，则**不产生任何效果**，甚至连`undefined`或`empty`都不会产生

```javascript
[...[], 1]
// [1]
[, 1]
// [empty, 1]

[...[]]
// []
```

注意，**只有函数调用时，扩展运算符才可以放在圆括号中**，否则会报错。

```javascript
(...[1, 2])
// Uncaught SyntaxError: Unexpected number

console.log((...[1, 2]))
// Uncaught SyntaxError: Unexpected number

console.log(...[1, 2])
// 1 2
```

### 扩展运算符的应用

- 复制数组

数组是复合的数据类型，直接复制的话，只是复制了指向底层数据结构的指针，而不是克隆一个全新的数组。ES5 只能用变通方法来复制数组。 

```javascript
const a1 = [1, 2];
const a2 = a1.concat();

a2[0] = 2;
a1 // [1, 2]
```

扩展运算符提供了复制数组的简便写法。

```javascript
const a1 = [1, 2];
// 写法一
const a2 = [...a1];
// 写法二
const [...a2] = a1;
```

- 合并数组

扩展运算符提供了数组合并的新写法。

```javascript
const arr1 = ['a', 'b'];
const arr2 = ['c'];
const arr3 = ['d', 'e'];

// ES5 的合并数组
arr1.concat(arr2, arr3);
// [ 'a', 'b', 'c', 'd', 'e' ]

// ES6 的合并数组
[...arr1, ...arr2, ...arr3]
// [ 'a', 'b', 'c', 'd', 'e' ]
```

 不过，这**两种方法都是浅拷贝**，使用的时候需要注意。 

```javascript
const a1 = [{ foo: 1 }];
const a2 = [{ bar: 2 }];

const a3 = a1.concat(a2);
const a4 = [...a1, ...a2];

a3[0] === a1[0] // true
a4[0] === a1[0] // true
```

- 与解构赋值结合

扩展运算符可以与解构赋值结合起来，用于生成数组。

```javascript
// ES5
a = list[0], rest = list.slice(1)
// ES6
[a, ...rest] = list
```

下面是另外一些例子。

```javascript
const [first, ...rest] = [1, 2, 3, 4, 5];
first // 1
rest  // [2, 3, 4, 5]

const [first, ...rest] = [];
first // undefined
rest  // []

const [first, ...rest] = ["foo"];
first  // "foo"
rest   // []
```

如果将**扩展运算符用于数组赋值，只能放在参数的最后一位**，否则会报错。

```javascript
const [...butLast, last] = [1, 2, 3, 4, 5];
// 报错

const [first, ...middle, last] = [1, 2, 3, 4, 5];
// 报错
```

- 字符串

扩展运算符还可以将字符串转为真正的数组。

```javascript
[...'hello']
// [ "h", "e", "l", "l", "o" ]
```

上面的写法，有一个重要的好处，那就是**能够正确识别四个字节的 Unicode 字符**。

```javascript
'x\uD83D\uDE80y'.length // 4
[...'x\uD83D\uDE80y'].length // 3
```

上面代码的第一种写法，JavaScript 会将四个字节的 Unicode 字符，识别为 2 个字符，采用扩展运算符就没有这个问题。因此，正确返回字符串长度的函数，可以像下面这样写。

```javascript
// 凡是涉及到操作四个字节的 Unicode 字符的函数，都有这个问题。因此，最好都用扩展运算符改写。
function length(str) {
  return [...str].length;
}

length('x\uD83D\uDE80y') // 3

let str = 'x\uD83D\uDE80y';

str.split('').reverse().join('')
// 'y\uDE80\uD83Dx'

[...str].reverse().join('')
// 'y\uD83D\uDE80x'
```

- 实现了 Iterator 接口的对象

 任何定义了遍历器（`Iterator`）接口的对象，都可以用扩展运算符转为真正的数组。 

```javascript
let nodeList = document.querySelectorAll('div');
let array = [...nodeList];
```

对于那些没有部署`Iterator`接口的类似数组的对象，扩展运算符就无法将其转为真正的数组。

```javascript
let arrayLike = {
  '0': 'a',
  '1': 'b',
  '2': 'c',
  length: 3
};

// TypeError: Cannot spread non-iterable object.
let arr = [...arrayLike];
```

`Map`和`Set`结构都具有`Iterator`接口，所以都能用扩展运算符遍历

```javascript
let map = new Map([
  [1, 'one'],
  [2, 'two'],
  [3, 'three'],
]);

let arr = [...map]; // [[1, 'one'], [2, 'two'], [3, 'three']]
```

`Generator`函数运行后，返回一个遍历器对象，因此也可以使用扩展运算符。

```javascript
const go = function*(){
  yield 1;
  yield 2;
  yield 3;
};

[...go()] // [1, 2, 3]
```

------

## Array.from()

`Array.from`方法用于将两类对象转为真正的数组：**类似数组的对象**（array-like object）和**可遍历的对象**（包括 ES6 新增的数据结构 Set 和 Map）。 

```javascript
let arrayLike = {
    '0': 'a',
    '1': 'b',
    '2': 'c',
    length: 3
};

// ES5的写法
var arr1 = [].slice.call(arrayLike); // ['a', 'b', 'c']

// ES6的写法
let arr2 = Array.from(arrayLike); // ['a', 'b', 'c']
```

实际应用中，常见的类似数组的对象是 DOM 操作返回的 NodeList 集合，以及函数内部的`arguments`对象。`Array.from`都可以将它们转为真正的数组。 

```javascript
// NodeList对象
let ps = document.querySelectorAll('p');
Array.from(ps).filter(p => {
  return p.textContent.length > 100;
});

// arguments对象
function foo() {
  var args = Array.from(arguments);
  // ...
}
```

**只要是部署了 Iterator 接口的数据结构**，`Array.from`都能将其转为数组。

```javascript
Array.from('hello')
// ['h', 'e', 'l', 'l', 'o']

let namesSet = new Set(['a', 'b'])
Array.from(namesSet) // ['a', 'b']
```

如果参数是一个真正的数组，`Array.from`会返回一个一模一样的新数组。

```javascript
Array.from([1, 2, 3])
// [1, 2, 3]
```

注意：扩展运算符（`...`）也可以将某些数据结构转为数组，两者区别如下

-   **扩展运算符背后调用的是遍历器接口**（`Symbol.iterator`），如果一个对象没有部署这个接口，就无法转换。 
-  `Array.from`方法还支持类似数组的对象。**所谓类似数组的对象(伪数组)，本质特征只有一点，即必须有`length`属性。**

```javascript
Array.from({ length: 3 });
// [ undefined, undefined, undefined ]
// 由于没有部署Interator接口,扩展运算符不能转换这个对象
```

对于还没有部署该方法的浏览器，可以用`Array.prototype.slice`方法替代。

```javascript
const toArray = (() =>
  // ES5 hack
  Array.from ? Array.from : obj => [].slice.call(obj)
)();
```

`Array.from`还可以接受第二个参数，作用类似于数组的`map`方法，用来对每个元素进行处理，将处理后的值放入返回的数组。

```javascript
Array.from(arrayLike, x => x * x);
// 等同于
Array.from(arrayLike).map(x => x * x);

Array.from([1, 2, 3], (x) => x * x)
// [1, 4, 9]
```

另一个例子是返回各种数据的类型。

```javascript
function typesOf () {
  return Array.from(arguments, value => typeof value)
}
typesOf(null, [], NaN)
// ['object', 'object', 'number']
```

如果`map`函数里面用到了`this`关键字，还可以传入`Array.from`的第三个参数，用来绑定`this`。 

`Array.from()`可以将各种值转为真正的数组，并且还提供`map`功能。这实际上意味着，只要有一个原始的数据结构，你就可以先对它的值进行处理，然后转成规范的数组结构，进而就可以使用数量众多的数组方法。

```javascript
Array.from({ length: 2 }, () => 'jack')
// ['jack', 'jack']
```

`Array.from()`的另一个应用是，将字符串转为数组，然后返回字符串的长度。注意，扩展运算符也能达到同样的效果。 

```javascript
function countSymbols(string) {
  return Array.from(string).length;
}

let str = '𠮷'
countSymbols(str) // 1
[...str].length // 1
```

------

## Array.of()

`Array.of`方法用于将一组值，转换为数组。

```javascript
Array.of(3, 11, 8) // [3,11,8]
Array.of(3) // [3]
Array.of(3).length // 1
```

这个方法的主要目的，是**弥补数组构造函数`Array()`的不足**。因为参数个数的不同，会导致`Array()`的行为有差异。 

```javascript
Array() // []
Array(3) // [, , ,]
Array(3, 11, 8) // [3, 11, 8]
```

`Array.of`基本上可以用来替代`Array()`或`new Array()`，并且不存在由于参数不同而导致的重载。它的行为非常统一。 

```javascript
Array.of() // []
Array.of(undefined) // [undefined]
Array.of(1) // [1]
Array.of(1, 2) // [1, 2]
```

`Array.of`方法可以用下面的代码模拟实现。

```javascript
function ArrayOf(){
  return [].slice.call(arguments);
}
```

------

## Array.prototype.copyWithin()

数组实例的`copyWithin()`方法，在当前数组内部，将指定位置的成员复制到其他位置（会覆盖原有成员），然后返回当前数组。也就是说，**使用这个方法，会修改当前数组**。

```javascript
Array.prototype.copyWithin(target, start = 0, end = this.length)
```

它接受三个参数。

- target（必需）：从该位置开始替换数据。如果为负值，表示倒数。
- start（可选）：从该位置开始读取数据，默认为 0。如果为负值，表示从末尾开始计算。
- end（可选）：到该位置前停止读取数据，默认等于数组长度。如果为负值，表示从末尾开始计算。

```javascript
// 将3号位复制到0号位
[1, 2, 3, 4, 5].copyWithin(0, 3, 4)
// [4, 2, 3, 4, 5]

// -2相当于3号位，-1相当于4号位
[1, 2, 3, 4, 5].copyWithin(0, -2, -1)
// [4, 2, 3, 4, 5]

// 将3号位复制到0号位
[].copyWithin.call({length: 5, 3: 1}, 0, 3)
// {0: 1, 3: 1, length: 5}

// 将2号位到数组结束，复制到0号位
let i32a = new Int32Array([1, 2, 3, 4, 5]);
i32a.copyWithin(0, 2);
// Int32Array [3, 4, 5, 4, 5]

// 对于没有部署 TypedArray 的 copyWithin 方法的平台
// 需要采用下面的写法
[].copyWithin.call(new Int32Array([1, 2, 3, 4, 5]), 0, 3, 4);
// Int32Array [4, 2, 3, 4, 5]
```

------

## Array.prototype.find()和Array.prototype.findIndex()

数组实例的`find`方法，用于找出第一个符合条件的数组成员。它的参数是一个回调函数，所有数组成员依次执行该回调函数，直到找出第一个返回值为`true`的成员，然后返回该成员。如果没有符合条件的成员，则返回`undefined`。  

```javascript
[1, 4, -5, 10].find(n => n < 0)
// -5

[1, 5, 10, 15].find(function(value, index, arr) {
  return value > 9;
}) // 10
```

数组实例的`findIndex`方法的用法与`find`方法非常类似，返回第一个符合条件的数组成员的位置，如果所有成员都不符合条件，则返回`-1`。 

```javascript
[1, 5, 10, 15].findIndex(function(value, index, arr) {
  return value > 9;
}) // 2
```

这两个方法都可以接受第二个参数，用来绑定回调函数的`this`对象。

```javascript
function f(v){
  // 这里的this指向person对象，this.age值位20
  return v > this.age;
}
let person = {name: 'John', age: 20};
[10, 12, 26, 15].find(f, person);    // 26
```

另外，这两个方法都可以发现`NaN`，因为`indexOf`没有回调函数，这两个方法以此进行了弥补。

```javascript
[NaN].indexOf(NaN)
// -1

[NaN].findIndex(y => Object.is(NaN, y))
// 0
```

------

## Array.prototype.fill()

`fill`方法使用给定值，填充一个数组。

```javascript
['a', 'b', 'c'].fill(7)
// [7, 7, 7]

new Array(3).fill(7)
// [7, 7, 7]
```

`fill`方法用于空数组的初始化非常方便。数组中已有的元素，会被全部抹去。 

`fill`方法还可以接受第二个和第三个参数，用于指定填充的起始位置和结束位置。其中第二个(start)和第三个(end)参数和`copyWithin()`方法规则相同。

```javascript
['a', 'b', 'c'].fill(7, 1, 2)
// 从 1 号位开始，向原数组填充 7，到 2 号位之前结束
// ['a', 7, 'c']
```

注意，如果**填充的类型为对象，那么被赋值的是同一个内存地址的对象，而不是深拷贝对象**。

```javascript
let arr = new Array(3).fill({name: "Mike"});
arr[0].name = "Ben";
arr
// [{name: "Ben"}, {name: "Ben"}, {name: "Ben"}]

let arr = new Array(3).fill([]);
arr[0].push(5);
arr
// [[5], [5], [5]]
```

------

## 数组实例的 entries()，keys() 和 values()

ES6 提供三个新的方法——`entries()`，`keys()`和`values()`——用于遍历数组。它们都返回一个遍历器对象，可以用`for...of`循环进行遍历，唯一的区别是`keys()`是对键名的遍历、`values()`是对键值的遍历，`entries()`是对键值对的遍历。 

```javascript
for (let index of ['a', 'b'].keys()) {
  console.log(index);
}
// 0
// 1

for (let elem of ['a', 'b'].values()) {
  console.log(elem);
}
// 'a'
// 'b'

for (let [index, elem] of ['a', 'b'].entries()) {
  console.log(index, elem);
}
// 0 "a"
// 1 "b"
```

如果不使用`for...of`循环，可以手动调用遍历器对象的`next`方法，进行遍历。

```javascript
let letter = ['a', 'b', 'c'];
let entries = letter.entries();
console.log(entries.next().value); // [0, 'a']
console.log(entries.next().value); // [1, 'b']
console.log(entries.next().value); // [2, 'c']
```

------

## Array.prototype.includes()

`Array.prototype.includes`方法返回一个布尔值，表示某个数组是否包含给定的值，与字符串的`includes`方法类似。 

```javascript
[1, 2, 3].includes(2)     // true
[1, 2, 3].includes(4)     // false
[1, 2, NaN].includes(NaN) // true
```

该方法的第二个参数表示搜索的起始位置，默认为`0`。如果第二个参数为负数，则表示倒数的位置，如果这时它大于数组长度（比如第二个参数为`-4`，但数组长度为`3`），则会重置为从`0`开始。如果是正数，但是大于数组长度 ，直接返回`false`

```javascript
[1, 2, 3].includes(3, 1);  // true
[1, 2, 3].includes(3, -1); // true

[1, 2, 3].includes(3, -4) // true
[1, 2, 3].includes(3, 4) // false
```

有了`includes`，可以用来代替`indexOf`判断是否包含元素，优点在于

- `indexOf`内部使用严格相等运算符`===`判断，会导致对`NaN`的误判；`includes`使用的是不一样的判断算法，不会出现这种情况
- `indexOf`不够语义化，它的含义是找到参数值的第一个出现位置，所以要去比较是否不等于`-1`，表达起来不够直观

下面代码用来检查当前环境是否支持该方法，如果不支持，部署一个简易的替代版本。

```javascript
const contains = (() =>
  Array.prototype.includes
    ? (arr, value) => arr.includes(value)
    : (arr, value) => arr.some(el => el === value)
)();
contains(['foo', 'bar'], 'baz'); // => false
```

------

## Array.prototype.flat()和Array.prototype.flatMap()

数组的成员有时还是数组，`Array.prototype.flat()`用于将嵌套的数组“拉平”，变成一维的数组。该方法返回一个新数组，对原数据没有影响。 

```javascript
[1, 2, [3, 4]].flat()
// [1, 2, 3, 4]
```

`flat()`默认只会“拉平”一层，如果想要“拉平”多层的嵌套数组，可以将`flat()`方法的参数写成一个整数，表示想要拉平的层数，默认为1。

```javascript
[1, 2, [3, [4, 5]]].flat()
// [1, 2, 3, [4, 5]]

[1, 2, [3, [4, 5]]].flat(2)
// [1, 2, 3, 4, 5]
```

如果不管有多少层嵌套，都要转成一维数组，可以用`Infinity`关键字作为参数。

```javascript
[1, [2, [3]]].flat(Infinity)
// [1, 2, 3]
```

如果原数组有空位，`flat()`方法会跳过空位。

```javascript
[1, 2, , 4, 5].flat()
// [1, 2, 4, 5]
```

以下是ES5的两种实现方式

```javascript
var flat = function (arr, depth = 1) {
  let res = [],
    depthNum = 0,
    flatArr = (arr) => {
      arr.forEach((item, index, array) => {
        // 如果是array,则递归调用flatArr函数
        if (Array.isArray(item)) {
          // 如果没有达到depth则继续向下递归调用,否则直接push
          if (depthNum < depth) {
            depthNum += 1
            flatArr(item)
          } else {
            res.push(item)
          }
        } else {
          res.push(item)
          // 查到层级最后一个元素,重置depthNum
          if (index === array.length - 1) {
            depthNum = 0
          }
        }
      })
    }
  flatArr(arr)
  return res
}
```

```javascript
function flat(arr, maxDepth = 1, currentDepth = 0) {
  let res = []
  arr.forEach((item, index, array) => {
    if (Array.isArray(item)) {
      if (maxDepth > currentDepth) {
        // 没有达到maxDepth的数组,递归调用flat函数,并将当前层级+1
        res = [...res, ...flat(item, maxDepth, currentDepth + 1)]
      } else {
        res.push(item)
      }
    } else {
      res.push(item)
    }
  })
  return res
}
```

`flatMap()`方法对原数组的每个成员执行一个函数（相当于执行`Array.prototype.map()`），然后对返回值组成的数组执行`flat()`方法。该方法返回一个新数组，不改变原数组。 

```javascript
// 相当于 [[2, 4], [3, 6], [4, 8]].flat(), 先执行map再执行flat
[2, 3, 4].flatMap((x) => [x, x * 2])
// [2, 4, 3, 6, 4, 8]
```

`flatMap()`**只能展开一层数组**。

```javascript
// 相当于 [[[2]], [[4]], [[6]], [[8]]].flat()
[1, 2, 3, 4].flatMap(x => [[x * 2]])
// [[2], [4], [6], [8]]
```

`flatMap`语法如下，里面的回调函数和`map`一致，也能在第二个参数上绑定遍历函数里的`this`

```javascript
arr.flatMap(function callback(currentValue[, index[, array]]) {
  // ...
}[, thisArg])
```

------

## 数组的空位

数组的空位指，数组的某一个位置没有任何值。比如，`Array`构造函数返回的数组都是空位。

```javascript
Array(3) // [, , ,] || [empty × 3]
```

ES5 对空位的处理，已经很不一致了，大多数情况下会忽略空位。

- `forEach()`, `filter()`, `reduce()`, `every()` 和`some()`都会跳过空位。
- `map()`会跳过空位，但会保留这个值
- `join()`和`toString()`会将空位视为`undefined`，而`undefined`和`null`会被处理成空字符串。

```javascript
// forEach方法
[,'a'].forEach((x,i) => console.log(i)); // 1

// filter方法
['a',,'b'].filter(x => true) // ['a','b']

// every方法
[,'a'].every(x => x==='a') // true

// reduce方法
[1,,2].reduce((x,y) => x+y) // 3

// some方法
[,'a'].some(x => x !== 'a') // false

// map方法
[,'a'].map(x => 1) // [,1]

// join方法
[,'a',undefined,null].join('#') // "#a##"

// toString方法
[,'a',undefined,null].toString() // ",a,,"
```

**ES6 则是明确将空位转为`undefined`。** 

`Array.from`方法会将数组的空位，转为`undefined`，也就是说，这个方法不会忽略空位。

```javascript
Array.from(['a',,'b'])
// [ "a", undefined, "b" ]
```

扩展运算符（`...`）也会将空位转为`undefined`。

```javascript
[...['a',,'b']]
// [ "a", undefined, "b" ]
```

`fill()`会将空位视为正常的数组位置。

```javascript
new Array(3).fill('a') // ["a","a","a"]
```

`for...of`循环也会遍历空位。

```javascript
let arr = [, ,];
for (let i of arr) {
  console.log(1);
}
// 1
// 1
```

`entries()`、`keys()`、`values()`、`find()`和`findIndex()`会将空位处理成`undefined`。

```javascript
// entries()
[...[,'a'].entries()] // [[0,undefined], [1,"a"]]

// keys()
[...[,'a'].keys()] // [0,1]

// values()
[...[,'a'].values()] // [undefined,"a"]

// find()
[,'a'].find(x => true) // undefined

// findIndex()
[,'a'].findIndex(x => true) // 0
```

 由于空位的处理规则非常不统一，所以建议避免出现空位。 

