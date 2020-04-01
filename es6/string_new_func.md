## String.fromCodePoint()

ES5 提供`String.fromCharCode()`方法，用于从 Unicode 码点返回对应字符，但是这个方法不能识别码点大于`0xFFFF`的字符。 

```javascript
String.fromCharCode(0x20BB7)
// "ஷ"
```

`String.fromCharCode()`不能识别大于`0xFFFF`的码点，所以`0x20BB7`就发生了溢出，最高位`2`被舍弃了，最后返回码点`U+0BB7`对应的字符，而不是码点`U+20BB7`对应的字符。 

ES6 提供了`String.fromCodePoint()`方法，可以识别大于`0xFFFF`的字符，弥补了`String.fromCharCode()`方法的不足。

```javascript
String.fromCodePoint(0x20BB7)
// "𠮷"
String.fromCodePoint(0x78, 0x1f680, 0x79) === 'x\uD83D\uDE80y'
// String.fromCodePoint(0x78) x
// String.fromCodePoint(0x1f680) 🚀
// String.fromCodePoint(0x79) y
// true
```

 如果`String.fromCodePoint`方法有多个参数，则它们会被**合并成一个字符串返回**。 

------

## 实例方法：codePointAt()

JavaScript 内部，字符以 UTF-16 的格式储存，每个字符固定为`2`个字节。对于那些需要`4`个字节储存的字符（Unicode 码点大于`0xFFFF`的字符），JavaScript 会认为它们是两个字符。 

```javascript
// UTF-16 编码为0xD842 0xDFB7（十进制为55362 57271）
var s = "𠮷";

s.length // 2

// charAt()方法无法读取整个字符
s.charAt(0) // ''
s.charAt(1) // ''

// charCodeAt()方法只能分别返回前两个字节和后两个字节的值。
s.charCodeAt(0) // 55362
s.charCodeAt(1) // 57271

var s = "𠮷a";
s.codePointAt(0) // 134071
s.codePointAt(1) // 57271

s.codePointAt(2) // 97
```

`codePointAt()`方法的参数，是字符在字符串中的位置（从 0 开始）。上面代码中，JavaScript 将“𠮷a”视为**三个字符**，codePointAt 方法在第一个字符上，正确地识别了“𠮷”，返回了它的十进制码点 134071（即十六进制的`20BB7`）。在第二个字符（即“𠮷”的后两个字节）和第三个字符“a”上，`codePointAt()`方法的结果与`charCodeAt()`方法相同。 

但是，此时的参数仍然不正确， 字符`a`在字符串`s`的正确位置序号应该是 1，但是必须向`codePointAt()`方法传入 2。解决这个问题的一个办法是使用`for...of`循环 。

```javascript
let s = '𠮷a';
for (let ch of s) {
  console.log(ch.codePointAt(0).toString(16));
}
// 20bb7
// 61
```

 另一种方法也可以，使用扩展运算符（`...`）进行展开运算。 

```javascript
let arr = [...'𠮷a']; // arr.length === 2
arr.forEach(
  ch => console.log(ch.codePointAt(0).toString(16))
);
// 20bb7
// 61
```

`codePointAt()`方法是测试一个字符由两个字节还是由四个字节组成的最简单方法。

```javascript
function is32Bit(c) {
  // 返回10进制，和16进制最大数进行比较
  return c.codePointAt(0) > 0xFFFF;
}

is32Bit("𠮷") // true
is32Bit("a") // false
```

------

## 实例方法：includes()，startsWith()，endsWith()

ES6之前，只有`indexOf`方法可以用来确定一个字符串是否包含在另一个字符串中。ES6 提供了三种新方法。

- **includes()**：返回布尔值，表示是否找到了参数字符串。
- **startsWith()**：返回布尔值，表示参数字符串是否在原字符串的头部。
- **endsWith()**：返回布尔值，表示参数字符串是否在原字符串的尾部。

```javascript
let s = 'Hello world!';

s.startsWith('Hello') // true
s.endsWith('!') // true
s.includes('o') // true
```

这三个方法都支持第二个参数，表示开始搜索的位置(position)或者字符长度(length)。

```javascript
let s = 'Hello world!';

s.startsWith('world', 6) // true
// 这里的n从0开始数，针对从第n个位置到字符串结束
// 相当于 'world'.startsWith('world')

s.endsWith('Hello', 5) // true
// 这里需要注意，endsWith的第二个参数，针对的是前n个字符
// 相当于 'Hello'.startsWith('world')

s.includes('Hello', 6) // false
// 相当于 'world'.includes('Hello')
```

------

## 实例方法：repeat()

 `repeat`方法返回一个新字符串，表示将原字符串重复`n`次。 

```javascript
'x'.repeat(3) // "xxx"
'hello'.repeat(2) // "hellohello"
'na'.repeat(0) // ""，0直接返回空字符串，相当于乘法规则
```

参数如果是小数，会被取整。大于0向下取整，在0到-1之间，则等同于0。如果是小于等于-1的参数，则会报错。

```javascript
'na'.repeat(2.9) // "nana"
'na'.repeat(-0.9) // ""，等同于'na'.repeat(0)
```

 参数如果是字符串会先转换成数字，`NaN`等同于 0。 

```javascript
'na'.repeat('3') // "nanana"
'na'.repeat(NaN) // ""
'na'.repeat('na') // ""， 'na'是NaN
```

------

## 实例方法：padStart()，padEnd()

`padStart()`用于头部补全，`padEnd()`用于尾部补全。如果某个字符串不够指定长度，会在头部或尾部补全。 

`padStart()`和`padEnd()`一共接受两个参数，第一个参数是字符串补全生效的最大长度，第二个参数是用来补全的字符串。 

```javascript
'x'.padStart(5, 'ab') // 'ababx'
'x'.padStart(4, 'ab') // 'abax'，达到长度后，会截取超出位数的补全字符串

'x'.padEnd(5, 'ab') // 'xabab'
'x'.padEnd(4, 'ab') // 'xaba'
```

如果原字符串的长度，等于或大于最大长度，则字符串补全不生效，返回原字符串。

```javascript
'xxx'.padStart(2, 'ab') // 'xxx'
'xxx'.padEnd(2, 'ab') // 'xxx'
```

如果省略第二个参数，**默认使用空格补全长度**。

```javascript
'x'.padStart(4) // '   x'
'x'.padEnd(4) // 'x   '
```

------

## 实例方法：trimStart()，trimEnd()

新增了`trimStart()`和`trimEnd()`这两个方法。它们的行为与`trim()`一致，`trimStart()`消除字符串头部的空格，`trimEnd()`消除尾部的空格。它们返回的都是新字符串，不会修改原始字符串。 

```javascript
const s = '  abc  ';

s.trim() // "abc"
s.trimStart() // "abc  "
s.trimEnd() // "  abc"
```

浏览器还部署了额外的两个方法，`trimLeft()`是`trimStart()`的别名，`trimRight()`是`trimEnd()`的别名。 