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

<h1 id="nginx_var">nginx 内置预定义变量</h1>

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

# 日志格式允许包含的变量

```
$remote_addr, $http_x_forwarded_for 记录客户端IP地址
$remote_user 记录客户端用户名称
$request 记录请求的URL和HTTP协议
$status 记录请求状态
$body_bytes_sent 发送给客户端的字节数，不包括响应头的大小； 该变量与Apache模块mod_log_config里的“%B”参数兼容。
$bytes_sent 发送给客户端的总字节数。
$connection 连接的序列号。
$connection_requests 当前通过一个连接获得的请求数量。
$msec 日志写入时间。单位为秒，精度是毫秒。
$pipe 如果请求是通过HTTP流水线(pipelined)发送，pipe值为“p”，否则为“.”。
$http_referer 记录从哪个页面链接访问过来的
$http_user_agent 记录客户端浏览器相关信息
$request_length 请求的长度（包括请求行，请求头和请求正文）。
$request_time 请求处理时间，单位为秒，精度毫秒； 从读入客户端的第一个字节开始，直到把最后一个字符发送给客户端后进行日志写入为止。
$time_iso8601 ISO8601标准格式下的本地时间。
$time_local 通用日志格式下的本地时间。
```

# nginx 开发环境编译参数

```
$ date
Mon Dec 22 09:29:21 CST 2014
$ ./configure --prefix=/Users/hy0kl/nginx \
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

# 扩展模块汇总

```
* lua-nginx-module
Embed the Power of Lua into NGINX HTTP servers
https://github.com/openresty
https://github.com/openresty/lua-nginx-module

* nginx-rtmp-module
NGINX-based Media Streaming Server
https://github.com/arut/nginx-rtmp-module

* naxsi
NAXSI is an open-source, high performance, low rules maintenance WAF for NGINX
https://github.com/nbs-system/naxsi

* nginx-push-stream-module
把 nginx 变成流式推服务器
https://github.com/wandenberg/nginx-push-stream-module

* nginx_http_push_module
把 nginx变成支持常连接的 HTTP 推服务器
https://pushmodule.slact.net/

* ngx_cache_purge
nginx module which adds ability to purge content from FastCGI, proxy, SCGI and uWSGI caches.
https://github.com/FRiCKLE/ngx_cache_purge

* HttpAccessKeyModule
防盗链
http://wiki.nginx.org/HttpAccessKeyModuleChs

* fastdfs-nginx-module
分存存储的 nginx 代理
https://code.google.com/p/fastdfs-nginx-module/
https://github.com/happyfish100/fastdfs-nginx-module

* nginx-upload-progress-module
上传文件
https://github.com/masterzen/nginx-upload-progress-module

* ngx_http_ssi_module
The ngx_http_ssi_module module is a filter that processes SSI (Server Side Includes) commands in responses passing through it. Currently, the list of supported SSI commands is incomplete.

* ngx_chunkin
HTTP 1.1 chunked-encoding request body support for Nginx.
http://wiki.nginx.org/HttpChunkinModule

* nginx-limit-upstream
limit the number of connections to upstream for NGINX
https://github.com/cfsego/nginx-limit-upstream

* nginx-module-vts
Nginx virtual host traffic status module
https://github.com/vozlt/nginx-module-vts

* rds-json-nginx-module
An nginx output filter that formats Resty DBD Streams generated by ngx_drizzle and others to JSON
https://github.com/openresty/rds-json-nginx-module

* ngx_pagespeed
Automatic PageSpeed optimization module for Nginx
https://github.com/pagespeed/ngx_pagespeed

* ngx_postgres
upstream module that allows nginx to communicate directly with PostgreSQL database.
https://github.com/FRiCKLE/ngx_postgres

* lua-resty-cookie
Lua library for HTTP cookie manipulations for OpenResty/ngx_lua
https://github.com/cloudflare/lua-resty-cookie
```
# nginx 配置相关

## nginx 配置文件中的单位记法

```
当指定空间时,可以使用的单位有:
    K 或者 k 千字节(kiloByte, KB)
    M 或者 m 兆字节(MegaByte, MB)

当指定时间时,可以使用的单位有:
    ms  毫秒
    s   秒
    m   分钟
    h   小时
    d   天
    w   周(包含7天)
    M   月(包含30天)
    y   年(包含365天)
```
## nginx 错误日志设置

```
语法: error_log /path/file level;
默认: error_log logs/erro.log error;

/path/file 可以是 /dev/null,这样不会输出任何日志,是关闭 error 日志的唯一手段.
/path/file 也可是 stderr,这样日志输出到标准错误文件中.


level 是日志的输出级别,取值范围是 debug, info, notice, warn, error, crit, alert, emerg, 从左到右级别依次增大.
当设定一个级别时,大于或等于该级别的日志都会被输出到 /path/file 文件中,小于该级别的日志则不会输出.

注意: 如果日志级别设定到 debug,必须在 configure 时加入 --with-debug 配置项.
```

## core dump 相关配置
### 限制 coredump 核心转储的大小

```
语法: worker_rlimit_core size;
```

### 指定 coredump 文件生成目录

```
语法: working_directory path;
```

## 指定 nginx worker 进程可以打开的最大句柄描述符

```
语法: worker_rlimit_nofile limit;
```

## 限制信号队列

```
语法: worker_rlimit_sigpending limit;
```

## 性能优化相关

### worker进程数

```
语法: worker_processes number;
默认: 1
```

### 绑定worker进程到指定的CPU内核

```
语法: worker_cpu_affinity cpumask [cpumask...]
```

### SSL硬件加速

```
语法: ssl_engine device;
```

### 系统调用`gettimeofday`的执行频率

```
语法: time_resolution t;
```

### worker进程优先级设置

```
语法: worker_priority nice;
默认: worker_priority 0;
```

## 事件类配置项

### 是否打开`accept`锁

```
语法: accept_mutex [on | off];
默认: accept_mutex on;
```

### lock文件路径

```
语法: lock_file path/file;
默认: lock_file logs/nginx.lock;
```

### 使用accept锁后到真正建立连接之间的延迟时间

```
语法: accept_mutex_delay Nms;
默认: accept_mutex_delay 500ms;
```

### 批量建立新连接

```
语法: multi_accept [on | off];
默认: multi_accept off;
```

### 选择事件模型

```
语法: use [kqueue | rtsig | epoll | /dev/poll | select | poll | eventport];
默认: nginx 会自动使用最适合的事件模型.
```

### 每个worker的最大连接数

```
语法: worker_connections number;
```

## 虚拟主机与请求分发

### 监听端口

```
语法: listen address:port [default(deprecated in 0.8.21) | default_server | [backlog=num] | rcvbuf=size | sndbuf=size | accept_filter=filter | deferred | bind | ipv6only=[on|off] | ssl];
默认: listen 80;
配置块: server
```

### 主机名

```
语法: server_name name [...];
配置块: server
```

如果`server_name`后跟着空字符串`"";`,那边表示匹配没有`Host`这个HTTP头部的请求.

### `server_name_hash_bucket_size`

```
语法: server_name_hash_bucket_size size;
默认: server_name_hash_bucket_size 32|64|128;
配置块: http, server, location
```

### `server_name_hash_max_size`

```
语法: server_name_hash_max_size size;
默认: server_name_hash_max_size 512;
配置块: http, server, location
```

### 重定向主机名称的处理

```
语法: server_name_in_redirect on | off;
默认: server_name_in_redirect on;
配置块: http, server 或者 location
```

### 文件路径的定义

```
语法: root path;
默认: root html;
配置块: http, server, location, if
```

### 以alias方式设置资源路径

```
语法: alias path;
配置块: location
```

alias 后面可以添加正则表达式.

### 访问首页

```
语法: index file ...;
默认: index index.html;
配置块: http, server, location
```

### 根据HTTP返回码重定向页面

```
语法: error_page code [code..] [= | =anser-code] uri | @named_location;
配置块: http, server, location, if

error_page 502 503 504 /50x.html;
error_page 404 =200 /empty.gif;
```

### 是否允许递归使用`error_page`

```
语法: recursive_error_pages [on | off];
默认: recursive_error_pages off;
配置块: http, server, location
```

### `try_files`

```
语法: try_files path1 [path2] uri;
配置块: server, location
```

## 内存及磁盘资源分配

### HTTP包体只存储到磁盘文件中

```
语法: client_body_in_file_only on | clean | off;
默认: client_body_in_file_only off;
配置块: http, server, location
```

### HTTP包体尽量写入到一个内存的buffer中

```
语法: client_body_in_single_buffer on | off;
默认: client_body_in_single_buffer off;
配置块: http, server, location
```

### 存储HTTP头部的内存buffer大小

```
语法: client_header_buffer_size size;
默认: client_header_buffer_size 1k;
配置块: http, server
```

### 存储超大HTTP头部的内存buffer大小

```
语法: large_client_header_buffers number size;
默认: large_client_header_buffers 4 8k;
配置块: http, server
```

### 存储HTTP包体的内存的buffer大小

```
语法: client_body_buffer_size size;
默认: client_body_buffer_size 8k/16k;
配置块: http, server, location
```

### HTTP包体的临时存放目录

```
语法: client_body_temp_path dir-path [level1 [level2 [level3]]];
默认: client_body_temp_path client_body_temp;
配置块: http, server, location
```

### `connection_pool_size`

```
语法: connection_pool_size size;
默认: connection_pool_size 256;
配置块: http, server
```

### `request_pool_size`

```
语法: request_pool_size size;
默认: request_pool_size 4k;
配置块: http, server
```

## 网络连接的设置

### 读取HTTP头部的超时时间

```
语法: client_header_timeout time(默认单位: 秒);
默认: client_header_timeout 60;
配置块: http, server, location

408 "Request timed out"
```

### 读取HTTP包体的超时时间

```
语法: client_body_timeout time;(默认单位: 秒)
默认: client_body_timeout 60;
配置块: http, server, location
```

### 发送响应的超时时间

```
语法: send_timeout time;
默认: send_timeout 60;
配置块: http, server, location
```

### `reset_timeout_connection`

```
语法: reset_timeout_connection on | off;
默认: reset_timeout_connection off;
配置块: http, server, location
连接超时后将通过向客户端发送RST包来直接重置连接.打开后nginx会在某个连接超时后,不是使用正常情形下的四次握手关闭TCP连接,而是直接向用户发送RST重置包,不再等待用户的应答,直接释放nginx服务器上关于这个套接字使用的所有缓存(如TCP滑动窗口),相比正常的关闭方式,它使得服务器避免产生许多处于FIN_WAIT1,FIN_WAIT2,TIME_WAIT状态的TCP连接.
注意,使用RST重置包关闭连接会带来一些问题,默认情况下不会开启.
```

### `lingering_close`

```
语法: lingering_close off | on | always;
默认: lingering_close on;
配置块: http, server, location
控制nginx关闭用户连接的方式.always表示关闭用户连接前必须无条件地处理连接上所有用户发送的数据.off表示关闭连接时完全不管连接上是否已经有准备就绪的来自用户数据.on是中间值,一般情况下在关闭连接前都会处理连接上的用户发送的数据,除了有些情况下业务上认定这之后的数据是不必要的.
```

### `lingering_time`

```
语法: lingering_time time;
默认: lingering_time 30s;
配置块: http, server, location
启用后对于上传大文件很有用.
当用户请求的Content-Lenght大client_max_body_size配置时,nginx服务会立刻向用户发送"413(Request entiry too large)"响应.但是很多客户端可能不管413返回值,仍然持续不断的上传HTTPbody,这时,经过了lingering_time设置的时间后,nginx将不管用户是否仍在上传,都会把连接关闭掉.
```

### `lingering_timeout`

```
语法: lingering_timeout time;
默认: lingering_timeout 5s;
配置块: http, server, location
lingering_close生效后,在关闭连接前,会检测是否有用户发送的数据到达服务器,如果超过lingering_timeout时间后还没有数据可读,就直接关闭连接;否则,必须在读取完连接缓冲区上的数据并丢弃后才会关闭连接.
```

### 对某些浏览器禁用`keepalive`功能

```
语法: keepalive_disable [msie6 | safari | none]...;
默认: keepalive_disable msie6 safari;
配置块: http, server, location
```

### keepalive超时时间

```
语法: keepalive_timeout time;(默认单位: 秒)
默认: keepalive_timeout 75;
配置块: http, server, location
```

### 一个keepalive长连接上允许承载的请求最大数

```
语法: keepalive_requests n;
默认: keepalive_requests 100;
配置块: http, server, location
```

### `tcp_nodelay`

```
语法: tcp_nodelay on | off;
默认: tcp_nodelay on;
配置块: http, server, location
确定对keepalive连接是否使用TCP_NODELAY选项
```

### `tcp_nopush`

```
语法: tcp_nopush on | off;
默认: tcp_nopush off;
配置块: http, server, location
在打开sendfile选项时,确定是否开启FreeBSD系统上的TCP_NOPUSH或Linux系统上的TCP_CORK功能.打开tcp_nopush后,将会在发送响应时把整个响应包头放到一个TCP包中发送.
```

## MIME类型设置

```
MIME type 与文件扩展的映射
语法: types {...}
配置块: http, server, location
例如:
types {
    text/html  html;
    text/html  conf;
    image/gif  gif;
    image/jpeg jpg;
}

默认MIME type
Syntax:  default_type mime-type;
Default: default_type text/plain;
Context: http, server, location

Syntax:  types_hash_bucket_size size;
Default: types_hash_bucket_size 64;
Context: http, server, location

Syntax:  types_hash_max_size size;
Default: types_hash_max_size 1024;
Context: http, server, location
```

## 文件操作优化

### sendfile系统调用

```
语法: sendfile on | off;
默认: sendfile off;
配置块: http, server, location
可以启用Linux上的sendfile系统调用来发送文件,它减少了内核态与用户态之间的两次内存复制,会从磁盘中读取文件后直接在内核发送到网卡设备,提高了发送文件的效率.
```

### AIO系统调用

```
语法: aio on | off;
默认: aio off;
配置块: http, server, location
此配置项表示是否在FreeBSD或Linux系统上启用内核级别的异步文件I/O功能.
注意: 它与sendfile功能是互斥的.
```

### directio

```
Syntax:  directio size | off;
Default: directio off;
Context: http, server, location
此配置项在FreeBSD和Linux系统上使用O_DIRECT选项去读取文件,缓冲区大小size,通常对大文件的读取速度有优化作用.
注意: 它与sendfile功能是互斥.
```

### `directio_alignment`

```
Syntax:  directio_alignment size;
Default: directio_alignment 512;
Context: http, server, location
它与directio配合使用,指定以directio方式读取文件时的对齐方式.一般情况下,512B已经足够了,但针对一些高情能的文件系统,如Linux下的XFS文件系统,可能需要设置到4KB作为对方方式.
```

### 打开文件缓存

```
语法: open_file_cache max=N [inactive=time] | off;
默认: open_file_cache off;
配置块: http, server, location
文件缓存会在内存中存储以下3种信息:
    文件句柄,文件大小和上次修改时间
    已经打开过的目录结构
    没有找到的或者没有权限操作的文件信息
```

### 是否缓存打开文件错误的信息

```
语法: open_file_cache_errors on | off;
默认: open_file_cache_errors off;
配置块: http, server, location
```

### 不被淘汰的最小访问次数

```
语法: open_file_cache_min_uses number;
默认: open_file_cache_min_uses 1;
配置块: http, server, location
```

### 检验缓存中元素的有效性的频率

```
语法: open_file_cache_valid time;
默认: open_file_cache_valid 60s;
配置块: http, server, location
```

## 对客户端特殊处理

```
# 忽略不合法的 HTTP 头部
ignore_invalid_headers on;
# HTTP 头部不允许下画线
underscores_in_headers off;
# 文件未找到时不记录error日志
log_not_found off;
# 合并相邻的"/"
merge_slashes on;
# 返回错误页面时不在server中注明nginx版本
server_tokens off;
```

# if 指令

```
-f, !-f 检查一个文件是否存在
-d, !-d 检查一个目录是否存在
-e, !-e 检查一个文件、目录、符号链接是否存在
-x, !-x 检查一个文件是否可执行
=, !=   比较的一个变量和字符串
~, ~*   与正则表达式匹配的变量，如果这个正则表达式中包含 }，; 则整个表达式需要用 " 或 ' 包围
```

# location 的匹配规则

```
语法: location [=|~|~*|^~|@] /uri/ {...}

1). = 表示把 URI 作为字符串,以便与参数中的 uri 做完全匹配.
2). ~ 表示匹配 URI 时字母大小写敏感.
3). ~* 表示匹配 URI 时忽略大小写.
4). ^~ 表示匹配 URI 时只需要其前半部分与 uri 参数匹配即可.
location ^~ /images/ {
    # 以 /images/ 开始的请求都会匹配上
    ...
}
5). @ 表示仅用于 nginx 服务内部请求之前的重定向,带有 @ 的 location 不直接处理用户请求.
```

# 对客户端的限制

## 按 HTTP 方法名限制用户请求

```
语法: limit_except METHOD ... {...}
配置块: location
nginx 通过 limit_except 后面指定的方法名来限制用户请求.方法名可取值包括: GET, HEAD, POST, PUT, DELETE, MKCOL, COPY, MOVE, OPTIONS, PROPFIND, PROPPATCH, LOCK, UNLOCK 或者 PATCH.

limit_except GET {
    allow 192.168.1.0/32;
    deny all;
}

注意,允许 GET 方法意味着也允许 HEAD 方法.上面的代码表示禁止 GET/HEAD 方法,但其他 HTTP 方法是允许的.
```

## HTTP请求包体的最大值

```
语法: client_max_body_size size;
默认: client_max_body_size 1m;
配置块: http, server, location
```

# 负载均衡的基本配置

```
(1) upstream 块
语法: upstream name {...}
配置块: http
upstream 定义一个上游服务器的集群,便于反向代理中的 proxy_pass 使用.例如:

upstream backend {
    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com;
}

server {
    location / {
        proxy_pass http://backend
    }
}

(2) server
语法: server name [parameters];
配置块: upstream
server 配置项指定了一台上游服务器的名字,可以是域名,IP 地址端口, UNIX 句柄等,其后可跟下列参数:
  weight=number:        设置这台服务器上游转发的权重,默认是 1;
  max_fails=number:     该选项与 fail_timeout 配合使用,指在 fail_timeout 时间内,如果向当前的上游服务器转发失败次数超过 number,则认为当前的 fail_timeout 时间内这台上游服务器不可用. max_fails 默认是 1,如果设置为 0,则表示不检查失败次数.
  fail_timeout=time:    表示该段时间内转发失败多少次后就认为上游服务器暂时不可用,用于优化反向代理功能.它与向上游服务器建立链接的超时时间,读取上流服务器的响应超时时间等完全无关. fail_timeout 默认为 10s.
  down:                 表示所在有上游服务器永久下线,只有使用了 ip_hash 配置项时才有用.
  backup:               在使用 ip_hash 配置项时它是无效的.它表示上游服务器只是个备份服务器,只有在所有的非备份上游服务器都失效后,才会向所在的上游服务器转发请求.
  例如:
upstream backend {
    server backend1.example.com weight=5;
    server 127.0.0.1:8080       max_fails=3 fail_timeout=30s;
    server unix:/tmp/backend3;
}

(3) ip_hash
语法: ip_hash;
配置块: upstream
将来自某一个用户的请求始终落到固定的一台上游服务器中.首先根据客户端的 ip 地址算出一个 key,将 key 按照 upstream 集群里的上游服务数量进行取模,然后以取模后的结果把请求转发到相应的上游服务器中.
ip_hash 与 weight(权重) 配置不可同时使用.如果 upstream 集群中有一台上游服务器暂时不可用,不能直接删除该配置,而是要用 down 参数标识,确保转发策略的一致性.
例如:

upstream backend {
    ip_hash;
    server backend1.example.com;
    server backend2.example.com down;
    server backend3.example.com;
}

(4) 记录日志时支持的变量
变量名                      意义
$upstream_addr              处理请求的上游服务器的地址
$upstream_cache_status      表示是否命中缓存,取值范围: MISS, EXPIRED, UPDATING, STALE, HIT
$upstream_status            上游服务器返回的响应中的 HTTP 响应码
$upstream_response_time     上游服务器的响应时间,精确到毫秒
$upstream_http_$HEADER      HTTP 的头部,如 upstream_http_host
```

# 源码阅读记录{#readSrc}

```
src/http/ngx_http_request.h 定义了 http 模块处理方法的返回值.

nginx 全局错误码
#define NGX_OK           0
#define NGX_ERROR       -1
#define NGX_AGAIN       -2
#define NGX_BUSY        -3
#define NGX_DONE        -4
#define NGX_DECLINED    -5
#define NGX_ABORT       -6
```

## 模块 ngx_http_handler_pt 方法返回值的意义

返回值 | 意义
------ | ---
NGX_OK | 如果在 nginx.conf 中配置了 satisfy all,那么将按照顺序执行下一个 ngx_http_handler_pt 处理方法;如果在 nginx.conf 中配置了 satisfy any,那么将执行下一个 ngx_http_phases 阶段中的第一个 ngx_http_handler_pt 处理方法.
NGX_DECLINED | 按照顺序执行下一个 ngx_http_handler_pt 处理方法
NGX_AGAIN <br /> NGX_DONE | 当前的 ngx_http_handler_pt 处理方法尚未结束,这意味着该处理方法在当前阶段中有机会再次被调用.这时会把控制权交还给事件模块,下次可写事件发生时会再次执行到该 ngx_http_handler_pt 处理方法.
NGX_HTTP_FORBIDDEN <br /> HGX_HTTP_UNAUTHORIZED | 如果在 nginx.conf 中配置了 satisfy any,那么将 ngx_http_request_t 中的 access_code 成员设为返回值,按照顺序执行下一个 ngx_http_handler_pt 处理方法;如果在 nginx.conf 中配置了 satisfy all,那么调用 ngx_http_finalize_request 结束请求.
NGX_ERROR <br /> 其他 | 需要调用 ngx_http_finalize_request 结束请求.

# ngx_log_error 日志接口 level 参数取值范围

级别名称 | 值 | 意义
-------- | -- | ---
NGX_LOG_STDERR  | 0 | 最高级别日志,日志的内容不会再写入 log 参数指定的文件,而是会直接将日志输出到标准错误设备,如控制台屏幕
NGX_LOG_EMERG   | 1 | 大于 NGX_LOG_ALERT 级别,而小于或等于 NGX_LOG_EMERG 级别的日志都会输出到 log 参数指定的文件中
NGX_LOG_ALERT   | 2 | 大于 NGX_LOG_CRIT 级别
NGX_LOG_CRIT    | 3 | 大于 NGX_LOG_ERR 级别
NGX_LOG_ERR     | 4 | 大于 NGX_LOG_WARN 级别
NGX_LOG_WARN    | 5 | 大于 NGX_LOG_NOTICE 级别
NGX_LOG_NOTICE  | 6 | 大于 NGX_LOG_INFO 级别
NGX_LOG_INFO    | 7 | 大于 NGX_LOG_DEBUG 级别
NGX_LOG_DEBUG   | 8 | 调试级别,最低级别日志

# ngx_log_debug 日志接口 level 参数的取值范围

级别名称 | 值 | 意义
-------- | -- | ---
NGX_LOG_DEBUG_CORE  | 0x010 | nginx 核心模块的调试日志
NGX_LOG_DEBUG_ALLOC | 0x020 | nginx 在分配内存时使用的调试日志
NGX_LOG_DEBUG_MUTEX | 0x040 | nginx 在使用进程锁时使用的调试日志
NGX_LOG_DEBUG_EVENT | 0x080 | nginx 事件模块的调试日志
NGX_LOG_DEBUG_HTTP  | 0x100 | nginx http 模块的调试日志
NGX_LOG_DEBUG_MAIL  | 0x200 | nginx 邮件模块的调试日志
NGX_LOG_DEBUG_MYSQL | 0x400 | nginx 表示与 MySQL 相关的 nginx 模块所使用的调试日志

# HTTP 框架为 11 个阶段实现的 checker 方法

阶段名称 | checker 方法
-------- | -----------
NGX_HTTP_POST_READ_PHASE | ngx_http_core_generic_phase
NGX_HTTP_SERVER_REWRITE_PHASE | ngx_http_core_rewrite_phase
NGX_HTTP_FIND_CONFIG_PHASE | ngx_http_core_find_config_phase
NGX_HTTP_REWRITE_PHASE | ngx_http_core_rewrite_phase
NGX_HTTP_POST_REWRITE_PHASE | ngx_http_core_post_rewrite_phase
NGX_HTTP_PREACCESS_PHASE | ngx_http_core_generic_phase
NGX_HTTP_ACCESS_PHASE | ngx_http_core_access_phase
NGX_HTTP_POST_ACCESS_PHASE | ngx_http_core_post_access_phase
NGX_HTTP_TRY_FILES_PHASE | ngx_http_core_try_files_phase
NGX_HTTP_CONTENT_PHASE | ngx_http_core_content_phase
NGX_HTTP_LOG_PHASE | ngx_http_core_generic_phase

## 链表容器

`src/core/ngx_list.h`

```
typedef struct ngx_list_part_s  ngx_list_part_t;

struct ngx_list_part_s {
    void             *elts;
    ngx_uint_t        nelts;
    ngx_list_part_t  *next;
};


typedef struct {
    ngx_list_part_t  *last;
    ngx_list_part_t   part;
    size_t            size;
    ngx_uint_t        nalloc;
    ngx_pool_t       *pool;
} ngx_list_t;
```

`ngx_list_t`描述整个链表,而`ngx_list_part_t`只描述链表的一个元素.每个链表元素`ngx_list_part_t`又是一个数组,拥有连续的内存,它既依赖于`ngx_list_t`里的`size`和`nalloc`来表示数组的容量,同时又依靠每个`ngx_list_part_t`成员中的`nelts`来表示数组当前已使用了多少容量.  `ngx_list_t`是一个链表容器,而链表中的元素又一个数组.事实上,`ngx_list_part_t`数组中的元素才是用户想要存储的东西,`ngx_list_t`链表能够容纳的元素数量由`ngx_list_part_t`数组元素的个数与每个数组所能容纳的元素相乘得到.

### 设计的优点

1. 链表中存储的元素是灵活的,它可以是任何一种数据结构
1. 链表元素需要占用的内存由`ngx_list_t`管理,它已经通过数组分配好了
1. 小块内存使用链表访问效率是低下的,使用用数组通过偏移量来直接访问内存则要高效的多


## 内存池

`src/core/ngx_palloc.h`

## `ngx_table_elt_t`hash表

`src/core/ngx_hash.h`

```
typedef struct {
    ngx_uint_t        hash;
    ngx_str_t         key;
    ngx_str_t         value;
    u_char           *lowcase_key;
} ngx_table_elt_t;
```

## `ngx_buf_t`

`src/core/ngx_buf.h`

```
typedef void *            ngx_buf_tag_t;

typedef struct ngx_buf_s  ngx_buf_t;

struct ngx_buf_s {
    u_char          *pos; 		/**  通常用来告诉使用者本次应该人 pos 这个位置开始处理内存中的数据,这样设置是因为同一个 ngx_buf_t 可能被多次反复处理.当然,pos 的含义是由使用它的模块定义的 */
    u_char          *last; 		/** 通常表示有效的内存到此为止.注意: pos 与 last 之间的内存是希望 nginx 处理的内容 */
    off_t            file_pos;	/** 处理文件时, file_pos 与 file_last 的含义与处理内存时的 pos 与 last 相同 */
    off_t            file_last;

    u_char          *start;         /* start of buffer 如果 ngx_buf_t 缓冲区用于内存,那么 start 指向这段内存的起始地址 */
    u_char          *end;           /* end of buffer 与 start 成员对应,指向缓冲区内存的末尾 */
    ngx_buf_tag_t    tag; 		/** 表示当前缓冲区的类型,例如由哪个模块使用就指向这个模块 ngx_module_t 变量的地址 */
    ngx_file_t      *file; 		/** 引用文件 */
    ngx_buf_t       *shadow;	/** 当前缓冲区的影子缓冲区 */


    /* the buf's content could be changed */
    unsigned         temporary:1;	/** 临时内存标志位,为 1 时表示数据内存中且这段内存可以修改 */

    /*
     * the buf's content is in a memory cache or in a read only memory
     * and must not be changed
     */
    unsigned         memory:1;	/** 标志位,为 1 时表示数据在内存中且这段内存不可以修改 */

    /* the buf's content is mmap()ed and must not be changed */
    unsigned         mmap:1;	/** 标志位,为 1 时表示这段内存是 mmap 系统调用映射过来的,不可以被修改 */

    unsigned         recycled:1;	/** 标志位,为 1 时表示可回收 */
    unsigned         in_file:1;		/** 标示位,为 1 时表示这段缓冲区处理的是文件而不是内存 */
    unsigned         flush:1;		/** 标志位,为 1 时表示需要执行 flush 操作 */
    unsigned         sync:1;		/** 标志位,对于操作这块缓冲区时是否使用同步方式,需谨慎考虑,这可能会阻塞nginx进程,nginx中所有操作几乎都是异步的,这是它支持高并发的关键.有些框架代码在sync为1时可能会有阻塞的方式进行I/O操作,它的意义视使用它的nginx模块而定 */
    unsigned         last_buf:1;	/** 标志位,表示是否是最后一块缓冲区,因为ngx_buf_t可以由ngx_chain_t链表串联起来,因此,当last_buf为1时,表示当前是最后一块待处理的缓冲区 */
    unsigned         last_in_chain:1;	/** 标志位,表示是否是ngx_chain_t中的最后一块缓冲区 */

    unsigned         last_shadow:1;		/** 标志位,表示是否是最后一个影子缓冲区,与shadow域配合使用 */
    unsigned         temp_file:1;		/** 标志位,表示当前缓冲区是否属于临时文件 */

    /* STUB */ int   num;
};
```

### `ngx_chain_t`数据结构

```
typedef struct ngx_chain_s           ngx_chain_t;

struct ngx_chain_s {
    ngx_buf_t    *buf;
    ngx_chain_t  *next;
};
```

## 处理方法返回值

`src/http/ngx_http_request.h`
