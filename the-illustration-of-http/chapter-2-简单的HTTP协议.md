### 简单的 HTTP 协议
在两台计算机之间使用 HTTP 协议通信时, 在一条通信线路上必定一端是客户端, 另一端是服务端

#### 通过请求和响应的交换达成通信
HTTP 协议规定, 请求从客户端发出, 最后服务端响应该请求并返回
- 请求的报文是由请求方法, 请求 URI, 协议版本, 可选的请求首部字段和内容实体构成
```
POST /from/entry HTTP/1.1
Host: www.example.com
Connection: keep-alive
Content-type: application/x-www-form-urlencoded
Content-Length: 16

name=ueno&age=37
```
- 响应报文是由协议版本, 状态码, 用以解释状态码的原因短语, 可选的响应首部字段和内容实体构成
```
HTTP/1.1 200 OK
Date: Tue, 10 Jul 2018 06:50:15 GMT
Content-Length: 362
Content-type: text/html

<html>...</html>
```

#### HTTP 是无状态协议
HTTP 是无状态协议, 自身不对请求和响应之间的通信状态进行保存; 这是为了更快的处理大量事务, 确保协议的可伸缩性, 特意将 HTTP 协议设计的简单

#### 请求 URI 定位资源
客户端请求访问资源而发送请求时, URI 需要将作为请求报文中的请求 URI 包含在内
```
GET /index.html HTTP/1.1
Host: Host: www.example.com
```
如果不是访问特定资源而是对服务器本身发起请求, 可以使用 * 来代替请求 URI
```
OPTIONS * HTTP/1.1
```

#### 告知服务器意图的 HTTP 方法
- GET: 获取资源
GET 方法用来请求访问已被 URI 识别的资源; 指定的资源经服务器端解析后返回响应内容
- POST: 传输实体主体
POST 方法用来传输实体的主体
- PUT: 传输文件
PUT 方法用来传输文件, 要求在请求报文的主体中包含文件内容, 然后保存到请求 URI 指定的位置
- HEAD: 获得报文首部
HEAD 方法和 GET 方法一样, 只是不反悔报文主体部分; 用于确认 URI 的有效性及资源更新的日期时间
- DELETE: 删除文件
DELETE 方法用来删除文件
- OPTIONS: 询问支持的方法
OPTIONS 方法用来查询针对请求 URI 指定的资源支持的方法
- TRACE: 追踪路径
TRACE 方法是让 Web 服务器将之前的请求通信返回给客户端的方法
- CONNECT: 要求用隧道协议连接代理
CONNECT 方法要求在与代理服务器通信时建立隧道, 实现用隧道协议进行 TCP 通信; 主要使用 SSL 和 TSL 协议把通信内容加密后经网络隧道传输

#### 使用方法下达命令

| 方法 | 说明 | 支持的 HTTP 协议版本 |
| :--- | :--- | :--- |
| GET | 获取资源 | 1.0, 1.1 |
| POST | 传输实体主体 | 1.0, 1.1 |
| PUT | 传输文件 | 1.0, 1.1 |
| HEAD | 获得报文首部 | 1.0, 1.1 |
| DELETE | 删除文件 | 1.0, 1.1 |
| OPTIONS | 询问支持的方法 | 1.1 |
| TRACE | 追踪路径 | 1.1 |
| CONNECT | 要求用隧道协议连接代理 | 1.1 |
| LINK | 建立和资源之间的联系 | 1.0 |
| UNLINK | 断开连接关系 | 1.0 |

#### 持久连接节省通信量
HTTP 协议的初始版本中, 每进行一次 HTTP 通信就要断开一次 TCP 连接
```
----------                              ----------
|        |   --------- SYN --------->   |        |
|        |   <-------- SYN/ACK ------   |        |
|        |   --------- ACK --------->   |        |
| 客户端 |   ------- HTTP 请求 ------>   | 服务端 |
|        |  <------ HTTP 响应 -------   |        |
|        |  <--------- FIN ---------    |        |
|        |  ---------- ACK -------->    |        |
|        |  ---------- FIN -------->    |        |
|        |  <-------- ACK ----------    |        |
----------                              ----------
```
当浏览一个包含多个图片的页面, 在发送请求访问页面资源的同时, 也会发送该 HTML 页面里包含的其他资源; 因此, 每次的请求都会造成无谓的 TCP 连接建立和断开
##### 持久连接
为解决上述问题, HTTP 1.1 提出了持久连接 (HTTP Persistent Connections, 也成为 HTTP keep-alive), 持久连接的特点是只要任意一端没有明确提出断开连接, 则保持 TCP 连接状态
```
----------                              ----------
|        |   --------- SYN --------->   |        |
|        |   <-------- SYN/ACK ------   |        |
|        |   --------- ACK --------->   |        |
|        |   ------- HTTP 请求 ------>  |        |
|        |  <------ HTTP 响应 -------   |        |
| 客户端 |   ------- HTTP 请求 ------>   | 服务端 |
|        |  <------ HTTP 响应 -------   |        |
|        |  --------- ...... ------>    |       |
|        |  <-------- ...... -------    |        |
|        |  <--------- FIN ---------    |        |
|        |  ---------- ACK -------->    |        |
|        |  ---------- FIN -------->    |        |
|        |  <-------- ACK ----------    |        |
----------                              ----------
```
持久连接的好处在于减少了 TCP 连接的重复建立和断开所造成的额外开销, 减轻了服务端的负载
##### 管线化
持久连接使得多数请求以管线化 (pipelining) 方法发送成为可能, 从前发送请求后需等待并收到响应, 才能发送下一请求; 管线化技术使得不用等待响应也可直接发送下一请求
```
----------                              ----------
|        |   --------- SYN --------->   |        |
|        |   <-------- SYN/ACK ------   |        |
|        |   --------- ACK --------->   |        |
|        |   ------- HTTP 请求 ------>  |        |
|        |   ------- HTTP 请求 ------>  |        |
| 客户端 |   <------ HTTP 响应 -------   | 服务端 |
|        |  <------- HTTP 响应 ------   |        |
|        |  --------- ...... ------>    |       |
|        |  <-------- ...... -------    |        |
|        |  <--------- FIN ---------    |        |
|        |  ---------- ACK -------->    |        |
|        |  ---------- FIN -------->    |        |
|        |  <-------- ACK ----------    |        |
----------                              ----------
```

#### 使用 Cookie 的状态管理
HTTP 是无状态协议, 不对之前发生过的请求和响应的状态进行管理, 无法根据之前的状态进行本次的请求处理; 在无状态协议上实现登录操作的问题, 引入了 Cookie 技术, Cookie 技术通过在请求和响应报文中写入 Cookie 信息来控制客户端的状态; Cookie 会根据从服务端发送的响应报文中的一个叫做 Set-Cookie 的首部字段信息, 通知客户端保存 Cookie, 下次客户端再往服务器发送请求时, 客户端会自动在请求报文中加入 Cookie 发送回去; 服务端发现客户端发送过来的 Cookie 后, 会得到客户端之前的状态信息
```
----------                                           ----------
|        |   ------- HTTP 请求 ------------------->  |        |
|        |  <------ HTTP 响应 (Set-Coookie) ------   |        |
| 客户端 |   ------- HTTP 请求 (Cookie) ---------->   | 服务端 |
|        |  <------ HTTP 响应 (Cookie) ----------    |        |
----------                                           ----------
```
