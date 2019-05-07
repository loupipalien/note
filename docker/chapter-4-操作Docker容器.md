### 操作 Docker 容器
容器是镜像的一个运行实例, 不同的是镜像是静态的只读文件, 而容器带有运行时需要的可写文件层; 如果认为虚拟机是模拟运行的一整套操作系统 (包括内核, 应用运行态环境和其他系统环境) 和跑在上面的应用, 那么 Docker 容器就是独立运行的一个 (或一组) 应用, 以及它们必须的运行环境

#### 创建容器
##### 新建容器
可以使用 docker create 命令新建一个容器, 例如
```
$ docker create -it ubuntu:latest
```
使用 docker create 命令新建的容器处于停止状态, 可以使用 docker start 命令来启动它
##### 启动容器
使用 docker start 命令来启动一个已经创建的容器
```
$ docker start container_id
```
##### 新建并启动容器
除了创建容器后通过 start 命令来启动, 也可以直接新建并启动容器; 所需要的命令主要为 docker run, 等价于先执行 docker create 命令, 再执行 docker start 命令
```
$ docker run ubuntu /bin/echo "Hello World"
```
当利用 docker run 来创建并启动容器时, Docker 在后台运行的标准操作包括
- 检查本地时候存在指定的镜像, 不存在就从公有仓库下载
- 利用镜像创建一个容器, 并启动该容器
- 分配一个文件系统给容器, 并在只读的镜像层外面挂在一层可读写层
- 从宿主机配置的网桥接口中桥接一个虚拟接口到容器中
- 从网桥的地址池配置一个 IP 地址给容器
- 执行用户指定的应用程序
- 执行完毕后容器被自动终止
##### 守护态运行
Docker 容器可以在后台以守护态形式运行, 可以通过添加 -d 参数来实现
```
$ docker run -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
```

#### 终止容器
```
$ docker stop container_id
```
处于终止状态的容器, 可以通过 docker start 命令重新启动

#### 进入容器
在使用 -d 参数时, 容器启动后会进入后台, 用户无法看到容器中的信息, 也无法进行操作; 以下介绍三种方法进入后台运行的容器
##### attach 命令
TODO
##### exec 命令
TODO
##### nsenter 工具
TODO

#### 删除容器
可以使用 docker rm 命令来删除处于终止或退出状态的容器, 命令格式为
```
docker rm [-f|--force] [-l|--link] [-v|--volumns] container_id
```

#### 导入和导出容器
##### 导出容器
导出容器是指导出一个已经创建的容器到一个文件, 不管此时这个容器是否处于运行状态, 都可以使用 docker export 命令, 其格式为
```
docker export [-o|--output[=""]] container_id
```
##### 导入容器
导出文件又可以使用 docker import 命令导入变成镜像, 该命令格式为
```
docker import [-c|--changer[=[]]] [-m|--message[=MESSAGE]] file|URL|-[REPOSITORY[:TAG]]
```
实际上, 即可以使用 docker load 命令来导入镜像存储文件到本地镜像库, 也可以使用 docker import 命令来导入一个容器快照到本地镜像库  
这两者的区别在于容器快照文件将丢弃所有的历史记录和元数据信息 (仅保存容器当时的快照状态), 而镜像存储文件将保存完整记录, 体积也更大; 此外, 从容器快照文件导入时可以重新指定标签等元数据信息
