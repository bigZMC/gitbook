## R的资源

- [R主页](https://www.r-project.org/)
- [CRAN](https://cran.r-project.org/)
- [R博客](https://www.r-bloggers.com/)

------

## 一些基本命令

```R
x <- 5 # 赋值
ls() # 获取当前环境存储的变量值
help.start() # 获取帮助文档首页
help('funcName') | ?funcName # 获取函数的帮助文档
getwd() # 获取当前工作空间
setwd() # 设置工作空间
history() # 当前工程操作历史
```

注意，最好**给不同的项目设置不同的工作空间**，否则每次都会加载工作空间都会加载所有变量，速度会变慢，变量也会互相覆盖。

------

