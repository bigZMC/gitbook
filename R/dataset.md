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

可以通过列下标或者列名取值

```R
patientsData[1:2]
#   patientID age
# 1         1  25
# 2         2  34
# 3         3  28
# 4         4  52

patientsData[c('diabetes', 'status')]
#   diabetes    status
# 1 TYPE1      Poor
# 2 TYPE2  Improved
# 3 TYPE1 Excellent
# 4 TYPE1      Poor
```

如果只想单独取一列，有两种方法

- 通过`$`符号对单独某一列取值
- **使用`attact()`函数将数据框加入`R`的搜索路径**，然后直接输入列名即可获得单独某一列的值

```R
# $符号取值
patientsData$age # 25 34 28 52

# attach()函数
attach(patientsData) # 将patientsData加入R搜索路径
age # 25 34 28 52
patientID # 1 2 3 4
```

还可以**通过`detach()`将数据框从R的搜索路径中移除**

```R
detach(patientsData) # 将patientsData从R搜索路径中移除
age # object 'age' not found
```

最后，还可以通过`with()`函数来快速访问数据框的某一列。**注意`with()`函数的大括号会形成一个作用域**，内部的变量不能在外层访问，**如果需要创建在`with()`以外的对象，使用`<<-`代替`<-`进行赋值**。

```R
with(patientsData, {
  scopeAge <- age # 将数据框中age列赋值给scopeAge
  scopeAge # 输出scopeAge
  scopePatientID <<- patientID
})

scopeAge # object 'scopeAge' not found
scopePatientID # 1 2 3 4
```

------

## 因子

`R`把表示分类的数据称为因子(`factor`)，因子具有因子水平(`level`)，用于限制因子的元素的取值范围。 

```R
# 语法
factor(x = character(), levels, labels = levels, exclude = NA, ordered = is.ordered(x), nmax = NA)
```

- x 向量，因子的数据源
- levels 水平，字符类型，用于设置`x`可能包含的唯一值，**默认是`x`的所有唯一值**。如果x不是字符向量，则用`as.character(x)`把`x`转换为字符向量。
- labels `level`的标签，字符类型，对因子水平重命名
- exclude 排除的字符
- ordered 逻辑值(`TRUE/FALSE`)，用于指定水平是否有序
- nmax 水平的上限数量

注意：通常情况下，在创建数据框时，**`R`隐式地把数据类型为字符的列创建为因子**

```R
class(patientsData$diabetes) # factor
```

### 因子水平

因为因子水平规定了因子的取值范围，每一个因子，都包含了水平信息。如果想单独查看因子水平，可以通过`levels()`函数来查看。

```R
patientsData$diabetes
# TYPE1 TYPE2 TYPE1 TYPE1
# Levels: TYPE1 TYPE2

patientsData$diabetes[1]
# TYPE1
# Levels: TYPE1 TYPE2

levels(patientsData$diabetes)
# TYPE1 TYPE2
```

可以看到，因子水平是`TYPE1`和`TYPE2`，如果把其他字符添加到`diabetes`列，`R`会抛出警告信息，并将错误赋值的元素设置为`NA`

```R
patientsData$diabetes[1] <- 'TYPE3'
# Warning message:
# In `[<-.factor`(`*tmp*`, 2, value = c(NA, 1L, 1L, 1L)) :
# invalid factor level, NA generated

patientsData$diabetes[1] # <NA>
```

在创建因子时，可以使用`labels`为每个因子水平添加标签，字符顺序要和`levels`参数的字符顺序一致

```R
sex <- factor(c('f','m','f','f','m'), levels=c('f','m'), labels=c('female','male'))

# female male female female male
# Levels: female male
```

### 有序因子

通常情况下，因子是无序的，可以通过`is.ordered()`函数判断

```R
is.ordered(sex) # FALSE
```

我们可以通过以下两种方法创建有序因子

- `ordered()`函数，缺点是不能指定特定因子水平的顺序

```R
ordered(sex)
# female male female female male
# Levels: female < male
```

- 在使用`factor()`函数创建时，添加`ordered=TRUE`参数

```R
sex <- factor(c('f','m','f','f','m'), levels=c('f','m'), ordered=TRUE)

# f m f f m
# Levels: f < m

# 如果一开始没有使用ordered=TRUE,也可以重新指定
sex <- factor(sex, levels=c('m','f'), ordered=TRUE)

# f m f f m
# Levels: m < f
```

------

## 列表

列表**包含不同类型**的元素，通过`list()`函数进行创建，列表能整合若干(可能无关的)对象到单个对象名下。例如，某个列表中可能是若干向量、矩阵、数据框，甚至是其他列表的组合。

从技术上讲，列表就是向量。之前我们接触过的**普通向量都称为“原子型(atomic)”向量**，就是说，向量的元素已经是 最小的、不可再分的。而**列表则属于“递归型(recursive)”向量**。 

```R
# 语法
list(key1=object1, key2=object2, ...)
```

下面是一些例子

```R
mylist <- list(id=27, name='TOM', scores=c(10, 15, 13))
# $id
# [1] 27

# $name
# [1] "TOM"

# $scores
# [1] 10 15 13
```

我们通常**通过双中括号(`[[]]`)的形式取值**，这是因为`list`类型的元素被看作是一个子列表，只有用双中括号才能取到相应的值

```R
mylist[1]
# $id
# [1] 27
mode(mylist[1]) # list

mylist[[1]] # 27
mode(mylist[[1]]) # numeric

# 还可以用$符号快速取值,和数据框语法一致
mylist$id # 27
```

通过`names`函数获得列表所有元素的名称

```R
names(mylist) # id name scores

# 还可以直接对names(mylist)赋值,修改名称
names(mylist) <- c('id', 'name', 'marks')

mylist
# $id
# [1] 27

# $name
# [1] "TOM"

# $marks
# [1] 10 15 13
```

对列表可以取出某一列进行赋值，如果列名存在就修改列值，如果不存在就是添加一个元素

```R
mylist$id <- 1
mylist$parents <- c('Mama', 'Baba')

mylist
# $id
# [1] 1

# $name
# [1] "TOM"

# $scores
# [1] 10 15 13

# $parents
# [1] "Mama" "Baba"
```

还可以通过`c()`函数合并两个列表

```R
otherlist <- list(age=20, sex='male')

otherlist
# $age
# [1] 20

# $sex
# [1] "male"

lt <- c(mylist, otherlist)
# $id
# [1] 1

# $name
# [1] "TOM"

# $scores
# [1] 10 15 13

# $parents
# [1] "Mama" "Baba"

# $scores
# [1] 10 15 13

# $parents
# [1] "Mama" "Baba"
```

列表还能转换成向量的元素，使用`unlist()`函数展开列表

```R
unlist(lt)

# id  name  scores1 scores2 scores3 parents1 parents2 age  sex 
# "1" "TOM" "10"    "15"    "13"    "Mama"   "Baba"   "20" "male" 
```

