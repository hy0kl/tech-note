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