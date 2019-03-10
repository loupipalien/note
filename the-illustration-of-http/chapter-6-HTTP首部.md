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
- 端到端首部 (End-to-end Header)
此类别中的首部会转发给请求/响应对应的最终接收目标, 且必须保存在由缓存生成的响应中, 另外规定它必须被转发
- 逐跳首部 (Hop-by-hop Header)
此类别中的首部只对单次转发有效, 会因通过缓存或代理而不再转发; HTTP/1.1 及以后版本如需使用 Hop-by-hop 首部则需提供 Connection 首部字段

除了以下 8 个逐跳首部字段, 其余所有字段都属于端到端首部
- Connection
- Keep-alive
- Proxy-Authenticate
- Proxy-Authorization
- Trailer
- TE
- Transfer-Encoding
- Upgrade

#### HTTP/1.1 通用首部字段
通用首部字段指请求和响应报文都会使用的首部

##### Cache-Control
通过指定首部字段 Cache-Control 的指令, 就能操作缓存的工作机制; 指令的参数是可选的, 多个指令参数之间通过逗号分隔
```
Cache-Control: private, max-age = 0, no-cache
```
###### Cache-Control 指令一览
- 缓存请求指令

| 指令 | 参数 | 说明 |
| :--- | :--- | :--- |
| no-cache | 无 | 强制向源服务器再次验证 |
| no-store | 无 | 不缓存请求或响应的任何内容 |
| max-age = [秒] | 必需 | 响应的最大 Age 值 |
| max-stale = ([秒]) | 可省略 | 接收已过期的响应 |
| min-fresh = [秒] | 必需 | 期望在指定时间内的响应仍有效 |
| no-transform | 无 | 代理不可更改媒体类型 |
| only-if-cached | 无 | 从缓存获取资源 |
| cach-extension | - | 新指令标记 (token) |

- 缓存响应指令

| 指令 | 参数 | 说明 |
| :--- | :--- | :--- |
| public | 无 | 可向任意方提供响应的缓存 |
| private  | 可省略 | 仅向特定用户返回响应 |
| no-cache | 无 | 强制向源服务器再次验证 |
| no-store | 无 | 不缓存请求或响应的任何内容 |
| no-transform | 无 | 代理不可更改媒体类型 |
| must-revalidate | 无 | 可缓存但必须再向服务器进行确认 |
| proxy-revalidae | 无 | 要求中间缓存服务器对缓存的响应有效性在进行确认 |
| max-age = [秒] | 必需 | 响应的最大 Age 值 |
| s-maxage = [秒] | 必需 | 公共缓存服务器响应的最大 Age 值 |
| cach-extension | - | 新指令标记 (token) |

- 表示是否能缓存的指令
```
Cache-Control: public
```
当指定为 public 时, 则表明其他用户也可利用缓存; 指定为 private 指令时, 响应只以特定的用户作为对象; 使用 no-cache 指令的目的是防止从缓存中返回过期的资源
- 控制可执行缓存的对象的
```
Cache-Control: no-store
```
使用 no-store 时表明请求或响应中包含机密信息, 因此该指令规定不能缓存在本地存储请求或响应的任一部分
- 指定缓存期限和认证的指令
```
Cache-Control: s-maxage=604800 (单位: 秒)
```
s-maxage 指令的功能和 max-age 的指令的功能相同, 不同的是 s-maxage 指令只适用于供多位用户使用的公共缓存服务器; max-age 指令要求缓存资源的缓存时间要比客户端指定的数值要小, 否则缓存服务器将会把请求发送给源服务器; min-fresh 指令要求缓存服务器返回至少还未过指定时间的缓存资源; max-stale 指令表示缓存资源即使过期了也照常接收; 使用 only-if-cached 指令表示客户端仅在服缓存服务器本地缓存目标资源的情况下才会要求其返回; must-reavalidate 指令表示会缓存服务器会向源服务器再次验证即将返回的响应缓存目前是否仍然有效; proxy-revalidate 指令要求所有的缓存服务器在接收到客户端带有该指令的请求返回响应之前, 必须再次验证缓存的有效性; no-transform 指令规定无论是在请求还是响应中, 缓存都不能改变实体主体的媒体类型
- Cache-Control 扩展
```
Cache-Control: private, community="UCI"
```
通过 cache-extension 标记可以扩展 Cache-Control 首部字段内的指令

##### Connnection
Connection 首部字段有如下两个作用
- 控制代理不再转发的首部字段
- 管理持久连接

##### Date
首部字段 Date 表明创建 HTTP 报文的日期和时间

##### Pragma
Pragma 是 HTTP/1.1 之前版本的历史遗留字段, 仅作为 HTTP/1.0 的向后兼容而定义; 规范定义的形式唯一, 如下
```
Pragma: no-cache
```
该字段属于通用首部字段, 但只用在客户端发送的请求中, 客户端要求所有中间服务器不返回缓存的资源

##### Trailer
首部字段 Trailer 会事先说明在报文主体后记录了哪些首部字段

##### Transfer-Encoding
首部字段 Transfer-Encoding 规定了传输报文主体时采用的编码方式

##### Upgrade
首部字段 Upgrade 用于检测 HTTP 协议及其他协议是否可使用更高版本进行通信, 其参数值可以用来指定一个完全不同的通信协议

##### Via
使用首部字段 Via 是为了追踪客户端和服务器之间的请求和响应报文的传输路径

##### Warning
HTTP/1.1 的 Warning 首部是从 HTTP/1.0 的响应首部 (Retry-After) 演变过来的, 该首部通常会告知用户一些与缓存相关的问题的警告; 其格式如下
```
Warning: [警告码] [警告的主机: 端口号] "[警告内容]" ([日期时间])
```

#### 请求首部字段
请求搜捕字段是从客户端往服务端发送请求报文中所使用的字段, 用于补充请求的附加信息, 客户端信息, 对响应内容相关的优先级排序等内容

##### Accept
```
Accept: text/html, application/xhtml+xml, application/xml;q=0.9,*/*;q=0.8
```
Accept 首部字段可通知服务器, 用户代理能够处理的媒体类型以及媒体类型的相对优先级

##### Accept-Charset
```
Accept-Charset: iso-8859-5, unicode-1-1;q=0.8
```
Accept-Charset 首部字段可用来通知服务器用户代理支持的字符集及字符集的相对优先顺序

##### Accept-Encoding
```
Accept-Encoding: gzip, deflate
```
Accept-Encoding 首部字段用来告知服务器用户代理支持的内容编码及内容编码的优先级顺序

##### Accept-Language
```
Accept-Language: zh-cn,zh;q=0.7,en-us,en;q=0.3
```
Accept-Language 首部字段用来告知服务器用户代理能够处理的自然语言集

##### Authorization
Authorization 首部字段是用来告诉服务器, 用户代理的认证信息

##### Expect
客户端使用首部字段 Expect 来告知服务器期望出现的某种特定行为

##### From
From 首部字段用来告知服务器, 用户代理的用户的电子邮箱地址

##### Host
首部字段 Host 会告知服务器, 请求的资源所处的互联网主机名和端口号; Host 首部字段在 HTTP/1.1 规范内唯一一个必须包含在内的首部字段

##### If-Match
形如 If-xxx 形式的请求首部字段, 都可称为条件请求, 服务器接收到附带条件的请求后, 只有判断指定条件为真时才会执行请求

##### If-Modified-Since
首部字段 If-Modified-Since 属附带条件之一, 它会告知服务器若 If-Modified-Since 字段值早于资源的更新时间, 则希望能处理该请求, 而在指定 If-Modified-Since 字段值的日期时间之后, 如果请求的资源都没有更新过, 则返回状态码 304 Not Modified 的响应

##### If-None-Match
首部字段 If-None-Match 属于附带条件之一, 和首部字段 If-Match 作用相反

##### If-Range
首部字段 If-Range 属于附带条件之一, 它告知服务器若指定的 If-Range 字段值和请求资源的 ETag 值或时间相一致时, 则作为范围处理, 否则返回全体资源

##### If-Unmodified-Since
首部字段 If-Unmodified-Since 和 If-Modified-Since 作用相反

##### Max-Forwards
通过 TRACE 和 OPTIONS 方法, 发送包含首部字段 Max-Forwards 的请求时, 该字段已十进制整数形式指定可经过的服务器最大的数目

##### Proxy-Authorization
接收到代理服务器发来的认证质询时, 客户端会发送包含首部字段 Proxy-Authorization 的请求, 以告知服务器认证所需要的信息

##### Range
```
Range: bytes=5001-10000
```
对于只需获取部分资源的范围请求, 包含首部字段 Range 即可告知服务器资源的指定范围

##### Referer
首部字段 Referer 会告知服务器请求的原始资源的 URI

##### TE
```
TE: gzip, deflate;q=0.5
```
首部字段 TE 会告知服务器客户端能够处理响应的传输编码方式及相对优先级

##### User-Agent
首部字段 User-Agent 会将创建请求的浏览器和用户代理名称等信息传达给服务器

#### 响应首部字段
响应首部字段是由服务器向客户端返回响应报文中所使用的字段, 用于补充响应的附加信息, 服务器信息, 以及对客户端的附加要求等信息

##### Accept-Ranges
首部字段 Accept-Ranges 是用来告知客户端服务是否能处理范围请求, 以指定获取服务端的某个部分的资源

##### Age
首部字段 Age 能告知客户端源服务器在多久前创建了响应; 字段值的单位为秒

##### ETag
首部字段 ETag 能告知客户端实体标识; 它是一种将资源以字符串形式作为唯一标识的方式; 服务器会为每份资源分配对应的 ETag 值 (ETag 中分强 ETag 和 弱 ETag)

##### Location
首部字段 Location 可以将响应接收方引导至某个与请求 URI 位置不同的资源

##### Proxy-Authenticate
```
Proxy-Authenticate:  Basic realm="Usagidesign Auth"
```
首部字段 Proxy-Authenticate 会把代理服务器所要求的认证信息发送给客户端

##### Retry-After
首部字段 Retry-After 告知客户端u应该在多久之后再次发送请求

##### Server
首部字段告知客户端当前服务器上安装的 HTTP 服务器应用程序的信息

##### Vary
首部字段 Vary 可对缓存进行控制, 源服务器会向代理服务器传达关于本地缓存使用方法的命令

##### WWW-Authenticate
首部字段 WWW-Authenticate 用于 HTTP 访问认; 它会告知客户端适用于访问请求 URI 所指定资源的认证方案

#### 实体首部字段
实体首部字段是包含在请求报文和响应报文中的实体部分所使用的首部, 用于补充内容的更新时间等与实体相关的信息

##### Allow
```
Allow: GET, HEAD
```
首部字段 Allow 用于通知客户端能够支持 Request-URI 指定资源的所有 HTTP 方法

##### Content-Encoding
首部字段 Content-Encoding 会告知客户端服务器对实体的主体部分选用的内容编码方式

##### Content-Language
首部字段 Content-Language 会告知客户端, 实体主体使用的自然语言

##### Content-Length
首部字段 Content-Length 表明了实体主体部分的大小; 对实体主体内容进行编码传输时, 不能再使用 Content-Length 首部字段

##### Content-Location
首部字段 Content-Location 给出的与报文主体部分相对应的 URI; 和 Location 不同, Content-Location 表示的是报文主体返回资源对应 URI

##### Content-MD5
首部字段 Content-MD5 是一串由 MD5 算法生成的值, 其目的是检查报文主体在传输过程中是否保持完整, 以及确认传输到达

##### Content-Range
首部字段 Content-Range 表示当前发送部分及整体的大小

##### Content-Type
首部字段 Content-Type 说明了实体主体内对象的媒体类型

##### Expires
首部字段 Expires 会将资源失效的日期告知客户端

##### Last-Modified
首部字段 Last-Modified 指明资源最终修改的时间

#### 为 Cookie 服务的首部字段
管理客户端与服务端之间状态的 Cookie

##### Set-Cookie
```
Set-Cookie: status=enable; expires=Tue, 05 Jul 2011 07:26:31 GMT; path=/; domain=.hacker.jp
```
以下表格为 Set-Cookie 的字段值

| 属性 | 说明 |
| :--- | :--- |
| NAME=VALUE | 赋予 Cookie 的名称和值 (必需项)|
| expires=DATE | Cookie 的有效期 (若不指定默认为浏览器关闭之前) |
| path=PATH | 将服务器上的文件目录作为 Cookie 的适用对象 (若不指定则默认为文档所在的文件目录) |
| domian=域名 | 作为 Cookie 适用对象的域名 (若不指定则默认为创建 Cookie 的服务器的域名) |
| Secure | 仅在 HTTPS 安全通信时才会发送 Cookie |
| HttpOnly | 加以限制, 使 Cookie 不能被 JavaScript  脚本访问 |

##### Cookie
```
Cookie: status=enable
```
首部字段 Cookie 会告知服务器, 当客户端想获得 HTTP 状态管理支持时, 就会在请求中包含从服务器接收到的 Cookie

#### 其他首部字段
HTTP 首部字段是可以自行扩展的

##### X-Frame-Options
首部字段 X-Frame-Options 属于 HTTP 响应首部, 用于控制网站内容在其他 Web 网站的 Frame 标签内的显示问题; 其主要目的是为了防止点击劫持 (clickjacking) 攻击; 其有两个可选值: DENY 和 SAMEORIGIN

##### X-XSS-Protection
首部字段 X-XSS-Protection 属于 HTTP 响应首部, 是针对跨站脚本攻击 (XSS) 的一种对策, 用于控制浏览器 XSS 防护机制的开关

##### DNT
首部字段 DNT (Do Not Track) 属于 HTTP 请求首部, 意为拒绝个人信息被收集, 是表示拒绝被精准广告追踪的一种方法

##### P3P
首部字段 P3P 属于 HTTP 响应首部, 通过利用 P3P 技术, 可以让 Web 网站上的个人隐私变成一种仅供程序理解的形式, 以达到保护用户隐私的目的
