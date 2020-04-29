## 手动输入

- 创建一个数据框`dataFrame`，不需要给定列的值，甚至不需要给定列名，这些都可以手动修改
- 使用`edit`函数修改`dataFrame`的值，并重新赋值给一个变量
- 或者使用`fix`函数直接修改，不需要重新赋值

```R
mydata <- data.frame(age=numeric(0)) # 可以只给列名,并指定列的数据类型

mydata < edit(mydata) # 需要将编辑后的值重新赋值给mydata
fix(mydata) # 不需要重新赋值,使用更方便
```

注意，**一般我们都是手动修改数据框，矩阵等数据结构**，向量的修改直接通过代码即可

------

## 文本文件导入

通过`read.table()`函数读取文件信息，这个方法**只能读取`ANSI`编码的文件**

```R
# 语法
data <- read.table(filePath, header=TRUE, sep=',')

# filePath 文本文件存放路径,不能使用中文
# header 逻辑值,如果是TRUE表示把文本的第一行取出来当作属性名
# sep 分隔符,表示按照分隔符拆分数据
```

------

## excel导入

将`excel`文件转换为`csv`文件，通过`read.csv()`函数读取csv文件

```R
# 语法
data <- read.csv(filePath, header=TRUE, sep=',')

# 参数和read.table()一致,值得注意的是,即使是csv文件,sep也要赋予','
```

------

## 通过数据库访问数据

- 安装`RODBC`包

```R
install.packages('RODBC')
```

- 到[`MYSQL`](https://dev.mysql.com/downloads/connector/odbc/)官网下载`mysql-connector-odbc`并安装
- 通过`windows`管理工具，配置`ODBC数据源`，添加一个用户数据源，选择刚才安装好的`MySQL ODBC 8.0 ANSI Driver`，配置好需要连接的数据库信息就可以了

```R
library(RODBC) # 引用下载好的RODBC包

myconn <- odbcConnect('R data', uid='root', pwd='1qazzaq1') # 创建数据库连接
# 第一个参数是创建用户数据源的时候自定义的名称
# uid和pwd代表数据库连接的账号和密码
```

下面介绍两种操作数据库的方式

- `sqlFetch`，通过表名获取表数据

```R
user <- sqlFetch(myconn, 't_user') # 通过sqlFetch函数获取表数据
# 第一个参数是刚才创建的数据库连接
# 第二个参数是数据库中的表名
```

- `sqlQuery`，通过`sql`语句操作数据库，相比`sqlFetch`更好用

```R
blog <- sqlQuery(myconn, 'select * from t_blog') # 通过sqlQuery函数获取表数据
# 第一个参数是刚才创建的数据库连接
# 第二个参数是sql语句
```

最后关闭数据库连接

```R
close(myconn) # 关闭数据库连接
```

