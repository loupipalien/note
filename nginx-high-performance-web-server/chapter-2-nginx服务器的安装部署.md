### nginx 服务器的安装部署

#### 如何获取 Nginx 服务器安装文件
TODO
#### 安装 Nginx 服务器和基本配置
TODO

#### Nginx 服务的启停控制
Nginx 服务在运行中, 会保持一个主进程和一个或多个 worker process 工作进程
TODO

#### Nginx 服务器基础配置指令
默认的 Nginx 服务器配置文件存放在 conf 目录中, 主配置文件名为 nginx.conf; 本节将介绍此文件的内容和基本配置方法, 以下是默认的 nginx.conf 文件中的完整内容 (部分省略)
```
worker_processes 1;  # 全局生效

events {
    worker_connections 1024;   # 在 events 部分中生效
}

http {  # 以下指令在 http 部分生效
    include mime.types;  
    default_type application/octet-stream;
    sendfile on;
    keepalive_timeout 65;
    server {  # 以下指令在 http 的 server 部分中生效
        listen 80;
        server_name localhost;
        location / {  # 以下指令在 http/server 的 location 中生效
            root html;
            index index.html index.htm;
        }
        error_page 500 502 503 504 /50x.html
        location = /50x.html {
            root html;
        }
    }
}
```
##### nginx.conf 的文件结构
从上面的文件配置内容, 可以归纳出 nginx.conf 文件的基本结构为
```
...                                          # 全局块
events {                                     # events 块
   ...
}
http {                                       # http 块
    ...                                      # http 全局块
    server {                                 # server 块
        ...                                  # server 全局块
        location [PATTERN] {                 # location 块
            ...
        }
        location [PATTERN] {                 # location 块
            ...
        }
    }
    server {                                 # server 块
        ...
    }
    ...                                      # http 全局块
}
...
```
配置文件支持大量可配置的指令, 绝大多数指令不是特定属于某个块的; 同一个指令放在不同层级的块中, 其作用域也不同; 在介绍配置指令之前, 先介绍各个块的作用
###### 全局块
是默认配置文件从开始到 events 块之间的一部分内容, 主要设置一些影响 Nginx 服务整体运行的配置指令; 通常包括配置运行的 Nginx 服务器的用户 (组), 允许生成的 worker process 数, Nginx 进程 PID 的存放路径, 日志的存放路径, 以及配置文引入等
###### events 块
events 块涉及的指令主要影响 Nginx 服务器与用户的网络连接; 如是否开启对多 worker process 下的网络连接进行序列化, 是否允许同时接收多个网络连接, 选择哪种事件驱动模型处理连接请求, 每个 worker process 可以同时支持最大的连接数
###### http 块
代理, 缓存, 日志定义等绝大多数功能和第三方模块的配置都可以放在这个模块中; http 块中包含全局块和 server 块; 可以在 http 全局块中配置的指令包括文件引入, MIME-Type 定义, 日志自定义, 是否使用 sendfile 传输文件, 连接超时时间, 单连接请求数上限等
###### server 块
server 块与 "虚拟主机" 的概念有密切联系; 每个 server 块就相当于一台虚拟主机, 它内部可以有多台主机联合提供服务, 一起对外提供在逻辑上关系密切的一组服务 (网站); 与 http 块相同, server 块也可以有自己的全局块, 同时包含多个 location 块
###### location 块
每个 server 块可以包含多个 location 块, 严格意义上来说, location 其实是 server 块的一个指令; location 块的主要作用是, 基于 Nginx 服务器接收到的请求字符串 (如: server_name/uri-string), 对除虚拟主机名称 (也可以是 IP 别名) 之外的字符串 (如 /uri-string) 进行匹配, 对特定的请求进行处理; 地址定向, 数据缓存, 应答控制等功能都是在这部分实现的; 许多第三方模块的配置也是在 location 块中提供功能

##### 配置运行 Nginx 服务器用户 (组)
用于配置运行 Nginx 服务器用户 (组) 的指令是 user, 其语法格式为
```
user user [group];
# user 是可运行 Nginx 的用户
# [group] 是可运行 Nginx 的用户组 (是可选项)
```
如果希望所有用户都可以运行, 可注释掉次指令行, 或者设置为以下值
```
user nobody nobody;
```
这也是 user 指令的默认值
>在 Nginx 配置文件中, 每一条指令配置都必须以分号结束

##### 配置允许生成的 worker process 数
worker process 是 Nginx 服务器实现并发处理服务的关键所在; 配置允许生成的 worker process 数的指令是 worker_processes, 其语法格式为
```
worker_processes number | auto;  
# number 为指定 Nginx 最多可产生的 worker process 数
# auto 值时 Nginx 进程将自动探测
```
##### 配置 Nginx 进程 PID 存放路径
Nginx 进程作为系统的守护进程运行, 需要在某文件中保存当前运行程序的主进程号; Nginx 支持对它的存放路径进行自定义配置, 其语法格式为
```
pid file;
# file 为指定存放的文件路径
```
配置文件默认将此文件存放在 Nginx 安装目录 logs 下, 名字为 nginx.pid; 可以是绝对路径, 也可以是相对于 Nginx 安装目录的相对路径

##### 配置错误日志的存放路径
在全局块, http 块, server 块中都可以对 Nginx 服务器的日志进行相关配置; 这里介绍全局块下的日志配置, 后两种情况的配置基本相同, 只是作用域不同; 使用的指令是 error_log, 其语法结构是
```
error_log file | stderr [debug | info | notice | warn | error | crit | alter | emerg];
```
以下是 Nginx 默认的日志存放和级别设置
```
error_log logs/error.log error;
```
此指令可以在全局块, http 块, server 块, location 块中配置

##### 配置文件的引入
在一些情况下, 可能需要将其他 Nginx 配置或者第三方模块的配置引用到当前的主配置文件中; Nginx 提供了 include 指令来完成配置文件的引入, 其语法结构为
```
include file;
```

##### 设置网络连接的序列化
在 <<UNIX 网络编程>> 第一卷中有一个 "惊群" 的问题 (Thundering herd problem); 大致意思是, 当某一时刻只有一个网络连接到来时, 多个睡眠进程会被同时叫醒, 但是只有一个进程可获得连接; 如果每次唤醒的进程数太多, 会影响一部分系统性能; Nginx 为了解决这个问题, 在配置中包含了这样一条指令 accept_mutex, 当其设置为开启时, 将会对多个 Nginx 进程接收连接进行序列化, 防止多个进程对连接的争抢; 其语法结构为
```
accept_mutex on | off;
```
此配置默认为开启状态, 只能在 events 块中配置

##### 设置是否允许同时接收多个网络连接
每个 Nginx 服务器的 worker process 都有能力同时接收多个新到达的网络连接, 但是需要在配置文件中进行设置, 其语法结构如下
```
multi_accept on | off;
```
此指令默认为关闭状态, 即每个 worker process 一次只能接收一个新到达的网络连接, 此指令只能在 events 块中进行配置

##### 事件驱动模型的选择
Nginx 服务器提供了多种事件驱动来处理网络消息, 配置文件提供了相关指令强制 Nginx 选择哪种事件驱动模型进行消息处理, 其语法结构为
```
use method;
```
其中 method 可选择的内容有: select, poll, kqueue, epoll, rtsig, /dev/poll 以及 eventport; 此指令只能在 events 块中进行配置

##### 配置最大连接数
指令 worker_connections 主要用来设置允许每一个 worker process 同时开启的最大连接数, 其语法结构为
```
worker_connections number;
```
此指令默认为 512 (不能大于操作系统支持打开的最大文件句柄数), 此指令只能在 events 块中配置

##### 定义 MIME-Type
在常用的浏览器中, 可以显示的内容有 HTML, XML, GIF 等多种类的文本或媒体资源, 浏览器为了区分这些资源需要使用 MIME-Type, 即 MIME-Type 是网络资源的媒体类型; Nginx 服务器作为 Web 服务器, 必须能够识别前端请求的资源类型  
在默认的 Nginx 配置文件中, 可以在 http 块中有以下两行配置
```
include mime.types;
default_type application/octet-stream;
```
第一行引用了外部 mime.types 文件, mime.types 文件中顶一个 types, 其中包含了浏览器能够识别的 MIME 类型以及对应与相关类型的文件后缀名  
```
# cat mime.types
types {
    text/html                   html html shtml;
    ...
    image/gif                   gif;
    ...
    application/x-javascript    js;
    ...
    audio/midi                  mid midi kar;
    ...
    video/3gpp                  3gpp 3gp;
    ...
}
```
第二行中使用指令 default_type 配置了用于处理前端请求的 MIME 类型, 其语法结构为
```
default_type mime-type;
```
其中 mime-type 为 types 中定义的 MIME-Type, 如果不加此指令, 默认值为 text/plain; 此指令可以在 http 块, server 块, location 块中进行配置

##### 自定义服务日志
记录 Nginx 服务器提供服务过程应答前端请求的日志, 将其称为服务日志; Nginx 服务器支持对服务日志的格式, 大小, 输出等进行配置, 需要使用两个指令, 分别是 access_log 和 log_format 指令; access_log 指令的语法结构为
```
access_log path[format[buffer=size]]
# path: 配置服务日志的文件存放的路径和名称
# format: 可选项, 自定义服务日志的格式字符串, 也可以通过 "格式串的名称" 使用 log_format 指令定义好的格式; "格式串的名称" 在 log_format 指令中定义
# size: 配置临时存放日志的内存缓存区大小
```
此指令可以在 http 块, server 块, location 块中进行配置; 默认的配置为
```
access_log logs/access_log combined;
```
其中 combined 为 log_format 指令默认定义的日志格式字符串的名称; 如果要取消记录服务日志的功能, 则使用
```
access_log off;
```
和 access_log 联合使用的另一个指令为 log_format, 它专门用于定义服务日志的格式, 并且为格式字符串定义一个名字, 以便 access_log 指令可以直接使用; 其语法格式为
```
log_format name string...;
# name: 格式字符串的名字, 默认的名字为 combined
# string：服务日志的格式字符串; 可以使用 Nginx 预设的一些变量获取相关内容, 变量名称使用双引号, string 整体使用单引号
```
以下是一个示例
```
log_format example_log '$remote_addr - [$time_local] $request $status $body_bytes_sent $ http_referer $http_user_agent';
```
此指令只能在 http 块中进行配置

##### 配置允许 sendfile 方式传输文件
这里主要学习配置 sendfile 传输方式的相关指令 sendfile 和 sendfile_max_chunk 以及它们的语法结构
```
sendfile on | off;
```
用于开启或关闭使用 sendfile() 传输文件, 默认值为 off; 可以在 http 块, server 块, location 块中进行配置
```
sendfile_max_chunk size;
```
其中 size 大于 0, Nginx 进程的每个 worker process 每次调用 sendfile() 传输的数据量最大值; 如果设置为 0 则无限制; 可以在 http 块, server 块, location 块中进行配置

##### 配置连接超时时间
与用户建立会话连接后, Nginx 服务器可以保持这些连接打开一段时间, 指令 keepalive_timeout 就是用来设置此时间的, 其语法结构为
```
keepalive_timeout timeout[header_timeout];
# timeout: 服务器对连接的保持时间, 默认值是 75s
# header_timeout: 可选项, 在应答报文头部的 KEEP-Alive 域设置超时时间 "KEEP-Alive: timeout=header_timeout"
```
可以在 http 块, server 块, location 块中进行配置

##### 单 连接请求数上限
Nginx 服务器端和用户端建立会话连接后, 用户端通过此连接发送请求; 指令 keepalive_requests 用于限制用户通过某一连接向 Nginx 服务器发送请求的次数, 其语法结构为
```
keepalive_requests number;
```
可以在 http 块, server 块, location 块中进行配置, 默认值是 100

##### 配置网络监听
