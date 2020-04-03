## Number.isFinite()，Number.isNaN()

 ES6 在`Number`对象上，新提供了`Number.isFinite()`和`Number.isNaN()`两个方法。 

`Number.isFinite()`用来检查一个数值是否为有限的（finite），即不是`Infinity`。传统的全局方法`isFinite()`会先调用`Number()`将非数值转为数值，再进行判断，而**新方法只对数值有效**。

```javascript
Number.isFinite(15); // true
Number.isFinite(0.8); // true
Number.isFinite(NaN); // false
Number.isFinite(Infinity); // false
Number.isFinite(-Infinity); // false
Number.isFinite('foo'); // false
Number.isFinite('15'); // false
Number.isFinite(true); // false

// 全局方法isFinite()
isFinite(15); // true
isFinite(0.8); // true
isFinite(NaN); // false
isFinite(Infinity); // false
isFinite(-Infinity); // false
isFinite('foo'); // false
// 最后两个例子就是Number.isFinite()和全局方法isFinite()的区别,'15'和true会先调用Number()转成数字15和1
isFinite('15'); // true
isFinite(true); // true
```

 注意，如果参数类型不是数值，`Number.isFinite`一律返回`false`。 

`Number.isNaN()`用来检查一个值是否为`NaN`。传统的全局方法`isNaN()`会先调用`Number()`将非数值转为数值，再进行判断，而**新方法只对数值有效**。

```javascript
Number.isNaN(NaN) // true
Number.isNaN("NaN") // false
Number.isNaN(15) // false
Number.isNaN('15') // false
Number.isNaN(true) // false
Number.isNaN(9/NaN) // true
Number.isNaN('true' / 0) // true
Number.isNaN('true' / 'true') // true

// 全局方法isNaN()
isNaN(NaN) // true
isNaN("NaN") // true,'NaN'经过Number()转换变成了NaN
```

------

## Number.parseInt()，Number.parseFloat()

ES6 将全局方法`parseInt()`和`parseFloat()`，移植到`Number`对象上面，行为完全保持不变。这样做的目的，是逐步减少全局性方法，使得语言逐步模块化。 

```javascript
Number.parseInt === parseInt // true
Number.parseFloat === parseFloat // true
```

------

## Number.isInteger()

`Number.isInteger()`用来判断一个数值是否为整数，只有在参数为`Number`类型的整数时才会返回`true`，不会自动调用`Number()`函数做转换。

```javascript
Number.isInteger(25) // true
Number.isInteger('25') // false，不会调用Number('25')
Number.isInteger() // false，不传参数或者参数不是Number类型，都返回false
Number.isInteger(null) // false
Number.isInteger(true) // false
```

注意：**JavaScript内部，整数和浮点数采用的是同样的储存方法**，所以 25 和 25.0 被视为同一个值。 

```javascript
Number.isInteger(25) // true
Number.isInteger(25.0) // true
```

还有一点，由于 JavaScript 采用 IEEE 754 标准，数值存储为64位双精度格式，数值精度最多可以达到 53 个二进制位（1 个隐藏位与 52 个有效位）。**如果数值的精度超过这个限度，第54位及后面的位就会被丢弃**，这种情况下，`Number.isInteger`可能会误判。 或者**当一个数值的绝对值小于`Number.MIN_VALUE`（5E-324），即小于 JavaScript 能够分辨的最小值，会被自动转为 0**。这时，`Number.isInteger`也会误判。

```javascript
Number.isInteger(3.0000000000000002) // true

Number.isInteger(5E-324) // false
Number.isInteger(5E-325) // true
```

所以对于数据精度要求较高的话，不建议使用`Number.isInteger()`

------

## Number.EPSILON

ES6 在`Number`对象上面，新增一个极小的常量`Number.EPSILON`。根据规格，它表示 **1 与大于 1 的最小浮点数之间的差**。 

对于 64 位浮点数来说，大于 1 的最小浮点数相当于二进制的`1.00..001`，小数点后面有连续 51 个零。这个值减去 1 之后，就等于 2 的 -52 次方。 

```javascript
Number.EPSILON === Math.pow(2, -52) // true
```

引入一个这么小的量的目的，在于为浮点数计算，设置一个误差范围。`Number.EPSILON`可以用来设置“**能够接受的误差范围**”。 

```javascript
function withinErrorMargin (left, right) {
  return Math.abs(left - right) < Number.EPSILON;
}

0.1 + 0.2 === 0.3 // false
withinErrorMargin(0.1 + 0.2, 0.3) // true

1.1 + 1.3 === 2.4 // false
withinErrorMargin(1.1 + 1.3, 2.4) // true
```

------

## Number.MAX_VALUE和Number.MIN_VALUE

`Number.MAX_VALUE` 属性值接近于 `1.79E+308`。大于 `MAX_VALUE` 的值代表 "`Infinity`"。 

`Number.MIN_VALUE` 属性表示在 JavaScript 中所能表示的最小的正值，而不是最小的负值。`MIN_VALUE` 的值约为 `5e-324`。小于 `MIN_VALUE` 的值将会转换为 0。  

一定要区分`Number.MIN_VALUE`和`Number.EPSILON`的区别，**`Number.MIN_VALUE`代表最小正值，而`Number.EPSILON`是最小精度。**

```javascript
Number.MIN_VALUE < Number.EPSILON // true
```

------

## 安全整数和Number.isSafeInteger()

JavaScript 能够准确表示的整数范围在`-2^53`到`2^53`之间（不含两个端点），超过这个范围，无法精确表示这个值。

```javascript
Math.pow(2, 53) // 9007199254740992

9007199254740992  // 9007199254740992
9007199254740993  // 9007199254740992

Math.pow(2, 53) === Math.pow(2, 53) + 1 // true
Math.pow(2, 53) === Math.pow(2, 53) + 2 // false
```

上面代码中，超出 2 的 53 次方之后，**一个数**就不精确了。 

ES6 引入了`Number.MAX_SAFE_INTEGER`和`Number.MIN_SAFE_INTEGER`这两个常量，用来表示这个范围的上下限。 

```javascript
Number.MAX_SAFE_INTEGER === Math.pow(2, 53) - 1
// true
Number.MAX_SAFE_INTEGER === 9007199254740991
// true

Number.MIN_SAFE_INTEGER === -Number.MAX_SAFE_INTEGER
// true
Number.MIN_SAFE_INTEGER === -9007199254740991
// true
```

`Number.isSafeInteger()`则是用来判断一个整数是否落在这个范围之内。

```javascript
Number.isSafeInteger('a') // false
Number.isSafeInteger(null) // false
Number.isSafeInteger(NaN) // false
Number.isSafeInteger(Infinity) // false
Number.isSafeInteger(-Infinity) // false

Number.isSafeInteger(3) // true
Number.isSafeInteger(1.2) // false
Number.isSafeInteger(9007199254740990) // true
Number.isSafeInteger(9007199254740992) // false

Number.isSafeInteger(Number.MIN_SAFE_INTEGER - 1) // false
Number.isSafeInteger(Number.MIN_SAFE_INTEGER) // true
Number.isSafeInteger(Number.MAX_SAFE_INTEGER) // true
Number.isSafeInteger(Number.MAX_SAFE_INTEGER + 1) // false
```

函数实现如下

```javascript
Number.isSafeInteger = function (n) {
  return (typeof n === 'number' &&
    Math.round(n) === n &&
    Number.MIN_SAFE_INTEGER <= n &&
    n <= Number.MAX_SAFE_INTEGER);
}
```

实际使用这个函数时，需要注意。验证运算结果是否落在安全整数的范围内，**不要只验证运算结果，而要同时验证参与运算的每个值**。 

```javascript
Number.isSafeInteger(9007199254740993)
// false
Number.isSafeInteger(990)
// true
Number.isSafeInteger(9007199254740993 - 990)
// true
9007199254740993 - 990
// 返回结果 9007199254740002
// 正确答案应该是 9007199254740003

// 正确的写法，验证每一个参与运算的值
function trusty (left, right, result) {
  if (
    Number.isSafeInteger(left) &&
    Number.isSafeInteger(right) &&
    Number.isSafeInteger(result)
  ) {
    return result;
  }
  throw new RangeError('Operation cannot be trusted!');
}

trusty(9007199254740993, 990, 9007199254740993 - 990)
// RangeError: Operation cannot be trusted!

trusty(1, 2, 3)
// 3
```

------

## Number.POSITIVE_INFINITY和Number.NEGATIVE_INFINITY

`Number.POSITIVE_INFINITY`和`Number.NEGATIVE_INFINITY`代表正负无穷大，和全局对象`Infinity`和其负值一致。

```javascript
Number.POSITIVE_INFINITY === Infinity // true
Number.NEGATIVE_INFINITY === -Infinity // true
```

------

## 指数运算符

ES2016 新增了一个指数运算符（`**`）。作用和`Math.pow`一致，不过`Math.pow`只有两个参数，指数运算符可以多个一起连用。

```javascript
2 ** 2 // 4
2 ** 3 // 8
```

 这个运算符的一个特点是**右结合**，而不是常见的左结合。**多个指数运算符连用时，是从最右边开始计算的**。 

```javascript
2 ** 3 ** 2
// 相当于 2 ** (3 ** 2)
// 512
```

指数运算符可以与等号结合，形成一个新的赋值运算符（`**=`）。

```javascript
let a = 1.5;
a **= 2;
// 等同于 a = a * a;

let b = 4;
b **= 3;
// 等同于 b = b * b * b;
```