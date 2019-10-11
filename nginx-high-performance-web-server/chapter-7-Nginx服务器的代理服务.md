### Nginx 服务器的代理服务

#### 正向代理和反向代理的概念
正向代理服务器用来让局域网客户机接入外网访问外网资源, 反向代理服务器用来让外网的客户端接入局域网中的站点以访问站点中的资源; 在正向代理服务器中, 我们的角色是客户端, 目的是要访问外网的资源; 在反向代理服务器中, 我们的角色是站点, 目的是把站点的资源发布出去让其他客户端能够访问

#### Nginx 服务器的正向代理服务

##### Nginx 服务器正向代理服务的配置的 3 个指令
###### resolver 指令
该指令用于指定 DNS 服务器的 IP 地址, DNS 服务器的主要工作是进行域名解析, 将域名映射为对应的 IP 地址; 该指令的语法结构为
```
resolver address ... [valid=time];
# address: DNS 服务器的 IP 地址, 如果不指定端口号, 默认使用 53 端口
# time: 设置数据包在网络中的有效时间
```
###### resolver_timeout 指令
该指令用于设置 DNS 服务器域名解析超时时间, 该语法结构为
```
resolver_timeout time;
```
###### proxy_pass 指令
该指令用于设置代理服务器的协议和地址, 它不仅仅用于 Nginx 服务器的代理服务, 更主要的是应用于反向代理服务; 该指令的语法的结构为
```
proxy_pass URL;
```
URL 即为设置的代理服务器协议和地址
##### Nginx 服务器正向代理服务的使用
```
server {
    resolver 8.8.8.8;
    listen 82;
    location / {
        proxy_pass http://$http_host$request_uri;
    }
}
```
#### Nginx 服务器的反向代理服务
配置 Nginx 服务反向代理用到的指令如果没有特别说明, 原则上可以出现在 Nginx 配置文件的 http, server, location 块中
##### 反向代理的基本设置的 21 个指令
###### proxy_pass 指令
该指令用来设置被代理服务器的地址, 可以是主机名称, IP 地址加端口号等形式, 其语法结构为
```
proxy_pass URL;
```
其中 URL 为要设置的被代理服务器的地址, 包含传输协议, 主机名称或 IP 地址加端口号等  
在使用该指令的过程中还要注意, URL 中是否包含有 URI, Nginx 服务器的处理方式是不同的, 如果 URL 中包含 URI, Nginx 服务器不会改变原地址 URI, 但是如果包含了 URI, Nginx 服务器将会使用新的 URI 替代原来的 URI  
另外还需要注意 proxy_pass 指令中的 URL 变量末尾是否加了斜杠 '/',  如 `proxy_pass http://192.168.1.1;` 和 `proxy_pass http://192.168.1.1/;`, 其中的区别就是后一个配置中的 URL 包含了 URI '/', 那么根据之前的叙述, 这一配置会改变源地址 URI, 会将 location 后的变量替换为 '/'

###### proxy_hide_header 指令
该指令用于设置 Nginx 服务器在发送 HTTP 响应时, 隐藏一些头域信息, 其语法结构为
```
proxy_hide_header field;
```
field 为需要隐藏的头域, 该指令可以在 http, server, location 块中进行配置
###### proxy_pass_header 指令
默认情况下, Nginx 服务器在发送响应报文时, 报文中不包含 "Date", "Server", "X-Accel" 等来自被代理服务器的头域信息; 该指令可以设置这些头域信息被发送, 其语法结构为
```
proxy_pass_header field;
```
field 为需要隐藏的头域, 该指令可以在 http, server, location 块中进行配置
###### proxy_pass_request_body 指令
该指令用于配置是否将客户端请求的请求体发送给代理服务器, 其语法结构为
```
proxy_pass_request_body on | off;
```
默认设置为开启 (on), 该指令可以在 http, server, location 块中进行配置
###### proxy_set_header 指令
该指令可以更改 Nginx 服务器接收到的客户端请求的请求头信息, 然后将新的请求头发送给被代理的服务器; 其语法结构为
```
proxy_set_header field value;
# field: 要更改的信息所在的头域
# value: 更改的值, 支持使用文本, 变量或者变量的组合
```
默认情况下, 该指令的设置为
```
proxy_set_header Host $proxy_host;
proxy_set_header connection close;
```
###### proxy_set_body 指令
该指令可以更改 Nginx 服务器接收到的客户端请求的请求体信息, 然后将新的请求体发送给被代理的服务器; 其语法结构为
```
proxy_set_body value;
```
value 为更改的值, 支持使用文本, 变量或者变量的组合
###### proxy_bind 指令
强制将与代理主机的连接绑定到指定的 IP 地址, 通俗的讲就是在配置了多个基于名称或者基于 IP 的主机的情况下, 如果我们希望代理连接由指定的主机处理, 就可以使用该指令进行配置, 其语法结构为
```
proxy_bind address;
```
其中 address 为指定主机的 IP 地址
###### proxy_connect_timeout 指令
该指令配置 Nginx 服务器与后端被代理服务器尝试建立连接的超时时间, 其语法结构为
```
proxy_connect_timeout time;
```
time 为设置的超时时间, 默认为 60s
###### proxy_read_timeout 指令
该指令配置 Nginx 服务器向后端被代理服务器 (组) 发出 read 请求后, 等待响应的超时时间, 其语法结构为
```
proxy_read_timeout time;
```
time 为设置的超时时间, 默认为 60s
###### proxy_send_timeout 指令
该指令配置 Nginx 服务器项后端被代理服务器 (组) 发出 write 请求后, 等待响应的超时时间, 其语法结构为
```
proxy_send_timeout time;
```
time 为设置的超时时间, 默认为 60s
###### proxy_http_version 指令
该指令用于设置用于 Nginx 服务器提供代理服务的 HTTP 协议版本, 其语法结构为
```
proxy_http_version 1.0 | 1.1;
```
###### proxy_method 指令
该指令用于设置 Nginx 服务器请求被代理服务器使用的请求方法, 一般为 POST 或 GET, 设置了该指令客户端的请求方法将被忽略; 其语法结构为
```
proxy_method method;
```
其中 method 可以设置为 POST 或 GET, 注意不加引号
###### proxy_ignore_client_abort 指令
该指令用于设置在客户端中断网络请求时, Nginx 服务器是否中断对被代理服务器的请求, 其语法结构为
```
proxy_ignore_client_abort on | off
```
默认值设置为 off, 当客户端中断网络请求时, Nginx 服务器中断对被代理服务器的请求
###### proxy_ignore_headers 指令
该指令用于设置一些 HTTP 响应头中的头域, Nginx 服务器接收到被大力服务器的响应数据后, 不会处理被设置的头域; 其语法结构为
```
proxy_ignore_headers field ...;
```
其中 field 为要设置的 HTTP 响应头的头域, 如 `X-Accel-Expires`, `Set-Cookie` 等
###### proxy_redirect 指令
该指令用于修改被代理服务器返回的响应头中的 Location 头域和 Refresh 头域, 与 proxy_pass 指令配合使用; 其语法结果为
```
proxy_redirect redirect replacement;
proxy_redirect default;
proxy_redirect off;
# redirect: 匹配 Location 头域值的字符串, 支持变量的使用和正则表达式
# replacement: 用于替换 redirect 变量内容的字符串, 支持变量的使用
```
###### proxy_intercept_errors 指令
该指令用于配置一个状态是开启还是关闭; 在开启该状态时, 如果被代理的服务器返回的 HTTP 状态码为 400 或者大于 400, 则 Nginx 服务器使用自己定义的错误页 (使用 error_page 指令); 如果是关闭该状态, Nginx 服务器直接将被代理服务器返回的 HTTP 状态返回给客户端; 其语法结构如下
```
proxy_intercept_errors on | off;
```
###### proxy_headers_hash_max_size 指令
该指令用于配置存放 HTTP 报文头的哈希表的容量, 其语法结构为
```
proxy_headers_hash_max_size size;
```
size 默认为 512 个字符
###### proxy_headers_hash_bucket_size 指令
该指令用于设置 Nginx 服务器申请存放 HTTP 报文头的哈希表容量的单位大小; 其语法结构如下
```
proxy_headers_hash_bucket_size size;
```
size 默认为 64 个字符
###### proxy_next_upstream 指令
在配置 Nginx 反向代理功能时, 如果使用 upstream 指令配置了一组服务器作为被代理服务器, 服务器中各服务器访问规则遵循 upstream 指令配置的轮询规则, 同时可以使用该指令配置在发生异常情况时, 将请求顺次交由下一个组内服务器处理; 该语法结构为
```
proxy_next_upstream status ...;
```
status 为设置服务器返回状态, 可以是一个或者多个, 这些状态包括
- error： 在建立连接时, 向被代理的服务器发送请求或者读取响应头时服务器发生错误
- timeout: - error： 在建立连接时, 向被代理的服务器发送请求或者读取响应头时服务器发生超时
- invalid_header: 被代理的服务器返回的响应头为空或者无效
- http_500 | http_502 | http_503 | http_504 | http_404: 被代理的服务器返回的状态码
- off: 无法将其请求发送给被代理的服务器
###### proxy_ssl_session_reuse 指令
该指令用于配置是否使用基于 SSL 安全协议的会话连接 (https://) 被代理的服务器, 其语法结构为
```
proxy_ssl_session_reuse on | off
```
默认为开启的状态

##### Proxy Buffer 的配置的 7 个指令
Proxy Buffer 启用以后, Nginx 服务器会异步地将被代理服务器的响应数据传递给客户端  
Nginx 服务器首先尽可能地从被代理服务器那里接收响应数据, 放置在 Proxy Buffer 中, Buffer 的大小由 proxy_buffer_size 指令和 proxy_buffers 指令决定; 如果在接收过程中, 发现 Buffer 没有足够大小接收一次响应的数据, Nginx 服务器会将部分接收到的数据临时存放在磁盘的临时文件中, 磁盘上的临时文件路径可以通过 proxy_temp_path 指令进行设置, 临时文件的大小由 proxy_max_temp_file_size 指令和 proxy_temp_file_write_size 指令决定; 一次响应数据被接收完成或者 Buffer 被装满后, Nginx 服务器开始向客户端传输数据  
每个 Proxy Buffer 装满数据后, 在从开始向客户端发送一直到其中数据全部传输给客户端的整个过程中, 它都处于 BUSY 状态, 期间对它进行的其他操作都会失败, 同时处于 BUSY 状态的 Proxy Buffer 总大小由 proxy_busy_buffers_size 指令限制, 不能超过该指令设置的大小  
当 Proxy Buffer 关闭时, Nginx 服务器只要接收到响应数据就会同步地传递给客户端
###### proxy_buffering 指令
该指令用于配置是否启用或关闭 Proxy Buffer, 其语法结构为
```
proxy buffering on | off;
```
默认设置为开启状态
###### proxy_buffers 指令
该指令用于配置接收一次被代理服务器响应数据 Proxy Buffer 个数和每个 Buffer 的大小, 其语法结构为
```
proxy_buffers number size;
# number: Proxy Buffer 的个数
# size: 每个 Buffer 的大小, 一般为内存页大小, 4 KB 或者 8 KB
```
###### proxy_buffer_size 指令
该指令用于配置从被代理服务器获取的第一部分响应数据的大小, 该数据中一般包含了 HTTP 响应头, Nginx 服务器通过它来响应数据和被代理服务器的一些必要信息; 该指令的语法结构为
```
proxy_buffer_size size
# size: 一般为 4 KB 或者 8 KB, 与 proxy_buffers 保持一致或者更小
```
###### proxy_busy_buffers_size 指令
该指令用于限制同时处于 BUSY 状态的 Proxy Buffer 的总大小; 该指令的语法结构为
```
proxy_busy_buffers_size size;
# size: 默认设置为 8KB 或 16 KB
```
###### Proxy_temp_path 指令
该指令用于配置磁盘上的一个文件路径, 该文件用于临时存放代理服务器的大体积响应数据
```
proxy_temp_path path [level1 [level2 [level3]]];
# path: 设置磁盘上存放临时文件的路径
# levelN: 设置在 path 变量的路径下第几级 hash 目录中存放临时文件
```
###### proxy_max_temp_file_size 指令
该指令用于配置所有临时文件的总体积大小; 其语法结构为
```
proxy_max_temp_file_size size;
```
默认设置为 1024 MB
###### proxy_temp_file_write_size 指令
该指令用于配置同时写入临时文件的数据量的总大小, 合理的设置可以避免磁盘 IO 负载过重导致系统性能下降; 其语法结构为
```
proxy_temp_file_write_size size;
```
一般为内存页大小, 4 KB 或者 8 KB

##### Proxy Cache 的配置的 12 个指令
在 Nginx 服务器中, Proxy Buffer (代理缓冲) 和 Proxy Cache (代理缓存) 都与代理服务器相关, 主要用来提供客户端与被代理服务器之间的效率; Proxy Buffer 实现了被代理服务器响应数据的异步传输, Proxy Cache 则主要实现了 Nginx 服务器对客户端请求的快速响应  
需要说明的是, Proxy Cache 机制依赖于 Proxy Buffer 机制, 只有在 Proxy Buffer 机制开启的情况下 Proxy Cache 的配置才发挥作用  
在 Nginx 服务器中还提供了另一种将被代理服务器数据缓存到本地的方法 Proxy Store, 与 Proxy Cache 的区别是, 它对来自被代理服务器的响应数据, 尤其是静态数据只进行简单的缓存, 不支持缓存过期更新, 内存索引等功能, 但支持设置用户或用户组对缓存数据的访问权限
###### proxy_cache 指令
该指令用于配置一块公用的内存区域的名称, 该区域可以存放缓存的索引数据; 该指令的语法结构为
```
proxy_cache zone | off;
# zone: 设置用于存放缓存索引的内存区域的名称
# off: 关闭该功能, 是默认配置
```
从 Nginx 0.7.66 开始, Proxy Cache 机制开启后会检查被代理服务器响应数据 HTTP 头中的 `Cache-Control` 头域, 当其值为 `no-cache`, `no-store`, `private`, `max-age=0`, 或者 `Expires` 头域包含一个过期的时间时, 该响应数据不被 Nginx 服务器缓存; 这样的主要目的是避免私有数据被其他客户端得到
###### proxy_cache_bypass 指令
该指令用于配置 Nginx 服务器项客户端发送响应数据时, 不从缓存获取中获取的条件; 这些条件支持使用 Nginx 配置的常用变量, 其语法结构为
```
proxy_cache_bypass string ...;
```
###### proxy_cache_key 指令
该指令用于设置 Nginx 服务器在内存中为缓存数据建立索引时使用的关键字; 其语法结构为
```
proxy_cache_key string;
```
Nginx 0.7.48 前为 `"$scheme$proxy_host$request_uri"`, 后为 `"$scheme$proxy_host$request_uri$is_args$args"`
###### proxy_cache_lock 指令
该指令用于设置是否开启缓存的锁功能, 在缓存中, 某些数据项可以同时被多个请求返回的响应数据填充; 开启该功能后, Nginx 服务器同时只能有一个请求填充缓存中的某一数据项, 这相当于给数据项上锁; 其他请求如果也想填充该项, 必须等待数据项的锁被释放; 这个等待时间由 proxy_cache_lock_timeout 指令配置
```
proxy_cache_lock on | off;
```
默认是关闭的
###### proxy_cache_lock_timeout 指令
该指令用于设置缓存的锁功能开启后锁的超时时间; 该指令的语法结构为
```
proxy_cache_lock_timeout time;
```
默认是 5s
##### proxy_cache_min_uses 指令
该指令用于设置客户端请求发送次数, 当客户端向代理服务器发送相同请求达到该指令设定的次数之后, Nginx 服务器才对请求的响应数据做缓存; 该指令的语法结构为
```
proxy_cache_min_uses number;
```
默认值为 1
###### proxy_cache_path 指令
该指令用于设置 Nginx 服务器存储缓存数据的路径以及和缓存缩影相关内容; 其语法结构为
```
proxy_cache_path path [levels=levels] keys_zone=name:size1 [inactive=time1] [max_size=size2] [Loader_files=number] [loader_sleep=time2] [loader_threshold=time3]
# TODO
```
###### proxy_cache_use_stale 指令
如果 Nginx 在访问被代理服务器过程中出现被代理的服务器无法访问或者访问错误等现象时, Nginx 服务器可以使用历史缓存响应客户端的请求; 该指令的语法结构为
```
proxy_cache_use_stale error | timeout | invalid_header | updating | http_500 | http_502 | http_503 | http_504 | http_404 | off ...;
````
默认值为 off
###### proxy_cache_valid 指令
该指令可以针对不同的 HTTP 响应状态设置不同的缓存时间, 其语法结构为
```
proxy_cache_valid [code ...] time;
# code: 设置 HTTP 响应的状态代码
# time: 设置缓存时间
```
###### proxy_no_cache 指令
该指令用于配置在什么情况下都不适用 cache 功能; 其语法结构为
```
proxy_no_cache string ...;
```
当 string 不为空或不为 `0` 时不启用 cache 功能
###### proxy_store 指令
该指令配置是否在本地磁盘存来自被代理服务器的响应数据; 该指令的语法结构为
```
proxy_store on | off | string;
# on | off: 设置是否开启 Proxy Store 功能
# string: 自定义缓存文件的存放路径
```
###### proxy_store_access 指令
该指令用于设置用户或用户组对 Proxy Store 缓存数据的访问权限; 其语法结构为
```
proxy_store_access users:permissions ...;
# users: 可以设置为 user, group, all
# permissions: 设置权限
```

#### Nginx 服务器的负载均衡
Nginx 服务器反向代理服务的一个重要用途是实现负载均衡; 现在的负载均衡技术主要实现和作用于网络的第四层或第七层, 完全独立于网络基础硬件设备; Nginx 服务器实现的负载均衡一般认为是七层负载均衡

TODO
