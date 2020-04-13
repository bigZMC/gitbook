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

