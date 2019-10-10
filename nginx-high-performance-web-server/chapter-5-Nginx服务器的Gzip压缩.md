### Nginx 服务器的 Gzip 压缩

#### 由 ngx_http_gzip_module 模块处理的 9 个指令
ngx_http_gzip_module 模块主要负责 Gzip 功能的开启和设置, 对响应数据进行在线实时压缩; 该模块主要包含以下主要指令
##### gzip 指令
该指令用于开启和关闭 Gzip 功能, 其语法结构为
```
gzip on | off;
```
默认情况下该指令设置为 off; 只有该指令为 on 时, 下列个指令才有效
##### gzip_buffers 指令
该指令用于设置 Gzip 压缩文件使用缓存空间的大小, 其语法结构如下
```
gzip_buffers number size;
# number：指定 Nginx 服务器需要项系统申请缓存空间的个数
# size: 指定每个缓存空间的大小
```
默认情况下, number * size = 128, 其中 size 的值取系统内存页一页的大小, 4KB 或 8KB
##### gzio_comp_level 指令
该指令用于设定 Gzip 压缩程度, 包括级别 1 到 9; 级别 1 表示压缩程度最低, 压缩效率最高; 其语法结构为
```
gzio_comp_level level;
```
默认级别为 1
##### gzip_disable 指令
针对不同种类客户端发起的请求, 可以选择性的开启和关闭 Gzip 功能, 该指令从 Nginx 0.6.23 启动, 用于设置一些客户端种类; Nginx 服务器在响应这些种类的客户端请求时, 不使用 Gzip 功能缓存响应输出数据, 其语法结构为
```
gzip_disable regex ...;
```
##### gzip_http_version 指令
早期一些浏览器可能不支持 Gzip 自解压, 因此会看到乱码, 所以针对不同的 HTTP 版本, 需要选择性开启或关闭 Gzip 功能; 该指令用于设置开启 Gzip 功能的最低 HTTP 协议版本, 其语法结构为
```
gzip_http_version 1.0 | 1.1
```
默认设置为 1.1, 使用该版本 HTTP 协议的客户端基本都支持 Gzip 自解压
##### gzip_min_length 指令
Gzip 压缩功能能对大数据的压缩效果明显, 但是如果压缩很小的数据, 可能出现越压缩越大的情况, 因此应该根据响应大小, 选择性的开始或关闭 Gzip 功能; 该指令设置页面的字节数, 当响应页面的大小大于该值时, 才启用 Gzip 功能; 响应页面的大小通过 HTTP 响应头部中的 Content-Length 指令获取, 但是如果使用了 Chunk 编码动态压缩, Content-Length 不存在或被忽略, 则该指令不起作用; 其语法结构为
```
gzip_min_length length;
```
默认值为 20, 设置为 0 时表示统统压缩, 建议设置值为 1024
##### gzip_proxied 指令
该指令在使用 Nginx 服务器的反向代理功能时有效, 前提是在后端服务器返回的响应页头部中, Requests 部分包含用于通知代理服务器的 Via 头域; 该指令主要用于设置 Nginx 服务器是否对后端服务器返回的结果进行压缩; 其语法结构为
```
gzip_proxied off | expired | no-cache | no-store | private | no_last_modified | no_etag | auth | any ...;
# off: 关闭 Nginx 服务器对后端服务器返回结果的 Gzip 压缩, 是默认设置
# expired: 当后端服务器响应页头部包含用于指示响应数据过期时间的 expired 头域时, 启用对响应数据的 Gzip 压缩
# no-cache: 当后端服务器响应页头部包含用于通知所有缓存机制是否缓存的 Cache-Control 头域, 其指令值为 no-cache 时, 启用对响应数据的 Gzip 压缩
# no-store: 当后端服务器响应页头部包含用于通知所有缓存机制是否缓存的 Cache-Control 头域, 其指令值为 no-store 时, 启用对响应数据的 Gzip 压缩
# private: 当后端服务器响应页头部包含用于通知所有缓存机制是否缓存的 Cache-Control 头域, 其指令值为 private 时, 启用对响应数据的 Gzip 压缩
# no_last_modified: 当后端服务器响应页头部不包含用于指明需要获取数据最后修改时间的 Last-Modified 头域时, 启用对响应数据的 Gzip 压缩
# no_etag: 当后端服务器响应页头部不包含用于标示请求变量的实体值的 ETag 头域时, 启用对响应数据的 Gzip 压缩
# auth: 当后端服务器响应页头部包含用于标示 HTTP 授权证书的 Authentication 头域时, 启用对响应数据的 Gzip 压缩
# any: 无条件启动对后端服务器响应数据的 Gzip 压缩
```
##### gzip_types 指令
Nginx 服务器可以根据响应页的 MIME 类型选择性的开启 Gzip 压缩功能, 该指令用来设置 MIME, 被设置的值会被压缩; 其语法结构为
```
gzip_types mime-type ...;
```
默认值为 text/html
##### gzip_vary 指令
该指令用于设置在使用 Gzip 功能时是否发送带有 "Vary: Accept-Encoding" 头域的响应头部; 该头域的主要功能是告诉接收方发送的数据经过了数据压缩; 其语法结构为
```
gzip_vary on | off;
```
默认值设置为 off (该指令在使用过程中存在 bug, 导致 IE 4 以上的浏览器的数据缓存功能失效)
##### 由 ngx_http_gzip_static_module 模块处理的指令
该模块主要负责搜索和发送经过 Gzip 功能预压缩的数据, 这些数据以 .gz 的后缀名存在服务器上; 如果客户端请求的数据在之前被压缩过, 并且客户端支持 Gzip 压缩, 就直接返回压缩后的数据  
该模块与 ngx_http_gzip_module 模块不同之处在于, 该模块使用的静态压缩, 在 HTTP 响应头部包含 Content-Length 头域来指明报文体的长度, 用于服务器可确定响应数据长度的情况, 而后者默认使用 Chunked 编码的动态压缩, 其主要适用于服务器无法确定响应数据长度的请况, 例如下载大文件  
该模块的有关指令主要有: `gzip_static, gzip_http_version, gzip_proxied, gzip_disable, gzip_vary` 等, 其中 gzip_static 指令用于开启和关闭该模块的功能, 其语法结构为
```
gzip_static on | off | always;
# on: 开启
# off: 关闭
# always: 一直发送 Gzip 预压缩文件, 不检查客户端是否支持
```
其他指令与 ngx_http_gzip_module 模块的使用方式相同, 但 gzip_proxied 指令只接受以下设置
```
gzip_proxied expired | no-cache | no-store | private | auth;
```
另外, gzip_vary 指令开启后值对为压缩的内容添加 "Vary: Accept-Encoding" 头域; 如果需要对所有响应头添加该头域, 可以通过 Nginx 设置的 add_header 指令实现

#### 由  ngx_http_gunzip_module 模块处理的两个指令
Nginx 服务器支持对响应输出数据流进行 Gzip 压缩, 这对客户端来说, 需要有能力解压和处理 Gzip 压缩数据, 但如果客户端本身不支持该功能, 就需要 Nginx 服务器在向其发送数据之前先将该数据解压; 主要的指令以有: `gunzip, gunzip_buffers, gzip_http_version, gzip_proxied, gzip_disable, gzip_vary`
##### gunzip 指令
该指令用于开启或关闭该模块的功能
```
gunzip_static on | off;
```
默认值为 off
#### gunzip_buffers 指令
与 ngx_http_gzip_module 模块下的指令使用方式相同

#### Gzip 压缩功能的使用
TODO

#### 本章小结
TODO
