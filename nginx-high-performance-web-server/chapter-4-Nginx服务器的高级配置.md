### Nginx 服务器的高级配置

#### 针对 IPv4 的内核 7 个参数的配置优化
这里提及的参数是和 IPv4 网络有关的 Linux 内核参数, 可以将这些参数追加到 `/etc/sysctl.conf` 文件中, 然后使用命令 `/sbin/sysctl -p` 让其生效

##### net.core.netdev_max_backlog 参数
表示当每个网络接口接收数据包的速率比内核处理这些包的速率块时, 允许发送到队列的数据包的最大数目; 一般默认值是 128, Nginx 服务器中定义的 NGX_LISTEN_BACKLOG 默认为 512; 可以调整为
```
net.core.netdev_max_backlog = 262144
```
##### net.core.somaconn 参数
该参数用于调节系统同时发起的 TCP 连接数, 一般为 128; 在客户端存在高并发请求的情况下, 该默认值较小, 可能导致连接超时或重传问题; 可按实际需要调整此值
```
net.core.somaconn = 262144
```
##### net.ipv4.tvp_max_orphans 参数
该参数用于设置系统中最多允许存在多少 TCP 套接字不被关联到任何一个用户文件句柄上; 如果超过这个数字, 没有与用户文件句柄关联的 TCP 套接字将立即复位, 同时给出警告信息; 这个限制只是为了防止简单的 DoS 攻击; 一般在内存较足的情况下可以增大这个值
```
net.ipv4.tvp_max_orphans = 262144
```
##### net.ipv4.tcp_max_syn_backlog 参数
该参数用于记录尚未收到客户端确认信息的连接请求的最大值, 此参数的默认值是 1024;  一般在内存较足的情况下可以增大这个值
```
net.ipv4.tcp_max_syn_backlog = 262144
```
##### net.ipv4.tcp_timestamps 参数
该参数用于设置时间戳, 这可以避免序列号的卷绕; 此值赋 0 时则禁用对于 TCP 时间戳的支持; 在默认情况下, TCP 协议会让内核接受这种 "异常", 建议关闭
```
net.ipv4.tcp_timestamps = 0
```
##### net.ipv4.tcp_synack_retries 参数
该参数用于设置内核放弃 TCP 连接之前向客户端发送 SYN + ACK 包的数量; 为了建立对端的连接服务, 服务器和客户端需要进行三次握手, 第二次握手期间, 内核需要发送 SYN 并附带一个回应前一个 SYN 的 ACK, 这个参数主要影响这进程, 一般赋值为 1, 即内核放弃连接之前发送一次 SYN + ACK 包, 可以设置为
```
net.ipv4.tcp_synack_retries = 1
```
##### net.ipv4.tcp_syn_retries 参数
该参数与上一个参数类似, 设置内核放弃建立连接之前发送 SYN 包的数量
```
net.ipv4.tcp_syn_retries = 1
```

#### 针对 CPU 的 Nginx 配置优化的 2 个指令
##### worker_processes 指令
用于设置 Nginx 服务器的进程数, 默认是 1, 建议为 CPU 核数的 1 ~ 2 倍; 例如四核 CPU
```
worker_processes 4;
```
##### worker_cpu_affinity 指令
用于为每个进程分配 CPU 的工作内核; worker_cpu_affinity 指令的值是由几组二进制值表示的, 每一组代表一个进程, 每组中的每一位表示该进程使用 CPU 的情况, 1 表示使用, 0 表示不使用; 主要注意的是, 二进制位排列顺序和 CPU 的顺序是相反的; 如下表示为四个工作进程分配四个 CPU 内核, 工作进程 1 使用 CPU 3, 以此类推
```
worker_cpu_affinity 0001 0010 0100 1000;
```

#### 与网络连接相关的配置的 4 个指令
##### keepalive_timeout 指令
这个指令支持两个选项, 中间用空格隔开; 第一个选项指定客户端连接保持活动的超时时间, 在这个时间之后服务器会关闭此连接; 第二个选项可选, 其指定了 Keep-Alive 消息头保持活动的有效时间, 如果不设置则 Nginx 不会想客户端发送 Keep-Alive 消息头以保持与客户端的连接, 超过设置的时间后客户端就可以关闭连接
```
keepalive_timeout 60 50;
```
##### send_timeout 指令
该指令用于设置 Nginx 服务器响应客户端的超时时间, 这个超时时间仅针对两个客户端和服务器之间建立连接后, 某次活动之间的时间; 如果这个时间后客户端没有任何活动, Nginx 服务器将会关闭连接
```
send_timeout 10s;
```
##### client_header_buffer_size 指令
该指令用于设置 Nginx 服务器允许的客户端请求头部的缓冲区大小, 默认为 1KB; Nginx 有时候返回 400 错误, 就是因为客户端请求头过大 (如 cookie 很大) 引起的
```
client_header_buffer_size 4k;
```
##### multi_accept 指令
该指令用于配置 Nginx 服务器是否尽可能多的接收客户端的网络连接请求, 默认值是 off

#### 与事件驱动模型相关的 8 个指令
##### use 指令
用于指定 Nginx 服务器使用的事件驱动模型
##### worker_connections 指令
该指令用于设置 Nginx 服务器的每个工作进程允许同时连接客户端的最大数量, 其语法结构为
```
worker_connections number;
```
##### worker_rlimit_sigpending 指令
该指令用于设置 Linux 2.6.6-mm2 版本之后 Linux 平台的事件信号队列长度上限, 其语法结构为
```
worker_rlimit_sigpending limit;
```
##### devpoll_changes 和 devpoll_events 指令
这两个指令用于设置在 /dev/poll 事件驱动模型下 Nginx 服务器可以与内核之间传递时间的数量; 前者设置传递给内核的数量, 后者设置从内核获取的事件数量; 默认值是 32
```
devpoll_changes number;
devpoll_events number;
```
##### kqueue_changes 和 kqueue_events 指令
这两个指令用于设置在 kqueue 事件驱动模型下 Nginx 服务器可以与内核之间传递时间的数量; 前者设置传递给内核的数量, 后者设置从内核获取的事件数量; 默认值是 512
```
kqueue_changes number;
kqueue_events number;
```
##### epoll_events 指令
该指令用于设置 epoll 事件驱动模型下 Nginx 服务器可以和内核之间传递事件的数量; 默认值是 512
```
epoll_changes number;
# 与其他事件驱动模型不同, epoll 的向内核传递事件数量和从内核传递数量是相同的
```
##### rtsig_signo 指令
该指令用于设置 rtsig 模式使用两个信号中的第一个, 第二个信号是在第一个信号的编号上加 1; 默认值是 SIGRTMIN + 10
```
rtsig_signo singo
```
##### rtsig_overflow_* 指令
表示 rtsig_overflow_events, rtsig_overflow_test, rtsig_overflow_threshold 指令, 其语法格式为
```
rtsig_overflow_* nunmber
# rtsig_overflow_events: 指定队列溢出时使用 poll 库处理的事件数
# rtsig_overflow_test: 指定 poll 库处理万第几件事件后将清空 rtsig 使用的信号队列, 默认值 32
# rtsig_overflow_threshold: 指定 rtsig 模式使用的信号队列中的事件超过多个时就需要清空队列了
```
