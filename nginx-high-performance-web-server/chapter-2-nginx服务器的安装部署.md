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
