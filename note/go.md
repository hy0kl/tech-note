# note

- panic, recover 参数类型为 interface{}, 因此可以抛出任何类型对象.panic 是一个内建函数，可以中断原有的控制流程，进入一个令人恐慌的流程中。当函数F调用panic，函数F的执行被中断，但是F中的延迟函数会正常执行，然后F返回到调用它的地方。在调用的地方，F的行为就像调用了panic。这一过程继续向上，直到发生panic的goroutine中所有调用的函数返回，此时程序退出。恐慌可以直接调用panic产生。也可以由运行时错误产生，例如访问越界的数组。recover 是一个内建的函数，可以让进入令人恐慌的流程中的goroutine恢复过来。recover仅在延迟函数中有效。在正常的执行过程中，调用recover会返回nil，并且没有其它任何效果。如果当前的goroutine陷入恐慌，调用recover可以捕获到panic的输入值，并且恢复正常的执行。
- 惯例: 在包内部使用 panic, 对外 API 使用 error 返回值.
- 值拷贝行为会造成性能问题,通常建议使用 slice,或数组指针.
- 内置函数 len() 和 cap() 都返回数组长度(元素数量).
- 从 map 中取回一个 value 是临时复制品,对其成员的修改是没有任何意义的.
- slice是引用类型，所以当引用改变其中元素的值时，其它的所有引用都会改变该值
  - `len`获取`slice`的长度
  - `cap`获取`slice`的最大容量
  - `append`向`slice`里面追加一个或者多个元素,然后返回一个和`slice`一样类型的`slice`
  - `copy`函数`copy`从源`slice`的`src`中复制元素到目标`dst`,并且返回复制的元素个数
- `new(T)`分配了零值填充的`T`类型的内存空间,并且返回其址,即一个`*T`类型的值.用Go的术语说,它返回了一个指针,指向新分配的类型`T`的零值.有一点非常重要:`new`返回指针.
- make只能创建slice、map和channel，并且返回一个有初始值(非零)的T类型，而不是`*T`。
- method的语法 `func (r ReceiverType) funcName(parameters) (results)`. Receiver 还可以是指针, 两者的差别在于, 指针作为 Receiver 会对实例对象的内容发生操作,而普通类型作为 Receiver 仅仅是以副本作为操作对象,并不对原实例对象发生操作。
- 名字开头字母大小写决定了名字在包外的可见性.如果一个名字是大写字母开头(必须是在函数外部定义的包级名字;包级函数名本身也是包级名字),那么它将被导出.
- 对于每个类型T,都有一个对应的类型转换操作T(x),用于将x转为T类型.
- 只有常量查以是无类型的
- 如果结构体的全部成员都是可以比较的,那么结构体也是可以比较的,那样的话两个结构体将可以使用`==`或`!=`运算符进行比较.相等比较运算符`==`将比较两个结构体的每个成员.
- 可比较的结构体类型和其他可比较的类型一样,可以用于map的key类型
- 匿名成员: 声明一个成员对应的数据类型而不指定成员的名字.匿名成员的数据类型必须是命名的类型或指向一个命名的类型的指针.
- `Printf`函数中`%v`参数包含的`#`副词,它表示用和Go语言类似的语法打印值.对于结构体类型来说,将包含每个成员的名字.
- 如果一个函数将所有返回值都显示的命名,那么该函数的`return`语句可以省略操作数,称之为`bare return`.
- `fmt.Errorf`定制出错信息.
- 对于一个给定的类型,其内部方法都必须有唯一的方法名.
- 当调用一个函数时,会对其每个参数值进行拷贝,如果一个函数需要更新一个变量,或者函数的其中一个参数实在太大而希望能够避免进行默认的拷贝,需要用到指针.对应到用来更新接收器的对象的方法,当接收者变量本身比较大时,可以用指针而不是对象来声明方法.
- `Nil`也是一个合法的接收器类型
- 当T是一个类型时,访求表达式可能会写`T.f`或者`(*T).f`,会返回一个函数"值",这种函数会将其第一个参数用作接收器,所以可用通常(译注:不写选择器)的方式来其进行调用.
- `T`类型的值不拥有所有`*T`指针的方法,它可能只实现更少的接口.
- 接口类型封装和隐藏具体类型和它的值.
- 空接口类型对实现它的类型没有要求,所以可以将任意一个值赋给空接口类型.
- 接口值可以使用`==`和`!=`来进行比较.两个接口值相等仅当它们都是`nil`值或者它们的动态类型相同并且动态值也根据这个动态类型的`==`操作相等.因为接口值是可比较的,所以它们可以用在map的键或者作为switch语句的操作数.
- 概念上讲一个接口的值,接口值,由两个部分组成,一个具体的类型和那个类型的值.被称为接口的动态类型和动态值.
- 如果两个接口值的动态类型相同,但是动态类型是不可比较的(比如切片),将它们进行比较就会失败并且`panic`
- 一个包含`nil`指针的接口不是`nil`接口
- 对于接口设计的一个好的标准是`ask only for what you need`(只考虑你需要的东西).
- "顺序通信进程"(communicating sequential processes),被简称为CSP.CSP是一种现代的并发编程模型,在这种编程模型中值会在不同的运行实例(goroutine)中传递,尽管大多数情况下仍然是被限制在单一实例中.
- `channel`是一个对应`make`创建的底层数据结构的引用.两个相同类型`channel`可以使用`==`运算比较,如果两个引用是相同的,比较结果为真,也可以和`nil`进行比较.
- 如果`channel`的容量大于零,那么该`channel`是带缓冲的`channel`.
- 当一个`channel`关闭后,再向该`channel`发送数据将导致`panic`异常.当一个被关闭的`channel`中已经发送的数据被成功接收后,后续的操作将不再阻塞,它们会立即返回一个零值.
- 试图重复关闭一个`channel`将导致`panic`异常,试图关闭一个`nil`值的`channel`也将导致`panic`异常.
- 单向`channel`,类型`chan<- T`表示一个只发送`T`的`channel`,只能发送不能接收;类型`<-chan T`表示一个只接收`T`的`channel`,只能接收不能发送.
- `cap`函数获取`channel`内部缓存的容量,`len`函数将返回`channel`内部缓存队列中有效的元素个数.
- 多个`goroutines`并发的向一个`channel`发送数据,或从同一个`channel`接收数据都是常见的用法.
- 无缓存`channel`更强地保证了每个发送操作与相应的同步接收操作;对于带缓存的`channel`,发送和接收是解耦的.
- 不要使用共享数据来通信;使用通信来共享数据
- 避免数据竞争:
  - 不要去写变量
  - 避免从多个`goroutine`访问变量
  - 互斥
    - 二元信号量(binary semaphore)
    - `sync.Mutex`互斥锁
    - `sync.RWMutex`读写锁
    - `sync.Once`初始化

## 包和命名

- 当创建一个包,一般要用短小的包名,但也不能太短导致难以理解
- 包名一般采用单数形式
- 要避免包名有其他含义
- 当设计一个包的时候,需要考虑包名和成员名两个部分如何很好地配合
- 专门用于保存包文档的源文件通常叫`doc.go`
- Go语言的构建工具对包含`internal`名字的路径段的包导入路径做了特殊处理.这种包叫`internal`包,一个`internal`包只能被和`internal`目录同一个父目录的包所导入.

## 测试

### go test

`got test`命令是一个按照一定的约定和组织的测试代码的驱动程序.在包目录内,所有以`_test.go`为后缀名的源文件并不是`go build`构建包的一部分,它们是`go test`测试的一部分.

在`*_test.go`文件中,有三种类型的函数:测试函数,基准测试函数,示例函数.

- 以`Test`为函数名前缀的函数,用于测试程序的一些逻辑行为是否正确
- 基准测试函数是以`Benchmark`为函数名前缀的函数,它们用于衡量一些函数的性能
- 示例函数是以`Example`为函数名前缀的函数,提供一个由编译器保证正确性的示例文档
- 每个测试函数必须导入`testing`包

### 测试函数

- 随机测试
- 白盒测试
- 扩展测试包
- 编写有效的测试
- 避免不稳定的测试

### 测试覆盖率

### 基准测试

## 反射

`reflect.Type`和`reflect.Value`

反射是由`reflect`包提供支持,它定义了两个重要的类型,`Type`和`Value`.一个`Type`表示一个Go类型,它是一个接口,有许多方法来区分类型和检查它们的组件.

函数`reflect.TypeOf`接受任意的`interface{}`类型,并返回对应动态类型的`reflect.Type`

一个`reflect.Value`可以持有一个任意类型的值.函数`reflect.ValueOf`接受任意的`interface{}`类型,并返回对应动态类型的`reflect.Value`.

## [golang下划线(underscore)的意义](https://www.jianshu.com/p/309f55a152db)

### 用在import

```
import  _  "net/http/pprof"
```

引入包，会先调用包中的初始化函数，这种使用方式仅让导入的包做初始化，而不使用包中其他功能

### 用在返回值

```
for _, v := range Slice{}
_, err := funcCall()
```

表示忽略某个值。单函数有多个返回值，用来获取某个特定的值

### 用在变量

```
type T struct{}
var _ I = T{}

// 其中I为interface
```

用来判断`type T`是否实现了`I`,用作类型断言，如果`T`没有实现接口`I`，则编译错误

### 用在函数定义返回值

```
func Parse(base *PosBase, src io.Reader, errh ErrorHandler, pragh PragmaHandler, mode Mode) (_ *File, first error) {
	var p parser
	p.init(base, src, errh, pragh, mode)
	p.next()
	return p.fileOrNil(), p.first
}
```

占位使用,减少变量声明.

# 常用库

- strings 提供了许多如字符串的查找,替换,比较,截断,拆分,和合并功能
- bytes 针对和字符串有着相同结构的[]byte类型
- unicode 提供了IsDigit,IsLetter,IsUpper,和IsLower等功能
- strconv 提供了包括布尔型,整型数,浮点型数和对应字符串的相互转换,还提供双引号转义相关的转换
- `math/big` 精密计算
- regexp 正则表达式
- fmt 格式化
- `net/http` 网络包
- `html/template`
  - func HTMLEscape(w io.Writer, b []byte) //把b进行转义之后写到w
  - func HTMLEscapeString(s string) string //转义s之后返回结果字符串
  - func HTMLEscaper(args ...interface{}) string //支持多个参数一起转义，返回结果字符串

## JSON

- `encoding/json`
- 将结构体slice转为JSON的过程叫编组(marshaling).编组通过调用`json.Marshal`函数完成
- `json.MarshalIndent`函数将产生整齐缩进的输出
- 一个结构体成员Tag是和在编译阶段关联到该成员的元信息字符串,可以是任意的字符串面值,通常是一系列用空格分隔的key:"value"键值对序列;因为值中含有双引号字符,因此成员Tag一般用原生字符串字面值的形式书写.
- 编码的逆操作是解码,对应将JSON数据解码为Go语言的数据结构,Go语中一般叫unmarshaling,通过`json.Unmarshal`函数完成.

- 流式解码器`json.Decoder`,流式编码器`json.Encoder`.

# Go语言设计与实现

## 编译原理

抽象语法树（Abstract Syntax Tree、AST）
静态单赋值（Static Single Assignment、SSA）是中间代码的特性

1. 词法与语法分析
1. 类型检查
    1. 常量、类型和函数名及类型；
    1. 变量的赋值和初始化；
    1. 函数和闭包的主体；
    1. 哈希键值对的类型；
    1. 导入函数体；
    1. 外部的声明；
1. 中间代码生成
1. 机器码生成

### 编译器入口

1. 检查常量、类型和函数的类型；
1. 处理变量的赋值；
1. 对函数的主体进行类型检查；
1. 决定如何捕获变量；
1. 检查内联函数的类型；
1. 进行逃逸分析；
1. 将闭包的主体转换成引用的捕获变量；
1. 编译顶层函数；
1. 检查外部依赖的声明；

有限自动机（Deterministic Finite Automaton、DFA）
Go 语言的词法解析是通过 src/cmd/compile/internal/syntax/scanner.go6 文件中的 cmd/compile/internal/syntax.scanner 结构体实现的
顶层声明有五大类型，分别是常量、类型、变量、函数和方法
Go 语言的解析器使用了 LALR(1) 的文法来解析词法分析过程中输出的 Token 序列20，最右推导加向前查看构成了 Go 语言解析器的最基本原理
词法分析器 cmd/compile/internal/syntax.scanner 作为结构体被嵌入到了 cmd/compile/internal/syntax.parser 中，所以这个方法中的 p.next() 实际上调用的是 cmd/compile/internal/syntax.scanner.next 方法，它会直接获取文件中的下一个 Token，所以词法和语法分析一起进行的。
复杂指令集（CISC）和精简指令集（RISC）
数组是否应该在堆栈中初始化是在编译期就确定了.

### 常用关键字

panic 只会触发当前 Goroutine 的延迟函数调用

## 反射三大法则

1. 从 interface{} 变量可以反射出反射对象；
1. 从反射对象可以获取 interface{} 变量；
1. 要修改反射对象，其值必须可设置；

## 运行时

### GMP模型

- G — 表示 Goroutine，它是一个待执行的任务；
- M — 表示操作系统的线程，它由操作系统的调度器调度和管理；
- P — 表示处理器，它可以被看做运行在线程上的本地调度器；

---

# golang 项目收集

# go gitlab
https://gogs.io/
https://github.com/gogits

# goim
https://github.com/Terry-Mao/goim

# docker
https://github.com/docker/docker

# 集群管理
https://github.com/kubernetes/kubernetes

# 基于 go 语言的分布式实时消息平台
https://github.com/bitly/nsq

# go logger
https://github.com/uber-go/zap
https://github.com/sirupsen/logrus
https://github.com/cihub/seelog
https://github.com/alecthomas/log4go

# go redis
https://github.com/garyburd/redigo
https://github.com/go-redis/redis

# go HTTP web framework
https://github.com/gin-gonic/gin
https://github.com/astaxie/beego

# golang protobuf
https://github.com/golang/protobuf

# JSON
https://github.com/bitly/go-simplejson

# golang WebSocket
https://github.com/gorilla/websocket

# Server-Sent Events implementation in Go
https://github.com/manucorporat/sse

# SQL
https://github.com/go-sql-driver/mysql
https://github.com/lib/pq
https://github.com/go-pg/pg

# curl 实用性不强,内核http包完全够用
https://github.com/golang-basic/go-curl
https://github.com/andelf/go-curl
https://github.com/mikemintang/go-curl
https://github.com/nareix/curl

# setproctitle() for Go
https://github.com/erikdubbelboer/gspt

# spider
https://github.com/henrylee2cn/pholcus
https://github.com/antchfx/antch
https://github.com/gocolly/colly
https://github.com/PuerkitoBio/gocrawl
https://github.com/PuerkitoBio/goquery

# ES client
https://github.com/olivere/elastic

# git-lfs
https://git-lfs.github.com/
https://github.com/git-lfs/git-lfs

# Modern SSH server for clusters and teams.
https://github.com/gravitational/teleport

# 原生数据库
https://github.com/cockroachdb/cockroach

# proxy
https://github.com/snail007/goproxy

# 标准库学习
https://github.com/polaris1119/The-Golang-Standard-Library-by-Example

# 代码分析
https://github.com/360EntSecGroup-Skylar/goreporter
https://github.com/alecthomas/gometalinter

# 图像处理
https://github.com/disintegration/imaging

# go 圣经
https://yar999.gitbooks.io/gopl-zh/content/
https://github.com/adonovan/gopl.io

# gopkg 参考例子
https://github.com/astaxie/gopkg

# 解析配置文件
https://github.com/go-ini/ini
https://github.com/go-gcfg/gcfg
https://github.com/kylelemons/go-gypsy
https://github.com/go-yaml/yaml
https://github.com/ghodss/yaml
https://github.com/micro/go-config
https://github.com/uber-go/config
https://github.com/emitter-io/config
https://github.com/zpatrick/go-config
https://github.com/astaxie/beego
https://github.com/larspensjo/config

# shadowsocks
https://github.com/shadowsocks/shadowsocks-go
https://github.com/shadowsocks/go-shadowsocks2 https://github.com/riobard/go-shadowsocks2

# 数据库管理
https://github.com/go-xorm/dbweb

# 热更新
https://github.com/fvbock/endless
