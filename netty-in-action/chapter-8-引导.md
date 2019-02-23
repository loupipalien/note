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
假设服务器正在处理一个客户端的请求, 这个请求需要它充当第三方系统的客户端, 在这种情况下需要从已经被接受的子 Channel 中引导一个客户端 Channel; 可以通过创建一个新的 Bootstrap 实例, 但这不是最高效的方式, 因为它将要求为每个新创建的客户端 Channel 定义另一个 EventLoop, 这会产生额外的线程, 以及在已被接受的子 Channel 和客户端 Channel 之间交换数据时不可避免的上下文切换; 一个更好的解决方案是, 通过将一被接受的子 Channel 的 EventLoop 传递给 Bootstrap 的 group() 方法来共享 EventLoop, 因为分配给 EventLoop 的所有 Channel 都使用相同的一个线程, 所以避免了额外的线程的创建以及线程上下文切换
```
---------------------                       
|        (1)        |               ---------------------     
| ServerBootstrap  bind()  ------>  | ServerChannel (2) |
|                   |               ---------------------
--------------------                        |
                                            |
        -------------------------------------                                            
        |
        v                  --------------
  ---------------          |            |                 ---------------
  | Channel (3) |  ------> | Bootstrap connect() -------> | Channel (5) |
  ---------------          |            |  (4)            ---------------
        |                   --------------                     |
        |                                                      |
        |                   --------------                     |
        ------------------> | EventLoop  | <--------------------
                            --------------
(1): 在 bind() 方法被调用时, ServerBootstrap 将创建一个新的 ServerChannel
(2): ServerChannel 接受新的连接, 并创建子 Channel 来处理它们
(3): 为已被接受的连接创建子 Channel
(4): 由子 Channel 创建的 Bootstrap 类的实例将在 connect() 方法被调用时创建新的 Channel
(5): EventLoop 在 ServerChannel 所创建子 Channel 以及由 connect() 方法创建 Channel 之间共享             
```
实现 EventLoop 共享涉及通过调用 group() 方法来设置 EventLoop
```
// 设置 EventLoopGroup, 其将提供用以处理 Channel 事件的 EventLoop
new ServerBootstrap().group(new NioEventLoopGroup(), new NioEventLoopGroup())
    .channel(NioServerSocketChannel.class)
    .childHandler(new SimpleChannelInboundHandler<ByteBuf>() {
        ChannelFuture connectFuture;

        @Override
        public void channelActive(ChannelHandlerContext ctx) throws Exception {\
            // 创建一个 Bootstrap 实例以连接远程主机
            Bootstrap bootstrap = new Bootstrap();
            connectFuture = bootstrap.channel(NioSocketChannel.class)
                .handler(new SimpleChannelInboundHandler<ByteBuf>() {
                    @Override
                    protected void channelRead0(ChannelHandlerContext channelHandlerContext, ByteBuf byteBuf) throws Exception {
                        System.out.println("Receive data.");
                    }
                }).group(ctx.channel().eventLoop()) // 使用与分配给被接受的子 Channel 相同的 EventLoop
                .connect(new InetSocketAddress("www.manning.com", 80));
        }

        @Override
        protected void channelRead0(ChannelHandlerContext channelHandlerContext, ByteBuf byteBuf) throws Exception {
            if (connectFuture.isDone()) {
                // do something with the data
            }
        }
    }).bind(new InetSocketAddress(8080))  // 绑定该 ServerSocketChannel
    .addListener(new ChannelFutureListener() {
        @Override
        public void operationComplete(ChannelFuture channelFuture) throws Exception {
            if (channelFuture.isSuccess()) {
                System.out.println("Server bind.");
            } else {
                System.err.println("Bind failed.");
                channelFuture.cause().printStackTrace();
            }
        }
    });
```
在 Netty 中尽可能的重用 EventLoop, 以减少线程创建所带来的开销

#### 在引导过程中添加多个 ChannelHandler
Netty 提供了一个特殊的 ChannelInboundHandlerAdapter 的子类, ChannelInitializer, 它的 initChannel() 方法提供了将多个 ChannelHandler 添加到一个 ChannelPipeline 中的能力; 只需要简单的向 ServerBootstrap 或者 Bootstrap 实例提供 ChannelInitializer 实现即可, 一旦 Channel 被注册到了它的 EventLoop 之后, 就会调用 ChannelInitializer 实现的 initChannel() 方法, 在该方法返回后, ChannelInitializer 的实例就会从 ChannelPipeline 中移除自己

#### 使用 Netty 的 ChannelOption 和属性
每个 Channel 创建时都手动配置可能会相当复杂, 这可以通过使用 option() 方法来将 ChannelOption 应用到引导, 所提供的值会被自动应用到引导所创建的所有 Channel; 可用的 ChannelOption 包括了底层连接的详细信息, 如 keep-alive 或者超时属性以及缓冲区设置, 如果没有对应的预置属性和数据, Netty 提供了 AttributeMap 抽象以及 AttributeKey<T>, 使用这些工具可以将任何类型数据项与 Channel 相关联了

#### 引导 DatagramChannel
除了基于 TCP 协议的 ServerSocket, Bootstrap 类也可以用于无连接协议, Netty 提供了各种 DatagramChannel 的实现; 唯一的区别就是不再调用 connect() 方法, 而是只调用 bind 方法
```
new Bootstrap().group(new OioEventLoopGroup())
    .channel(OioDatagramChannel.class)
    .handler(new SimpleChannelInboundHandler<ByteBuf>() {
        @Override
        protected void channelRead0(ChannelHandlerContext channelHandlerContext, ByteBuf byteBuf) throws Exception {
            // do something with the data
        }
    }).bind(new InetSocketAddress(0))  // 调用 bind() 方法, 因为该协议是无连接的
    .addListener((ChannelFuture future) -> {
        if (future.isSuccess()) {
            System.out.println("Server bind.");
        } else {
            System.err.println("Bind failed.");
            future.cause().printStackTrace();
        }
    });
```

#### 关闭
关闭 Netty 最重要的是关闭 EventLoopGroup, 它将处理任何挂起的事件和任务, 并随后释放所有活动的线程, 这就是调用 EventLoopGroup.shutdownGracefully() 方法的作用; 需要注意的是该方法是一个异步方法, 所以需要阻塞等待直到它完成, 或者向返回的 Future 注册一个监听器以在关闭完成时获得通知

#### 小结
TODO
