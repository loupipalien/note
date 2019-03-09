### HTTP 首部
#### HTTP 报文首部
```
-----------------
| 报文首部       | 客户端和服务器处理需要的信息
-----------------
| 空行 (CR+LF)  |
----------------
| 报文主体      | 用户需要的信息
|              |
----------------
```

#### HTTP 首部字段
##### HTTP 首部字段传递重要信息
HTTP 首部字段是 HTTP 报文的要素之一; 在客户端与服务器之间以 HTTP 协议进行通信的过程中, 无论是请求还是响应都会使用首部字段, 它起到传递额外重要信息的作用
##### HTTP 首部字段结构
HTTP 首部字段是由首部字段名和字段值构成的, 中间用冒号 ":" 分隔, 单个首部字段可以有多个值
###### 4 种 HTTP 首部字段
- 通用首部字段 (General Header Fields)
请求报文和响应报文双方都会使用的首部
- 请求首部字段 (Request Header Fields)
从客户端向服务器发送请求报文时使用的首部; 补充了请求的附加内容, 客户端信息, 响应内容相关优先级等信息
- 响应首部字段 (Response Header Fields)
从服务器向客户端发送请求报文时使用的首部; 补充了响应的附加内容, 也会要求客户端附加额外的内容信息
- 实体首部字段 (Entity Header Fields)
针对请求报文和响应报文的实体部分使用的首部; 补充了资源内容更新时间等与实体有关的信息

##### HTTP/1.1 首部字段一览
通用首部字段

| 首部字段名 | 说明 |
| :--- | :--- |
| Cache-Control | 控制缓存的行为 |
| Connection | 逐跳首部, 连接的管理 |
| Date | 创建报文的日期时间 |
| Pragma | 报文指令 |
| Trailer | 报文末端的首部一览 |
| Transfer-Encoding | 指定报文主体的传输编码方式 |
| Upgrade | 升级为其他协议 |
| Via | 代理服务器的相关信息 |
| Waring | 错误通知 |

请求首部字段

| 首部字段名 | 说明 |
| :--- | :--- |
| Accept | 用户代理可处理的媒体类型 |
| Accept-Charset | 优先的字符集 |
| Accept-Encoding | 优先的内容编码 |
| Accept-Language | 优先的语言 |
| Authorization | Web 认证信息 |
| Expect | 期待服务器的特定行为 |
| From | 用户的电子邮箱地址 |
| Host | 请求资源所在服务器 |
| If-Match | 比较实体标记 (Etag) |
| If-Modified-Since | 比较资源的更新时间 |
| If-None-Match | 比较实体标记 (与 If-Match 相反) |
| If-Range | 资源为更新时发送实体 Byte 的范围请求 |
| If-Unmodified-Since | 比较资源的更新时间 (与 If-Modified-Since 相反) |
| Max-Forwards | 最大传输逐跳数 |
| Proxy-Authorization | 代理服务器要求客户端的认证信息 |
| Range | 实体的字节范围请求 |
| Referer | 对请求中 URI 的原始获取方 |
| TE | 传输编码的优先级 |
| User-Agent | HTTP 客户端程序的信息 |

响应首部字段

| 首部字段名 | 说明 |
| :--- | :--- |
| Accept-Ranges | 是否接受字节范围请求 |
| Age | 推算资源创建经过时间 |
| ETag | 资源的匹配信息 |
| Location | 令客户端重定向至指定 URI |
| Proxy-Authenticate | 代理服务器对客户端的认证信息 |
| Retry-After | 对再次发起请求的时机要求 |
| Server | HTTP 服务器的安装信息 |
| Vary | 代理服务器缓存的管理信息 |
| WWW-Authenticate | 服务器对客户端的认证信息 |

实体首部字段

| 首部字段名 | 说明 |
| :--- | :--- |
| Allow | 资源可支持的 HTTP 方法 |
| Content-Encoding | 实体主体使用的编码方式 |
| Content-Language | 实体主体的自然语言 |
| Content-Length | 实体主体的大小 (字节) |
| Content-Location | 替代对应资源的 URI |
| Content-MD5 | 实体主体的报文摘要 |
| Content-Type | 实体主体的媒体类型 |
| Expires | 实体主体过期的日期时间 |
| Last-Modified | 资源的最后修改日期时间 |

##### 非 HTTP/1.1 首部字段
HTTP 通信协议中使用的首部字段不限于 RFC2616 中的 47 种, 还有 RFC4229 中的非正式首部字段

##### End-to-end 首部和 Hop-by-hop 首部
HTTP 首部字段将定义成缓存代理和非缓存代理的行为, 分为两种类型
