# php 扩展开发相关

- MINIT: Module Initialization, 内核中预置了 PHP_MINIT_FUNCTION() 宏函数
- RINIT: Request Initialization, PHP_RINIT_FUNCTION()
- RSHUTDOWN: Request Shutdown, PHP_RSHUTDOWN_FUNCTION()
- MSHUTDOWN: Module Shutdown, PHP_MSHUTDOWN_FUNCTION()
- 一个PHP实例，无论是从 init 脚本中调用的，还是从命令行启动的，都会依次进行 Module init、Request init、Request Shutdown、Module shutdown 四个过程

# php 的生命周期

1.CLI/CGI

![PHP的生命周期](http://www.walu.cc/phpbook/image/01fig01.jpg)

2.多进程模式

![多进程模式](http://www.walu.cc/phpbook/image/01fig02.jpg)
![从apache的视角来看多进程工作模式下的PHP](http://www.walu.cc/phpbook/image/01fig03.jpg)

3.多线程模式

![多线程模式](http://www.walu.cc/phpbook/image/01fig04.jpg)

# TSRM(Thread Safe Resource Management)

- 在扩展的Module Init里，扩展可以调用 ts_allocate_id() 来告诉 TRSM 自己需要多少资源
- ./configure 的 enable-maintainer-zts 指令

# 8种数据类型

```
IS_NULL
IS_BOOL
IS_LONG
IS_DOUBLE
IS_STRING
IS_ARRAY
IS_OBJECT
IS_RESOURCE
```
# 检测 php 变量类型,宏定义

```
Z_TYPE      参数是zval型
Z_TYPE_P    参数是 *zval 型变量
Z_TYPE_PP   参数是**zval

以上三个宏的定义在 Zend/zend_operators.h 里,定义分别是:
#define Z_TYPE(zval)        (zval).type
#define Z_TYPE_P(zval_p)    Z_TYPE(*zval_p)
#define Z_TYPE_PP(zval_pp)  Z_TYPE(**zval_pp)
```

# 变量的值

```
Z_DVAL
ZDVAL_P
ZDVAL_PP

string 型变量比较特殊
Z_STRVAL
Z_STRVAL_P
Z_STRVAL_PP
Z_STRLEN
Z_STRLEN_P
Z_STRLEN_PP

例子:
void display_string(zval *zstr)
{
    if (Z_TYPE_P(zstr) != IS_STRING) {
        php_printf("这个变量不是字符串!\n");
        return;
    }
    PHPWRITE(Z_STRVAL_P(zstr), Z_STRLEN_P(zstr));
    //这里用了PHPWRITE 宏,只要知道它是从 Z_STRVAL_P(zstr) 地址开始,输出 Z_STRLEN_P(zstr) 长度的字符就可以了
}

数组
Z_ARRVAL
Z_ARRVAL_P
Z_ARRVAL_PP

有关值操作的宏都定义在 Zend/zend_operators.h 文件里
64
//操作整数的
#define Z_LVAL(zval)            (zval).value.lval
#define Z_LVAL_P(zval_p)        Z_LVAL(*zval_p)
#define Z_LVAL_PP(zval_pp)      Z_LVAL(**zval_pp)

//操作IS_BOOL布尔型的
#define Z_BVAL(zval)            ((zend_bool)(zval).value.lval)
#define Z_BVAL_P(zval_p)        Z_BVAL(*zval_p)
#define Z_BVAL_PP(zval_pp)      Z_BVAL(**zval_pp)

//操作浮点数的
#define Z_DVAL(zval)            (zval).value.dval
#define Z_DVAL_P(zval_p)        Z_DVAL(*zval_p)
#define Z_DVAL_PP(zval_pp)      Z_DVAL(**zval_pp)

//操作字符串的值和长度的
#define Z_STRVAL(zval)          (zval).value.str.val
#define Z_STRVAL_P(zval_p)      Z_STRVAL(*zval_p)
#define Z_STRVAL_PP(zval_pp)    Z_STRVAL(**zval_pp)

#define Z_STRLEN(zval)          (zval).value.str.len
#define Z_STRLEN_P(zval_p)      Z_STRLEN(*zval_p)
#define Z_STRLEN_PP(zval_pp)    Z_STRLEN(**zval_pp)

#define Z_ARRVAL(zval)          (zval).value.ht
#define Z_ARRVAL_P(zval_p)      Z_ARRVAL(*zval_p)
#define Z_ARRVAL_PP(zval_pp)    Z_ARRVAL(**zval_pp)

//操作对象的
#define Z_OBJVAL(zval)          (zval).value.obj
#define Z_OBJVAL_P(zval_p)      Z_OBJVAL(*zval_p)
#define Z_OBJVAL_PP(zval_pp)    Z_OBJVAL(**zval_pp)

#define Z_OBJ_HANDLE(zval)          Z_OBJVAL(zval).handle
#define Z_OBJ_HANDLE_P(zval_p)      Z_OBJ_HANDLE(*zval_p)
#define Z_OBJ_HANDLE_PP(zval_p)     Z_OBJ_HANDLE(**zval_p)

#define Z_OBJ_HT(zval)          Z_OBJVAL(zval).handlers
#define Z_OBJ_HT_P(zval_p)      Z_OBJ_HT(*zval_p)
#define Z_OBJ_HT_PP(zval_p)     Z_OBJ_HT(**zval_p)

#define Z_OBJCE(zval)           zend_get_class_entry(&(zval) TSRMLS_CC)
#define Z_OBJCE_P(zval_p)       Z_OBJCE(*zval_p)
#define Z_OBJCE_PP(zval_pp)     Z_OBJCE(**zval_pp)

#define Z_OBJPROP(zval)         Z_OBJ_HT((zval))->get_properties(&(zval) TSRMLS_CC)
#define Z_OBJPROP_P(zval_p)     Z_OBJPROP(*zval_p)
#define Z_OBJPROP_PP(zval_pp)   Z_OBJPROP(**zval_pp)

#define Z_OBJ_HANDLER(zval, hf)         Z_OBJ_HT((zval))->hf
#define Z_OBJ_HANDLER_P(zval_p, h)      Z_OBJ_HANDLER(*zval_p, h)
#define Z_OBJ_HANDLER_PP(zval_p, h)     Z_OBJ_HANDLER(**zval_p, h)

#define Z_OBJDEBUG(zval,is_tmp)     (Z_OBJ_HANDLER((zval),get_debug_info)?  \
                        Z_OBJ_HANDLER((zval),get_debug_info)(&(zval),&is_tmp TSRMLS_CC): \
                        (is_tmp=0,Z_OBJ_HANDLER((zval),get_properties)?Z_OBJPROP(zval):NULL))
#define Z_OBJDEBUG_P(zval_p,is_tmp)     Z_OBJDEBUG(*zval_p,is_tmp)
#define Z_OBJDEBUG_PP(zval_pp,is_tmp)   Z_OBJDEBUG(**zval_pp,is_tmp)

//操作资源的
#define Z_RESVAL(zval)          (zval).value.lval
#define Z_RESVAL_P(zval_p)      Z_RESVAL(*zval_p)
#define Z_RESVAL_PP(zval_pp)    Z_RESVAL(**zval_pp)
```

# 创建 php 变量

```
宏申请内存
MAKE_STD_ZVAL()

赋值
ZVAL_NULL(pzv);
ZVAL_BOOL(pzv, b);
ZVAL_TRUE(pzv);
ZVAL_FALSE(pzv);
ZVAL_LONG(pzv, l);
ZVAL_DOUBLE(pzv, d);
ZVAL_STRINGL(pzv, str, len, dup);
ZVAL_STRING(pzv, str, dup);
ZVAL_RESOURCE(pzv, res);
```

# 变量的检索

```
{
    zval **fooval;

    if (zend_hash_find(
            EG(active_symbol_table), // 这个参数是地址,如果我们操作全局作用域,则需要 &EG(symbol_table)
            "foo",
            sizeof("foo"),
            (void**)&fooval
        ) == SUCCESS
    )
    {
        php_printf("成功发现$foo!");
    }
    else
    {
        php_printf("当前作用域下无法发现$foo.");
    }
}
```

# 类型转换

```
//将任意类型的zval转换成字符串
void change_zval_to_string(zval *value)
{
    convert_to_string(value);
}

//其它基本的类型转换函数
ZEND_API void convert_to_long(zval *op);
ZEND_API void convert_to_double(zval *op);
ZEND_API void convert_to_null(zval *op);
ZEND_API void convert_to_boolean(zval *op);
ZEND_API void convert_to_array(zval *op);
ZEND_API void convert_to_object(zval *op);

ZEND_API void _convert_to_string(zval *op ZEND_FILE_LINE_DC);
#define convert_to_string(op) if ((op)->type != IS_STRING) { _convert_to_string((op) ZEND_FILE_LINE_CC); }
```

# 内存管理

C语言原生函数 | PHP内核封装后的函数
----------- | ----------------
void *malloc(size_t count); | void *emalloc(size_t count); <br /> void *pemalloc(size_t count, char persistent);
void *calloc(size_t count); | void *ecalloc(size_t count); <br /> void *pecalloc(size_t count, char persistent);
void *realloc(void *ptr, size_t count); | void *erealloc(void *ptr, size_t count); <br /> void *perealloc(void *ptr, size_t count, char persistent);
void *strdup(void *ptr); | void *estrdup(void *ptr); <br /> void *pestrdup(void *ptr, char persistent);
void free(void *ptr); | void efree(void *ptr); <br /> void pefree(void *ptr, char persistent);

```
void *estrndup(void *ptr，int len);
void *safe_emalloc(size_t size, size_t count, size_t addtl);
void *safe_pemalloc(size_t size, size_t count, size_t addtl, char persistent);
```

# 注意点

为什么有的扩展的开启方式是 --enable-extname 的形式，有的则是 --with-extname 的形式呢？其实两者并没有什么本质的不同，只不过 enable 多代表不依赖外部库便可以直接编译，而 with 大多需要依赖于第三方的lib。

# INTERNAL_FUNCTION_PARAMETERS 宏

```
#define INTERNAL_FUNCTION_PARAMETERS int ht, zval *return_value, zval **return_value_ptr, zval *this_ptr, int return_value_used TSRMLS_DC
```

- int ht
- zval *return_value，我们在函数内部修改这个指针，函数执行完成后，内核将把这个指针指向的zval返回给用户端的函数调用者。
- zval **return_value_ptr，
- zval *this_ptr，如果此函数是一个类的方法，那么这个指针的含义和PHP语言中$this变量差不多。
- int return_value_used，代表用户端在调用此函数时有没有使用到它的返回值。

# 内核提供的 RETURN_* 系列宏

```
//这些宏都定义在Zend/zend_API.h文件里
#define RETVAL_RESOURCE(l)              ZVAL_RESOURCE(return_value, l)
#define RETVAL_BOOL(b)                  ZVAL_BOOL(return_value, b)
#define RETVAL_NULL()                   ZVAL_NULL(return_value)
#define RETVAL_LONG(l)                  ZVAL_LONG(return_value, l)
#define RETVAL_DOUBLE(d)                ZVAL_DOUBLE(return_value, d)
#define RETVAL_STRING(s, duplicate)         ZVAL_STRING(return_value, s, duplicate)
#define RETVAL_STRINGL(s, l, duplicate)     ZVAL_STRINGL(return_value, s, l, duplicate)
#define RETVAL_EMPTY_STRING()           ZVAL_EMPTY_STRING(return_value)
#define RETVAL_ZVAL(zv, copy, dtor)     ZVAL_ZVAL(return_value, zv, copy, dtor)
#define RETVAL_FALSE                    ZVAL_BOOL(return_value, 0)
#define RETVAL_TRUE                     ZVAL_BOOL(return_value, 1)

#define RETURN_RESOURCE(l)              { RETVAL_RESOURCE(l); return; }
#define RETURN_BOOL(b)                  { RETVAL_BOOL(b); return; }
#define RETURN_NULL()                   { RETVAL_NULL(); return;}
#define RETURN_LONG(l)                  { RETVAL_LONG(l); return; }
#define RETURN_DOUBLE(d)                { RETVAL_DOUBLE(d); return; }
#define RETURN_STRING(s, duplicate)     { RETVAL_STRING(s, duplicate); return; }
#define RETURN_STRINGL(s, l, duplicate) { RETVAL_STRINGL(s, l, duplicate); return; }
#define RETURN_EMPTY_STRING()           { RETVAL_EMPTY_STRING(); return; }
#define RETURN_ZVAL(zv, copy, dtor)     { RETVAL_ZVAL(zv, copy, dtor); return; }
#define RETURN_FALSE                    { RETVAL_FALSE; return; }
#define RETURN_TRUE                     { RETVAL_TRUE; return; }
```

# 编译时的传递引用 Compile-time Pass-by-ref

```
在 Zend Engine 2 (PHP5+)中，arginfo 的数据是由多个 zend_arg_info 结构体构成的数组，数组的每一个成员即每一个 zend_arg_info 结构体处理函数的一个参数。zend_arg_info 结构体的定义如下：

typedef struct _zend_arg_info {
    const char *name;               /* 参数的名称*/
    zend_uint name_len;             /* 参数名称的长度*/
    const char *class_name;         /* 类名 */
    zend_uint class_name_len;       /* 类名长度*/
    zend_bool array_type_hint;      /* 数组类型提示 */
    zend_bool allow_null;           /* 是否允许为NULL　*/
    zend_bool pass_by_reference;    /* 是否引用传递 */
    zend_bool return_reference;     /* 返回值是否为引用形式 */
    int required_num_args;          /* 必要参数的数量 */
} zend_arg_info;

生成 zend_arg_info 结构的数组比较繁琐，为了方便 PHP 扩展开发者，内核已经准备好了相应的宏来专门处理此问题，首先先用一个宏函数来生成头部，然后用第二个宏生成具体的数据，最后用一个宏生成尾部代码。

#define ZEND_BEGIN_ARG_INFO(name, pass_rest_by_reference)   ZEND_BEGIN_ARG_INFO_EX(name, pass_rest_by_reference, ZEND_RETURN_VALUE, -1)
#define ZEND_BEGIN_ARG_INFO_EX(name, pass_rest_by_reference, return_reference, required_num_args)   \
    static const zend_arg_info name[] = {                                                                       \
        { NULL, 0, NULL, 0, 0, 0, pass_rest_by_reference, return_reference, required_num_args },


#define ZEND_ARG_INFO(pass_by_ref, name)        { #name, sizeof(#name)-1, NULL, 0, 0, 0, pass_by_ref, 0, 0 },
#define ZEND_ARG_PASS_INFO(pass_by_ref)         { NULL, 0, NULL, 0, 0, 0, pass_by_ref, 0, 0 },
#define ZEND_ARG_OBJ_INFO(pass_by_ref, name, classname, allow_null) { #name, sizeof(#name)-1, #classname, sizeof(#classname)-1, 0, allow_null, pass_by_ref, 0, 0 },
#define ZEND_ARG_ARRAY_INFO(pass_by_ref, name, allow_null) { #name, sizeof(#name)-1, NULL, 0, 1, allow_null, pass_by_ref, 0, 0 },


#define ZEND_END_ARG_INFO()     };
```

# 函数的参数

```
最简单的获取函数调用者传递过来的参数便是使用zend_parse_parameters()函数,对应的宏 ZEND_NUM_ARGS() TSRMLS_CC
*注意两者之间有个空格,但是没有逗号.

需要传递给zend_parse_parameters()函数的参数是一个用于格式化的字符串

type_spec是格式化字符串，其常见的含义如下：
参数    代表着的类型
b       Boolean
l       Integer 整型
d       Floating point 浮点型
s       String 字符串
r       Resource 资源
a       Array 数组
o       Object instance 对象
O       Object instance of a specified type 特定类型的对象
z       Non-specific zval 任意类型～
Z       zval**类型
f       表示函数、方法名称，PHP5.1里貌似木有... ...

参数    对应C里的数据类型
b       zend_bool
l       long
d       double
s       char*, int 前者接收指针，后者接收长度
r       zval*
a       zval*
o       zval*
O       zval*, zend_class_entry*
z       zval*
Z       zval**

其它的三个参数来增强接收参数的能力
Type Modifier   Meaning
|               它之前的参数都是必须的，之后的都是非必须的，也就是有默认值的。
!               如果接收了一个 PHP 语言里的 null 变量，则直接把其转成 C 语言里的 NULL，而不是封装成 IS_NULL 类型的 zval。
/               如果传递过来的变量与别的变量共用一个 zval，而且不是引用，则进行强制分离，新的 zval 的 is_ref__gc == 0, and refcount__gc == 1.

举个栗子:
ZEND_FUNCTION(sample_hello_world) {
    char *name;
    int name_len;
    char *greeting = "Mr./Mrs.";
    int greeting_len = sizeof("Mr./Mrs.") - 1;


    if (zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "s|s",
      &name, &name_len, &greeting, &greeting_len) == FAILURE) {
        RETURN_NULL();
    }

    php_printf("Hello ");
    PHPWRITE(greeting, greeting_len);
    php_printf(" ");
    PHPWRITE(name, name_len);
    php_printf("!\n");
}
```

# 使用HashTable

## 创建并初始化一个HashTable

```c
int zend_hash_init(
    HashTable *ht,
    uint nSize,
    hash_func_t pHashFunction,
    dtor_func_t pDestructor,
    zend_bool persistent
);
```

- *ht 是指针，指向一个HashTable，我们既可以&一个已存在的HashTable变量， 也可以通过emalloc()、pemalloc()等函数来直接申请一块内存， 不过最常用的方法还是用ALLOC_HASHTABLE(ht)宏来让内核自动的替我们完成这项工作。 ALLOC_HASHTABLE(ht)所做的工作相当于ht = emalloc(sizeof(HashTable));
- nSize 代表着这个HashTable可以拥有的元素的最大数量(HashTable能够包含任意数量的元素， 这个值只是为了提前申请好内存，提高性能，省的不停的进行rehash操作)。 在我们添加新的元素时，这个值会根据情况决定是否自动增长，有趣的是， 这个值永远都是2的次方，如果你给它的值不是一个2的次方的形式， 那它将自动调整成大于它的最小的2的次方值。 它的计算方法就像这样：nSize = pow(2, ceil(log(nSize, 2)));
- pHashFunction 是早期的Zend Engine中的一个参数，为了兼容没有去掉它， 但它已经没有用处了，所以我们直接赋成NULL就可以了。在原来， 它其实是一个钩子，用来让用户自己hook一个散列函数，替换php默认的DJBX33A算法实现。
- pDestructor 也代表着一个回调函数，当我们删除或者修改HashTable中其中一个元素时候便会调用， 它的函数原型必须是这样的：void method_name(void pElement);这里的pElement是一个指针，指向HashTable中那么将要被删除或者修改的那个数据，而数据的类型往往也是个指针。
- persistent 是最后一个参数，它的含义非常简单。 如果它为true，那么这个HashTable将永远存在于内存中，而不会在RSHUTDOWN阶段自动被注销掉。 此时第一个参数ht所指向的地址必须是通过pemalloc()函数申请的。

## 四个常用的函数来完成 添加/修改 操作

```
int zend_hash_add(
    HashTable *ht,      //待操作的ht
    char *arKey,        //索引，如"my_key"
    uint nKeyLen,       //字符串索引的长度，如6
    void **pData,       //要插入的数据，注意它是void **类型的。int *p,i=1;p=&i,pData=&p;。
    uint nDataSize,
    void *pDest         //如果操作成功，则pDest=*pData;
);

int zend_hash_update(
    HashTable *ht,
    char *arKey,
    uint nKeyLen,
    void *pData,
    uint nDataSize,
    void **pDest
);

int zend_hash_index_update(
    HashTable *ht,
    ulong h,
    void *pData,
    uint nDataSize,
    void **pDest
);

int zend_hash_next_index_insert(
    HashTable *ht,
    void *pData,
    uint nDataSize,
    void **pDest
);
```

## 查找

```c
int zend_hash_find(HashTable *ht, char *arKey, uint nKeyLength, void **pData);
int zend_hash_index_find(HashTable *ht, ulong h, void **pData);

if( zend_hash_exists(EG(active_symbol_table),"foo", sizeof("foo")) == SUCCESS )
{
    /* $foo is set */
}
else
{
    //FAILURE
    /* $foo does not exist */
}

ulong zend_get_hash_value(char *arKey, uint nKeyLen);

int zend_hash_quick_add(
    HashTable *ht,
    char *arKey,
    uint nKeyLen,
    ulong hashval,
    void *pData,
    uint nDataSize,
    void **pDest
);

int zend_hash_quick_update(
    HashTable *ht,
    char *arKey,
    uint nKeyLen,
    ulong hashval,
    void *pData,
    uint nDataSize,
    void **pDest
);

int zend_hash_quick_find(
    HashTable *ht,
    char *arKey,
    uint nKeyLen,
    ulong hashval,
    void **pData
);

int zend_hash_quick_exists(
    HashTable *ht,
    char *arKey,
    uint nKeyLen,
    ulong hashval
);
```

## 复制与合并(Copy And Merge)

```c
void zend_hash_copy(
    HashTable *target,
    HashTable *source,
    copy_ctor_func_t pCopyConstructor,
    void *tmp,
    uint size
);
```

- *source 中的所有元素都会通过pCopyConstructor函数Copy到*target中去，我们还是以PHP语言中的数组举例，pCopyConstructor这个hook使得我们可以在copy变量的时候对他们的ref_count进行加一操作。target中原有的与source中索引位置的数据会被替换掉，而其它的元素则会被保留，原封不动。
- tmp 参数是为了兼容PHP4.0.3以前版本的，现在赋值为NULL即可。
- size参数代表每个元素的大小，对于PHP语言中的数组来说，这里的便是sizeof(zval*)了。

```c
void zend_hash_merge(
    HashTable *target,
    HashTable *source,
    copy_ctor_func_t pCopyConstructor,
    void *tmp,
    uint size,
    int overwrite
    // overwrite参数，当其值非0的时候，两个函数的工作是完全一样的；如果overwrite参数为0，则zend_hash_merge函数就不会对target中已有索引的值进行替换了。
);
```

```c
typedef zend_bool (*merge_checker_func_t)(HashTable *target_ht, void *source_data, zend_hash_key *hash_key, void *pParam);
void zend_hash_merge_ex(
    HashTable *target,
    HashTable *source,
    copy_ctor_func_t pCopyConstructor,
    uint size,
    merge_checker_func_t pMergeSource,
    void *pParam
);
```

### 例子:

```c
zend_bool associative_only(HashTable *ht, void *pData,zend_hash_key *hash_key, void *pParam)
{
    //如果是字符串索引
    return (hash_key->arKey && hash_key->nKeyLength);
}

void merge_associative(HashTable *target, HashTable *source)
{
    zend_hash_merge_ex(target, source, zval_add_ref,sizeof(zval*), associative_only, NULL);
}
```

## 遍历

```c
typedef int (*apply_func_t)(void *pDest TSRMLS_DC);
void zend_hash_apply(HashTable *ht,apply_func_t apply_func TSRMLS_DC);

typedef int (*apply_func_arg_t)(void *pDest,void *argument TSRMLS_DC);
void zend_hash_apply_with_argument(HashTable *ht,apply_func_arg_t apply_func, void *data TSRMLS_DC);
```

回调函数的返回值

Constant | Meaning
-------- | -------
ZEND_HASH_APPLY_KEEP    |    结束当前请求，进入下一个循环。与PHP语言forech语句中的一次循环执行完毕或者遇到continue关键字的作用一样。
ZEND_HASH_APPLY_STOP    |    跳出，与PHP语言forech语句中的break关键字的作用一样。
ZEND_HASH_APPLY_REMOVE  |    删除当前的元素，然后继续处理下一个。相当于在PHP语言中：unset($foo[$key]);continue;

### 例子

```c
int php_sample_print_zval(zval **val TSRMLS_DC)
{
    //重新copy一个zval，防止破坏原数据
    zval tmpcopy = **val;
    zval_copy_ctor(&tmpcopy);

    //转换为字符串
    INIT_PZVAL(&tmpcopy);
    convert_to_string(&tmpcopy);

    //开始输出
    php_printf("The value is: ");
    PHPWRITE(Z_STRVAL(tmpcopy), Z_STRLEN(tmpcopy));
    php_printf("\n");

    //毁尸灭迹
    zval_dtor(&tmpcopy);

    //返回，继续遍历下一个～
    return ZEND_HASH_APPLY_KEEP;
}

//生成一个名为arrht、元素为zval*类型的HashTable
zend_hash_apply(arrht, php_sample_print_zval TSRMLS_CC);

typedef int (*apply_func_args_t)(void *pDest,int num_args, va_list args, zend_hash_key *hash_key);
void zend_hash_apply_with_arguments(HashTable *ht,apply_func_args_t apply_func, int numargs, ...);

int php_sample_print_zval_and_key(zval **val,int num_args,va_list args,zend_hash_key *hash_key)
{
    //重新copy一个zval，防止破坏原数据
    zval tmpcopy = **val;
    /* tsrm_ls is needed by output functions */
    TSRMLS_FETCH();
    zval_copy_ctor(&tmpcopy);
    INIT_PZVAL(&tmpcopy);

    //转换为字符串
    convert_to_string(&tmpcopy);

    //执行输出
    php_printf("The value of ");
    if (hash_key->nKeyLength)
    {
        //如果是字符串类型的key
        PHPWRITE(hash_key->arKey, hash_key->nKeyLength);
    }
    else
    {
        //如果是数字类型的key
        php_printf("%ld", hash_key->h);
    }

    php_printf(" is: ");
    PHPWRITE(Z_STRVAL(tmpcopy), Z_STRLEN(tmpcopy));
    php_printf("\n");

    //毁尸灭迹
    zval_dtor(&tmpcopy);
    /* continue; */
    return ZEND_HASH_APPLY_KEEP;
}

zend_hash_apply_with_arguments(arrht, php_sample_print_zval_and_key, 0);
```

## 向前遍历HashTable

```c
/* reset() */
void zend_hash_internal_pointer_reset(HashTable *ht);

/* key() */
int zend_hash_get_current_key(HashTable *ht,char **strIdx, unit *strIdxLen,ulong *numIdx, zend_bool duplicate);

/* current() */
int zend_hash_get_current_data(HashTable *ht, void **pData);

/* next()/each() */
int zend_hash_move_forward(HashTable *ht);

/* prev() */
int zend_hash_move_backwards(HashTable *ht);

/* end() */
void zend_hash_internal_pointer_end(HashTable *ht);

/* 其他的...... */
int zend_hash_get_current_key_type(HashTable *ht);
int zend_hash_has_more_elements(HashTable *ht);
```

## 另一个例子

```c
void php_sample_print_var_hash(HashTable *arrht)
{

    for(
        zend_hash_internal_pointer_reset(arrht);
        zend_hash_has_more_elements(arrht) == SUCCESS;
        zend_hash_move_forward(arrht))
    {
        char *key;
        uint keylen;
        ulong idx;
        int type;
        zval **ppzval, tmpcopy;

        type = zend_hash_get_current_key_ex(arrht, &key, &keylen,&idx, 0, NULL);
        if (zend_hash_get_current_data(arrht, (void**)&ppzval) == FAILURE)
        {
            /* Should never actually fail
             * since the key is known to exist. */
            continue;
        }

        //重新copy一个zval，防止破坏原数据
        tmpcopy = **ppzval;
        zval_copy_ctor(&tmpcopy);
        INIT_PZVAL(&tmpcopy);

        convert_to_string(&tmpcopy);

        /* Output */
        php_printf("The value of ");
        if (type == HASH_KEY_IS_STRING)
        {
            /* String Key / Associative */
            PHPWRITE(key, keylen);
        } else {
            /* Numeric Key */
            php_printf("%ld", idx);
        }
        php_printf(" is: ");
        PHPWRITE(Z_STRVAL(tmpcopy), Z_STRLEN(tmpcopy));
        php_printf("\n");
        /* Toss out old copy */
        zval_dtor(&tmpcopy);
    }
}
```

## zend_hash_get_current_key()函数的返回值

Constant | Meaning
-------- | -------
HASH_KEY_IS_STRING      |        当前元素的索引是字符串类型的。therefore, a pointer to the element's key name will be populated into strIdx, and its length will be populated into stdIdxLen. If the duplicate flag is set to a nonzero value, the key will be estrndup()'d before being populated into strIdx. The calling application is expected to free this duplicated string.
HASH_KEY_IS_LONG        |        当前元素的索引是数字型的。
HASH_KEY_NON_EXISTANT   |        HashTable中的内部指针已经移动到尾部，不指向任何元素。

## 删除HashTable元素的函数

```c
int zend_hash_del(HashTable *ht, char *arKey, uint nKeyLen);
int zend_hash_index_del(HashTable *ht, ulong h);
void zend_hash_clean(HashTable *ht);
void zend_hash_destroy(HashTable *ht);
```

### 例子

```c
int sample_strvec_handler(int argc, char **argv TSRMLS_DC)
{
    HashTable *ht;

    //分配内存
    ALLOC_HASHTABLE(ht);

    //初始化
    if (zend_hash_init(ht, argc, NULL,ZVAL_PTR_DTOR, 0) == FAILURE) {
        FREE_HASHTABLE(ht);
        return FAILURE;
    }

    //填充数据
    while (argc) {
        zval *value;
        MAKE_STD_ZVAL(value);
        ZVAL_STRING(value, argv[argc], 1);
        argv++;
        if (zend_hash_next_index_insert(ht, (void**)&value,
                            sizeof(zval*)) == FAILURE) {
            /* Silently skip failed additions */
            zval_ptr_dtor(&value);
        }
    }

    //完成工作
    process_hashtable(ht);

    //毁尸灭迹
    zend_hash_destroy(ht);

    //释放ht 为什么不在destroy里free呢，求解释！
    FREE_HASHTABLE(ht);
    return SUCCESS;
}
```

## 排序、比较and Going to the Extreme(s)

```c
typedef int (*compare_func_t)(void *a, void *b TSRMLS_DC);
int zend_hash_minmax(HashTable *ht, compare_func_t compar,int flag, void **pData TSRMLS_DC);
int zend_hash_compare(HashTable *hta, HashTable *htb, compare_func_t compar, zend_bool ordered TSRMLS_DC);

typedef void (*sort_func_t)(void **Buckets, size_t numBuckets,size_t sizBucket, compare_func_t comp TSRMLS_DC);
int zend_hash_sort(HashTable *ht, sort_func_t sort_func, compare_func_t compare_func, int renumber TSRMLS_DC);
zend_hash_sort(target_hash, zend_qsort, array_data_compare, 1 TSRMLS_CC);
```

### 例子

```c
//先定义一个比较函数，作为zend_hash_minmax的回调函数。
int fname_compare(zend_function *a, zend_function *b TSRMLS_DC)
{
    return strcasecmp(a->common.function_name, b->common.function_name);
}

void php_sample_funcname_sort(TSRMLS_D)
{
    zend_function *fe;
    if (zend_hash_minmax(EG(function_table), fname_compare, 0, (void **)&fe) == SUCCESS)
    {
        php_printf("Min function: %s\n", fe->common.function_name);
    }
    if (zend_hash_minmax(EG(function_table), fname_compare, 1, (void **)&fe) == SUCCESS)
    {
        php_printf("Max function: %s\n", fe->common.function_name);
    }
}
```

## 操作{数组}

```c
array_init(arrval);

add_assoc_long(zval *arrval, char *key, long lval);
add_index_long(zval *arrval, ulong idx, long lval);
add_next_index_long(zval *arrval, long lval);

//add_assoc_*系列函数：
add_assoc_null(zval *aval, char *key);
add_assoc_bool(zval *aval, char *key, zend_bool bval);
add_assoc_long(zval *aval, char *key, long lval);
add_assoc_double(zval *aval, char *key, double dval);
add_assoc_string(zval *aval, char *key, char *strval, int dup);
add_assoc_stringl(zval *aval, char *key,char *strval, uint strlen, int dup);
add_assoc_zval(zval *aval, char *key, zval *value);

//备注：其实这些函数都是宏，都是对add_assoc_*_ex函数的封装。

//add_index_*系列函数：
ZEND_API int add_index_long     (zval *arg, ulong idx, long n);
ZEND_API int add_index_null     (zval *arg, ulong idx           );
ZEND_API int add_index_bool     (zval *arg, ulong idx, int b    );
ZEND_API int add_index_resource (zval *arg, ulong idx, int r    );
ZEND_API int add_index_double   (zval *arg, ulong idx, double d);
ZEND_API int add_index_string   (zval *arg, ulong idx, const char *str, int duplicate);
ZEND_API int add_index_stringl  (zval *arg, ulong idx, const char *str, uint length, int duplicate);
ZEND_API int add_index_zval     (zval *arg, ulong index, zval *value);

//add_next_index_long函数：
ZEND_API int add_next_index_long        (zval *arg, long n  );
ZEND_API int add_next_index_null        (zval *arg          );
ZEND_API int add_next_index_bool        (zval *arg, int b   );
ZEND_API int add_next_index_resource    (zval *arg, int r   );
ZEND_API int add_next_index_double      (zval *arg, double d);
ZEND_API int add_next_index_string      (zval *arg, const char *str, int duplicate);
ZEND_API int add_next_index_stringl     (zval *arg, const char *str, uint length, int duplicate);
ZEND_API int add_next_index_zval        (zval *arg, zval *value);
```

### 例子

```c
ZEND_FUNCTION(sample_array)
{
    zval *subarray;

    array_init(return_value);

    /* Add some scalars */
    add_assoc_long(return_value, "life", 42);
    add_index_bool(return_value, 123, 1);
    add_next_index_double(return_value, 3.1415926535);

    /* Toss in a static string, dup'd by PHP */
    add_next_index_string(return_value, "Foo", 1);

    /* Now a manually dup'd string */
    add_next_index_string(return_value, estrdup("Bar"), 0);

    /* Create a subarray */
    MAKE_STD_ZVAL(subarray);
    array_init(subarray);

    /* Populate it with some numbers */
    add_next_index_long(subarray, 1);
    add_next_index_long(subarray, 20);
    add_next_index_long(subarray, 300);

    /* Place the subarray in the parent */
    add_index_zval(return_value, 444, subarray);
}
```

# PHP中的资源类型

```c
//每一个资源都是通过它来实现的。
typedef struct _zend_rsrc_list_entry
{
    void *ptr;
    int type;
    int refcount;
}zend_rsrc_list_entry;
```

# 启动与终止的那点事

## PHP扩展中的全局变量

Accessor Macro | Associated Data
-------------- | ---------------
EG() |  这个宏可以用来访问符号表，函数，资源信息和常量。
CG() |  用来访问核心全局变量。
PG() |  PHP全局变量。我们知道php.ini会映射一个或者多个PHP全局结构。举几个使用这个宏的例子：PG(register_globals), PG(safe_mode), PG(memory_limit)
FG() |  文件全局变量。大多数文件I/O或相关的全局变量的数据流都塞进标准扩展出口结构。

# INI设置

```c
static zend_ini_entry ini_entries[] = {
           {0,0,NULL,0,NULL,NULL,NULL,NULL,NULL,0,NULL,0,0,NULL} };
```

PHP总共有 4 个指令配置作用域:

Parameter |  Meaning
--------- | --------
PHP_INI_PERDIR | 指令可以在php.ini、httpd.conf或.htaccess文件中修改
PHP_INI_SYSTEM | 指令可以在php.ini 和 httpd.conf 文件中修改
PHP_INI_USER   | 指令可以在用户脚本中修改
PHP_INI_ALL | 指令可以在任何地方修改
