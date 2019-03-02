### 了解 ZooKeeper

#### ZooKeeper 基础
ZooKeeper 采用类似于文件系统的层级树状结构进行管理数据, 其中的节点称作为 znode

##### API 概述
znode 节点可能含有数据, 也可能没有; 如果一个 znode 节点包含任何数据, 那么数据存储为字节数组, 其中字节数组的格式特定于每个应用的实现, ZooKeeper  并不直接提供解析支持; ZooKeeper 的 API 暴露了以下方法

| 方法 | 描述 |
| :--- | :--- |
| create /path date | 创建一个名为 /path 的 znode 节点, 并包含数据 data |
| delete /path | 删除名为 /path 的 znode |
| exist /path | 检查是否存在名为 /path 的节点 |
| setData /path data | 设置名为 /path 的 znode 的数据为 data |
| getData /path | 返回名为 /path 节点的数据信息 |
| getChildren /path | 返回 /path 节点的所有子节点列表 |

需要注意的是, ZooKeeper 不允许局部写入或读取 znode 节点的数据, 只能整个替换和读取

##### znode 的不同类型
当新建 znode 时, 还需要指定节点的类型, 不同类型决定了 znode 节点的行为方式

###### 持久节点和临时节点
znode 节点可以是持久节点 (persistent) 节点, 还可以是临时 (ephemeral) 节点; 持久的 znode 只能通过调用 delete 来进行删除, 而临时节点不但可以通过调用 delete 删除, 当创建该节点的客户端崩溃或者关闭了与 ZooKeeper 的连接时, 临时节点就会被删除

###### 有序节点
一个 znode 还可以设置为有序 (sequential) 节点, 一个有序 znode 节点被分配唯一一个单调递增的整数; 有序 znode 通过提供了创建具有唯一名称的 znode 的简单方式, 同时也通过这种方式可以直观的查看 znode 的创建顺序  
总之 znode 有四种类型: 持久的 (persistent), 临时的 (ephemeral), 持久有序的 (persistent sequential), 临时有序的 (ephemeral sequential)

##### 监视与通知
ZooKeeper 通常以远程服务的方式被访问, 如果每次访问 znode 时, 客户端都需要获得节点中的内容, 这样会导致较高的延迟, 而且 ZooKeeper 需要更多的操作; 考虑以下的例子
