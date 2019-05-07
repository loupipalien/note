### 使用 Docker 镜像
Docker 运行容器前需要本地存在对应的镜像, 如果镜像没保存在本地, Docker 会尝试先从默认镜像仓库下载 (默认使用 Docker Hub 公共注册服务器中的仓库, 也可以通过配置自定义镜像仓库

#### 获取镜像
可以使用 `docker pull NAME[:TAG]`, 其中 NAME 是镜像仓库的 (全限定) 名称, TAG 是镜像的标签; 如果不显示指定 TAG, 则默认会选择 latest 标签, 这会下载仓库中最新版本的镜像  
下载过程中可以看出, 镜像文件一般由若干层组成, 每一层有 64 个十六进制字符组成的唯一 id; 使用 `docker pull` 命令下载时会获取并输出镜像的各层信息, 当不同的镜像包括相同的层时, 本地仅存储层的一份内容, 减少了需要的存储空间

#### 查看镜像信息
##### 使用 image 命令列出镜像
```
$ docker images
```
##### 使用 tag 命令添加镜像标签
```
docker tag ubuntu:latest myubuntu:latest
```
##### 使用 inspect 命令查看详细信息
```
$ docker inspect ubuntu:latest
```
##### 使用 history 命令查看镜像历史
```
$ docker history ubuntu:latest
```

#### 搜索镜像
使用 docker search 命令可以搜索远端仓库中共享的镜像, 默认搜索官方仓库中的镜像
```
$ docker search ubuntu
```

#### 删除镜像
##### 使用标签删除镜像
```
$ docker rmi myubuntu:latest
```
当同一个镜像拥有多个标签的时候, docker rmi 命令只是删除该镜像多个标签中的指定标签而已, 并不影响镜像文件
##### 使用镜像 ID 删除镜像
当使用 docker rmi 命令, 并且在后面跟上镜像的 ID 时, 会先尝试删除所有指向该镜像的标签, 然后删除该镜像文件本身; 当有该镜像创建的容器存在时, 镜像文件默认是无法被删除的

#### 创建镜像
创建镜像的方法主要有三种: 基于已有镜像的容器创建, 基于本地模板导入, 基于 Dockerfile 创建
##### 基于已有镜像的容器创建
TODO
##### 基于本地模板导入
TODO

#### 存出和载入镜像
用户可以使用 docker save 和 docker load 命令来存出和载入镜像
##### 存储镜像
如果要导出镜像到本地文件, 可以使用 docker save 命令
```
$ docker save -o ubuntu-latest.tar ubuntu:latest
```
##### 载入镜像
可以使用 docker load 将导出的 tar 文件再导入到本地镜像库
```
$ docker load --input ubuntu-latest.tar
```

#### 上传镜像
可以使用 docker push 命令上传镜像到仓库, 默认上传到 Docker Hub 官方仓库 (需要登录); 命令格式为
```
docker push NAME[:TAG] [REGISTRY_HOST[:REGISTRY_PORT]]/NAME[:TAG]
```
