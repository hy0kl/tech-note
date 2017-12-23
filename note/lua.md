# 拾遗

- 所有逻辑操作符将`false`和`nil`视作假,其他任何值视为真,对于`and`和`or`短路求值,对于`not`永远返回`true`或者`false`
- `string.format`格式化
- 多分支结构中`else`和`if`是连在一起的,若将`else`与`if`写成`else if`则相当于在`else`里嵌套另一个`if`语句
- 泛型`for`循环与数字`for`循环的共同点:
    - 循环变量是循环的局部变量
    - 决不应该对循环变量作任何赋值
- 具名参数: lua支持通过名称来指定实参,这时需要把所有的实参组织到一个`table`中,并将这个`table`作为唯一的实参传给函数
- 当函数的参数是`table`类型时,传递进来的是实际参数的引用
- 在常用基本类型中,除了`table`是按址传递类型外,其他的都是按值传递参数.
- 在lua中,数组下标从1开始计数
- `table.getn`获取长度.取长度的操作符写作一元操作`#`
- 不要在lua的`table`中使用`nil`值,如果一个元素要删除,直接`remove`,不要用`nil`去代替.
- **不推荐使用继承**
- **不使用`local`显式定义的变量就是全局变量**

## 元表

在lua5.1语言中,元表(metatable)的表现行为类似于C++语言中的操作符重载.

- `setmetatable(table, metatable)`: 此方法用于为一个表设置元表
- `getmetatable(table)`: 此方法用于获取表的元表对象
- 假如想保护对象使其使用者既看不到也不能修改`metatable`,可以对`metatable`设置`__metatable`的值,`getmetatable`将返回这个域的值,而调用`setmetatable`将会报错.

## 面向对象

```lua
-- account.lua

local _M = {}
local mt = {__index = _M}


function _M.deposit(self, v)
    self.balance = self.balance + v
end


function _M.withdraw(self, v)
    if self.balance >= v then
        self.balance = self.balance - v
    else
        error("insufficient funds")
    end
end


function _M.new(self, balance)
    local balance = balance or 0

    return setmetatable(balance = balance, mt)
end


return _M
```

## table判空

`next`指令不能被`LuaJIT`编译优化

```
function is_table_empty(t)
    return t == nil or next(t) == nil
end
```

# lua中冒号(:)与点号(.)的区别

[参考一](http://my.oschina.net/lonewolf/blog/173065)
[参考二](http://www.cnblogs.com/youxilua/archive/2011/07/28/2119059.html)

```
可以把点号(.)作为静态方法来看待，冒号(:)作为成员方法来看待。
用lua进行面向对象的编程, 声明方法和调用方法统一用冒号,对于属性的调用全部用点号
```

## OpenResty


# OpenResty中可能的阻塞情况

在`OpenResty`中,所有具备`set_keepalive`的类,库函数,说明都是支持连接池的.

- 高`CPU`调用(压缩,解压缩,加解密等)
- 高磁盘的调用(所有文件操作)
- 非OpenResty提供的网络操作(luasocket等)
- 系统命令行调用(os.execute等)

## 定是任务

- `ngx.timer.at(delay, handler)`
- `ngx.timer.every()`

## 组件

cosocket = coroutine + socket

- `lua-resty-core`
- `lua-resty-redis`
- `lua-resty-mysql`
- `lua-resty-upload`
- `lua-resty-http`
- `lua-resty-dns`
- `lua-resty-websocket`
- `lua-resty-limit-traffic`
- `lua-resty-test`测试

## 代码风格

[参考](https://github.com/openresty/lua-resty-string/blob/master/lib/resty/string.lua)

1. 没有全局变量,所有变量均使用`local`限制作用域
2. 提取公共函数到本地变量,使用本地变量缓存函数指针,加速下次使用
3. 函数名称全部小写,使用下划线进行分割
4. 两个函数之间距离两个空行

# lua -fPIC error

```
现象:
$ gcc luacurl.c -Wl,-E /home/work/local/lib/liblua.a /home/work/local/lib/libcurl.a  -fPIC -shared -o t.so
/usr/bin/ld: /home/work/local/lib/liblua.a(lapi.o): relocation R_X86_64_32 against `luaO_nilobject_' can not be used when making a shared object; recompile with -fPIC
/home/wok/local/lib/liblua.a: error adding symbols: Bad value
collect2: error: ld returned 1 exit status
```

[参考](http://www.cppblog.com/colorful/archive/2013/04/23/199659.html)

```
解决方法
如果服务器是64位的,这时要调整一下 Makefile: vi src/Makefile, 在 CFLAGS 里加上-fPIC.
```

# luacurl 手动编译安装

自带的`cmake`编译出来的是静态`*.a` 库.因为就一个文件,手工编译成动态共享库.

libcrypto.a 依赖 libssl.a,所有参数先后有顺序

```
$ wget http://files.luaforge.net/releases/luacurl/luacurl/luacurl-1.2.1/luacurl-1.2.1.zip
$ unzip luacurl-1.2.1.zip
$ cd luacurl-1.2.1
$ gcc luacurl.c -Wl,-E \
-L/usr/lib/x86_64-linux-gnu \
/home/work/local/lib/liblua.a \
/home/work/local/lib/libcurl.a \
/home/work/local/lib/libssl.a \
/home/work/local/lib/libcrypto.a  \
-lrtmp \
-lldap \
-lidn \
-fPIC -shared -o luacurl.so
```

# 让 ctags 更好支持 lua

[see](http://blog.csdn.net/zdl1016/article/details/9118579)

```
ctags --langdef=MYLUA --langmap=MYLUA:.lua --regex-MYLUA="/^.*\s*function\s*(\w+):(\w+).*$/\2/f/" --regex-MYLUA="/^\s*(\w+)\s*=\s*[0-9]+.*$/\1/e/" --regex-MYLUA="/^.*\s*function\s*(\w+)\.(\w+).*$/\2/f/" --regex-MYLUA="/^.*\s*function\s*(\w+)\s*\(.*$/\1/f/" --regex-MYLUA="/^\s*(\w+)\s*=\s*\{.*$/\1/e/" --regex-MYLUA="/^\s*module\s+\"(\w+)\".*$/\1/m,module/" --regex-MYLUA="/^\s*module\s+\"[a-zA-Z0-9._]+\.(\w+)\".*$/\1/m,module/" --languages=MYLUA --excmd=number -R .
```

# ctags lua 规则增强

[see](https://gist.github.com/yongkangchen/10120546)

```
--regex-LUA=/^.*\s*function[ \t]*([a-zA-Z0-9_]+):([a-zA-Z0-9_]+).*$/\2/f,function/
--regex-LUA=/^.*\s*function[ \t]*([a-zA-Z0-9_]+)\.([a-zA-Z0-9_]+).*$/\2/f,function/
--regex-LUA=/^.*\s*function[ \t]*([a-zA-Z0-9_]+)\s*\(.*$/\1/f,function/

--regex-LUA=/([a-zA-Z0-9_]+) = require[ (]"([^"]+)"/\1/r,require/

--regex-LUA=/[ \t]{1}([a-zA-Z0-9_]+)[ \t]*[=][^=]/\1/v,variable/

--regex-LUA=/[ \t]*([a-zA-Z0-9_]+)[ \t]*=[ \t]*module_define.*$/\1/m,module/
--regex-LUA=/func_table\[ msg\.([A-Z_]+) \].+/\1/
--regex-LUA=/\([ \t]*msg.([A-Z_]+)[ \t]*\)/\1/


ctags lua 规则增强,将以上部分保存至目录：
/ctags.cnf (on MSDOS, MSWindows only)
/etc/ctags.conf
/usr/local/etc/ctags.conf
$HOME/.ctags
$HOME/ctags.cnf (on MSDOS, MSWindows only)
.ctags
ctags.cnf (on MSDOS, MSWindows only)

支持识别：
function xxx.yyy() end
function xxx:yyy() end
modules("xxx",...)
全局变量

已测试系统：
mac os x 10.9
centos 6.4

参考文档：http://ctags.sourceforge.net/ctags.html
```
