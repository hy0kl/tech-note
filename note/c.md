# 多级指针的个人理解

```
以 char 当作书本为例子,能更形象的说明

char    p       代表书的具体一个字
char   *p       代表书的具体一行内容
char  **p       代表书的一页的内容
char ***p       代表书有多页内容,可以翻页

更多层级可以理解为 书本(换一本书)->书柜(换一个书柜) 等等.
也可以通过 点,线,面,三维立体 的具象来理解
```

# 函数参数

c 语言函数参数是形式,在调用函数时,参数实例,是在栈上新分配空间,将实参的值去初始化形参.
如果想在函数内部改变外边的变量的值,需要参数实例化时取一级指针.

[例子与详解](https://github.com/hy0kl/algorithm/blob/master/other/func_args.c)

# 移位操作溢出

[参见](http://stackoverflow.com/questions/4201301/warning-left-shift-count-width-of-type)

```
unsigned long long int t = 1 << 32;
编译时报如下错.原因是 1 是 int 型,左移 32 位溢出.
warning: shift count >= width of type [-Wshift-count-overflow]

解决方法:
unsigned long long int t = 1ULL << 32;
显式指定更宽的位数.
```

