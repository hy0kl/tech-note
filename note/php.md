# mac php configure 参数

```
$ uname -a
Darwin MacPro.local 13.3.0 Darwin Kernel Version 13.3.0: Tue Jun  3 21:27:35 PDT 2014; root:xnu-2422.110.17~1/RELEASE_X86_64 x86_64

$ ./configure --prefix=/usr \
    --mandir=/usr/share/man \
    --infodir=/usr/share/info \
    --disable-dependency-tracking \
    --sysconfdir=/private/etc \
    --with-apxs2=/usr/sbin/apxs \
    --enable-cli \
    --with-config-file-path=/etc \
    --with-config-file-scan-dir=/Library/Server/Web/Config/php \
    --with-libxml-dir=/usr \
    --with-openssl=/usr \
    --with-kerberos=/usr \
    --with-zlib=/usr \
    --enable-bcmath \
    --with-bz2=/usr \
    --enable-calendar \
    --disable-cgi \
    --with-curl=/usr \
    --enable-dba \
    --enable-ndbm=/usr \
    --enable-exif \
    --enable-fpm \
    --enable-ftp \
    --with-gd \
    --with-freetype-dir=/BinaryCache/apache_mod_php/apache_mod_php-87.2~1/Root/usr/local \
    --with-jpeg-dir=/BinaryCache/apache_mod_php/apache_mod_php-87.2~1/Root/usr/local \
    --with-png-dir=/BinaryCache/apache_mod_php/apache_mod_php-87.2~1/Root/usr/local \
    --enable-gd-native-ttf \
    --with-icu-dir=/usr \
    --with-ldap=/usr \
    --with-ldap-sasl=/usr \
    --with-libedit=/usr \
    --enable-mbstring \
    --enable-mbregex \
    --with-mysql=mysqlnd \
    --with-mysqli=mysqlnd \
    --without-pear \
    --with-pdo-mysql=mysqlnd \
    --with-mysql-sock=/var/mysql/mysql.sock \
    --with-readline=/usr \
    --enable-shmop \
    --with-snmp=/usr \
    --enable-soap \
    --enable-sockets \
    --enable-sqlite-utf8 \
    --enable-sysvmsg \
    --enable-sysvsem \
    --enable-sysvshm \
    --with-tidy \
    --enable-wddx \
    --with-xmlrpc \
    --with-iconv-dir=/usr \
    --with-xsl=/usr \
    --enable-zend-multibyte \
    --enable-zip \
    --with-pcre-regex=/usr
```

```
开发环境的 php 编译参数
./configure --prefix=/home/work/php \
    --enable-fpm \
    --enable-bcmath \
    --enable-ftp \
    --enable-mbstring \
    --enable-soap \
    --enable-sockets \
    --enable-zip \
    --enable-mysqlnd \
    --enable-exif \
    --with-libxml-dir=/usr/lib \
    --with-openssl=/usr/local/ssl \
    --with-pcre-regex=/usr \
    --with-pcre-dir=/usr \
    --with-zlib=/usr \
    --with-bz2=/usr \
    --with-gd=/usr \
    --with-vpx-dir=/usr \
    --with-xpm-dir=/usr \
    --with-jpeg-dir=/usr \
    --with-freetype-dir=/usr \
    --with-mysql=/usr \
    --with-mysqli=/usr/bin/mysql_config \
    --with-pdo-mysql=/usr/bin/mysql_config \
    --enable-sysvsem \
    --enable-sysvmsg
```

```
$ uname -a
Darwin MacPro.local 14.0.0 Darwin Kernel Version 14.0.0: Fri Sep 19 00:26:44 PDT 2014; root:xnu-2782.1.97~2/RELEASE_X86_64 x86_64
./configure --prefix=/usr \
    --mandir=/usr/share/man \
    --infodir=/usr/share/info \
    --disable-dependency-tracking \
    --sysconfdir=/private/etc \
    --with-apxs2=/usr/sbin/apxs \
    --enable-cli \
    --with-config-file-path=/etc \
    --with-config-file-scan-dir=/Library/Server/Web/Config/php \
    --with-libxml-dir=/usr \
    --with-openssl=/usr \
    --with-kerberos=/usr \
    --with-zlib=/usr \
    --enable-bcmath \
    --with-bz2=/usr \
    --enable-calendar \
    --disable-cgi \
    --with-curl=/usr \
    --enable-dba \
    --with-ndbm=/usr \
    --enable-exif \
    --enable-fpm \
    --enable-ftp \
    --with-png-dir=no \
    --with-gd \
    --with-jpeg-dir=/BinaryCache/apache_mod_php/apache_mod_php-93~55/Root/usr/local \
    --enable-gd-native-ttf \
    --with-icu-dir=/usr \
    --with-ldap=/usr \
    --with-ldap-sasl=/usr \
    --with-libedit=/usr \
    --enable-mbstring \
    --enable-mbregex \
    --with-mysql=mysqlnd \
    --with-mysqli=mysqlnd \
    --without-pear \
    --with-pear=no \
    --with-pdo-mysql=mysqlnd \
    --with-mysql-sock=/var/mysql/mysql.sock \
    --with-readline=/usr \
    --enable-shmop \
    --with-snmp=/usr \
    --enable-soap \
    --enable-sockets \
    --enable-sysvmsg \
    --enable-sysvsem \
    --enable-sysvshm \
    --with-tidy \
    --enable-wddx \
    --with-xmlrpc \
    --with-iconv-dir=/usr \
    --with-xsl=/usr \
    --enable-zend-multibyte \
    --enable-zip \
    --with-pcre-regex=/usr
```

php 版本升级变化,编译参数会有变化,粘别人的编译参数,或者沿用旧的记录都有问题,从网上粘的问题就更大了.需要视当前版本支持的参数变动.

php5.3 | php5.5
------ | ------
--enable-ndbm |  --with-ndbm
--without-pear | --without-pear <br /> --with-pear=no

# php 开发接口时空数组和空对象

现象描述:
和后端约定 ext 字段是对象,但初始化为 array(),如果后端有没有结果,最终给 app 输出 json 时 ext 的值变成了 [],即变成数组了.

解决方法:
将 ext 初始化对象 new ArrayObject().

# php 低版兼容

> 不用也可,只做个记录,毕竟用低版的概率很小了

```php
if (PHP_VERSION < '4.1.0')
{
    $_GET    = & $HTTP_GET_VARS;
    $_POST   = & $HTTP_POST_VARS;
    $_COOKIE = & $HTTP_COOKIE_VARS;
    $_SERVER = & $HTTP_SERVER_VARS;
    $_ENV    = & $HTTP_ENV_VARS;
    $_FILES  = & $HTTP_POST_FILES;
}
```

# php 中验证电子邮箱地址是否合法

一上来很自然会想到正则,今天刚好有个同事遇到 phpmailer 的一个问题,就顺便翻了下其源码,发现原来除了正则,还有更好的方法.源码摘抄如下:

```php
function ValidateAddress($address)
{
    if (function_exists('filter_var'))
    { //Introduced in PHP 5.2
        if(filter_var($address, FILTER_VALIDATE_EMAIL) === FALSE)
        {
            return false;
        }
        else
        {
            return true;
        }
    }
    else
    {
        return preg_match('/^(?:[\w\!\#\$\%\&\'\*\+\-\/\=\?\^\`\{\|\}\~]+\.)*[\w\!\#\$\%\&\'\*\+\-\/\=\?\^\`\{\|\}\~]+@(?:(?:(?:[a-zA-Z0-9_](?:[a-zA-Z0-9_\-](?!\.)){0,61}[a-zA-Z0-9_-]?\.)+[a-zA-Z0-9_](?:[a-zA-Z0-9_\-](?!$)){0,61}[a-zA-Z0-9_]?)|(?:\[(?:(?:[01]?\d{1,2}|2[0-4]\d|25[0-5])\.){3}(?:[01]?\d{1,2}|2[0-4]\d|25[0-5])\]))$/', $address);
    }
}
```

# 为 composer 提速
[gc_disable()](http://php.net/manual/zh/function.gc-disable.php)
[参考](http://segmentfault.com/q/1010000002402696)
[github commit](https://github.com/composer/composer/commit/ac676f47f7bbc619678a29deae097b6b0710b799)

Can be very useful for big projects, when you create a lot of objects that should stay in memory. So GC can't clean them up and just wasting CPU time.

composer 在运行的时候会创建大量的对象，这些对象会触发 GC 机制，而这些对象需要被使用，所以 GC 无法清除，因此，使用 gc_disable 禁用 GC 之后，会节省 cpu 时间，效率更高。

# mac下编译 php5.6

```
编译参数:
./configure --prefix=/Users/hy0kl/php \
    #--disable-dependency-tracking \
    --enable-cli \
    --with-libxml-dir=/usr \
    --with-openssl=/usr \
    --with-kerberos=/usr \
    --with-zlib=/usr \
    --enable-bcmath \
    --with-bz2=/usr \
    --enable-calendar \
    --disable-cgi \
    --with-curl=/usr \
    --enable-dba \
    --with-ndbm=/usr \
    --enable-exif \
    --enable-fpm \
    --enable-ftp \
    --with-gd \
    --with-jpeg-dir=/Users/hy0kl/local \
    --with-png-dir=/Users/hy0kl/local \
    --enable-gd-native-ttf \
    --with-icu-dir=/usr \
    --with-ldap=/usr \
    --with-ldap-sasl=/usr \
    --with-libedit=/usr \
    --enable-mbstring \
    --enable-mbregex \
    --with-mysql=mysqlnd \
    --with-mysqli=mysqlnd \
    --without-pear \
    --with-pear=no \
    --with-pdo-mysql=mysqlnd \
    --with-readline=/usr \
    --enable-shmop \
    --with-snmp=/usr \
    --enable-soap \
    --enable-sockets \
    --enable-sysvmsg \
    --enable-sysvsem \
    --enable-sysvshm \
    --with-tidy \
    --enable-wddx \
    --with-xmlrpc \
    --with-iconv-dir=/usr \
    --with-xsl=/usr \
    #--enable-zend-multibyte \
    --enable-zip \
    --with-pcre-regex=/Users/hy0kl/local

configure出错:
checking whether to enable JIS-mapped Japanese font support in GD... no
If configure fails try --with-vpx-dir=<DIR>
configure: error: jpeglib.h not found.

1. 下载 libjpeg http://libjpeg.sourceforge.net/
2. ./configure --enable-shared
编译时报个错: configure: error: cannot find macro directory `m4'
创建个 m4 的目录即可.源码包里面没带这个目录
但安装成功后出以下错:
$ pbcopy -v
dyld: Symbol not found: __cg_jpeg_resync_to_restart
  Referenced from: /System/Library/Frameworks/ImageIO.framework/Versions/A/Resources/libTIFF.dylib
  Expected in: /Users/hy0kl/local/lib/libJPEG.dylib
 in /System/Library/Frameworks/ImageIO.framework/Versions/A/Resources/libTIFF.dylib
Trace/BPT trap: 5
貌似其他命令没有受到影响.

If configure fails try --with-vpx-dir=<DIR>
checking for jpeg_read_header in -ljpeg... yes
configure: error: png.h not found.

下载 libpng http://libpng.sourceforge.net/index.html
编译安装即可

执行完报个警告:
configure: WARNING: unrecognized options: --disable-dependency-tracking, --enable-zend-multibyte
查看 php5.6 的编译参数时,发现这两个参数已经不存了.拿掉不支持参数再走一遍.

configure 过了后 make 通常问题不大,但若真出问题,那可不好折腾了,换思想吧.
```

# yaf 相关点收集

## yaf 下注册本地类

在 Bootstrap.php 中初始化

```php
/* 注册本地类名前缀, 这部分类名将会在本地类库查找 */
Yaf_Loader::getInstance()->registerLocalNameSpace(
    array(
         'Tool',
         'Cache',
         'Page',
         'Model',
    )
);
```

## yaf 导入预置类

在 Bootstrap.php 中初始化

```php
Yaf_loader::import(APPLICATION_PATH . '/application/actions/BaseAction.php');
```

## yaf 禁止自动渲染模板

```php
Yaf_Dispatcher::getInstance()->disableView();
```

## yaf 中 ajax, iframe 页面出现反复刷新或闪动现象

```
原因是 yaf 框架代理页面时,没有遵行浏览器头缓存协议,在对应的 action 中加入以下代码可解决:
$this->getView()->setLayout(null);
```

