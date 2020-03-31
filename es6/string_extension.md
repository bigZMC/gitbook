## 模板字符串

模板字符串（template string）是增强版的字符串，用反引号（`）标识。它可以当作普通字符串使用，也可以用来定义多行字符串，或者在字符串中嵌入变量。 

```javascript
// 普通字符串
`In JavaScript '\n' is a line-feed.`

// 多行字符串
`In JavaScript this is
 not legal.`

console.log(`string text line 1
string text line 2`);

// 字符串中嵌入变量
let name = "Bob", time = "today";
`Hello ${name}, how are you ${time}?`
```

如果在模板字符串中需要使用反引号，则前面要用反斜杠转义。 

```javascript
let greeting = `\`Yo\` World!`;
```

如果使用模板字符串表示**多行字符串**，**所有的空格和缩进都会被保留在输出之中**。  如果你不想要这个换行，可以使用`trim`方法消除它。注意，这里的`trim`方法，去除的是首尾空格。

```javascript
$('#list').html(`
<ul>
  <li>first</li>
  <li>second</li>
</ul>
`.trim());
```

模板字符串中嵌入变量， 大括号内部可以放入任意的 JavaScript 表达式，可以进行运算，以及引用对象属性，还能调用函数。 

```javascript
let x = 1;
let y = 2;

`${x} + ${y} = ${x + y}`
// "1 + 2 = 3"

`${x} + ${y * 2} = ${x + y * 2}`
// "1 + 4 = 5"

let obj = {x: 1, y: 2};
`${obj.x + obj.y}`
// "3"

function fn() {
  return "Hello World";
}

`foo ${fn()} bar`
// foo Hello World bar
```

如果模板字符串中的变量没有声明，将报错。 

```javascript
// 变量place没有声明
let msg = `Hello, ${place}`;
// 报错
```

 模板字符串甚至还能嵌套。 但是必须是嵌套在变量中，否则会被认为是模板字符的结束。

```javascript
const tmpl = addrs => `
  <table>
  ${addrs.map(addr => `
    <tr><td>${addr.first}</td></tr>
    <tr><td>${addr.last}</td></tr>
  `).join('')}
  </table>
`;

const data = [
  { first: '<Jane>', last: 'Bond' },
  { first: 'Lars', last: '<Croft>' },
];

console.log(tmpl(data));
// <table>
//
//   <tr><td><Jane></td></tr>
//   <tr><td>Bond</td></tr>
//
//   <tr><td>Lars</td></tr>
//   <tr><td><Croft></td></tr>
//
// </table>
```