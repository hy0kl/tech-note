# 记一次数据不完整的事故

现象描述: 一个 JSON 数据接口,用浏览器访问,数据正常.但通过 php curl 的脚本来调用时,总是返回不完整的数据.确认不存在代码改动的干扰,查无果.后来发现开发机磁盘满了,却没有证据表明是直接原因.查看 nginx error log 时,发现如下记录:

```
2014/05/30 14:31:30 [crit] 11046#0: *81985 writev() "/usr/local/webserver/nginx/fastcgi_temp/9/66/0000010669" failed (28: No space left on device) while reading upstream, client: 192.168.25.32, server: , request: "POST /api/countries HTTP/1.1", upstream: "fastcgi://unix:/tmp/php-cgi.sock:", host: "192.168.3.200:8086"
```

由于确认是磁盘满导致的.
但遗憾的是,出问题的开发机把 php notice 关了,结果有用的错误信息被淹没在大量 php notice 错误之中了.可见开发环境关掉 notice 是多么屌爆的一件事.

# 打开 nginx 的 rewrite 日志

```
在http段加入:

rewrite_log on;
error_log logs/error.log notice;
```

# nginx 内置预定义变量

[FROM](http://www.nginx.cn/273.html)

```conf
nginx的配置文件中可以使用的内置变量以美元符$开始，也有人叫全局变量。其中，部分预定义的变量的值是可以改变的。

$arg_PARAMETER 这个变量值为：GET请求中变量名PARAMETER参数的值。

$args 这个变量等于GET请求中的参数。例如，foo=123&bar=blahblah;这个变量只可以被修改

$binary_remote_addr 二进制码形式的客户端地址。

$body_bytes_sent 传送页面的字节数

$content_length 请求头中的Content-length字段。

$content_type 请求头中的Content-Type字段。

$cookie_COOKIE cookie COOKIE的值。

$document_root 当前请求在root指令中指定的值。

$document_uri 与$uri相同。

$host 请求中的主机头(Host)字段，如果请求中的主机头不可用或者空，则为处理请求的server名称(处理请求的server的server_name指令的值)。值为小写，不包含端口。

$hostname  机器名使用 gethostname系统调用的值

$http_HEADER   HTTP请求头中的内容，HEADER为HTTP请求中的内容转为小写，-变为_(破折号变为下划线)，例如：$http_user_agent(Uaer-Agent的值), $http_referer...;

$sent_http_HEADER  HTTP响应头中的内容，HEADER为HTTP响应中的内容转为小写，-变为_(破折号变为下划线)，例如： $sent_http_cache_control, $sent_http_content_type...;

$is_args 如果$args设置，值为"?"，否则为""。

$limit_rate 这个变量可以限制连接速率。

$nginx_version 当前运行的nginx版本号。

$query_string 与$args相同。

$remote_addr 客户端的IP地址。

$remote_port 客户端的端口。

$remote_user 已经经过Auth Basic Module验证的用户名。

$request_filename 当前连接请求的文件路径，由root或alias指令与URI请求生成。

$request_body 这个变量（0.7.58+）包含请求的主要信息。在使用proxy_pass或fastcgi_pass指令的location中比较有意义。

$request_body_file 客户端请求主体信息的临时文件名。

$request_completion 如果请求成功，设为"OK"；如果请求未完成或者不是一系列请求中最后一部分则设为空。

$request_method 这个变量是客户端请求的动作，通常为GET或POST。
包括0.8.20及之前的版本中，这个变量总为main request中的动作，如果当前请求是一个子请求，并不使用这个当前请求的动作。

$request_uri 这个变量等于包含一些客户端请求参数的原始URI，它无法修改，请查看$uri更改或重写URI。

$scheme 所用的协议，比如http或者是https，比如rewrite ^(.+)$ $scheme://example.com$1 redirect;

$server_addr 服务器地址，在完成一次系统调用后可以确定这个值，如果要绕开系统调用，则必须在listen中指定地址并且使用bind参数。

$server_name 服务器名称。

$server_port 请求到达服务器的端口号。

$server_protocol 请求使用的协议，通常是HTTP/1.0或HTTP/1.1。

$uri 请求中的当前URI(不带请求参数，参数位于$args)，不同于浏览器传递的$request_uri的值，它可以通过内部重定向，或者使用index指令进行修改。不包括协议和主机名，例如/foo/bar.html
```

# nginx 开发环境编译参数

```
./configure --prefix=/Users/hy0kl/nginx \
    --with-http_ssl_module \
    --with-http_spdy_module \
    --with-http_realip_module \
    --with-http_addition_module \
    --with-http_xslt_module \
    --with-http_sub_module \
    --with-http_dav_module \
    --with-http_flv_module \
    --with-http_mp4_module \
    --with-http_gunzip_module \
    --with-http_gzip_static_module \
    --with-http_auth_request_module \
    --with-http_random_index_module \
    --with-http_secure_link_module \
    --with-http_stub_status_module \
    --with-pcre=/Users/hy0kl/src/pcre-8.35 \
    --with-zlib=/Users/hy0kl/src/zlib-1.2.8 \
    --with-openssl=/Users/hy0kl/src/openssl-1.0.1h \
    --add-module=/Users/hy0kl/src/nginx-ext/ngx_devel_kit-0.2.19 \
    --add-module=/Users/hy0kl/src/nginx-ext/lua-nginx-module-0.9.11 \
    --add-module=/Users/hy0kl/src/nginx-ext/echo-nginx-module-0.55 \
    --with-debug

```

# nginx 日志切割

[参考](http://www.nginx.cn/255.html)

nginx的日志文件没有rotate功能。如果你不处理，日志文件将变得越来越大，还好我们可以写一个nginx日志切割脚本来自动切割日志文件。

第一步就是重命名日志文件，不用担心重命名后nginx找不到日志文件而丢失日志。在你未重新打开原名字的日志文件前，nginx还是会向你重命名的文件写日志，linux是靠文件描述符而不是文件名定位文件。

第二步向nginx主进程发送USR1信号。

nginx主进程接到信号后会从配置文件中读取日志文件名称，重新打开日志文件(以配置文件中的日志名称命名)，并以工作进程的用户作为日志文件的所有者。

重新打开日志文件后，nginx主进程会关闭重名的日志文件并通知工作进程使用新打开的日志文件。

工作进程立刻打开新的日志文件并关闭重名名的日志文件。

然后你就可以处理旧的日志文件了。

nginx日志按日期自动切割脚本如下

```shell
# nginx日志切割脚本
# author: http://www.nginx.cn

#!/bin/bash
# 设置日志文件存放目录
logs_path="/usr/local/nginx/logs/"
# 设置pid文件
pid_path="/usr/local/nginx/nginx.pid"

# 重命名日志文件
mv ${logs_path}access.log ${logs_path}access_$(date -d "yesterday" +"%Y%m%d").log

# 向nginx主进程发信号重新打开日志
kill -USR1 `cat ${pid_path}`
```

保存以上脚本nginx_log.sh,或者点此下载

crontab 设置作业

0 0 * * * bash /usr/local/nginx/nginx_log.sh

这样就每天的0点0分把nginx日志重命名为日期格式，并重新生成今天的新日志文件。

# 隐藏Nginx版本号

[参考](http://blog.cunss.com/?p=299)

一、 关于Nginx版本隐藏，有2种方法：

 1.编译安装时，可以源码包里面修改

 具体在 src/core/nginx.h 文件下

```c
#define nginx_version      1004004
#define NGINX_VERSION      "1.0.5"
#define NGINX_VER          "nginx/" NGINX_VERSION
```

修改这里面的版本号，当然可以伪造为其他，然后再进行编译

2.不想重新编译，可直接在Nginx配置文件中进行修改

在http{}段添加全局参数：

```conf
server_tokens off;
```

如果使用Tengine，除了完全兼容Nginx外，还有其独有的配置方法

同样在http{}段

使用全局参数：

```conf
server_tag off;
```

当然可以对server_tag 进行伪造，如

server_tag BAT/1.1;
以下为设置前后对比：

off状态

```
[root@cunss ~]# curl -I http://blog.cunss.com/
HTTP/1.1 200 OK
Date: Fri, 11 Apr 2014 15:53:50 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
Vary: Accept-Encoding
X-Pingback: http://blog.cunss.com/xmlrpc.php
```

设置为其他值

```
[root@cunss ~]# curl -I http://blog.cunss.com/
HTTP/1.1 200 OK
Server: BAT/1.1
Date: Fri, 11 Apr 2014 15:54:04 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
Vary: Accept-Encoding
X-Pingback: http://blog.cunss.com/xmlrpc.php
```

二、除了Nginx，apache也可以隐藏版本信息

需要在httpd.conf加入

```conf
ServerTokens ProductOnly
ServerSignature Off
```

三、同时wordpress使用php,因此php也需要进行相关安全设置

在php.ini中

```ini
expose_php = Off  #隐藏php版本号
disable_functions = system,exec,shell_exec,passthru,popen,dl,phpinfo #禁用危险函数
display_errors = Off    #关闭错误日志
allow_url_fopen = Off   # 禁用打开远程url
safe_mode = On     #打开php的安全模式
```

# linux nginx 安装 lua 扩展报错

```
-Wl,-E -lpthread -lcrypt -llua -lm -lpcre -lcrypto -lcrypto -lz
/usr/bin/ld: //usr/local/lib/liblua.a(loadlib.o): undefined reference to symbol 'dlclose@@GLIBC_2.2.5'
//lib/x86_64-linux-gnu/libdl.so.2: error adding symbols: DSO missing from command line
collect2: error: ld returned 1 exit status
```

```
谷歌后有解决方法:
$ vim objs/Makefile
找到 -Wl,-E 所在行, 加入 -ldl 即可.
```

# 扩展模块

```
* nginx-push-stream-module
https://github.com/wandenberg/nginx-push-stream-module
把 nginx 变成流式推服务器

* nginx_http_push_module
https://pushmodule.slact.net/
把 nginx变成支持常连接的 HTTP 推服务器

* ngx_cache_purge
https://github.com/FRiCKLE/ngx_cache_purge
nginx module which adds ability to purge content from FastCGI, proxy, SCGI and uWSGI caches.

* HttpAccessKeyModule
http://wiki.nginx.org/HttpAccessKeyModuleChs
防盗链

* fastdfs-nginx-module
https://code.google.com/p/fastdfs-nginx-module/
分存存储的 nginx 代理

* nginx-upload-progress-module
https://github.com/masterzen/nginx-upload-progress-module
上传文件
```
