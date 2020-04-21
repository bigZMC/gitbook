## 操作MYSQL

- 安装`mysql`库

```javascript
cnpm install --save mysql
```

- 使用`node`操作数据库

```javascript
const mysql = require('mysql')

// 创建连接对象
const con = mysql.createConnection({
  host: 'localhost', // 连接地址
  user: 'root', // 用户名
  password: '1qazzaq1', // 密码
  port: '3306', // 端口
  database: 'myblog' // 数据库名称
})

// 开始连接
con.connect()

// 执行sql语句
const sql = 'select id, username from t_user;'
con.query(sql, (err, result) => {
  if(err) {
    console.error('执行sql语句错误', err)
    return
  }
  console.log('执行sql语句结果为 ====>', result)
})

// 关闭连接
con.end()
```

------

## 将数据库连接方法封装成公共函数

- 创建`conf.js`文件，**根据环境参数不同，返回不同的数据库连接**

```javascript
const env = process.env.NODE_ENV // 环境参数

// 配置
let MYSQL_CONF

if (env === 'dev') {
  MYSQL_CONF = {
    // 开发环境连接数据库配置信息
  }
} else if (env === 'production') {
  MYSQL_CONF = {
    // 生产环境连接数据库配置信息
  }
}

module.exports = {
  MYSQL_CONF
}
```

- 创建`db.js`文件，引入`MYSQL_CONF`返回的数据库连接

```javascript
const mysql = require('mysql')
const {
  MYSQL_CONF
} = require('../conf/db')

// 创建连接对象
const con = mysql.createConnection(MYSQL_CONF)
```

- 使用`promise`封装`query`函数的返回值

```javascript
// 执行sql的函数
function exec(sql) {
  const promise = new Promise((resolve, reject) => {
    con.query(sql, (err, result) => {
      if (err) {
        reject(err)
        return
      }
      resolve(result)
    })
  })
  return promise
}
```

- 最后，导出`exec`函数，作为统一执行`sql`的函数

```javascript
module.exports = {
  exec
}
```

注意，这里没有使用`con.end()`进行数据库连接关闭，是由于我们只引用一次`db.js`，而不是每次使用都去创建一次连接，可以看作是**单例模式**。而且`exec`函数是异步的，如果关闭后就不能再执行`sql`语句。

