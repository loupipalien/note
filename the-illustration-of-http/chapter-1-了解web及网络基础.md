### 了解 Web 及网络基础
#### 使用 HTTP 协议访问 Web
Web 使用一种名为 HTTP (HyperText Transfer Protocol, 超文本传输协议) 的协议作为规范, 完成客户端到服务端等一系列运作流程

#### HTTP 的诞生
##### 为知识共享而规划 Web
蒂姆-伯纳斯-李 (Tim Berners Lee) 提出的构想; 构建 WWW (World Wide Web, 万维网) 的三项技术是: 把 SGML (Standard Generalized Markup Language, 标准通用标记语言) 作为页面的文本标记语言的 HTML (HyperText Markup Language, 超文本标记语言), 作为文档传递协议的 HTTP, 指定文档所在地址的 URL (Uniform Resource Locator, 统一资源定位符)

##### Web 成长时代
TODO

##### 驻足不前的 HTTP
- HTTP/0.9
自 1990 年问世到 HTTP/1.0 前
- HTTP/1.0
1996 年 5 月发布 [HTTP/1.0](https://www.ietf.org/rfc/rfc1945.txt) 协议版本
- HTTP/1.1
1997 年 1 月发布 [HTTP/1.1](https://www.ietf.org/rfc/rfc2616.txt) 协议版本

#### 网络基础 TCP/IP
通常使用的网络是在 TCP/IP 协议族的基础上运行的; HTTP 协议是其一个子集

##### TCP/IP 协议族
TODO

##### TCP/IP 的分层管理
TCP/IP 协议族按层次分为以下四层: 应用层, 传输层, 网络层, 数据链路层; 其各层作用如下
- 应用层
应用层决定了向用户提供服务时通信的活动; 此层包括了 FTP (File Transfer Protocol, 文件传输协议), DNS (Domain Name System, 域名系统) 等协议, HTTP 协议也属于此层
- 传输层
传输层提供网络连接中两台计算机之间的数据传输; 此层有两个性质不同的协议: TCP (Transmission Control Protocol, 传输控制协议) 和 UDP (User Data Protocol, 用户数据报协议)
- 网络层
网络层用于处理网络上流动的数据包; 数据包是网络传输的最小数据单位, 该层规定了通过怎样的路径到达对方计算机, 并把数据传送给对方
- 链路层
链路层用来处理连接网络的硬件部分; 包括控制操作系统, 硬件的设备驱动, NIC (Network Interface Card, 网络适配器), 光纤等等

##### TCP/IP 通信传输流
```
    应用层       传输层  网络层  链路层        链路层   网络层  传输层      应用层
HTTP (客户端) <-> TCP <-> IP <-> 网络 <------> 网络 <-> IP <-> TCP <-> HTTP (服务端)
```
客户端在应用层发出一个请求 (HTTP 协议); 为了方便传输, 在传输层 (TCP 协议) 把从应用层收到的数据 (HTTP 请求报文) 进行分割, 并在各个报文上打上标记序号以及端口号后转发给网络层; 在网络层 (IP 协议) 增加作为通信目的地的 MAC 地址后转发给链路层; 服务端在链路层接收到数据按序向上发送直到应用层  
发送端在层与层之间传输数据时, 每经过一层时必定会被打上一个该层所属的首部信息, 反之接收端在层与层传输数据时, 每经过一层会把对应的首部消去

#### 与 HTTP 关系密切的协议: IP, TCP, DNS
##### 负责传输的 IP 协议
TODO
##### 确保可靠性的 TCP 协议
TODO
##### 负责域名解析的 DNS 服务
TODO

#### 各种协议与 HTTP 协议的关系
TODO

#### URI 和 URL
##### 统一资源标识符
URI 是 Uniform Resource Identifier 的缩写, [RFC2369](https://www.ietf.org/rfc/rfc2369.txt) 对这三个单词定义如下
- Uniform: 规定统一的格式可方便处理多种不同类型的资源, 而不用根据上下文环境来识别资源指定的访问方式
- Resource: 资源的定义是 "可标识的任何东西"
- Identifier: 表示可标识的对象, 也称标识符

URI 用字符串表示某一互联网资源, 而 URL 表示资源的地点 (互联网上所处的位置), 即 URL 是 URI 的子集

##### URI 格式
```
http://user:pass@www.example.com:8080/dir/index.html?uid=1#ch1
协议名, 登录信息, 服务器地址, 服务器端口号, 文件路径, 查询字符串, 片段标识符
```
协议名表示获取访问资源时指定的协议类型
- 登录信息: 指定用户名和密码作为从服务端获取资源的凭证; 此项是可选项
- 服务器地址: 可以是域名, 或则 IP 地址; 此项是必选项
- 服务器端口号: 服务器连接的网络端口号; 此项是可选项
- 文件路径: 指定服务器上文件路径来定位特定资源; 此项是可选项
- 查询字符串: 可以传入查询参数; 此项是可选项
- 片段标识符: 使用片段标识符可以标记出已获取资源中的子资源 (文档内的某个位置); 此项是可选项
