### Docker 核心实现技术
从操作系统功能上看, 目前 Docker 底层依赖的核心技术主要包括 Linux 操作系统的命名空间 (Namespace), 控制组 (Control Group), 联合文件系统 (Union File System) 和 Linux 网络虚拟化支持

#### 基本架构
Docker 目前采用 C/S 架构, 客户端和服务端可以运行在同一台服务器上, 也可以运行在不同的机器上通过 socket 或 restful API 来通信
##### 服务端
Docker Deamon 一般在宿主机后台运行, 作为服务端接受来自客户的请求, 并处理这些请求 (创建, 运行, 分发容器); Docker 服务端默认监听本地的 unix:///var/run/docker.sock 套接字, 只允许本地的 root 用户或 docker 用户成员访问; 可以通过 -H 选项来修改监听的方式
##### 客户端
Docker 客户端提供一系列可执行的命令, 用户使用这些命令与 Docker Deamon 交互; 用户使用的 Docker 可执行命令即为客户端程序, 与 Docker Deamon 不同的是, 客户端发送命令后, 等待服务端返回, 一旦收到返回后, 客户端立刻执行结束并退出; 用户执行新的命令, 需要再次调用客户端命令
##### 新的架构设计
C/S 架构给 Docker 基本功能的实现带来了许多便利, 但同时也引入了一些限制  
例如在使用 Docker 时, 必须要启动并保持 Docker Deamon 的正常运行, 但一旦 Docker Deamon 服务不正常则已启动的容器将无法使用; 在新的版本 (1.11.0+) 中, 开始将维护容器运行的任务放到一个单独的组件 containerd 中管理, 并且支持 OCI 的 runc 规范

#### 命名空间
命名空间 (namespace) 是 Linux 内核的一个强大的特性, 为容器虚拟化的实现带来了极大便利; 利用这一特性, 每个容器都可以拥有自己单独的命名空间, 运行在其中的应用都像是在独立的操作系统环境中一样; 命名空间机制保证了容器之间的彼此互不影响
##### 进程命名空间
Linux 通过命名空间管理进程号, 对于同一进程 (即同一个 task_struct), 在不同的命名空间中, 看到的进程号不相同, 每个进程命名空间有一套自己的进程号管理办法; 进程命名空间是一个父子关系的结构, 子空间中的进程对于父空间是可见的; 新 fork 出的进程在父命名空间和子命名空间将分别有一个进程号来对应
##### 网络命名空间
有了 pid 命名空间, 那么每个命名空间中的进程就可以相互隔离, 但是网络端口还是共享本地系统的端口  
通过网络命名空间, 可以实现网络隔离; 网络命名空间为进程提供了一个完全独立的网络协议栈的视图; Docker 采用虚拟网络设备 (Vitual NetWork Device) 的方式, 将不同命名空间的网络设备连接到一起; 默认情况下, 容器中的虚拟网卡将同本机上的 docker0 网桥连接在一起
##### IPC 命名空间
容器中进程交互还是采用了 Linux 常见的进程间交互办法 (Interprocess Commiunication, IPC), 包括信号量, 消息队列和共享内存等; PID 命名空间和 IPC 命名空间可以组合一起使用, 同一个 IPC 命名空间内的进程可以彼此可见, 允许进行交互,  不同命名空间的进程则无法交互
##### 挂载命名空间
类似与 chroot, 将一个进程放到一个特定的目录执行; 挂载命名空间允许不同命名空间的进程看到的文件结构不同, 这样每个命名空间中的进程看到的文件目录彼此则被隔离
##### UTS 命名空间
UTS (UNIX Time-sharing System) 命名空间允许每个容器拥有独立的主机名和域名, 从而可以虚拟出一个有独立主机名和网络空间的环境, 就跟网络上一台独立的主机一样
##### 用户命名空间
每个容器可以有不同的用户和组 id, 也就是说可以在容器内使用特定的内部用户执行程序, 而非本地系统上存在的用户

#### 控制组
控制组 (CGroup) 是 Linux 内核的一个特性, 主要用来对共享资源进行隔离, 限制, 审计等; 只有能控制分配到容器的资源, 才能避免多个容器同时运行时对宿主机系统的资源竞争; 控制组提供
- 资源限制 (Resource limiting): 可以将组设置为不超过设定的内存限制
- 优先级 (Prioritization): 通过优先级让一些组优先得到更多的 CPU 资源
- 资源审计 (Accounting): 用来统计系统实际上把多少资源用到适合的目的上, 可以使用 cpuacct 子系统记录某个进程组使用的 CPU 时间
- 隔离 (isolation): 为组隔离命名空间, 这样一个组不会看到另一个组的进程, 网络连接和文件系统
- 控制 (Control): 挂起, 恢复和重启动等

#### 联合文件系统
联合文件系统 (UnionFS) 是一种轻量级的高性能分层文件系统, 它支持将文件系统中的修改信息作为一次提交, 并层层叠加, 同时可以将不同目录挂载到同一虚拟文件系统下, 应用看到的是挂载的最终结果; 联合文件系统是实现 Docker 镜像的技术基础, Docker 镜像可以通过分层来进行继承
##### Docker 存储
Docker 镜像自身就是由多个文件层组成, 每一层有唯一的编号 (层 ID), 可以通过 `docker history` 查看一个镜像由哪些层组成; 对于 Docker 镜像来说, 这些层的内容都是不可修改的, 只读的; 而当 Docker 利用镜像启动一个容器时, 将在镜像文件系统的最顶端再挂载一个新的可读写层给容器; 容器中的内容更新将会发生在可读写层; 一般推荐将容器的修改数据通过 volume 方式挂载, 而不是直接修改镜像内数据  
Docker 所有存储都在 Docker 目录下, 以 Ubuntu 系统为例, 默认路径是 /var/lib/docker
##### 多种文件系统比较
Docker 目前支持文件系统种类包括: AUFS, OverlayFS, brtfs, vfs, zfs, Device Mapper

#### Linux 网络虚拟化
Docker 的本地网络实现其实就是利用了 Linux 上的网络命名空间和虚拟网络设备 (特别是 veth pair)
##### 基本原理
直观上看要实现网络通信, 机器需要至少一个网络接口 (物理接口和虚拟接口) 与外界相通, 并可以收发数据包; 此外如果不同子网之间要进行通信, 需要额外的路由机制  
Docker 中的网络接口默认都是虚拟的接口, 虚拟接口的最大优势就是转发效率极高; 这是因为 Linux 通过在内核中进行数据复制来实现虚拟接口之间的数据转发, 即返送接口的发送缓存中的数据包将直接复制到接收接口的接收缓存中, 而无需通过外部物理网络设备进行交换; 对于本地系统和容器系统来看, 虚拟网口跟一个正常的以太网卡相比并无区别, 只是它速度要快的多  
Docker 容器网络就很好的利用了 Linux 虚拟网络技术, 在本地主机和容器内分别创建一个虚拟接口, 并让它们彼此联通 (这样的一对接口叫做 veth pair)
##### 网络创建过程
- 创建一对虚拟接口, 分别放到本地注解和新容器的命名空间中
- 本地主机一端的虚拟接口连接到默认的 docker0 网桥或指定网桥上, 并具有一个以 veth 开头的唯一名
- 容器一端的虚拟接口将放到新创建的容器中, 并修改名字作为 eth0, 这个接口只在容器的命名空间可见
- 从网桥可用地址段中获取一个空闲地址分配给容器的 eth0, 并配置默认路由网关为 docker0 网卡的内部接口 docker0 的 IP 地址
##### 手动配置网络
TODO
 
