## cookie

`cookie`是存储在浏览器的一段字符串，**会放在请求头中和`server`端通信**，这是和`web`本地存储`sessionStorage`和`localStorage`最大的区别

- 有大小限制，最大为`4kb`，一般不超过20个键值对
- 跨域不共享，`SameSite`同源策略
- `cookie`格式为`k1=v1;k2=v2;k3=v3`，因此可以存储结构化数据
- 可以设置过期时间，如果不设置则和`sessionStorage`生命周期一致，关闭浏览器窗口时销毁

`server`端操作`cookie`

```javascript
// 获取cookie过期时间,设置在一天后
const getCookieExpires = () => {
  const d = new Date()
  d.setTime(d.getTime() + (24 * 60 * 60 * 1000))
  return d.toGMTString() // 返回cookie固定时间格式GMT
}

// 设置cookie
res.setHeader('Set-Cookie', `username=zmc;path=/;httpOnly;expires=${getCookieExpires()}`)

// httpOnly 代表cookie只能由服务端修改,前端的任何修改无效
// path 指定cookie生效的路由
// expires 设置过期时间,不设置默认浏览器关闭时销毁cookie
```

