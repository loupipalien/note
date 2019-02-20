### 你的第一款 Netty 应用程序

#### 设置开发环境
TODO

#### Netty 客户端/服务端概览
```
 ----------      ABC     ---------               ---------
| 客户端 1 |  <------->  | 连接 1 |  <--------->  |       |
 ----------      ABC     ---------               |       |
                                                 |       |
 ----------      CDE     ---------               |       |
| 客户端 2 |  <------->  | 连接 2 |  <--------->  | 服务端 |
 ----------      CDE     ---------               |       |      
                                                 |       |
 ----------      XYZ     ---------               |       |
| 客户端 3 |  <------->  | 连接 3 |  <--------->  |       |
 ----------      XYZ     ---------               ---------
```

#### 编写 Echo 服务器
所有的 Netty 服务器都需要以下两部分
- 至少需要一个 ChannelHandler, 该组件实现了服务器从客户端接收的数据的处理, 即它的业务逻辑
- ServerBootstrap, 这是配置服务器的启动代码, 它会将服务器绑定到它要监听连接请求的端口上

##### ChannelHandler 和业务逻辑
ChannelHandler 的实现负责接收并响应事件通知, 在 Netty 应用程序中, 所有的数据处理逻辑都包含在这些核心抽象的实现中  
Echo 服务器会响应传入的消息, 所以需要实现 ChannelInboundHandler 接口, 这里可以直接使用此接口的默认实现 ChannelInboundHandlerAdapter; Echo 服务器中使用到的方法有
- channelRead(): 对每个传入的消息都要调用
- channelReadComplete(): 通知 ChannelInboundHandler 最后一次对 channelRead() 的调用是当前批量读取中的最后一条消息
- exceptionCaught(): 在读取操作期间, 有异常抛出时会调用

EchoServerHandler 代码
```
// Sharable 注解标识一个 ChannelHandler 可以被多个 Channel 安全的共享
@Sharable
public class EchoServerHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        // 将接收到的消息写给发送者, 而不冲刷出站消息
        ctx.write(msg);
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) {
        // 将未决消息冲刷到远程节点, 并且关闭该 Channel
        ctx.flush();
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        // 打印异常
        cause.printStackTrace();
        // 关闭该 Channel
        ctx.close();
    }
}
```  
>**如果不捕获异常, 会发生什么**
每个 Channel 都拥有一个与之相关联的 ChannelPipeline, 其持有一个 ChannelHandler 的实例链; 在默认情况下, ChannelHandler 会把对它的方法调用转发给链中的下一个 ChannelHandler; 因此, 如果 exceptionCaught() 方法没有被该链中的某处实现, 那么所接收的异常将会被传递到 ChannelPipeline 的尾端并被记录; 为此, 应用程序应该至少有一个实现了 exceptionCaught() 方法的 ChannelHandler

请记住以下这些关键点
- 针对不同类型的事件来调用 ChannelHandler
- 应用程序通过实现或扩展 ChannelHandler 来挂钩事件到生命周期, 并且提供自定义的应用程序逻辑
- 在架构上, ChannelHandler 有助于保持业务逻辑和网络处理代码的分离; 这简化了开发过程, 因为代码必须不断地演化以响应不断变化的需求

##### 引导服务器
EchoServerHandler 实现了核心业务逻辑了, 引导服务器涉及以下内容
- 绑定到服务器将在其上监听并接受传入连接请求的端口
- 配置 Channel, 以将有关的入站消息通知给 EchoServerHandler 实例

EchoServer 代码
```
public final class EchoServer {

    static final boolean SSL = System.getProperty("ssl") != null;
    static final int PORT = Integer.parseInt(System.getProperty("port", "8007"));

    public static void main(String[] args) throws Exception {
        // Configure SSL.
        final SslContext sslCtx;
        if (SSL) {
            SelfSignedCertificate ssc = new SelfSignedCertificate();
            sslCtx = SslContextBuilder.forServer(ssc.certificate(), ssc.privateKey()).build();
        } else {
            sslCtx = null;
        }

        // 创建 EventLoopGroup
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        final EchoServerHandler serverHandler = new EchoServerHandler();
        try {
            ServerBootstrap b = new ServerBootstrap();
            b.group(bossGroup, workerGroup)
             // 指定所使用的 NIO 传输的 Channel
             .channel(NioServerSocketChannel.class)
             .option(ChannelOption.SO_BACKLOG, 100)
             .handler(new LoggingHandler(LogLevel.INFO))
             // 添加一个 EchoServerHandler 到子 Channel 的 ChannelPipeline
             .childHandler(new ChannelInitializer<SocketChannel>() {
                 @Override
                 public void initChannel(SocketChannel ch) throws Exception {
                     ChannelPipeline p = ch.pipeline();
                     if (sslCtx != null) {
                         p.addLast(sslCtx.newHandler(ch.alloc()));
                     }
                     // EchoServerHandler 标注了 @Sharable, 即总是使用相同的实例
                     p.addLast(serverHandler);
                 }
             });

            // 异步的绑定服务器, 调用 sync() 方法阻塞直到绑定完成
            ChannelFuture f = b.bind(PORT).sync();

            // 获取 Channel 的 closeFuture, 并阻塞当前线程直到它完成
            f.channel().closeFuture().sync();
        } finally {
            // 关闭 EventLoopGroup, 释放所有的资源
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```

#### 编写 Echo 客户端
Echo 客户端将会
- 连接到服务器
- 发送一个或多个消息
- 对于每个消息, 等待并接收从服务器发回的相同的消息
- 关闭连接

##### 通过 ChannelHandler 实现客户端逻辑
客户端将拥有一个用来处理数据的 ChannelInboundHandler, 这里使用扩展 SimpleChannelInboundHandler 类处理任务, 需要重写以下方法
- channelActive(): 在到服务器的连接已经建立后将被调用
- channelRead0(): 当从服务器接收到一条消息时被调用
- exceptionCaught(): 在处理过程中引发异常时被调用

EchoClientHandler 代码
```
@Sharable // 标记该类的实例可以被多个 Channel 共享
public class EchoClientHandler extends SimpleChannelInboundHandler<ByteBuf> {

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        // 当被通知 Channel 是活跃的时候, 发送一条消息
        ctx.writeAndFlush(Unpooled.copiedBuffer("Netty rocks!", CharsetUtil.UTF_8));
    }

    @Override
    protected void channelRead0(ChannelHandlerContext channelHandlerContext, ByteBuf byteBuf) {
        // 记录已接受的消息的转储
        System.out.println("Client received:" + byteBuf.toString(CharsetUtil.UTF_8));
    }
}
```

##### 引导客户端
EchoClient 代码
```
public final class EchoClient {

    static final boolean SSL = System.getProperty("ssl") != null;
    static final String HOST = System.getProperty("host", "127.0.0.1");
    static final int PORT = Integer.parseInt(System.getProperty("port", "8007"));
    static final int SIZE = Integer.parseInt(System.getProperty("size", "256"));

    public static void main(String[] args) throws Exception {
        // Configure SSL.git
        final SslContext sslCtx;
        if (SSL) {
            sslCtx = SslContextBuilder.forClient()
                .trustManager(InsecureTrustManagerFactory.INSTANCE).build();
        } else {
            sslCtx = null;
        }

        // 指定 EventLoopGroup 用以处理客户端事件
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            Bootstrap b = new Bootstrap();
            b.group(group)
             .channel(NioSocketChannel.class)
             .option(ChannelOption.TCP_NODELAY, true)
             .handler(new ChannelInitializer<SocketChannel>() {
                 @Override
                 public void initChannel(SocketChannel ch) throws Exception {
                     ChannelPipeline p = ch.pipeline();
                     if (sslCtx != null) {
                         p.addLast(sslCtx.newHandler(ch.alloc(), HOST, PORT));
                     }
                     // 向 ChannelPipeline 添加一个 EchoClientHandler 实例
                     p.addLast(new EchoClientHandler());
                 }
             });

            // 连接服务器的
            ChannelFuture f = b.connect(HOST, PORT).sync();

            // 阻塞到知道 Channel 关闭
            f.channel().closeFuture().sync();
        } finally {
            // Shut down the event loop to terminate all threads.
            group.shutdownGracefully();
        }
    }
}
```

#### 构建和运行 Echo 服务器和客户端
TODO

#### 小结
TODO
