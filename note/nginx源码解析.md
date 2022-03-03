# 源码阅读记录

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

# 关于nginx中host, server_name, http_host的区别

[see](http://schin.space/nginx/NGINX-%E5%85%B3%E4%BA%8Enginx%E4%B8%AD$host-$server_name-$http_host%E7%9A%84%E5%8C%BA%E5%88%AB/)

```
总结：$server_name 是一个directive,用来匹配请求的host头，$http_host与$host表示请求的host头，区别在于$http_host会带有端口号(非80/443)

/$server_name

Server names are defined using the server_name directive and determine which server block is used for a given request

server_name 是nginx配置文件中的一个指令（directive), 支持通配符，用来匹配请求到来时，该用哪个server block来处理请求

$host

in this order of precedence: host name from the request line, or host name from the “Host” request header field, or the server name matching a request

$host是nginx配置文件中的一个变量，其值按如下顺序来确定，如果request line (写过socket的同学应该了解建立TCP连接后会发送类似’GET / HTTP 1.1’类似的就是请求行)中有的话就是这个值，否则从请求头中的Host值获取(http/1.0中，http_host有可能为空哦，http/1.1，host必须存在），否则从匹配到server_name（server_name有可能是*.xxxx.com这种呢，如果用这种值proxy_pass到某个域名的话，会报400哦)来确定

$http_host

nginx官方并没有对其的解释, 不过测试发现$host 与 $http_host的区别在于当使用非80/443端口的时候，$http_host = $host:$port

另外在做反向代理的时候，RS（real server)有时需要知道请求的host头，此时需要用proxy_set_header host $host; 指令来对转发请求增加一个请求头，否则后端收到的host会是""
```
