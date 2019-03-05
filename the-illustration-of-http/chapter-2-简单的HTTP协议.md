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
HTTP 是无状态协议, 自身步队请求和响应之间的通信状态进行保存; 这是为了更快的处理大量事务, 确保协议的可伸缩性, 特意将 HTTP 协议设计的简单

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
