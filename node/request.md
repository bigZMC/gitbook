## http请求概述

- DNS解析（通过域名解析ip），建立TCP连接（三次握手），发送http请求
- server端接收到http请求，处理并返回
- 客户端接收到返回数据，处理数据

------

## nodejs处理http请求

1. ### get请求和querystring

- `get`请求，即客户端要向`server`端获取数据
- 通过`querystring`来传递参数
- 浏览器直接访问，就发送`get`请求

```javascript
const http = require('http')
const querystring = require('querystring')

// 通过直接访问浏览器地址栏, get请求
// localhost:8000/api/blog/list?author=zmc&age=26
const server = http.createServer((req, res) => {
  console.log('method: ', req.method) // GET
  const url = req.url
  console.log('url: ', url) // /api/blog/list?author=zmc&age=26
  req.query = querystring.parse(url.split('?')[1])
  console.log('query: ', req.query) // { author: 'zmc', age: '26' }
  res.end(JSON.stringify(req.query))
})

server.listen(8000)
```

2. ### post请求和post data

- `post`请求，即客户端向 `server` 端传递数据
- 通过`post data`传递数据

```javascript
const http = require('http')

const server = http.createServer((req, res) => {
  if (req.method === 'POST') {
    // req 数据格式
    console.log('req content-type', req.headers)
    // 接收数据
    let postData = ''
    req.on('data', chunk => {
      postData += chunk.toString()
    })
    req.on('end', () => {
      console.log('postData: ', postData, typeof postData)
      res.end('Hello World')
    })
  }
})

server.listen(8000)
```

------

## 搭建开发环境

- 使用`nodemon`检测文件变化，自动重启`node`

```javascript
cnpm install --save-dev nodemon
```

安装好后，修改`package.json`中的命令，使用`nodemon`代替`node`命令，还可以单独进行`nodemon.json`的配置，可以参照[官网]( https://github.com/remy/nodemon/blob/master/doc/sample-nodemon.md )的例子

```javascript
"scripts": {
	"dev": "nodemon ./bin/www.js"
}
```

- 使用`cross-env`设置环境变量

当我们使用`NODE_ENV = production`来设置环境变量的时候，大多数`windows`命令会提示将会阻塞或者异常，或者`windows`不支持`NODE_ENV=development`的这样的设置方式，会报错。 所以，我们就可以使用`cross-env`命令，这样我们就不必担心平台设置或使用环境变量了。也就是说**`cross-env`能够提供一个设置环境变量的`scripts`，这样我们就能够以`unix`方式设置环境变量，然而在`windows`上也能够兼容的**。 

```javascript
cnpm install --save-dev cross-env
```

再次修改`package.json`的命令，设置不同的`NODE_ENV`

```javascript
"scripts": {
	"dev": "cross-env NODE_ENV=dev nodemon ./bin/www.js",
  "prod": "cross-env NODE_ENV=production nodemon ./bin/www.js"
}
```

