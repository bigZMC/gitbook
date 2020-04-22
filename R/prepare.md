## R的资源

- [R主页](https://www.r-project.org/)
- [CRAN](https://cran.r-project.org/)
- [R博客](https://www.r-bloggers.com/)

------

## 一些基本命令

```R
x <- 5 # 赋值

help.start() # 获取帮助文档首页
help('funcName') | ?funcName # 获取函数的帮助文档
help(package='packageName') # 列出包中可用函数和数据集
example('funcName') # 获取函数的使用示例

ls() # 获取当前环境存储的变量值
rm(list=ls()) # 清空工作空间

getwd() # 获取当前工作空间
setwd() # 设置工作空间

history() # 当前工程操作历史
```

注意，最好**给不同的项目设置不同的工作空间**，否则每次都会加载工作空间都会加载所有变量，速度会变慢，变量也会互相覆盖。

------

## 包的使用                                                                                                                                                                                                                                                                                    

包(`package`)是R函数、数据、预编译代码以一种定义完善的格式组成的集合。存储包的目录成为库(`library`)。

```R
install.packages(packageName) # 安装包,如果没有packageName,会在选择了CRAN镜像站点后列出所有可用包,选择安装
update.packages() # 更新包
installed.packages() # 查看所有安装的包

library() # 查看依赖的包
library(packageName) # 载入某个包
help(package='package_name') # 输入某个包的简短描述
```

