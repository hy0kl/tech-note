# lua中冒号(:)与点号(.)的区别

[参考一](http://my.oschina.net/lonewolf/blog/173065)
[参考二](http://www.cnblogs.com/youxilua/archive/2011/07/28/2119059.html)

```
可以把点号(.)作为静态方法来看待，冒号(:)作为成员方法来看待。
用lua进行面向对象的编程, 声明方法和调用方法统一用冒号,对于属性的调用全部用点号
```

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

自带的 cmake 编译出来的是静态 *.a 库.因为就一个文件,手工编译成动态共享库.

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
