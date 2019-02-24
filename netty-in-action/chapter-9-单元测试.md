### 单元测试
EmbeddedChannel 是 Netty 专门为改进针对 ChannelHandler 的单元测试而提供的

#### EmbeddedChannel 的概述
Netty 提供了所谓的 Embedded 传输, 用于测试 ChannelHandler; 这个传输是一个特殊的实现, 提供了通过 ChannelPipeline 传播事件的简便方法; 这个想法是直截了当的: 将入站数据或者出站数据写到 EmbeddedChannel 中, 然后检查是否有任何东西到达了 ChannelPipeline 的尾端; 以这种方式来确定消息是否已经被编码或者被编码过了, 以及是否触发了任何的 ChannelHandler 动作; 以下是 EmbeddedChannel 的相关方法

| 方法 | 描述 |
| :--- | :--- |
| writeInbound(Object... msg) | 将入站消息写到 EmbeddedChannel 中, 如果可以通过 readInbound() 方法从 EmbeddedChannel 中读取数据, 则返回 true |
| readInbound() | 从 EmbeddedChannel 中读取一个入站消息, 任何返回的东西都穿越了整个 ChannelPipeline; 如果没有任何可供读取的则返回 null |
| writeOutbound(Object... msg) | 将出站消息写到 EmbeddedChannel 中, 如果可以通过 readOutbound() 方法从 EmbeddedChannel 中读取数据, 则返回 true |
| readOutbound() | 从 EmbeddedChannel 中读取一个出站消息, 任何返回的东西都穿越了整个 ChannelPipeline; 如果没有任何可供读取的则返回 null |
| finish() | 将 EmbeddedChannel 标记为完成的, 并且如果有可被读取的入站数据或者出站数据, 则返回 true; 这个方法将还会调用 EmbeddedChannel 上的 close() 方法 |

下图展示了使用 EmbeddedChannel 的方法, 数据是如何流经 ChannelPipeline 的
```
        EmbeddedChannel
        ---------------
        |             |
        |       writeOutbound() -------------------------------------------
        |             |         --- ChannelPipeline --------------------  |
        |             |         |                                      |  |
   readOutbound() <--------------------------- 出站处理器 <-----------------
        |             |         |                                      |
        |       writeInbound() ---> 入站处理器 ------------> 入站处理器 -----  
        |             |         |                                      |  |
        |             |         ----------------------------------------  |
   readInbound() <---------------------------------------------------------
        |             |
        ---------------
```

#### 使用 EmbeddedChannel 测试 ChannelHandler
TODO

#### 测试异常处理
TODO

#### 小结
TODO
