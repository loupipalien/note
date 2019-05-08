### Docker 数据管理
容器中管户据主要有两种方式
- 数据卷 (Data Volumns): 容器内数据直接映射到本地主机环境
- 数据卷容器 (Data Volumn Containers): 使用特定容器维护数据卷

#### 数据卷
数据卷是一个可供容器使用的特殊目录, 它将主机操作系统目录直接映射进容器, 类似 Linux 的 mount 操作; 数据卷可以提供很多有用的特性, 如下所示
- 数据卷可以在容器之间共享和重用, 容器间传递数据将变得高效方便
- 对数据卷内数据的修改会立马生效, 无论是容器内操作还是本地操作
- 对数据卷的更新不会影响镜像, 解耦了应用和数据
- 卷会一直存在, 直到没有容器使用, 可以安全地卸载它
##### 在容器内创建一个数据卷
在用 docker run 命令的时候, 使用 -v 标记可以在容器内创建一个容器卷, 多次重复使用 -v 标记可以创建多个数据卷
```
$ docker run -d -P --name web -v /webapp traing/webapp python app.py
```
##### 挂在一个主机目录作为数据卷 (推荐)
使用 -v 标记指定挂载一个本地的已有目录到容器中作为数据卷
```
$ docker run -d -P --name web -v /src/webapp:/opt/webapp traing/webapp python app.py
```
##### 挂在一个本地主机文件作为数据卷 (不推荐)
-v 标记也可以从主机挂在单个文件到容器中作为数据卷
```
$ docker run -d -P --name web -v ~/.bash_history:/.bash_history traing/webapp python
```

#### 数据卷容器
如果用户需要在多个容器之间共享一些持续更新的数据, 最简单的方式是使用数据卷容器; 数据卷容器也是一个容器, 但是它的目的是专门用来提供其他容器挂载
```
$ docker run -it -v /dbdata --name dbdata ubuntu
```
然后可以在其他容器中使用 --volumns-from 来挂载 dbdata 容器中的数据卷
```
$ docker run -it --volumns-from dbdata --name db1 ubuntu
$ docker run -it --volumns-from dbdata --name db2 ubuntu
```
可以多次使用 --volumns-from 参数来从多个容器挂载多个数据卷, 还可以从其他已经挂载了容器卷的容器来挂载数据卷
```
$ docker run --name db3 --volumns-from db1 traing/postgres
```
如果删除了挂载的容器 (包括 dbdata, db1 和 db2), 数据卷并不会被自动删除; 如果要删除一个数据卷, 必须在删除最后一个还挂载它的容器时显式使用 docker rm -v 命令来指定同时删除关联的容器

#### 利用数据卷容器来迁移数据
可以利用数据卷容器对其中的数据卷进行备份, 恢复, 以实现数据的迁移
##### 备份
例如使用以下命令来备份 dbdata 数据卷容器内的数据
```
$ docker run --volumns-from dbdata -v ${pwd}:/backup --name ubuntu tar -cvf /backup/backup.tar /dbdata
```
##### 恢复
如果要恢复数据, 首先要创建一个带有数据卷的容器 dbdata2
```
$ docker run -v /dbdata --name dbdata2 ubuntu /bin/bash
```
然后创建另一个新容器, 挂载 dbdata2 的容器, 并使用 untar 解压备份文件到所挂载的容器中
```
$ docker run --volumns-from dbdata2 -v ${pwd}:/backup busybox tar xvf /backup/backup.tar
```
