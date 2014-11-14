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

# php 开发接口时空数组和空对象

现象描述:
和后端约定 ext 字段是对象,但初始化为 array(),如果后端有没有结果,最终给 app 输出 json 时 ext 的值变成了 [],即变成数组了.

解决方法:
将 ext 初始化对象 new ArrayObject().

# yaf 中 ajax, iframe 页面出现反复刷新或闪动现象

```
原因是 yaf 框架代理页面时,没有遵行浏览器头缓存协议,在对应的 action 中加入以下代码可解决:
$this->getView()->setLayout(null);
```

# php 低版兼容

不用也可,只做个记录,毕竟用低版的概率很小了

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

