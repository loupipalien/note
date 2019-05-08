### 访问 Docker 仓库
仓库 (Repository) 是集中存放镜像的地方, 分公共仓库和私有仓库; 一个容易与之混淆的概念是注册服务器 (Registry); 实际上注册服务器是存放仓库的具体服务器, 一个注册服务器上可以有多个仓库, 而每个仓库下可以有多个镜像; 可以将仓库看作一个具体的项目或者目录; 例如对于仓库地址 private-docker.com/ubuntu 来说, private-docker.com 是注册服务器地址, ubuntu 是仓库名

#### Docker Hub 公共镜像市场
TODO

#### 时速云镜像市场
TODO

#### 搭建本地私有仓库
##### 使用 registry 镜像创建私有仓库
安装 Docker 后, 可以通过官方提供的 registry 镜像来简单搭建一套本地私有仓库环境
```
$ docker run -d -p 5000:5000 registry
```
默认情况下, 会将仓库创建在容器的 /tmp/registry 目录下, 可以通过 -v 参数来将镜像文件存放在本地的指定路径
##### 管理私有仓库
TODO
