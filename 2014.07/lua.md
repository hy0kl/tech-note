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
/usr/bin/ld: /home/baihe/local/lib/liblua.a(lapi.o): relocation R_X86_64_32 against `luaO_nilobject_' can not be used when making a shared object; recompile with -fPIC
/home/wok/local/lib/liblua.a: error adding symbols: Bad value
collect2: error: ld returned 1 exit status
```

[参考](http://www.cppblog.com/colorful/archive/2013/04/23/199659.html)

```
解决方法
如果服务器是64位的,这时要调整一下 Makefile: vi src/Makefile, 在 CFLAGS 里加上-fPIC.
```
