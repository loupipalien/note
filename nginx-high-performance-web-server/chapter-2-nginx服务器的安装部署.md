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
配置监听使用指令 listen, 其配置方法主要有三种, 语法结构如下
```
# 第一种
listen address[:port] [default_server] [setlib=number] [backlog=number] [rcvbuf=size] [sndbuf=size] [deferred] [accept_filter=filter] [bind] [ssl];

# 第二种
listen port [default_server] [setlib=number] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [deferred] [bind] [ipv6only=on|off] [ssl];

# 第三种
listen unix:path [default_server] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [deferred] [bind] [ssl];

# address: IP 地址, 如果是 IPv6 的地址, 需要使用 "[]" 括起来
# port: 端口号, 如果只定义了 IP 地址没有定义端口号, 就使用 80 端口
# path: socket 文件路径, 如 /var/run/nginx.sock 等
# default_server: 标识符, 将此虚拟主机设置为 address:port 的默认主机
```

##### 基于名称的虚拟主机配置
这里的主机就是指 server 块对外提供的虚拟主机; 设置了主机的名称并配置好 DNS, 用户就可以使用这个名称向虚拟主机发送请求了; 其语法结构为
```
server_name name ...;
```
对于 name 来说可以只有一个名称, 也可以由多个名称并列, 之间用空格隔开; 每个名字就是一个域名, 由两端或者三段组成, 之间由点号 "." 隔开; 以下是一个简单的示例
```
server_name myserver.com www.myserver.com;
```
Nginx 规定, 第一个名称作为此虚拟主机的主要名称;  除此之外, name 还支持使用通配符 `*`, 但通配符只能用在由三段字符串组成名称的首段或尾段, 或者由两段字段组成的名称的尾段; 示例如下
```
server_name *.myserver.com www.myserver.*;
```
在 name 中还可以使用正则表达式, 并使用波浪号 `~` 作为正则表达式字符串的开始标记
```
server_name ~^www\+d\.myserver\.com$;
```
由于 name 支持使用通配符和正则表达式两种配置名称方式, 因此在包含多个虚拟主机的配置文件中, 可能会出现一个名称被多个虚拟主机的 name 匹配成功; Nginx 中对于匹配方式的不同, 按照以下优先级选择虚拟主机
- 准确匹配 name 的
- 通配符在开始时匹配 name 的
- 通配符在结尾时匹配 name 的
- 正则表达式匹配 name 的
- 有多个同一优先级匹配 name 的, 首次匹配的生效

##### 基于 IP 的虚拟主机匹配
与基于名称的虚拟主机匹配类似, 只是 name 为 IP 地址, 也无需考虑通配符和正则的问题

##### 配置 location 块
Nginx 的官方文档中定义的 location 的语法结构为
```
location [ = | ~ | ~* | ^~ ] uri { ... }
# 匹配前, 会将 uri 中的符号进行编码处理 (例如空格, 问号等)
```
其中 uri 变量是待匹配的请求字符串, 可以是不含正则表达式的字符串 (标准 uri), 也可以是包含正在表达式的字符串 (正则 uri);  在括号里的是可选项, 用来改变请求字符串与 uri 的匹配方式  
在不添加可选项时, Nginx 首先在 server 块的多个 location 中搜索是否有标准 uri 和请求字符串匹配的, 如果有多个选择记录匹配度最高的, 然后再与 location 块中的正则 uri 匹配, 当第一个正则匹配则结束, 并使用这个 location 块处理请求, 如果正则全部匹配失败, 就使用刚才记录的匹配度最高的 location 块处理此请求  
四个可选项的含义如下
- `=`: 用于标准 uri 前, 要求请i去字符串与 uri 严格匹配, 如果匹配成功则使用此 location 块处理此请求
- `~`: 用于表示 uri 包含正则表达式, 并区分大小写
- `~*`: 用于表示 uri 包含正则表达式, 并不区分大小写
- `^~`: 用于标准 uri 前, 表示找到标准 uri 和请求字符串匹配度最高的 location 后, 立即使用此 location 块处理请求, 不再继续搜索正则 uri 和请求字符串做匹配

##### 配置请求的根目录
Web 服务器接收到网络请求后, 首先要在服务器端指定目录中寻找请求资源; 在 Nginx 服务器中, 指令 root 就是用来配置这个根目录的, 其语法结构为
```
root path;
```
其中 path 为 Nginx 服务器接收到请求后查找资源的根目录路径; path 变量可以包含 Nginx 服务器预设的大多数变量, 只有 `$document_root` 和 `$realpath_root` 不可以使用; 此指令而可以在 http 块, server 块, location 块配置, 由于使用 Nginx 服务器多数情况下要配置多个 location 块对不同的请求分别做出处理, 因此通常配置在 location 块中
```
location /data/ {
    root /locationtest1
}
```
到 location 接收到 `/data/index.htm` 的请求时, 将在 /locationtest1/data/ 目录下找到 index.htm 响应请求

##### 更改 location 的 URI
在 location 块中, 除了使用 root 指令指明请求处理根目录, 还可以使用 alias 指令改变 location 接收到 URI 的请求路径, 其语法为
```
alias path;
```
path 为修改后的根路径, 同样也可以包含除了 `$document_root` 和 `$realpath_root` 之外的 Nginx 服务器预设变量
```
location ~^/data/(.+\.(html|htm))$ {
    root /locationtest1/other/$1
}
```
到 location 接收到 `/data/index.htm` 的请求, 匹配成功, 之后根据 alias 指令的配置, 将在 /locationtest1/other/ 目录下找到 index.htm 并响应请求

##### 设置网站的默认首页
Nginx 设置网站错误页面的指令为 error_page, 语法结构为
```
error_page code ... [=[response]] uri;
# code: 要处理的 HTTP 错误代码
# response: 可选项, 将 code 指定错误代码转为新的代码错误 response
# uri: 错误页面的路径或者网站地址; 如果设置为路径, 则是以 Nginx 服务器安装路径下的 html 目录为根路径的相对路径; 如果设置为网址, 则 Nginx 服务器会直接访问该网址获取错误页面, 并返回给用户端
```
设置 Nginx 服务器使用 `${NGINX_HOME/html}/404.html` 页面响应 404 错误
```
error_page 404 /404.html
```
设置 Nginx 服务器产生 410 的 HTTP 消息时, 使用 `${NGINX_HOME/html}/empty.gif` 返回给用户表段 301 消息
```
error_page 410 =301 /empty.gif
```
如果不想将错误页面放在 Nginx 服务器的安装路径下, 那么可以使用一个 location 指令定向错误页面到新的路径下
```
error_page 404 /404.html
...
location /404.html {
    root /myserver/errorpages/
}
```

##### 基于 IP 配置的 Nginx 的访问权限
Nginx 配置通过两种途径支持基本访问权限的控制, 一种是由 HTTP 标准模块支持的 ngx_http_access_module 支持的, 另一种是通过 IP 判断客户端是否有对 Nginx 的访问权限  
- allow 指令, 用于设置允许访问 Nginx 的客户端 IP, 语法结构为
```
allow address | CIDR | all;
# address: 允许访问的客户端 IP, 不支持同时设置多个, 如果有多个 IP 需要设置, 需要重复使用 allow 指令
# CIDR: 允许访问的客户端的 CIDR 地址, 如 202.80.18.23/25
# all: 允许所有客户端访问
```
- deny 指令, 用于设置禁止访问 Nginx 的客户端 IP
```
deny address | CIDR | all;
# address: 禁止访问的客户端 IP, 不支持同时设置多个, 如果有多个 IP 需要设置, 需要重复使用 deny 指令
# CIDR: 禁止访问的客户端的 CIDR 地址, 如 202.80.18.23/25
# all: 禁止所有客户端访问
```

这两个指令可以在 http 块, server 块, location 块中配置; 以下示例
```
location / {
    deny 192.168.1.1;
    allow 192.168.1.0/24;
    deny all;
}
```
当有多个访问权限的配置时, 从上到下依次匹配, 遇到匹配的配置时就不再继续匹配

##### 基于密码配置的 Nginx 的访问权限
Nginx 还支持基于 HTTP Basic Authentication 协议的认证; 该协议是一种 HTTP 性质的认证办法, 需要识别用户名和密码, 认证失败的客户端不拥有访问 Nginx 服务器的权限; 该功能由 HTTP 标准模块 ngx_http_auth_basic_module 支持
- auth_basic 指令, 用于开启或关闭该认证功能, 语法结构为
```
auth_basic string | off;
# string: 开启该认证功能, 并配置验证时的指示信息
# off: 关闭该认证功能
```
- auth_basic_user_file, 用于设置包含用户名和密码信息的文件路径
```
auth_basic_user_file file;
```
这里密码文件支持明文或加密后的文件, 其文件格式如下
```
name1:password1
name2:password2:comment
```
加密密码可以使用 crypt() 函数进行密码加密的格式, Linux 平台可以使用 htpasswd 命令生成

#### Nginx 服务器基础配置实例
TODO
