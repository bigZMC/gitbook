## String.prototype.concat()

 **`concat()`** 方法将一个或多个字符串与原字符串连接合并，形成一个**新的字符串**并返回，并不影响原字符串。

```javascript
// 语法
str.concat(string2, string3[, ..., stringN])
```

```javascript
var hello = "Hello, ";
console.log(hello.concat("Kevin", " have a nice day.")); /* Hello, Kevin have a nice day. */
```

 另：强烈建议使用**赋值操作符**（+, +=）代替 `concat` 方法。 

------

## String.prototype.indexOf()

`indexOf()` 方法返回调用它的 `String` 对象中第一次出现的指定值的索引，从 `fromIndex` 处进行搜索。如果未找到该值，则返回 -1。

```js
// 语法
str.indexOf(searchValue [, fromIndex = 0])
```

- 如果`searchValue`没有提供值，那么会被设置为`undefined`
- `fromIndex`默认为0，如果`fromIndex`小于0，等同于为空的情况，使用默认值0。如果大于`str.length`，则会直接返回-1。

```javascript
let str = 'hello world'
str.indexOf('o', -5) // 返回 4,相当于str.indexOf('o', 0)
str.indexOf('o', str.length) // 返回 -1
```

如果`searchValue`是一个空字符串，会产生以下结果。

```js
let str = 'hello world'

// 如果fromIndex值为空，或者小于str.length，返回fromIndex的值，可以理解为在fromIndex的索引处就能找到空字符串
str.indexOf('') // 返回 0
str.indexOf('', 0) // 返回 0
str.indexOf('', 3) // 返回 3
str.indexOf('', 8) // 返回 8

// 如果fromIndex大于等于str.length，直接返回str.length
str.indexOf('', 11) // 返回 11
str.indexOf('', 13) // 返回 11
str.indexOf('', 22) // 返回 11
```

------

## String.prototype.lastIndexOf()

`lastIndexOf()` 方法返回调用它的 `String` 对象中最后一次出现的指定值的索引，从 `fromIndex` 处从后向前进行搜索。如果未找到该值，则返回 -1。

```javascript
// 语法
str.lastIndexOf(searchValue [, fromIndex = 0])
```

- 如果`searchValue`没有提供值，那么会被设置为`undefined`
- `fromIndex`的值也是从左向右找到的索引，默认为`+Infinity`，但实际上代表搜索整个字符串。所以如果`fromIndex`大于等于str.length，则会搜索整个字符串。如果`fromIndex`小于0，则等同于0，相当于只**搜索字符串的第一个字符为开头的字符串**。

```javascript
let str = 'canal'

str.lastIndexOf('a');     
// returns 3 （没有指明fromIndex则从末尾l处开始反向检索到的第一个a出现在l的后面，即index为3的位置）

str.lastIndexOf('a', 2);  
// 相当于 'can'.lastIndexOf('a')
// returns 1（指明fromIndex为2则从n处反向向回检索到其后面就是a，即index为1的位置）

str.lastIndexOf('a', 0);
// 相当于 'c'.lastIndexOf('a')
// returns -1(指明fromIndex为0则从c处向左回向检索a发现没有，故返回-1)

str.lastIndexOf('x');     
// returns -1

str.lastIndexOf('c', -5); 
// 相当于 'canal'.lastIndexOf('c', 0) => 'c'.lastIndexOf('c')
// returns 0（指明fromIndex为-5则视同0，从c处向左回向查找发现自己就是，故返回0）

str.lastIndexOf('');      // returns 5
str.lastIndexOf('', 2);   // returns 2
```

注意： **`fromIndex`只限制待匹配字符串的开头。** 

```javascript
'abadefgabm'.lastIndexOf('ab', 7) 
// 返回7，因为第7位是a，第8位是b，虽然超过了fromIndex的显示，但是其只限制开头，所以仍能连接成ab，所以返回7
```

------

## String.prototype.slice()

 **`slice()`** 方法提取某个字符串的一部分，并返回一个新的字符串，且不会改动原字符串。 

```javascript
// 语法
str.slice(beginIndex[, endIndex])
```

- `beginIndex`默认值为0，如果小于0，会被当作`str.length+beginIndex`看待。
- `endIndex`可选，如果省略该参数，`slice()`会一直提取到字符结尾。如果小于0，会被当作`str.length+endIndex`看待。
- 注意：`slice()`提取的新字符串，包括`beginIndex`但不包括 `endIndex`。如果`beginIndex`大于等于`endIndex`，则直接返回空字符串。

```javascript
var str1 = 'The morning is upon us.', // str1 的长度 length 是 23。
    str2 = str1.slice(1, 8),
    str3 = str1.slice(4, -2),
    str4 = str1.slice(12),
    str5 = str1.slice(30);
console.log(str2); // 输出：he morn
console.log(str3); // 输出：morning is upon u
console.log(str4); // 输出：is upon us.
console.log(str5); // 输出：""
```

------

## String.prototype.split()

`**split()** `方法使用指定的分隔符字符串将一个`String`对象分割成子字符串数组，以一个指定的分割字串来决定每个拆分的位置。  

```javascript
// 语法
str.split([separator[, limit]])
```

- `separator`指定表示每个拆分应发生的点的字符串。`separator`可以是字符串也可以是正则表达式。如果`separator`不传或者在原字符串中找不到，则返回一个数组，包含一个由整个字符串组成的元素。如果分隔符为空字符串，则将原字符串中每个字符的数组形式返回。
- `limit`限制分割片段数量。如果分割后的`arr.length < limit`，那么结果不变。如果`arr.length > limit`，执行`arr.length = limit`操作进行截断。

```javascript
let myString = "Hello World. How are you doing?";

myString.split(" ", 3); // ["Hello", "World.", "How"]
myString.split(" ", 10); // ["Hello", "World.", "How", "are", "you", "doing?"]
```

如果使用一个数组作为分隔符，相当于先调用`toString()`方法

```javascript
const myString = 'ca,bc,a,bca,bca,bc';

const splits = myString.split(['a','b']); 
// myString.split(['a','b']) is same as myString.split(String(['a','b'])) 
// 相当于 const splits = myString.split('a,b'); 

console.log(splits);  //["c", "c,", "c", "c", "c"]
```

------

## String.prototype.substring()

`substring() `方法返回一个字符串在开始索引到结束索引之间的一个子集, 或从开始索引直到字符串的末尾的一个子集。 

```javascript
// 语法
str.substring(indexStart[, indexEnd])
```

- `indexStart`表示需要截取的第一个字符的索引，截取的字符串**包含**该索引对应的字符
- `indexEnd`可选，一个 0 到字符串长度之间的整数，截取的字符串**不包含**该索引对应的字符
-  如果 `indexStart` 等于 `indexEnd`，`substring` 返回一个空字符串。 
-  如果任一参数小于 0 或为 `NaN`，则被当作 0。 
-  如果任一参数大于 `str.length`，则被当作 `str.length`。 
-  如果 `indexStart` 大于 `indexEnd`，则 `substring` 的执行效果就像两个参数调换了一样。 

```javascript
let anyString = "Mozilla";

// 输出 "Moz"
console.log(anyString.substring(0,3));
// indexStart > indexEnd,两个参数对调,相当于anyString.substring(0,3)
console.log(anyString.substring(3,0)); 
// indexEnd < 0,直接看作0,相当于anyString.substring(3, 0),然后由于indexStart > indexEnd,参数对调
console.log(anyString.substring(3,-3));
// NaN也视作0
console.log(anyString.substring(3,NaN));
console.log(anyString.substring(-2,3));
console.log(anyString.substring(NaN,3));

// 输出 "lla"
console.log(anyString.substring(4,7));
console.log(anyString.substring(7,4));

// 输出 ""
console.log(anyString.substring(4,4));

// 输出 "Mozill"
console.log(anyString.substring(0,6));

// 输出 "Mozilla"
console.log(anyString.substring(0,7));
// endIndex > str.length,相当于anyString.substring(0,anyString.length)
console.log(anyString.substring(0,10));
```

小技巧：使用length属性来截取字符串最后n位字符

```javascript
let anyString = 'Mozilla';

anyString.substring(anyString.length - 4); // 'illa'
```

------

## String.prototype.toLowerCase()，String.prototype.toUpperCase()

```javascript
// 语法
str.toLowerCase()
str.toUpperCase()
```

`toLowerCase` 会将调用该方法的字符串值转为小写形式，`toUpperCase` 会将调用该方法的字符串值转为大写形式 ，均不会影响字符串本身的值。 

```javascript
console.log('中文简体 zh-CN || zh-Hans'.toLowerCase());
// 中文简体 zh-cn || zh-hans

console.log( "ALPHABET".toLowerCase() ); 
// "alphabet"

console.log( "alphabet".toUpperCase() ); 
// "ALPHABET"
```

------

## String.prototype.toString()，String.prototype.valueOf()

```javascript
// 语法
str.toString()
str.valueOf()
```

String对象的`valueOf`方法返回一个String对象的**原始值**。该值等同于`String.prototype.toString()`。

```javascript
x = new String("Hello world");
x.valueOf() // "Hello world"
x.toString() // "Hello world"
```

------

## String.prototype.search()

`search()` 方法执行正则表达式和 `String`对象之间的一个搜索匹配。如果找不到，返回-1。

```javascript
// 语法
str.search(regexp)
```

-  regexp：正则表达式对象。如果传入一个非正则表达式对象，则会使用`new RegExp(obj)`进行隐式转换。如果是一个字符串，那么返回值等于`indexOf`方法。

```javascript
let str = "hey JudE";

const re = /[A-Z]/g;
const re2 = /[.]/g;
console.log(str.search(re)); // returns 4, which is the index of the first capital letter "J"
console.log(str.search(re2)); // returns -1 cannot find '.' dot punctuation
```

------

## String.prototype.replace()

`replace()` 方法返回一个由替换值（`replacement`）替换一些或所有匹配的模式（`pattern`）后的新字符串。模式可以是一个字符串或者一个正则表达式，替换值可以是一个字符串或者一个每次匹配都要调用的回调函数。 

```javascript
// 语法
str.replace(regexp|substr, newSubStr|function)
```

- `regexp(pattern)`：一个正则表达式对象或其字面量。
- `substr(pattern)`：一个被`newSubStr`替换的字符串。
- `newSubStr(replacement)`：用于替换掉第一个参数在原字符串中的匹配部分的字符串。*该字符串中可以内插一些特殊的变量名*。
- `function(replacement)`：一个用来创建新子字符串的函数，该函数的返回值将替换掉第一个参数匹配到的结果。

注意：没有`replaceAll`方法，进行全局搜索替换时，正则表达式需包含`g`标识。

```javascript
var str = 'Twas the night before Xmas...';
var newstr = str.replace(/xmas/i, 'Christmas'); // i表示忽略大小写
console.log(newstr);  // Twas the night before Christmas...
```

```javascript
var re = /apples/gi;
var str = "Apples are round, and apples are juicy.";
var newstr = str.replace(re, "oranges");

// oranges are round, and oranges are juicy.
console.log(newstr);
```

```javascript
function styleHyphenFormat(propertyName) {
  function upperToHyphenLower(match) {
    return '-' + match.toLowerCase();
  }
  return propertyName.replace(/[A-Z]/g, upperToHyphenLower);
}

// 通过行内函数匹配大写字母
styleHyphenFormat('borderTop') // border-top
```

------

## String.prototype.match()

`match()` 方法检索返回一个字符串匹配正则表达式的的结果。 如果找不到则返回`null`。

```javascript
// 语法
str.match(regexp)
```

- regexp：一个正则表达式对象。如果传入的是非正则表达式对象，则会隐式地使用 `new RegExp(obj)` 将其转换为一个 `RegExp` 。

```js
var str = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz';
var regexp = /[A-E]/gi;
var matches_array = str.match(regexp);

console.log(matches_array);
// ['A', 'B', 'C', 'D', 'E', 'a', 'b', 'c', 'd', 'e']
```

注意： 如果正则表达式不包含 `g `标志，`str.match()` 将返回与 `RegExp.exec()`. 相同的结果。 

- `groups`: 一个捕获组数组 或 `undefined`。
- `index`: 匹配的结果的开始位置
- `input`: 搜索的字符串.

```javascript
var str = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz';
var regexp = /[A-E]/i;
var matches_array = str.match(regexp);

console.log(matches_array);
// ["A", index: 0, input: "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz", groups: undefined]
```

如果你没有给出任何参数并直接使用match() 方法 ，你将会得到一个包含空字符串的 `Array` ：[''] 。 

```javascript
var str = "Nothing will come of nothing.";

str.match();   // returns [""]
```

当参数是一个字符串或一个数字，它会使用`new RegExp(obj)`来隐式转换成一个 `RegExp`。如果它是一个有正号的正数，`RegExp()`方法将忽略正号。 

```js
var str1 = "NaN means not a number. Infinity contains -Infinity and +Infinity in JavaScript.",
    str2 = "My grandfather is 65 years old and My grandmother is 63 years old.",
    str3 = "The contract was declared null and void.";

str1.match("number");   // "number" 是字符串。返回["number"]
str1.match("grandfather");   // "grandfather" 没有找到,返回null
str1.match(NaN);        // NaN的类型是number。返回["NaN"]
str1.match(Infinity);   // Infinity的类型是number。返回["Infinity"]
str1.match(+Infinity);  // 返回["Infinity"]
str1.match(-Infinity);  // 返回["-Infinity"]
str2.match(65);         // 返回["65"]
str2.match(+65);        // 有正号的number。返回["65"]
str3.match(null);       // 返回["null"]
```

------

## String.prototype.matchAll()

`matchAll()` 方法返回一个包含所有匹配正则表达式的结果及分组捕获组的**迭代器**。 

```javascript
// 语法
str.matchAll(regexp)
```

- regexp：一个正则表达式对象。如果传入的是非正则表达式对象，则会隐式地使用 `new RegExp(obj)` 将其转换为一个 `RegExp` 。

```javascript
const string = 'test1test2test3';

// g 修饰符加不加都可以
const regex = /t(e)(st(\d?))/g;

for (const match of string.matchAll(regex)) {
  console.log(match);
}
// ["test1", "e", "st1", "1", index: 0, input: "test1test2test3"]
// ["test2", "e", "st2", "2", index: 5, input: "test1test2test3"]
// ["test3", "e", "st3", "3", index: 10, input: "test1test2test3"]
```

由于`string.matchAll(regex)`返回的是遍历器，所以可以用`for...of`循环取出。相对于返回数组，返回遍历器的好处在于，如果匹配结果是一个很大的数组，那么遍历器比较节省资源。 