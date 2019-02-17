### Netty-异步和事件驱动

#### Java 网络编程
阻塞 I/O 示例
```
// 创建一个新的 ServerSocket, 用以监听指定端口上的连接请求
ServerSocket serverSocket = new ServerSocket(portNumber);
// (1): 对 accept() 方法的调用被阻塞, 直到一个连接建立
Socket clientSocket = serverSocket.accept();
// (2): 这些流对象都派生与该套接字的流对象
BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true);
String request, response;
// (3): 处理循环开始
while ((request = in.readLine()) != null) {
    // 如果客户端发送了 "Done", 则退出循环
    if ("Done".equals(request)) {
        break;
    }
    // (4): 请求被传递给服务器的处理方法
    response = processRequest(request);
    // 服务器的响应被发送给了客户端
    out.println(response);
} // 继续执行处理循环
```
以上代码实现了 Socket API 的基本模式之一, 需要注意以下几点
- ServerSocket 上的 accept() 方法将会一直阻塞到一个连接建立 (1), 随后返回一个新的 Socket 用于客户端和服务端之间的通信; 该 ServerSocket 将继续监听传入的连接
- BufferedReader 和 PrintWriter 都衍生于 Socket 的输入输出流 (2)
- readLine() 方法将会阻塞, 直到在 (3) 处一个由换行符或者回车符结尾的字符串被读取
- 客户端的请求已经被处理

这段代码将只能同时处理一个连接, 要管理多个并发的客户端, 需要为每个新的客户端 Socket 创建一个新的 Thread
```
----------    ----------     ----------
| Socket |    | Socket |     | Socket |
----------    ----------     ----------
    |             |              |
    v             v              v
----------    ----------     ----------
|  读写  |    |  读写  |     |  读写  |
----------    ----------     ----------
    |             |              |
    v             v              v
----------    ----------     ----------
| Thread |    | Thread |     | Thread |
----------    ----------     ----------
```
这种方案可以管理多个并发客户端, 但是也存在着大量的资源浪费
- 在任何时候都可能有大量的线程处于休眠状态, 知识等待输入或者输出数据就绪
- 需要为每个线程的栈分配内存, JDK 线程栈默认值大小围日 255KB
- JVM 可以支持非常多的线程, 但是远在到达该极限之前, 上下文切换所带来的开销就会十分耗时

所以多线程并发方案对于支持中小数量的客户端来说还可以接受, 但是更大数量级的并发连接所需要的资源并不理想

##### Java NIO
本地套接字库也提供了非阻塞调用 (java.nio 包), 其为网络资源的利用率提供了相当多的控制
- 可以使用 setsockopt() 方法配置套接字, 以便读写调用在没有数据的时候立即返回
- 可以使用操作系统的事件通知 API 注册一组非阻塞套接字, 以确定它们中时候有任何的套接字已经有数据可供读写

>**新的还是非阻塞的**
NIO 最开始是新的输入/输出 (New Input/Output) 的英文缩写, 但是该 Java API 已经出现足够长的时间了, 不再是新的了; 因此, 如今大多数的用户认为 NIO 代表非阻塞 I/O (Non-blocking I/O), 而阻塞 I/O (blocking I/O) 是旧的输入/输出 (old input/output, OIO), 也被称作普通 I/O (plain I/O)

##### 选择器
```
----------    ----------     ----------
| Socket |    | Socket |     | Socket |
----------    ----------     ----------
    |             |              |
    v             v              v
----------    ----------     ----------
|  读写  |    |  读写  |     |  读写  |
----------    ----------     ----------
    |             |              |
    ------------------------------
                  |
              ----------
              |Selector|
              ----------
                  |
              ----------
              | Thread |
              ----------      
```
Selector 是 Java 的非阻塞 I/O 实现的关键; 它使用了事件通知 API 以确定在一组非阻塞套接字中有哪些已经就绪能够进行 I/O 相关的操作, 可以在任何时间检查任意的读操作或写操作的完成状态, 由此单个线程便可以处理多个并发的连接; 这种模型提供了更好的资源管理
- 使用较少的线程便可以处理许多连接, 因此也减少了内存管理和上下文切换所带来的开销
- 当没有 I/O 操作需要处理的时候, 线程也可以被用于其他任务

但在高负载下可靠和高效的处理和调度 I/O 操作是一项繁琐且容易出错的任务, 最好由以封装好的 Netty 操作

#### Netty 简介
Netty 的特性总结
| 分类 | Netty 的特性 |
| :--- | :--- |
| 设计 | 统一的 API, 支持多种传输类型, 阻塞的和非阻塞的; 简单而强大的线程模型; 真正的无连接数据报套接字支持; 链接逻辑组件以支持复用|
|易于使用|详实的 Javadoc 和大量示例集|
|性能|拥有比 Java 的核心 API 更高的吞吐量以及更低的延迟; 得益于池化和复用, 拥有更低的资源消耗; 最少的内存复制|
|健壮性|不会因为慢速, 快速和超载的连接而导致 OutOfMemoryError; 消除在高速网络中 NIO 应用程序常见的不公平读写比率|
|安全性|完整的 SSL/TLS 以及 StartTLS 支持; 可用于受限环境下, 如 Applet 和 OSGI|
|社区驱动|发布快速且频繁|

##### 谁在使用 Netty
TODO

##### 异步和事件驱动
一个既是异步的又是事件驱动的系统会表现出一种特殊的, 极其有价值的行为: 它可以是任意的顺序响应在任意的时间点产生的事件  
这种能力对于实现最高级别的可伸缩性至关重要, 定义为: "一种系统, 网络或者进程在需要处理的工作不断增长时, 可以通过某种可行的方式或者扩大它的处理能力来适应这种增长的能力"; 异步和可伸缩性之间的联系是
- 非阻塞网络调用使得可以不必等待一个操作的完成, 完全异步的 I/O 正是基于这个特性构建的, 并且更进一步的是: 异步方法会立即返回, 并且在它完成时会直接或者在稍后的某个时间点通知用户
- 选择器使得能够通过较少的线程便可监视许多连接上的事件

将这些元素结合在一起, 与使用阻塞 I/O 来处理大量时间相比, 使用阻塞 I/O 处理来的更快更经济

#### Netty 的核心组件
Netty 的主要构件块
- Channel
- 回调
- Future
- 事件和 ChannelHandler

这些构建块代表了不同类型的构造: 资源, 逻辑, 通知; 应用程序将使用它们来访问网络以及流经网络的数据

##### Channel
Channel 是 Java NIO 的一个基本构造: 它是一个到实体 (如一个硬件设备, 一个文件, 一个网络套接字或者一个能够执行一个或者多个不同的 I/O 操作的程序组件) 的开放连接, 如读操作和写操作; 可以把 Channel 看作是传入 (入站) 或者传出 (出站) 数据的载体

##### 回调
一个回调其实就是一个方法, 一个指向已经被提供给另外一个方法的方法的引用; 这使得后者可以在适当的时候调用前者; Netty 在内部使用了回调来处理事件, 当一个回调被触发时, 相关的事件可以被接口 ChannelHandler 的一个实现处理; 以下示例展示当一个新的连接被建立时, ChannelHandler 的 channelActive() 回调方法被调用
```
public class ConnectHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        // 当一个新的连接已经被建立时, channelActive() 将会被调用
        System.out.println("Client" + ctx.channel().remoteAddress() + "connected");
    }
}
```

##### Future
Future 提供了另一种在操作完成时通知应用程序的方式, 这个对象可以看作是一个异步操作的结果占位符, 它将在未来的某个时刻完成, 并提供对其结果的访问  
JDK 预置了 Future 接口, 但是其所提供的实现, 只允许手动检查对应的操作是否已经完成, 或者一直阻塞到它完成; 这是非常繁琐的, 所以 Netty 提供了它自己的实现 ChannelFuture, 用于在执行异步操作的时候使用  
ChannelFuture 提供了几种额外的方法, 这些方法使得能够注册一个或多个 ChannelFutureListener 实例, 监听器的回调方法 operationComplete() 将会在对应的操作完成时被调用  
每个 Netty 的出站 I/O 操作都将返回一个 ChannelFuture, 这就是说这些操作都不会被阻塞; 以下示例展示一个 ChannelFuture 作为一个 I/O 操作的一部分返回的例子, connect() 方法将会直接返回而不一会阻塞, 该调用将会在后台完成, 连接何时被完成取决于若干因素, 但是这个关注点已经从代码中抽象出来了; 因为线程不用阻塞以等待对应的操作完成, 所以它可以同时做其他的工作, 从而有效的利用资源
```
Channel channel = ...;
// 异步连接到远程节点, 非阻塞
ChannelFuture future = channel.connect(new InetSocketAddress("localhost", 8080));
// 注册一个 ChannelFutureListener 以便在操作完成时获得通知
future.addListener(new ChannelFutureListener() {
    @Override
    public void operationComplete(ChannelFuture future) throws Exception {
        if (future.isSuccess()) {
            // 如果操作是成功的, 则创建一个 ByteBuf 以持有数据
            ByteBuf buffer = Unpooled.copiedBuffer("Hello", Charset.defaultCharset());
            // 将数据异步的发送到远程节点, 返回一个 ChannelFuture
            ChannelFuture wf = future.channel().writeAndFlush(buffer);
        } else {
            // 如果发生错误, 则访问描述原因的 Throwable
            Throwable cause = future.cause();
            cause.printStackTrace();
        }
    }
});
```
ChannelFutureListener 可以看作是一个更精细的回调版本

##### 事件和 ChannelHandler
Netty 使用不同的事件来通知状态的改变或者是操作的状态, 这使得能够基于已经发生的事件来触发适当的动作, 这些动作可能是:
- 记录日志
- 数据转换
- 流控制
- 应用程序逻辑

Netty 是一个网络编程框架, 所以事件是按照它们与入站或出站数据流的相关性进行分类的, 可能由入站数据或者相关的状态更改而触发的事件包括
- 连接已被激活或者连接失活
- 数据读取
- 用户事件
- 错误事件
出站事件是未来将会触发的某个动作的操作结果, 这些动作包括
- 打开或者关闭到远程节点的连接
- 将数据写到或者冲刷到套接字

每个事件都可以被分发给接口 ChannelHandler 的一个实现的方法, 这是一个很好的将事件驱动范式直接转换为应用程序构件块的例子
```
<------- 出站事件 --- 出站处理器 --- 出站事件 --- 出站处理器 --- 出站事件

入站事件 --- 入站处理器 --- 入站事件 --- 入站处理器 --- 入站事件 ------->
```
Netty 的 ChannelHandler 为处理器提供了基本的抽象, 并提供了大量的开箱即用的 ChannelHandler 实现

##### 把它们放在一起
###### Future, 回调和 ChannelHandler
Netty 的异步编程模型是建立在 Future 和回调的概念之上的, 而将事件派发到 ChannelHandler 的方法则发生在更深的层次; z这些结合在一起提供了一个处理环境, 使得应用程序逻辑可以独立于任何网络操作相关的顾虑而独立的演变  
拦截操作以及高速转换入站数据和出站数据, 都只需要提供回调或者利用操作所返回的 Future, 这使得链接操作变得简单又高效, 并促进了可重用代码的编写
###### 选择器, 事件和 EventLoop
Netty 通过触发事件将 Selector 从应用程序中抽象出来, 消除了所有本来需要手动编写的派发代码; 在内部, 将会为每个 Channel 分配一个 EventLoop, 用以处理所有事件, 包括
- 注册感兴趣的事件
- 将事件派发给 ChannelHandler
- 安排进一步的动作

EventLoop 只由一个线程驱动, 其处理一个 Channel 的所有 I/O 事件, 并且在该 EventLoop 的整个生命周期内都不会改变, 这个简单而强大的设计消除了可能有的在 ChannelHandler 实现中需要进行同步的任何顾虑
