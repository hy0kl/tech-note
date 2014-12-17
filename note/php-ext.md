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

