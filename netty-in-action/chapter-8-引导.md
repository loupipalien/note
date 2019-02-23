### 引导

#### Bootstrap 类
引导类的层次结构包括一个抽象的父类和两个具体的引导子类, 两个具体类分别对应于服务端和客户端; 服务端致力于使用一个父 Channel 来接受客户端的连接, 并创建子 Channel 用于它们之间的通信; 客户端可能只需要一个单独的, 没有父 Channel 的 Channel 来用于所有的网络交互
>**为什么引导类是 Cloneable 的**
因为有时可能需要创建多个具有类似配置或者完全相同配置的 Channel, AbstractBootstrap 被标记为 Cloneable, 就可以在一个已完成引导类实例上调用 clone() 方法返回一个立即可用的引导类实例; 需要注意的是这种方式创建的引导类实例的 EventLoopGroup 是一个浅拷贝, 所以后者将在所有克隆的 Channel 实例之间共享

AbstractBootstrap 类的完整声明如下, 其中子类型是父类型的一个类型参数
```
public abstract class AbstractBootstrap<B extends AbstractBootstrap<B,C>, C extends Channel>
```

#### 引导客户端和无连接协议
Bootstrap 类用于客户端或者使用了无连接协议的应用程序中; 以下是它的部分 API 列表

| 方法 | 描述 |
| :--- | :--- |
| TODO | - |

##### 引导客户端
Bootstrap 类负责为客户端和使用无连接的应用程序创建 Channel
```
-------------                  -----------
|       (1):bind()       --->  | Channel |
|           |                  -----------
| Bootstrap |               
|           |                  -----------
|      (2):connect()     --->  | Channel |
-------------                   -----------
(1): Bootstrap 类将会在 bind() 方法被调用后创建一个新的 Channel, 在这之后将会调用 connect() 方法以创建连接
(2): 在 connect() 方法被调用后, Bootstrap 类将会创建一个新的 Channel
```

##### Channel 和 EventLoopGroup 的兼容性
对于 NIO 以及 OIO 传输两者来说, 都有相关的 EventLoopGroup 和 Channel 实现
```
channel
   |
   |--- nio: NioEventLoopGroup
   |
   |--- oio: OioEventLoopGroup
   |
   |--- socket
           |
           |--- nio: NioDatagramChannel, NioServerSocketChannel, NioSocketChannel
           |
           |--- oio: OioDatagramChannel, OioServerSocketChannel, OioSocketChannel
```
必须保持这种兼容性, 不能混用不同前缀的组件
>**关于 IllegalStateException 的更多讨论**
在引导的过程中, 在调用 bind() 或者 connect() 方法之前, 必须调用以下方法来设置所需的组件
group(), channel() 或者 channelFactory, handler(); 如果不这样做, 则会导致 IllegalStateException, 对 handler() 方法的调用尤其重要, 因为它需要配置好 ChannelPipeline

#### 引导服务器

##### ServerBootstrap 类
以下是它的部分 API 列表

| 方法 | 描述 |
| :--- | :--- |
| TODO | - |

##### 引导服务器
ServerChannel 的实现负责创建子 Channel, 这些子 Channel 代表了以被接受的连接, 而负责引导 ServerChannel 的 ServerBootstrap 提供了这些方法, 以简化将设置应用到已被接受的子 Channel 的 ChannelConfig 的任务  
下图展示了 ServerBootstrap 在 bind() 方法调用时创建了一个 ServerChannel, 并且该 ServerChannel 管理了多个子 Channel
```
---------------------                       
|                   |               -----------------          -------------------------
| ServerBootstrap  bind()  ------>  | ServerChannel | -------> | Channel, Channel, ... |
|                   |               -----------------          -------------------------
--------------------                        
```

##### 从 Channel 引导客户端
假设服务器正在处理一个客户端的请求, 这个请求需要它充当第三方系统的客户端, 在这种情况下需要从已经被接受的子 Channel 中引导一个客户端 Channel; 可以通过创建一个新的 Bootstrap 实例, 但这不是最高效的方式, 因为它将要求为每个新创建的客户端 Channel 定义另一个 EventLoop, 这会产生额外的线程, 以及在已被接受的子 Channel 和客户端 Channel 之间交换数据时不可避免的上下文切换; 一个更好的解决方案是, 
