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

