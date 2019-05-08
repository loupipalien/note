### 端口映射与容器互联
Docker 提供了两个方便的功能来满足服务访问的基本需求: 一个是允许映射容器内应用的服务端口到本地宿主机, 另一个是互联机制实现多个容器间通过容器名来快速访问

#### 端口映射实现访问容器
#### 从外部访问容器应用
在启动容器的时候, 如果不指定对应的参数, 在容器外部是无法通过网络来访问容器内的网络应用和服务的; 可以通过 -P 和 -p 参数来指定端口映射; 当使用 -P 标记时, Docker 会随即映射一个 49000-49900 的端口到内部容器开放的网络端口; -p 可以指定要映射的端口, 并且在一个指定端口上只可以绑定一个容器, 支持的格式有 IP:HostPort:ContainerPort | IP::ContainerPort | HostPort:ContainerPort
##### 映射所有接口地址
使用 HostPort:ContainerPort 格式将本地的 5000 端口映射到容器的 5000 端口
```
$ docker run -d -p 5000:5000 traing/webapp python app.py
```
此时默认会绑定本地所有接口上的所有地址, 可以多次使用 -p 标记绑定多个端口
##### 映射到指定地址的指定端口
可以使用 IP:HostPort:ContainerPort 格式指定映射使用一个特定地址
```
$ docker run -d -p 127.0.0.1:5000:5000 traing/webapp python app.py
```
##### 映射到指定地址的任意端口
使用 IP::ContainerPort 绑定 localhost 的任意端口到容器的 5000 端口, 本地主机会自动分配一个端口
```
$ docker run -d -p 127.0.0.1::5000 traing/webapp python app.py
```
##### 查看映射端口配置
使用 docker port 命令来查看当前映射的端口配置, 也可以查看绑定的地址

#### 互联实现便捷互访
容器的互联 (linking) 是一种让多个容器中应用进行快速交互的方式; 它会在源和接收容器之间创建连接关系, 接收容器可以通过容器名快速访问到源容器, 而不用指定具体的 IP 地址
##### 自定义容器命名
使用 --name 标记可以为容器自定义命名
```
$ docker run -d -P --name web traing/webapp python app.py
```
##### 容器互联
使用 --link 参数可以让容器之间安全的进行交互; 以下先创建一个新的数据库容器, 再创建一个 web 容器连接它
```
$ docker run -d --name db traing/postgres
$ docker run -d -P --name web --link db:db traing/webapp python app.py
```
--link 参数的格式为 --link name:alias, 其中 name 是要连接的容器名称, alias 是这个连接的别名  
Docker 相当于在两个互联的容器之间创建了一个虚机通道, 而且不用映射它们的端口到宿主主机上; 在启动 db 时没有使用 -p 或 -P 标记, 从而避免了暴露数据库服务端口到外部网络上  
Docker 通过两种方式为容器公开连接信息
- 更新环境变量
- 更新 /etc/hosts 文件

TODO
