## 数据集

数据集通常是**由数据构成的一个矩形数组**，行表示记录，列表示属性（字段）。

我们在进行数据分析时，**数据集就是我们的数据源按照某种数据结构存储的数据**，我们可以根据个人需要的格式创建数据集，有以下两步：

- 选择一种数据结构来存储数据
- 将数据输入或导入到这个数据结构中

------

## 向量

向量是用于储存**数值型**、**字符型**或**逻辑型**数据的一维数组。执行组合功能的函数`c()`可用来创建向量 

```R
a <- c(1, 3, 5, 7, 9) # 1 3 5 7 9
b <- c('one', 'two', 'three') # 'one' 'two' 'three'
d <- c(TRUE, FALSE, TRUE) # TRUE FALSE TRUE
```

注意，**单个向量中数据必须是一样的数据类型**，如果混合使用，会产生以下效果

```R
a <- c(1, 3, 5, '7', '9', TRUE) # '1' '3' '5' '7' '9' 'TRUE' 包含字符型数据,其余所有数据都会被转成字符型
b <- c(1, 3, 5, TRUE, FALSE) # 1 3 5 1 0 只有数值型和逻辑型数据,逻辑型会被转成数值型,TRUE转成1,FALSE转成0
```

向量可以通过下标来访问，注意，**`R`中下标从1开始计数**

```R
a <- c(1, 2, 3, 4, 5, 6, 7)
a[1] # 1
a[5] # 5
a[c(1, 3, 6)] # 1, 3, 6 中括号中间是一个向量,返回下标
a[2:4] # 2, 3, 4 使用[startIndex: endIndex]的语法取startIndex到endIndex中的数据
```

**标量是只含一个元素的向量**，用于保存常量

```R
f <- 3 # 3
g <- 'US' # US
```

------

## 矩阵

矩阵是一个二维数组， 其中**每个元素都拥有相同的模式** ，和向量一样，用于存储**数值型**、**字符型**或**逻辑型**数据。通过函数`matrix()`来创建矩阵。

```R
# 语法
matrix(data = NA, nrow = 1, ncol = 1, byrow = FALSE, dimnames = NULL)

# data 矩阵的元素
# nrow&ncol 指定了行数和列数
# byrow 是否按行填充元素
# dimnames 每行每列的名称,NULL或者是一个长度为2的列表list(rownames, colnames)
```

下面是一些例子

```R
rnames <- c('R1', 'R2')
cnames <- c('C1', 'C2')
x <- matrix(5:8, nrow=2, ncol=2, byrow = TRUE, dimnames=list(rnames, cnames))

#    C1 C2
# R1  5  6
# R2  7  8
```

访问起中的某个元素，也是通过下标的形式完成的

```R
x[2] # 6
x[4] # 8,通过一个下标,访问的是矩阵创建时,源数据的元素
x[1,] # 5 6 第一行
x[, 2] # 6 8 第二列的所有元素
x[2, 2] # 8 第二行第二列的元素

x[,]
#    C1 C2
# R1  5  6
# R2  7  8
# 所有元素,相当于直接访问x
```

------

## 数组

数组(`array`)类似于矩阵，但是它的**维度可以大于 2**。 

```R
# 语法
array(data = NA, dim = length(data), dimnames = NULL)

# data 数组的元素

# dim 维度
# 如果是一个常量n,返回的就是长度为n的向量
# 如果是一个长度为2的向量,返回的是矩阵
# 长度为3及以上的向量,返回的就是一个多维度的数组

# dimnames 每行每列的名称,用法和矩阵一致,只不过列表长度不一定是2,根据数组的维度调整
```

下面是一些例子

```R
dim1 <- c('A1', 'A2', 'A3')
dim2 <- c('B1', 'B2')
dim3 <- c('C1', 'C2', 'C3', 'C4')

arr <- array(1: 24, c(3, 2, 4), dimnames=list(dim1, dim2, dim3))
```

和上面两种数据结构一样，**只能存放单一类型的元素，也能通过下标访问元素**。

```R
arr[3] # 3,通过单一下标访问,访问的是源数据的元素
arr[1,] # Error in arr[1, ] : 量度数目不对

arr[2,,]
#    C1 C2 C3 C4
# B1  2  8 14 20
# B2  5 11 17 23
arr[,,2]
#    B1 B2
# A1  7 10
# A2  8 11
# A3  9 12

arr[2, 2, 2] # 11
```

------

## 数据框

数据框和矩阵的数据结构类似，但是**不同的列可以包含不同的模式的数据**。

```R
# 语法
data.frame(col1, col2, col3...)

# 注意,这里的col1,col2会被直接当做列名,相当于下例中的patientID,age等
```

下面是一些例子

```R
patientID <- c(1, 2, 3, 4)
age <- c(25, 34, 28, 52)
diabetes <- c('TYPE1', 'TYPE2', 'TYPE1', 'TYPE1')
status <- c('Poor', 'Improved', 'Excellent', 'Poor')

# 创建数据框
patientsData <- data.frame(patientID, age, diabetes, status)
#   patientID age diabetes    status
# 1         1  25    TYPE1      Poor
# 2         2  34    TYPE2  Improved
# 3         3  28    TYPE1 Excellent
# 4         4  52    TYPE1      Poor
```

