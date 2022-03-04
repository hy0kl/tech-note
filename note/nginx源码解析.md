# 源码阅读记录

## configure 中默认不会编译到 nginx 中的 HTTP 模块参数

可选的 HTTP  模块 | 意义
------ | ---
--with-http_ssl_module | 安装 http ssl module.该模块使 nginx 支持 SSL 协议,提供 HTTPS 服务.<br />注意: 该模块的安装依赖于 OpenSSL  开源软件,即首先确保已经在之前的参数中配置了 OpenSSL
--with-http_realip_module | 安装 http realip module.该模块可以从客户端请求里的 header 信息(如 X-Real-IP 或者 X-Forwarded-For)中获取真正的客户端 IP 地址
--with-http_addition_module | 安装 http addtion module.该模块可以在返回客户端的 HTTP 包体头部或尾部增加内容
--with-http_xslt_module | 安装 http xslt module.这个模块可以使XML格式的数据在发给客户端前加入 XSL 渲染<br />注意: 这个模块依赖于 libxml2 和 libxslt 库,安装它前首先确保上述两个软件已经安装
--with-http_image_filter_module | 安装 http image filter module.这个模块将符合配置的图片实时压缩为指定大小(width * height)的缩略图再发送给用户,目前支持 JPEG,PNG,GIF 格式.<br />注意: 这个模块依赖于开源的 libgd 库.
--with-http_geoip_module | 安装 http geoip module.该模块可以依据 MaxMind GeoIP 的 IP 地址数据对客户端的 IP 地址得到实际的地理位置信息.<br />注意: 这个模块依赖于 MaxMind GeoIP 的库文件,访问 https://www.maxmind.com/en/accounts/current/geoip/downloads 获取.
--with-http_sub_module | 安装 htt sub module.该模块可以在 nginx 返回客户的 HTTP 响应包中将指定的字符串替换为自己需要的字符串.
--with-http_dav_module | 安装 http dav module.这个模块可以让 nginx 支持 Webdav 标准,如支持 Webdav 协议中的 PUT, DELETE, COPY, MOVE, MKCOL 等请求
--with-http_flv_module | 安装 http flv module.这个模块可以在向客户端返回响应时,对 FLV 格式的视频文件在 header 头做一些处理,使得客户端可以观看,拖动 FLV 视频
--with-http_mp4_module | 安装 http mp4 module.该模块使客户端可以观看,拖动 MP4 视频
--with-http_gzip_static_module | 安装 http gzip static module.如果采用 gzip 模块把一些文档进行 gzip 格式压缩后再返回给客户端,那么对同一个文件每次都会重新压缩,这是比较消耗服务器 CPU 资源的. gzip static 模块可以在做 gzip 压缩前,先查看相同位置是否已经做过 gzip 压缩的 .gz 文件,如果有,就直接返回.这样可以预先在服务器上做好文件的压缩,给 CPU 减负
--with-http_random_index_module | 安装 http random index module.该模块在客户端访问某个目录时,随机返回该目录下的任意文件
--with-http_secure_link_module | 安装 http secure link module.该模块提供一种验证请求是否有效的机制.例如,它会验证 URL 中需要加入的 token 参数是否属于特定客户端发发来的,以及检查时间戳是否过期
--with-http_degradation_module | 安装 http degradation module.该模块针对一些特殊的系统调用(如 sbrk)做一些优化,如直接返回 HTTP 响应码为204或者444.目前不支持 Linux 系统
--with-http_stub_status_module | 安装 http stub status module.该模块可以让运行中的 nginx 提供性能统计页面,获取相关的并发连接,请求的信息
--with-google_perftools_module | 安装 google perftools module.该模块提供 google 的性能测试工具

`ngx_modules.c`是一个关键文件,用来定义`ngx_modules`数组,它指明了每个模块在nginx中的优先级,当一个请求同时符合多个模块的处理规则时,将按照它们在`ngx_modules`数组中顺序选择最靠前的模块优先处理.对于 HTTP 过滤模块而言则是相反的,因为 HTTP 框架在初始化时,会在`ngx_modules`数组中将过滤模块按先后顺序向过滤链表中添加,但每次都是添加到链表的表头,因此,对 HTTP 过滤模块而言,在`ngx_modules`数组中越是靠后的模块反而会首先处理 HTTP 响应.


# nginx 配置项相关

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

## nginx 服务的基本配置

### 用于调试进程和定位问题的配置项

1) 是否以守护进程方式运行nginx

```
语法: daemon on | off;
默认: daemon on;
```

2) 是否以 master/worker 方式工作

```
语法: master_process on | off;
默认: master_process on;
```

3) error 日志设置

```
语法: error_log /path/file level;
默认: error_log logs/error.log error;
```

level是日志的输出级别,取值范围是 debug,info,notice,warn,error,crit,alert,emerg,从左至右级别依次增大.当设定一个级别时,大于或等于该级别的日志都会被输出到`/path/file`文件中,小于该级别的日志则不会输出.

注意: 如果日志级别设定到debug,必须在 configure 时加入 --with-debug 配置项.

4) 是否处理几个特殊的调试点

```
语法: debug_points [stop | abort]
```

nginx在一些关键的错误逻辑中设置了调试点.如果设置了`debug_points`为 stop,那么 nginx 的代码执行到这些调试点时会发出`SIGSTOP`信号以用于调试.如果`debug_points`设置为 abort,则会产生一个 coredump 文件.

5) 仅对指定的客户端输出 debug 级别日志

```
语法: debug_connection [IP | CIDR]
```

此配置项必须放在 events {...} 中才有效.

6) 限制 coredump 核心转储文件的大小

```
语法: worker_rlimit_core size;
```

7) 指定 coredump 文件生成目录

```
语法: working_directory path;
```

### 正常运行的配置项

1) 定义环境变量

```
语法: env VAR=VALUE
```

2) 嵌入其他配置文件

```
语法: include /path/file;
```

参数既可以是绝对路径,也可以是相对路径(相对于 nginx 的配置目录,即 nginx.conf 所在的目录).参数的值可以是一个明确的文件名,也可以是通配置符 * 的文件名,同时可以一次嵌入多个配置文件.

3) pid 文件的路径

```
语法: pid path/file;
默认: pid logs/nginx.pid;
```

4) nginx worker 进程运行的用户及用户组

```
语法: user username [groupname];
默认: user nobody nobody;
```

若用户在 configure 命令执行时使用了参数 --user=username 和 --group=groupname,些时 nginx.conf 将使用参数中指定的用户和用户组.

5) 指定 nginx worker 进程可以打开的最大句柄描述符个数

```
语法: worker_rlimit_nofile limit;
```


`src/http/ngx_http_request.h`定义了 http 模块处理方法的返回值.

```
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
-------- | --- | ---
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
-------- | --- | ---
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

```c
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

```c
typedef struct {
    ngx_uint_t        hash;
    ngx_str_t         key;
    ngx_str_t         value;
    u_char           *lowcase_key;
} ngx_table_elt_t;
```

## `ngx_buf_t`

`src/core/ngx_buf.h`

```c
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

```c
typedef struct ngx_chain_s           ngx_chain_t;

struct ngx_chain_s {
    ngx_buf_t    *buf;
    ngx_chain_t  *next;
};
```

## 处理方法返回值

`src/http/ngx_http_request.h`
