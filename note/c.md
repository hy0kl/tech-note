# 多级指针的个人理解

```
以 char 当作书本为例子,能更形象的说明

char    p       代表书的具体一个字
char   *p       代表书的具体一行内容
char  **p       代表书的一页的内容
char ***p       代表书有多页内容,可以翻页

更多层级可以理解为 书本(换一本书)->书柜(换一个书柜) 等等.
也可以通过 点,线,面,三维立体 的具象来理解

type_t *p   指向 type_t 类型的指针
```

数组的`array[i]`和`*(array + i)`等价.


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

# 开发环境编译参数

```
export CFLAGS="-Wall -g"
```

# 内存泄漏工具

```
valgrind --track-origins=yes ./pro
```

# 拾遗

- 除了`sizeof`,`&`操作和声明之外,数组名称都会被编译器推导为指向其首个元素的指针.对于这些情况,不要用"是"这个词,而是要用"推导".
- `struct_obj->member`是`(*struct_obj).member`的简写.
- 不想显式释放内存又想避免内存泄漏的办法是引入`libGC`库.需要把所有的`malloc`换成`GC_malloc`,然后把所有的`free`删掉.
- '%p'占位符,打印出内存地址.
- 理清内存的最简单的方式是遵守这条原则:如果你的变量并不是从`malloc`中获取的,也不是从一个从`malloc`获取的函数中获取的,那么它在栈上.

- 关于栈和堆的主要问题:
  - 如果你从`malloc`获取了一块内存,并且把指针放在了栈上,那么当函数退出时,指针会被弹出而丢失.
  - 如果你在栈上存放了大量数据(比如大结构体和数组),那么会产生"栈溢出"并且程序会中止.
  - 如果你获取了指向栈上变量的指针,并且将它用于传参或从函数返回,接收它的函数会产生"段错误".因为实际的数据被弹出而消失,指针也会指向被释放的内存.
- 函数指针的主要用途是向其他函数传递"回调",或者模拟类和对象.

