# app api 接口设计

## 客户端

### 必传参数

参数名 | 参数类型 | 说明
------ | -------- | ---
version | int | api版本号
platform | string | 取值为: pc, h5, iphone, ipad, android, other 之一
token | string(32) | 用户需要持token才能访问受保护的数据,没有持有时请传空`""`
`app_id` | string | app 维护的唯一 id
rtime | bigint | 用户发起请求的时间戳

### 发送请求加密算法

1. 动态生成`AES`加解密需要的16位(ASCII码)`key`和`iv`
1. 将所有待传送参数放入字典中
1. 将字典生成为一个标准`JSON`字符串
1. 使用`AES`的`key`和`iv`,采用`AES-256-CBC`对`JSON`字符串加密,并进行`base64`编码
1. 将生成的密文串赋值给`data`
1. 将`key`和`iv`使用`$$$$`连接起来,然后使用`RSA`公钥加密,并进行`base64`编码
1. 将生成的密文串赋值给`signature`
1. 通过`HTTP POST`方式将报文(data=AES密文&signature=RSA密文)发送到服务端

### 接收响应解密算法

1. 解析服务端的`JSON`数据,如果`code`不等于`0`,则异常处理
1. 将`signature`字段的内容`base64`解码后使用`RSA`公钥解密,得到服务端`AES`密钥串,以`$$$$`切开得`key`,`iv`
1. 将`data`字段内容`base64`解码后使用`AES`的`key`,`iv`,采用`AES-256-CBC`解密,得到服务器端有效的响应结果集

## 服务端

### 接收请求解密算法

1. 将`signature`字段的内容`base64`解码后使用`RSA`私钥解密,得到客户端`AES`密钥报文
1. 将客户端`AES`密钥报文用`$$$$`切开,得到客户端`AES`密钥串的`key`和`iv`
1. 将`data`字段内容`base64`解码后使用`AES`的`key`,`iv`,采用`AES-256-CBC`解密,得到客户端请求参数和值
1. 解析原始报文(`JSON`字符串)

### 响应请求加密算法

1. 动态生成服务器端`AES`加解密需要的16位(ASCII码)`key`和`iv`
1. 将业务结果集放入一个字典
1. 将字典生成为一个标准`JSON`字符串
1. 使用`AES`的`key`和`iv`,采用`AES-256-CBC`对`JSON`字符串加密,并进行`base64`编码
1. 将生成的密文串赋值给响应结构体的`data`
1. 将`key`和`iv`使用`$$$$`连接起来,然后使用`RSA`私钥加密,并进行`base64`编码
1. 将生成的密文串赋值给响应结构体的`signature`
1. 将响应结构体组织成`JSON`数据,发送给客户端

### 响应请求数据结构说明

- 所有接口返回JSON,包含5个基本元素: `code`, `message`, `signature`, `data`, `server_time`
- 仅当`code`等于`0`,说明接口响应正确,其他值均为出错,请接口调用方根据业务自行处理后续
- `message`为出错时错误提示,可直接用于向用户展示出错信息
- `signature`: 服务端私钥加密后的密文,异常是为空字符串
- `data`服务端使`AES`算法加密后的密文,异常是为空字符串
- `server_time`服务端响应请求的unix时间戳

