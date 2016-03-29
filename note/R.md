# 资源

http://www.biosino.org/R/R-doc/



# 随记

- 当某个元素或者数值从统计角度讲是"不可用"(not avaliable)或者"缺失值"(missing value)时,它们在向量中的位置将被保留,同时被赋值为一个特殊的值`NA`.一般来讲一个NA的任何操作都将返回NA.
- 函数`is.na(v)`返回一个逻辑向量,这个向量与v有相同的长度,并且由相应位置的元素是否是NA来决定这个逻辑向量相应位置的元素是`TRUE`还是`FALSE`
- NaN(Not a Number),是一种"缺失"值,是由数值运算产生的.
- 函数`is.na(v)`对于`NA`和`NaN`值都返回`TRUE`.要想区分,函数`is.nan(v)`只对`NaN`值返回`TRUE`.

## 对象的其他类型

- 矩阵(matrices)或者更一般的说`数组`是向量在多维情况下的一般形式.
- 因子(factors)提供一种处理分类数据的更简介的方式.
- 列表(list)是向量的一种一般形式,并不需要保证其中的元素都是相同的的类型,而且其中的元素经常是向量和列表本身.
- 数据框(data frames)是一种与矩阵相似的结构,其中的列可以是不同的数据类型.可以把数据框看作一种数据"矩阵",它的每行是一个观测单位,而且(可能)同时包含数值型和分类的变量.
- 函数(functions)是能够在R的workspace中存储的对象.

## 对象及它们的模式和属性

- R的对象类型包括数值型(numeric),复数型(comlex),逻辑型(logical),字符型(character),原味型(raw)
- 向量必须保证它的所有元素是一样的模式.
- 列表可以为任何模式的对象的有序序列,列表被认为是一种“递归”结构而不是原子结构因为它们的元素可以以它们各自的方式单独列出.
- 另外两种递归结构是函数(function)和表达式(expression).
- as.`something()`函数,主要用于对象模式的强制转换,或者赋于某个对象一些先前没有的功能.
- 读取和设置属性

  函数attributes(object) 给出对象当前定义的非内在属性(non-intrinsic at- tributes)的列表。函数attr(object, name) 可以用来选择特定的属性。

- R 里面的所有对象都属于一个类(class),可以通过函数class 查看。对于简 单的向量,就是对应的模式"numeric","logical","character" 或者"list",但 是"matrix","array","factor" 和"data.frame" 就可能是其他值。
- 用函数unclass() 临时去掉一个对象的类作用。

## 有序因子和无序因子

- 因子(factor)是一个对等长的其他向量元素进行分类(分组)的向量对象。 R 同时提供有序(ordered)和无序(unordered)因子。
- 函数levels() 可以用来得到因子的水平(levels)。

### 函数tapply() 和不规则数组

## 数组和矩阵

- 维度向量(dimension vector)是一个正整数向量。

### 向量和数组混合运算以及循环使用原则

- 表达式运算是从左到右进行的;
- 短的向量操作数将会被循环使用以达到其他操作数的长度;
- 有且只有短的向量和数组在一起,数组必须有一样的属性dim,否则返回一个错误;
- 向量操作数比矩阵或者数组操作数长时会引起错误;
- 如果数组结构给定,同时也没有关于向量的错误信息和强制转换操作,结果将是一个和它的数组操作数属性dim 一致的数组。
- 数组一个非常重要的运算就是外积运算(outer product)。外积是通过特别的操作符%o%实现.备选方案:`ab <- outer(a, b, "*")`,命令中的乘法操作符可以被任意一个双变量函数代替。

### 数组的广义转置,函数`aperm()`

### 矩阵工具

- 矩阵相乘,操作符%*% 用于矩阵相乘。
- 函数crossprod() 可以完成“矢积”(crossproduct)运算,也就是说crossprod(X,y) 和t(X) %*% y 等价,但是在运算上更为高效。如果crossprod() 第二个参数忽略 了,它将默认和第一个参数一样,即第一个参数和自己进行运算。

### 线性方程和求逆`solve()`

### 特征值和特征向量

函数eigen(Sm) 用来计算矩阵Sm 的特征值和特征向量。这个函数的返回值是一个含有values 和vectors 两个分量的列表。

### 奇异值分解和行列式

- 函数svd(M) 可以把任意一个矩阵M作为一个参数, 且对M 进行奇异值分解。
- R 有一个计算行列式(包括符号)的内置函数det 和另外一个给出符号和模(对数坐标可选)的函数。

### 最小二乘法拟合和QR分解

- 函数lsfit() 返回最小二乘法拟合(Least squares fitting)的结果列表。

### 用cbind() 和rbind() 构建分块矩阵

cbind() 把矩阵横向合并成一个大矩阵(列方式),而rbind() 是纵向合并(行方式).

### 对数组实现连接操作的函数c()

- 将一个数组强制转换成简单向量的标准方法是用函数as.vector()。

### 因子的频率表

函数table() 可以从等长的不同因子中计算出频率表。

## 列表和数据框

### 列表

- R 的列表(list)是一个以对象的有序集合构成的对象。列表中包含的对象又称为它的分量(components)。
- 通过函数list() 将已有的对象构建成列表。
- 当连接函数c() 的参数中有列表对象时,结果就是一个列表模式的对象。

### 数据框

- 数据框(data frame)是一个属于"data.frame" 类的列表。
- 符合数据框限制的列表可被函数`as.data.frame()`强制转换成数据框。
- 从外部文件读取一个数据框最简单的方法是使用函数`read.table()`。
- `attach()`和`detach()`

## 从文件中读取数据

### `read.table()`函数

可以直接读取整个数据框,外部文件常常要求有特定的格式。

## 概率分布

概率分布 | R 对应的名字 | 附加参数
-------  | ------------ | -------
β分布 | beta | shape1, shape2, ncp
二项式分布 | binom | size, prob
Cauchy 分布 | cauchy | location, scale
卡方分布 | chisq | df, ncp
指数分布 | exp | rate
F分布 | f | df1, df1, ncp
γ分布 | gamma | shape, scale
几何分布 | geom | prob
超几何分布 | hyper | m,n,k
对数正态分布 | lnorm | meanlog, sdlog
logistic 分布 | logis | location, scale
负二项式分布 | nbinom | size, prob
正态分布 | norm | mean, sd
Poisson 分布 | pois | lambda
t 分布 | t | df, ncp
均匀分布 | unif | min, max
Weibull 分布 | weibull | shape, scale
Wilcoxon 分布 | wilcox | m, n

不同的名字前缀表示不同的含义,d表示概率密度函数,p 表示累积分布函数(cumulative distribution function,CDF),q 表示分位函数以及r 表示随机模拟(random deviates)或者随机数发生器。dxxx 的第一个参数是x,pxxx是q,qxxx是p,和rxxx的是n (rhyper 和rwilcox例外,二者的参数是nn)。偏态指数(noncentrality parameter)ncp 现在仅用于累积分布函数.

pxxx 和qxxx 函数都有逻辑参数lower.tail 和log.p, dxxx 也有一个逻辑参数log。

## 成组,循环和条件控制

### 成组表达式

### 控制语句

```
if (expr1 ) expr2 else expr3
```

“短路”(short-circuit)操作符&& 和|| 常常用于if 语句的条件控制部分。这里要注意& 和| 将作用于向量的所有元素1,而&& 和|| 仅用于长度为1的向量,并且必要时 才对第二个参数求值.

R 提供了if/else 条件语句向量形式的函数ifelse。它的使用方式是ifelse(condition, a, b),最终返回一个和最长的参数向量同长的向量。

### 循环控制:for循环,repeat 和while

```
for (name in expr 1) expr 2repeat expr

while (condition) expr```

关键字break可以用于结束任何循环,甚至是非常规的。它是结束repeat 循环的 唯一办法。

关键字next 可以用来结束一次特定的循环,然后直接跳入“下一次”循环。

## 编写函数

```
name <- function(arg ￼ ￼ 1 , arg ￼ 2 , ...) expression
```

### 参数命名和默认值

- 如果被调用函数的参数以`name = object`方式给出,它们可以用任何顺序设置。
- 参数赋值序列可能以未命名的,位置特异性的方式给出,同时也有可能在这些位置特异性的参数后加上命名参数赋值。
- 默认参数与其他语言类似

### ... 参数

### 在函数中赋值

如果想在一个函数里面全局赋值或者永久赋值,可以采用“强赋值”(superassignment)操作符`<<-`或者采用函数`assign()`。

### 递归式的数值积分

### 类,泛型函数和面向对象

## R中的统计模型

## 图形工具

## 包

