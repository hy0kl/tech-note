# 一个动态库报错的问题记录

```
在用户目录下安装了 libjpeg 后, pbcopy 报错了.
$ pbcopy
dyld: Symbol not found: __cg_jpeg_resync_to_restart
  Referenced from: /System/Library/Frameworks/ImageIO.framework/Versions/A/Resources/libTIFF.dylib
  Expected in: /Users/hy0kl/local/lib/libJPEG.dylib
 in /System/Library/Frameworks/ImageIO.framework/Versions/A/Resources/libTIFF.dylib
Trace/BPT trap: 5
```

[参考](http://stackoverflow.com/questions/17643509/conflict-between-dynamic-linking-priority-in-osx)

```
You should not set library paths using DYLD_LIBRARY_PATH. As you've discovered, that tends to explode. Executables and libraries should have their library requirements built into them at link time. Use otool -L to find out what the file is looking for:

$ otool -L /System/Library/Frameworks/ImageIO.framework/Versions/A/ImageIO
/System/Library/Frameworks/ImageIO.framework/Versions/A/ImageIO:
    /System/Library/Frameworks/ImageIO.framework/Versions/A/ImageIO (compatibility version 1.0.0, current version 1.0.0)
    ...
    /usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1197.1.1)
For an example of one of my homebrew-built programs:

$ otool -L /usr/local/bin/gifcolor
/usr/local/bin/gifcolor:
    /usr/local/Cellar/giflib/4.1.6/lib/libgif.4.dylib (compatibility version 6.0.0, current version 6.6.0)
    /usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 159.1.0)
Note that it references /usr/local. If you've built it in such a way that it references the wrong library, I recommend rebuilding and pointing it to the correctly library.

If that's impossible, it is possible to edit what path is used using install_name_tool, but there are cases where this doesn't work, such as if the new path is longer than the old path and you didn't link it with -header_pad_max_install_names. Rebuilding with the correct path is preferred.

Note that there are a few "special" paths available that allow libraries to be found relative to their loader. See @executable_path/ and its kin in the dyld(1) man page.
```

# aclocal: command not found

need install automake.[see](http://stackoverflow.com/questions/9575989/install-autoreconf-on-osx-lion)

[ftp下载](ftp://ftp.gnu.org/gnu/automake/)

标准安装

# pkg-config: command not found

[下载](http://pkgconfig.freedesktop.org/releases/)

[see](http://blog.csdn.net/yuanya/article/details/8801736)

```
1.检测环境是否已安装 pkg-config
再命令行中输入: pkg-config 若未安装, 则提示命令未找到.

2.安装pkg-config

curl http://pkgconfig.freedesktop.org/releases/pkg-config-0.28.tar.gz -o pkg-config-0.28.tar.gz

tar -xf pkg-config-0.28.tar.gz
cd pkg-config-0.28

./configure  --with-internal-glib

make
sudo  make install
```

# liblzma

[主页](http://tukaani.org/xz/)

# shuf 命令不存在

```
$ shuf
-bash: shuf: command not found
```

谷歌说要安装 `gnu coreutils`,下载了一份拷贝后发现,文件数挺多,为了一个 shuf 命令折腾一大圈,有点不划算.

[Coreutils - GNU core utilities](https://www.gnu.org/software/coreutils/)

先用 awk 代替,实在不成再来折腾.

# darwin 源码编译 openssl

```
如果出现 openssl 在64位编译不支持时,形如:

ld: warning: in /Users/hy0kl/src/openssl-1.0.1c/.openssl/lib/libssl.a, file was built for unsupported file format which is not the architecture being linked (x86_64)
ld: warning: in /Users/hy0kl/src/openssl-1.0.1c/.openssl/lib/libcrypto.a, file was built for unsupported file format which is not the architecture being linked (x86_64)
Undefined symbols:
  "_SSL_write", referenced from:

先到 openssl 下执行 ./config --help,再修改 Makefile 文件, ./Configure darwin64-x86_64-cc.
```

# Mac 编译 nginx + openssl 出错

```
$uname -a
Darwin MacPro.local 13.3.0 Darwin Kernel Version 13.3.0: Tue Jun  3 21:27:35 PDT 2014; root:xnu-2422.110.17~1/RELEASE_X86_64 x86_64

出错摘要:
  "_i2a_ASN1_INTEGER", referenced from:
      _ngx_ssl_get_serial_number in ngx_event_openssl.o
  "_i2d_OCSP_REQUEST", referenced from:
      _ngx_ssl_certificate_status_callback in ngx_event_openssl_stapling.o
  "_i2d_OCSP_RESPONSE", referenced from:
      _ngx_ssl_stapling in ngx_event_openssl_stapling.o
  "_i2d_SSL_SESSION", referenced from:
      _ngx_ssl_new_session in ngx_event_openssl.o
  "_sk_num", referenced from:
      _ngx_ssl_stapling in ngx_event_openssl_stapling.o
  "_sk_value", referenced from:
      _ngx_ssl_stapling in ngx_event_openssl_stapling.o
ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make[1]: *** [objs/nginx] Error 1
make: *** [build] Error 2

解决方法: (http://www.oschina.net/question/96568_121563)
对于mac os x - x86_64 系统，提供另外一个办法（如果export KERNEL_BITS=64不生效的话）

进入nginx src 目录
$ ./configure

手动修改 objs/Makefile:
./config --prefix=/Users/xxx/Downloads/openssl-1.0.1e/.openssl no-shared  no-threads
改成
./Configure darwin64-x86_64-cc --prefix=/Users/xxx/Downloads/openssl-1.0.1e/.openssl no-shared  no-threads

$ make
```

# mac 编译 openssl 出错

```
openssl/ssl/man/man3/hmac.3: Too many levels of symbolic links
```

[参考](http://trac.nginx.org/nginx/ticket/583)

```
A workaround is to delete the folder ../openssl-1.0.1h/.openssl, at which point `make` in nginx-1.7.1 works as expected.
```

# 安装 PostgreSQL php 扩展

标准安装时报错:

```
checking for pg_config... not found
configure: error: Cannot find libpq-fe.h. Please specify correct PostgreSQL installation path
```

解决方法[see](http://stackoverflow.com/questions/6588174/enabling-postgresql-support-in-php-on-mac-os-x):

```shell
$ cd php-src/ext/pgsql
$ ./configure --with-php-config=/Users/hy0kl/php/bin/php-config  --with-pgsql=/Library/PostgreSQL/9.4/

$ cd php-src/pdo_pgsql
$ ./configure --with-php-config=/Users/hy0kl/php/bin/php-config  --with-pdo-pgsql=/Library/PostgreSQL/9.4/

$ ~/php/bin/php -m | grep pg
pdo_pgsql
pgsql
```

# 命令行快速删除

```
ctrl + u[w] 从当前快速删除到开头
ctrl + d 从当前光标向后删除一个字符
ctrl + k 从当前光标向后删除所有内容
```

# 命令行定位

```
ctrl + a 将光标置于开头
ctrl + e 将光标置于行尾
ctrl + f 光标向前移动一个字符
ctrl + b 光标向后移动一个字符
```
#  php Segmentation fault: 11

```
Generating phar.php
/bin/sh: line 1: 76451 Segmentation fault: 11  ` if test -x "/Users/MLS/src/php-5.6.8/sapi/cli/php"; then /Users/MLS/src/php-5.6.8/build/shtool echo -n -- "/Users/MLS/src/php-5.6.8/sapi/cli/php -n"; if test "x" != "x"; then /Users/MLS/src/php-5.6.8/build/shtool echo -n -- " -d extension_dir=/Users/MLS/src/php-5.6.8/modules"; for i in bz2 zlib phar; do if test -f "/Users/MLS/src/php-5.6.8/modules/$i.la"; then . /Users/MLS/src/php-5.6.8/modules/$i.la; /Users/MLS/src/php-5.6.8/build/shtool echo -n -- " -d extension=$dlname"; fi; done; fi; else /Users/MLS/src/php-5.6.8/build/shtool echo -n -- "/Users/MLS/src/php-5.6.8/sapi/cli/php"; fi;` -d 'open_basedir=' -d 'output_buffering=0' -d 'memory_limit=-1' -d phar.readonly=0 -d 'safe_mode=0' /Users/MLS/src/php-5.6.8/ext/phar/build_precommand.php > ext/phar/phar.php
make: *** [ext/phar/phar.php] Error 139
```

# dyld: Library not loaded: libmysqlclient.18.dylib

```
$ sudo ln -s /usr/local/mysql/lib/libmysqlclient.18.dylib /usr/local/lib/libmysqlclient.18.dylib
```

# config.status: error: cannot find input file: `Makefile.in'

```
https://github.com/rscada/libmbus/issues/68

autoheader \
    && aclocal \
    && libtoolize --ltdl --copy --force \
    && automake --add-missing --copy \
    && autoconf \
    && ./configure

在我机器上没有解决

http://stackoverflow.com/questions/8560997/installing-iulib-config-status-error-cannot-find-input-file-makefile-in

> Try
>
> automake --add-missing
> or just
>
> automake
> It should create your Makefile.in.

$ vim configure.ac

AC_PROG_CC
AC_PROG_CXX
AC_PROG_GCC_TRADITIONAL
+ AM_PROG_CC_C_O

这样能过
```

# mac 下源码编译 libzookeeper

```
由于要安装 php 扩展 zookeeper,安装过程依赖 libzookeeper.
转而安装 libzookeeper 时问题一大堆.
先从 http://mirrors.hust.edu.cn/apache/zookeeper/ 下载最新版.
linux 下传统安装没有问题, mac 下报错:

./include/recordio.h:76:9: error: conflicting types for '__builtin_constant_p'

原因是有个函数在 mac 下出现了重复定义,产生了冲突.

编辑 include/recordio.h, src/recordio.c 注释掉 int64_t htonll(int64_t v); 对应的申明还定义,再编译安装就 ok 了.
```

# 命令行参数自动补齐

[bash-completion](https://bash-completion.alioth.debian.org/)

mac 下需要升级 bash 到 4.0 以上.

# possibly undefined macro: AM_INIT_AUTOMAKE

[see](http://blog.sina.com.cn/s/blog_a1cafb9901016bzr.html)

```
执行 autoconf -i 是遇到 possibly undefined macro: AM_INIT_AUTOMAKE

尝试运行： autoreconf --install
```

# gcc 报错
`Agreeing to the Xcode/iOS license requires admin privileges, please re-run as root via sudo.`
[解决方法](http://stackoverflow.com/questions/26197347/agreeing-to-the-xcode-ios-license-requires-admin-privileges-please-re-run-as-r)

```
$ sudo xcodebuild -license
```

# mac 下远程桌面访问 Ubuntu
## X11
```
这是最简单最方便的方法，不需要在 Ubuntu 端做任何配置，不过在 Mac 端必须已装有 X11，在 Terminal 上敲（把 ubuntu 换成对应的服务器 IP 地址或域名）：
$ Xnest -geometry 1280x800 :1 & DISPLAY=:1 ssh -X ubuntu  gnome-session
```

# OS X添加用户到以存在的组？
[see](http://www.zhihu.com/question/23375941)

```
$ sudo dseditgroup -o edit -a username -t user www
```

# How can I install 7zip so I can run it from Terminal on OS X

```
mac app `The Unarchiver`

1. http://sourceforge.net/projects/p7zip/files/p7zip/
2. tar xf *.bz2
3. cd
4. make
5. cp bin/* ~/local/bin

实际操作总是 make 报错:

p7zip_9.38.1
ld: internal error: atom not found in symbolIndex(__ZN13CObjectVectorI9CMyComPtrI19ISequentialInStreamEED2Ev) for architecture x86_64

p7zip_15.09
ld: internal error: atom not found in symbolIndex(__ZN12NCoderMixer29CBindInfoaSERKS0_) for architecture x86_64

p7zip_9.20.1
ld: internal error: atom not found in symbolIndex(__ZN13CObjectVectorI11CStringBaseIwEE3AddERKS1_) for architecture x86_64

下载: http://p7zip.en.softonic.com/mac/download
```

# mac R  Warning messages

[参考](http://stackoverflow.com/questions/27299420/how-to-get-rid-of-warning-messages-after-installing-r)

```
During startup - Warning messages:
1: Setting LC_CTYPE failed, using "C"
2: Setting LC_COLLATE failed, using "C"
3: Setting LC_TIME failed, using "C"
4: Setting LC_MESSAGES failed, using "C"
5: Setting LC_MONETARY failed, using "C"

$ defaults write org.R-project.R force.LANG en_US.UTF-8
```

# [在Mac OSX 编译tree命令](http://rockyfeng.me/mac_tree.html)

在Linux中一个tree命令就可以把文件夹的目录结构很好的展现，但Mac下默认是没有 这个命令的，所以得自己编译。

从 http://mama.indstate.edu/users/ice/tree/ 下载最新得tree, 现在的版本 是1.7.0

```
tar xf tree-1.7.0.tgz
cd tree-1.7.0
make
```

本来以为这样就会成功，结果出现了以下错误

```
Undefined symbols for architecture x86_64:
  "_strverscmp", referenced from:
      _versort in tree.o
ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```

查看一下Makefile, 如果是Mac系统的话，需要去掉一下的注释，

```
# Uncomment for OS X:
CC=cc
CFLAGS=-O2 -Wall -fomit-frame-pointer -no-cpp-precomp
LDFLAGS=
MANDIR=/usr/share/man/man1
OBJS+=strverscmp.o
```

从新来一遍make

```
make
sudo make install
```

# 常见错误解决思路

```
Undefined symbols for architecture x86_64:
...
ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```

经验表明,出现以上编译错误,大概率是软件版本与系统不匹配造成的,可适当向前或向前几个版本试试.

# 源码编译安装Python-3.5.1

默认编译参数报错:

```
Ignoring ensurepip failure: pip 7.1.2 requires SSL/TLS
```

指定新的编译参数后编译又报错:

```
./configure --prefix=/Users/hy0kl/local/python3.5.1 --with-libs='-lssl' --disable-ipv6 --with-ensurepip=install

Include/pyport.h:261:13: error: "This platform's pyconfig.h needs to define PY_FORMAT_LONG_LONG"
#           error "This platform's pyconfig.h needs to define PY_FORMAT_LONG_LONG"

vim Include/pyport.h
打开此宏
256 #ifndef PY_FORMAT_LONG_LONG
257 #   define PY_FORMAT_LONG_LONG "I64"
258 #endif
```

发现安装还是报一样的提示.

[安装最新的openssl](http://mac-dev-env.patrickbougie.com/openssl/)

```
./configure darwin64-x86_64-cc --prefix=/User/hy0kl/local/openssl
make depend
make
make install
```


```
vi Modules/Setup.dist

SSL=/Users/hy0kl/local/openssl
_ssl _ssl.c \
    -DUSE_SSL -I$(SSL)/include -I$(SSL)/include/openssl \
    -L$(SSL)/lib -lssl -lcrypto

./configure --prefix=/Users/hy0kl/local/python3.5.1
```

重新编译安装:

```
Python build finished successfully!
The necessary bits to build these optional modules were not found:
_gdbm                 _ssl                  ossaudiodev
spwd
To find the necessary bits, look in setup.py in detect_modules() for the module's name.
```

# [compile php with openssl on mac osx error](https://upliu.net/compile-php-with-openssl-on-max-osx-error.html)

从源码手动编译 PHP 时出现如下错误：

```
Undefined symbols for architecture x86_64:
  "_PKCS5_PBKDF2_HMAC", referenced from:
      _zif_openssl_pbkdf2 in openssl.o
  "_TLSv1_1_client_method", referenced from:
      _php_openssl_setup_crypto in xp_ssl.o
  "_TLSv1_1_server_method", referenced from:
      _php_openssl_setup_crypto in xp_ssl.o
  "_TLSv1_2_client_method", referenced from:
      _php_openssl_setup_crypto in xp_ssl.o
  "_TLSv1_2_server_method", referenced from:
      _php_openssl_setup_crypto in xp_ssl.o
ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make: *** [libs/libphp5.bundle] Error 1
```

解决办法

MakeFile 里面找到类似下面这一行：

```
EXTRA_LIBS = -lresolv -lmcrypt -lltdl -liconv-lm -lxml2 -lcurl -lssl -lcrypto
```

删除所有的 -lssl 和 -lcrypto 然后添加 libssl.dylib 和 libcrypto.dylib 的路径（如果你安装了 brew，那么则是 /usr/local/opt/openssl/lib/），重新运行 make 命令，done。

附上我修改后的 MakeFile EXTRA_LIBS 那一行：

```
EXTRA_LIBS = -lz -lresolv -lmcrypt -lltdl -lstdc++ -liconv -liconv -lpng -lz -lcurl -lz -lm -lxml2 -lz -licucore -lm -lcurl -lxml2 -lz -licucore -lm -licui18n -licuuc -licudata -licuio -lxml2 -lz -licucore -lm -lxml2 -lz -licucore -lm -lxml2 -lz -licucore -lm -lxml2 -lz -licucore -lm /usr/local/opt/openssl/lib/libssl.dylib /usr/local/opt/openssl/lib/libcrypto.dylib
```

# pod 报警告

```
$ pod
/System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/lib/ruby/2.0.0/universal-darwin16/rbconfig.rb:213: warning: Insecure world writable dir /usr/local/mysql in PATH, mode 040777
```

解决方法:

```
$ cd /usr/local
$ sudo chmod go-w mysql-5.6.19-osx10.7-x86_64
```

