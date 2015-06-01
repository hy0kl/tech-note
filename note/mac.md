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
