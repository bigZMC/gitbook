## 自定义函数

通过`function`语法进行创建

```R
myfunction <- function(arg1, arg2, ...) {
  statements
  return(object)
}
```

下面是几个例子

```R
# 通过switch判断type类型输出不同格式的时间
# 使用cat函数捕获除long和short外的情况
mydate <- function(type) {
  switch(type,
         long = format(Sys.time(), '%A %B %d %Y'),
         short = format(Sys.time(), '%m-%d-%y'),
         cat(type, 'is not recognized type\n')
         )
}
```

```R
# 传入num,通过for循环进行累加
# 注意,return的时候必须给x加上括号,否则会报错 unexpected symbol
sum <- function(num) {
  x <- 0
  for(i in 1:num) {
    x <- x + i
  }
  return (x)
}
```

