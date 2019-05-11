### Docker 核心实现技术
从操作系统功能上看, 目前 Docker 底层依赖的核心技术主要包括 Linux 操作系统的命名空间 (Namespace), 控制组 (Control Group), 联合文件系统 (Union File System) 和 Linux 网络虚拟化支持

#### 基本架构
Docker 目前采用 C/S 架构, 客户端和服务端可以运行在同一台服务器上, 也可以运行在不同的机器上通过 socket 或 restful API 来通信
