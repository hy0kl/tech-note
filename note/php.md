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
