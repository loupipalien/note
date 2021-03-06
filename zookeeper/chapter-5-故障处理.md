### 故障处理
故障的发生点主要有三个: ZooKeeper 服务, 网络, 应用程序

#### 可恢复的故障
ZooKeeper 呈现给使用某些状态的所有客户端进程一致性的状态视图, 当一个客户端从 ZooKeeper 获得响应时, 客户端可以非常肯定这个响应信息与其他响应信息或其他客户端所接收的响应保持一致; 客户端与 ZooKeeper 服务的连接丢失, 客户端会使用 DisConnected 事件和 ConnectionLossException 异常来表示自己无法了解当前系统的z状态; 客户端会努力进行重连操作, 一旦重新连接, ZooKeeper 会产生一个 SyncConnected 事件, 还会注册之前已经注册过的监视点, 并会对失去连接这段时间发生的变更产生监视事件  
如果客户端没有进行中的请求, 这种情况只会对客户端产生很小的影响, 紧随 DisConnected 事件之后是 SyncConnected 事件, 客户端不会注意到这个变化, 但是如果存在进行中的请求, 连接丢失就会产生很大的影响

##### 已存在的监视点与 DisConnected 事件
为了使连接断开与重新建立会话之间更加平滑, ZooKeeper 客户端会在新的服务器上重新建立所有已经存在的监视点; 当客户端连接 ZooKeeper 的服务器, 客户端会发送监视点列表和最后已知的 zxid (最终状态时间戳), 服务器会接受这些监视点并检查 znode 节点的修改时间戳与这些监视点是否对应, 如果任何已经监视的 znode 节点的修改时间戳晚于最后已知的 zxid, 服务器就会触发这个监视点

#### 不可恢复的故障
有时一些更糟的事情发生, 导致会话无法恢复而且必须被关闭; 这种情况最常见的原因是会话过期, 另一个原因是已认证的会话无法再次与 ZooKeeper 完成认证, 这两种情况下 ZooKeeper 都会丢弃会话的状态  
处理不可恢复故障的最简单方法就是中止进程并重启, 这样可以使进程恢复原状, 通过一个新的会话重新初始化自己的状态, 如果该进程继续工作, 首先必须要清除与旧会话关联的应用内容的进程状态信息, 然后重新初始化新的状态

#### 群首选举和外部资源
客户端与 ZooKeeper 进行的任何交互操作, ZooKeeper 都会保持同步; 然而 ZooKeeper 无法保护与外部设备的交互操作
```
             create /leader   t1                                t4   t5
Client1 ----------------------------------------------------------------------------->
                  |           ^                                  |    |    ^
                  v           |                                  |    v    |
ZooKeeper ------------------------------------------------------------------------------>
                                    |        ^          |        |
                                    v        |          v        |
Client2 ---------------------------------------------------------------------------------->
                                   t2  create /leader   t3   |   |
                                                             v   v
DB --------------------------------------------------------------------------------------->
t1: 客户端 C1 开始进行垃圾回收
t2: ZooKeeper 声明 C1 终止
t3: 客户端 C2 成为群首, 通过数据库进行同步并更新数据
t4: 之前已经队列化但延迟的更新被发送给数据库
t5: 客户端 C1 重新连接到 ZooKeeper, 发现其会话已经过期
```

#### 小结
