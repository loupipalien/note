### 预置的 ChannelHandler 和编解码器

#### 通过 SSL/TLS 保护 Netty 程序
为了支持 SSL/TLS, Java 提供了 javax.net.ssl 包, 它的 SSLContext 和 SSLEngine 类使得实现解密和加密相当直接简单; Netty 通过一个名为 SslHandler 的 ChannelHandler 实现利用了这个 API, 其中 SslHandler 在内部使用 SSLEngine 来完成实际的工作
>**Netty 的 OpenSSL/SSLEngine 的实现**
Netty 还提供了使用 OpenSSL 工具包的 SSLEngine 的实现, 这个 OpenSslEngine 类提供了比 JDK 的 SSLEngine 实现更好的性能; 如果 OpenSSL 库可用, 可将 Netty 应用程序配置为默认使用的 OpenSslEngine, 如果不可用则回退到 JDK 实现

以下是使用 SslHandler 的数据流
```
            加密 (1) --------------  解密 (2)
         ----------> | SslHandler | ---------> INBOUND
OUTBOUND <---------- |            | <---------
            加密 (3) --------------   原始数据 (4)
(1): SslHandler 拦截了加密的入站数据
(2): SslHandler 对数据进行了解密, 并且将它定向到入站端
(3): 出站数据被传递通过 SslHandler
(4): SslHandler 对数据进行了加密, 并且传递给出站端
```
以下代码展示如何使用 ChannelInitializer 将 SslHandler 添加到 ChannelPipeline 中
```
public class SslChannelInitializer extends ChannelInitializer<Channel> {
    private final SslContext context;
    private final boolean startTls;

    public SslChannelInitializer(SslContext context, boolean startTls) {
        // 传入要使用的 SslContext
        this.context = context;
        // 如果设置为 true, 第一个写入的消息将不会被加密 (客户端应该设置为 true)
        this.startTls = startTls;
    }

    @Override
    protected void initChannel(Channel channel) throws Exception {
        // 对于每个 SslHandler 实现, 都使用 Channel 的 ByteBufAllocator 从 SslContext 获取一个新的 SSLEngine
        SSLEngine engine = context.newEngine(channel.alloc());
        // 将 SslHandler 作为第一个 ChannelHandler 添加到 ChannelPipeline 中
        channel.pipeline().addFirst("ssl", new SslHandler(engine, startTls));
    }
}
```
大多数情况下, SslHandler 将是 ChannelPipeline 中的第一个 ChannelHandler, 这确保了只有在所有其他 ChannelHandler 将它们的逻辑应用到数据之后, 才会进行加密; SslHandler 还有一些其他有用的方法, 例如在握手阶段, 两个节点将相互验证并确定一种加密方式; 可以通过配置 SslHandler 来修改它的行为, 或者在 SSL/TLS 握手一旦完成后提供通知, 握手阶段完成后, 所有的数据都将会被加密

| 方法 | 描述 |
| :--- | :--- |
| TODO | - |

#### 构建基于 Netty 的 HTTP/HTTPS 应用程序
##### HTTP 解码器, 编码器和编解码器
HTTP 是基于请求/响应模式的: 客户端向服务器发送一个 HTTP 请求, 然后服务器将会返回一个 HTTP 响应; 下图是 HTTP 的请求部分 (HTPP 的响应部分与其类似)
```
                    FullHttpRequest
-------------------------------------------------------------
|     (1)           (2)                           (3)       |
| HttpRequest   HttpContent   HttpContent   LastHttpContent |
|                                                           |
-------------------------------------------------------------
(1): HTTP 请求的第一个部分, 包含了 HTTP 的头部信息
(2): HttpContent 包含了数据, 后面可能还跟着一个或多个 HttpContent 部分
(3): LastHttpContent 标记了该 HTTP 请求的结束, 可能还包含了尾随的 HTTP 头部信息
```
下表简要的介绍了处理和生成这些消息的 HTTP 编码器和解码器

| 方法 | 描述 |
| :--- | :--- |
| HttpRequestEncoder | 将 HttpRequest, HttpContent, LastHttpContent 消息编码为字节 |
| HttpResponseEncoder | 将 HttpResponse, HttpContent, LastHttpContent 消息编码为字节 |
| HttpRequestDecoder | 将字节解码为 HttpRequest, HttpContent, LastHttpContent 消息 |
| HttpResponseDecoder | 将字节编码为 HttpResponse, HttpContent, LastHttpContent 消息 |

HttpPipelineInitializer 类展示了将 HTTP 支持添加到应用中
```
public class HttpPipelineInitializer extends ChannelInitializer<Channel> {
    private final boolean client;

    public HttpPipelineInitializer(boolean client) {
        this.client = client;
    }

    @Override
    protected void initChannel(Channel channel) throws Exception {
        ChannelPipeline pipeline = channel.pipeline();
        if (client) {
            // 如果是客户端, 则添加 HttpResponseDecoder 以处理来自服务器的响应
            pipeline.addLast("decoder", new HttpResponseDecoder());
            // 如果是客户端, 则添加 HttpRequestEncoder 以向服务器发送请求
            pipeline.addLast("encoder", new HttpRequestEncoder());
        } else {
            // 如果是服务端, 则添加 HttpRequestDecoder 以接收来自客户端的请求
            pipeline.addLast("decoder", new HttpRequestDecoder());
            // 如果是服务端, 则添加 HttpResponseEncoder 以向客户端发送请求
            pipeline.addLast("encoder", new HttpResponseEncoder());
        }
    }
}
```

##### 聚合 HTTP 消息
在 ChannelInitializer 将 ChannelHandler 安装到 ChannelPipeline 中后, 便可以处理不同类型的 HttpObject 消息了; 但是由于 HTTP 的请求和响应可能由许多部分组成, 因此需要聚合它们以形成完整的消息; Netty 提供了聚合器, 可以将多个消息合并为 FullHttpRequest 或者 FullHttpResponse 消息; 由于消息分段需要被缓冲, 直到可以转发一个完整的消息给下一个 ChannelInboundHandler, 所以这个操作是有轻微开销的, 但所带来的好处便是不必关心碎片了
```
public class HttpAggregatorInitializer extends ChannelInitializer<Channel> {
    private final boolean isClient;

    public HttpAggregatorInitializer(boolean isClient) {
        this.isClient = isClient;
    }

    @Override
    protected void initChannel(Channel channel) throws Exception {
        ChannelPipeline pipeline = channel.pipeline();
        if (isClient) {
            pipeline.addLast("codec", new HttpClientCodec());
        } else {
            pipeline.addLast("codec", new HttpServerCodec());
        }
        // 将最大的消息大小为 512 KB 的聚合器添加到 ChannelPipeline
        pipeline.addLast("aggregator", new HttpObjectAggregator(512 * 1024));
    }
}
```

##### HTTP 压缩
当使用 HTTP 时, 建议开启压缩功能以尽可能多的减小传输数据的大小, 虽然压缩会带来一些 CPU 时钟周期上的开销, 但通常来说都是一个好主意, 尤其是对文本数据来说
>** HTTP 请求的头部信息 **
客户端可以通过提供以下头部信息来指示服务器它所支持的压缩格式
GET /encrypted-area HTTP/1.1
Host: www.example.com
Accept-encoding: gzip, deflate
但是需要注意的是, 服务器没有义务压缩它所发送的数据

```
public class  HttpCompressorInitializer extends ChannelInitializer<Channel> {
    private final boolean isClient;

    public HttpCompressorInitializer(boolean isClient) {
        this.isClient = isClient;
    }

    @Override
    protected void initChannel(Channel channel) throws Exception {
        ChannelPipeline pipeline = channel.pipeline();
        if (isClient) {
            pipeline.addLast("codec", new HttpClientCodec());
            pipeline.addLast("decompressor", new HttpContentCompressor());
        } else {
            pipeline.addLast("codec", new HttpServerCodec());
            pipeline.addLast("compressor", new HttpContentCompressor());
        }
    }
}
```

##### 使用 HTTPS
启用 HTTPS 只需要将 SslHandler 添加到 ChannelPipeline 的 ChannelHandler 组合中
```
public class HttpsCodecInitializer extends ChannelInitializer<Channel> {
    private final SslContext context;
    private final boolean startTls;

    public SslChannelInitializer(SslContext context, boolean startTls) {
        this.context = context;
        this.startTls = startTls;
    }

    @Override
    protected void initChannel(Channel channel) throws Exception {
        ChannelPipeline pipeline = channel.pipeline();
        SSLEngine engine = context.newEngine(channel.alloc());
        pipeline.addFirst("ssl", new SslHandler(engine, startTls));

        if (isClient) {
            pipeline.addLast("codec", new HttpClientCodec());
        } else {
            pipeline.addLast("codec", new HttpServerCodec());
        }
    }
}
```

##### WebSocket
WebSocket 提供了在单个的 TCP 连接上提供了双向的通信, 在服务器和客户端之间提供了真正的双向数据交换; 下图展示了客户端和服务端使用 WebSocket 通信的过程
```
                      (1)
----------      -------------      ------------
|        | ---> | HTTP      | ---> |        |
|        |      | WebSocket |      |        |
|        | <--- | 握手      | <--- |        |
| 客户端 |       ------------       | 服务端  |
|        |      -------------      |        |
|        | <--- |           | ---> |        |
|        |      | WebSocket |      |        |
|        | ---> |           | <--- |        |
----------      -------------      ----------
                      (2)
(1): 客户端通过 HTTP(S) 向服务器发起 WebSocket 握手, 并等待确认
(2): 连接协议升级到 WebSocket
```
以下是 WebSocketFrame 协议

| 方法 | 描述 |
| :--- | :--- |
| BinaryWebSocketFrame | 数据帧: 二进制数据 |
| TextWebSocketFrame | 数据帧: 文本数据 |
| ContinuationWebSocketFrame | 数据帧: 属于上一个 BinaryWebSocketFrame 或者 TextWebSocketFramed 的文本的或者二进制数据 |
| CloseWebSocketFrame | 控制帧: 一个 CLOSE 请求, 关闭的状态码以及关闭原因 |
| PingWebSocketFrame | 控制帧: 请求一个 PongWebSocketFrame |
| PongWebSocketFrame | 控制帧: 对 PingWebSocketFrame 请求的响应 |

以下代码展示了一个使用 WebSocketServerProtocolHandler 的示例
```
public class WebSocketServerInitializer extends ChannelInitializer<Channel> {

    @Override
    protected void initChannel(Channel channel) throws Exception {
        channel.pipeline().addLast(
                new HttpClientCodec(),
                new HttpObjectAggregator(65535),
                new WebSocketServerProtocolHandler("/websocket"),
                new TextFrameHandler(),
                new BinaryFrameHandler(),
                new ContinuationFrameHandler()
        );
    }

    public static class TextFrameHandler extends SimpleChannelInboundHandler<TextWebSocketFrame> {

        @Override
        protected void channelRead0(ChannelHandlerContext channelHandlerContext, TextWebSocketFrame textWebSocketFrame) throws Exception {
            // Handle text frame
        }
    }

    public static class BinaryFrameHandler extends SimpleChannelInboundHandler<BinaryWebSocketFrame> {

        @Override
        protected void channelRead0(ChannelHandlerContext channelHandlerContext, BinaryWebSocketFrame binaryWebSocketFrame) throws Exception {
            // Handle binary frame
        }
    }

    public static class ContinuationFrameHandler extends SimpleChannelInboundHandler<ContinuationWebSocketFrame> {

        @Override
        protected void channelRead0(ChannelHandlerContext channelHandlerContext, ContinuationWebSocketFrame continuationWebSocketFrame) throws Exception {
            // Handle continuation frame
        }
    }
}
```

#### 空闲的连接和超时
检测空闲连接以及超时对于及时释放资源来说是至关重要的, Netty 为此提供了几个 ChannelHandler 的实现

| 方法 | 描述 |
| :--- | :--- |
| IdleStateHandler | 当连接空闲时间太长, 将会触发一个 IdleStateEvent 事件, 然后可以通过在 ChannelInboundHandler 中重写 userEventTrigged() 方法来处理该 IdleStateEvent 事件 |
| ReadTimeoutHandler | 如果在指定的时间间隔内没有收到任何的入站数据, 则抛出一个 ReadTimeoutException 并关闭对应的 Channel, 可以通过重写 ChannelHandler 中的 exceptionCaught() 方法来检测该 ReadTimeoutException |
| WriteTimeoutHandler | 如果在指定的时间间隔内没有任何出站数据, 则抛出一个 WriteTimeoutException 并关闭对应的 Channel, 可以通过重写 ChannelHandler 中的 exceptionCaught() 方法来检测该 ReadTimeoutException |

以下代码展示了发送心跳消息到远程节点, 如果在 60 miao 内没有接收或者发送任何的数据, 将如何得到通知; 如果没有响应则连接会被关闭
```
public class IdleStateHandlerInitializer extends ChannelInitializer<Channel> {

    @Override
    protected void initChannel(Channel channel) throws Exception {
        ChannelPipeline pipeline = channel.pipeline();
        // IdleStateHandler 将在被触发时发送一个 IdleStateEvent 事件
        pipeline.addLast(new IdleStateHandler(0, 0, 60, TimeUnit.SECONDS));
        pipeline.addLast(new HeartbeatHandler());
    }

    public static final class HeartbeatHandler extends ChannelInboundHandlerAdapter {
        private static final ByteBuf HEARTBEAT_SEQUENCE = Unpooled.unreleasableBuffer(Unpooled.copiedBuffer("HEARTBEAT", CharsetUtil.ISO_8859_1));

        @Override
        public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
            if (evt instanceof IdleStateEvent) {
                // 发送心跳信息失败时关闭连接
                ctx.writeAndFlush(HEARTBEAT_SEQUENCE.duplicate()).addListener(ChannelFutureListener.CLOSE_ON_FAILURE);
            } else {
                // 不是 IdleStateEvent 则传递给下一个 ChannelHandler
                super.userEventTriggered(ctx, evt);
            }
        }
    }
}
```
如果连接超过 60 秒没有接受或者发送任何的数据, 那么 IdleStateHandler 将会使用一个 IdleStateEvent 事件来调用 fireUserEventTriggered() 方法, HeartbeatHandler 实现了 userEventTriggered() 方法, 如果这个方法检测到 IdleStateEvent 事件, 它将会发送心跳信息, 并且添加一个将在发送操作失败时关闭该连接的 ChannelFutureListener

#### 解码基于分隔符的协议和基于长度的协议
##### 基于分隔符的协议
基于分隔符的 (delimited) 消息协议使用定义的字符来标记消息或者消息段 (通常称为帧) 的开头或结尾; 以下是 Netty 提供的解码器

| 类名 | 描述 |
| :--- | :--- |
| DelimiterBasedFrameDecoder | 使用任何由用户提供的分隔符来提取帧的通用解码器 |
| LineBasedFrameDecoder | 提取由行尾符 (\n 或者 \r\n) 分隔的帧的解码器,这个解码器比 DelimiterBasedFrameDecoder 更快 |

下图展示了当帧由行尾序列 \r\n 分隔是如何被处理的
```
       字节流                   帧          帧
------------------         ----------- -----------
| ABC\r\nDEF\r\n | ------> | ABC\r\n | | DEF\r\n |
------------------         ----------- -----------
```
以下代码展示如何使用 LineBasedFrameDecoder
```
public class LineBasedHandlerInitializer extends ChannelInitializer<Channel> {

    @Override
    protected void initChannel(Channel channel) throws Exception {
        ChannelPipeline pipeline = channel.pipeline();
        // 该 LineBasedFrameDecoder 将提取的帧转发给下一个 ChannelInboundHandler
        pipeline.addLast(new LineBasedFrameDecoder(64 * 1024));
        pipeline.addLast(new FrameHandler());
    }

    public static final class FrameHandler extends SimpleChannelInboundHandler<ByteBuf> {
        // 传入单个帧的内容
        @Override
        protected void channelRead0(ChannelHandlerContext channelHandlerContext, ByteBuf byteBuf) throws Exception {
            // do something
        }
    }
}
```

##### 基于长度的协议
基于长度的协议将通过对它的长度编码到帧的头部来定义帧, 而不是使用特殊的分隔符来标记帧的结束; 以下是 Netty 提供的用于处理此类协议的两个解码器

| 类名 | 描述 |
| :--- | :--- |
| FixedLengthFrameDecoder | 提取在调用构造函数时指定的定长帧 |
| LengthFieldBasedFrameDecoder | 根据编码进帧头部中的长度值提取帧, 该字段的偏移量以及长度在构造函数中指定 |

下图展示了 FixedLengthFrameDecoder 指定帧长为 8 的处理
```
       字节流                   帧          帧
------------------         ---------  --------- ---------  ---------
|     32 字节    | ------> | 8 字节 | | 8 字节 | | 8 字节 | | 8 字节 |
------------------         ---------  --------- ---------  ---------
```

#### 写大型数据
Netty 是异步框架, 其写操作是非阻塞的, 所以即使没有写出所有的数据, 写操作也会在完成时返回并通知 ChannelFuture; 当这种情况发生时, 如果仍然不停的写入, 就有内存耗尽的风险; 所以写大型数据需要准备好处理到远程节点的连接是慢连接的情况, 这种情况会导致内存释放的延迟  
Netty 的 NIO 支持零拷贝特性, 这种特性消除了将文件内容从文件系统移动到网络栈的复制过程; 以下利用零拷贝实现传输一个文件的内容
```
FileInputStream in = new FileInputStream(file);
FileRegion region = new DefaultFileRegion(in.getChannel(), 0, file.length());
channel.writeAndFlush(region).addListener((ChannelFuture future) -> {
    if (!future.isSuccess()) {
        // 处理失败
        Throwable cause = future.cause();
    }
});
```
这个示例只适合于文件内容的传输, 不包括应用程序对数据的任何处理; 在需要将数据从文件系统复制到用户内存时, 可以使用 ChunkedWriteHandler, 它支持异步写大量数据, 又不会导致大量的内存消耗  
Netty 预置了四个 ChankedInput<B> 接口的实现 (其中 B 代表的是 readChunk() 方法返回的类型), 每个都代表了一个将由 ChunkedWriteHandler 处理的不定长度的数据流

| 类名 | 描述 |
| :--- | :--- |
| ChunkedFile | 从文件中逐块获取数据, 当平台不支持零拷贝或者在需要转换数据时使用 |
| ChunkedNioFile | 和 ChunkedFile 类似, 只是它使用了 FileChannel |
| ChunkedStream  | 从 InputStream 中逐块传输内容 |
| ChunkedNioStream | 从 ReadableByteChannel 中逐块传输内容 |

当 Channel 的状态变为活动时, WriteStreamHandler 将会逐块的把来自文件中的数据作为 ChunkedStream 写入
```
public class ChunkedWriteHandlerInitializer extends ChannelInitializer<Channel> {
    private final File file;
    private final SslContext sslCtx;

    public ChunkedWriteHandlerInitializer(File file, SslContext sslCtx) {
        this.file = file;
        this.sslCtx = sslCtx;
    }

    @Override
    protected void initChannel(Channel channel) throws Exception {
        ChannelPipeline pipeline = channel.pipeline();
        pipeline.addLast(new SslHandler(sslCtx.newEngine(channel.alloc())));
        // 添加 ChunkedWriteHandler 以处理作为 ChunkedInput 传入的数据
        pipeline.addLast(new ChunkedWriteHandler());
        pipeline.addLast(new WriteStreamHandler());
    }

    public final class WriteStreamHandler extends ChannelInboundHandlerAdapter {
        @Override
        public void channelActive(ChannelHandlerContext ctx) throws Exception {
            super.channelActive(ctx);
            // 当连接建立时, WriteStreamHandler 就开始写数据
            ctx.writeAndFlush(new ChunkedStream(new FileInputStream(file)));
        }
    }
}
```

#### 序列化数据
##### 使用 JDK 序列化
TODO
##### 使用 JBoss Marshalling 序列化
TODO
##### 使用 Protocol Buffers 序列化
TODO

#### 小结
TODO
