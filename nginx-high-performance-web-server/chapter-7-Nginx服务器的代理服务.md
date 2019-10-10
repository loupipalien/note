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
