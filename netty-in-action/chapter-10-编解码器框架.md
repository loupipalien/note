### 编解码器框架
#### 什么是编解码器
编码器就是将消息转换为适合于传输的格式, 解码器就是将网络字节流转换回应用程序的消息格式

#### 解码器
Netty 提供的解码器分以下两大类
- 将字节解码为消息: ByteToMessageDecoder 和 ReplayingDecoder
- 将一种消息类型解码为另一种: MessageToMessageDecoder

##### 抽象类 ByteToMessageDecoder
Netty 提供了将字节解码为消息的抽象基类: ByteToMessageDecoder; 由于不知道远程节点是否会一次性发送一个完成的消息, 所以这类会对入站消息进行缓冲, 直到它准备好处理; 以下是它的最重要的两个方法

| 方法 | 描述 |
| :--- | :--- |
| decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) | decode() 方法被调用时将会传入一个包含了传入数据的 ByteBuf, 以及一个用来添加解码消息的 List; 对这个方法的调用将会重复进行, 直到确定没有新的元素被添加到该 List, 或者该 ByteBuf 中没有更多可读取的字节时为止; 然后 List 不为空, 那么它的内容将会被传递给 ChannelPipeline 中的下一个 ChannelInboundHandler |
| decodeLast(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) | 当 Channel 的状态变为非活动状态时, 这个方法将会被调用一次, 可以重写该方法以提供特殊的处理 |

以下展示一个接收 int 的字节流
```
public class ToIntegerDecoder extends ByteToMessageDecoder {

    @Override
    protected void decode(ChannelHandlerContext channelHandlerContext, ByteBuf byteBuf, List<Object> list) throws Exception {
        // 检查是否有足够的字节读取
        if (byteBuf.readableBytes() >= 4) {
            // 从入站 ByteBuf 中读取一个 int, 并将其添加到解码消息的 List 中
            list.add(byteBuf.readInt());
        }
    }
}
```
>**编解码器中的引用计数**
一旦消息被编码或者解码, 它就会被 ReferenceCountUtil.release(msg) 调用而自动释放; 如果需要保留引用以便后续使用, 则可以调用 ReferenceCountUtil.retain(message) 方法, 这会增加该引用计数, 从而防止该消息被释放

##### 抽象类  ReplayingDecoder
ReplayingDecoder 扩展了 ByteToMessageDecoder 类, 使得不必调用 readableBytes() 方法; 它通过使用一个自定义的 ByteBuf 实现, ReplayingDecoderByteBuf 包装了传入的 ByteBuf 实现了这一点, 其将在内部执行 readableBytes() 的调用
```
// 类型参数用于指定状态管理的类型, Void 代表不需要状态管理
pubic class ToIntegerDecoder2 extends ReplayingDecoder<Void> {

    @Override
    protected void decode(ChannelHandlerContext channelHandlerContext, ByteBuf byteBuf, List<Object> list) throws Exception {
        // 传入的 ByteBuf 是 ReplayingDecoderByteBuf
        list.add(byteBuf.readInt());
    }
}
```

##### 抽象类 MessageToMessageDecoder
MessageToMessageDecoder 抽象基类用于在两个消息格式之间进行转换 (例如一种 POJO 类型转换为另一种)
```
public abstract class MessageToMessageDecoder<I> extends ChannelInboundHandlerAdapter
```
其中类型参数 I 制定了 decode() 方法的输入参数 msg 的类型

| 方法 | 描述 |
| :--- | :--- |
| decode(ChannelHandlerContext ctx, I msg, List<Object> out) | 对于每个需要被解码为另一种格式的入站消息来说, 该方法都会被调用; 解码消息随后会被传递给 ChannelPipeline 中的下一个 ChannelInboundHandlerAdapter |

以下代码展示了将 Integer 类型转换为 String 的表示形式
```
// 拓展了 MessageToMessageDecoder<Integer>
public class IntegerToStringDecoder extends MessageToMessageDecoder<Integer> {

    @Override
    protected void decode(ChannelHandlerContext channelHandlerContext, Integer integer, List<Object> list) throws Exception {
        list.add(String.valueOf(integer));
    }
}
```

##### TooLongFrameException 类
Netty 是一个异步框架, 所以需要在字节可以编码之前在内存中缓存它们, 因此不能让解码器缓冲大量数据以至于耗尽可用内存; 为了解除这个顾虑, Netty 提供了 TooLongFrameException 类, 其将有解码器在帧超出指定的大小限制时抛出; 可以设置一个缓存最大字节数的阈值, 当超出时则抛出异常 (随后会被 ChannelHandler.exceptionCaught() 方法捕获处理); 以下是一个示例
```
public class SafeByteToMessageDecoder extends ByteToMessageDecoder {

    private  static final int MAX_FRAME_SIZE = 1024;
    @Override
    protected void decode(ChannelHandlerContext channelHandlerContext, ByteBuf byteBuf, List<Object> list) throws Exception {
        int readable = byteBuf.readableBytes();
        if (readable > MAX_FRAME_SIZE) {
            // 跳过所有可读字节, 抛出异常
            byteBuf.skipBytes(readable);
            throw new TooLongFrameException("Frame too big!");
        }
    }
}
```

#### 编码器
Netty 提供的编码器分以下两大类
- 将消息编码为字节
- 将消息编码为消息

##### 抽象类 MessageToByteEncoder

| 方法 | 描述 |
| :--- | :--- |
| encode(ChannelHandlerContext ctx, I msg, ByteBuf out) | encode() 方法是需要实现的唯一抽象方法, 它被调用时将会传入要被该类编码为 ByteBuf 的 (类型为 I 的) 出站消息; 该 ByteBuf 随后将会被转发给 ChannelPipeline 中的下一个 ChannelOutboundHandler |

以下代码展示如何写出 Short 类型消息
```
public class ShortToByteEncoder extends MessageToByteEncoder<Short> {

    @Override
    protected void encode(ChannelHandlerContext channelHandlerContext, Short aShort, ByteBuf byteBuf) throws Exception {
        // 将 Short 写入 ByteBuf 中
        byteBuf.writeShort(aShort);
    }
}
```

##### 抽象类 MessageToMessageEncoder

| 方法 | 描述 |
| :--- | :--- |
| encode(ChannelHandlerContext ctx, I msg, List<Object> out) | 每个通过 write() 方法写入的消息都将会被传递给 encode() 方法, 以编码一个或多个出站消息; 随后这些出站消息将会被转发给 ChannelPipeline 中的下一个 ChannelOutboundHandler |

#### 抽象的编解码器类
##### 抽象类 ByteToMessageCodec
适用于需要将字节解码为某种形式的消息 (可能是 POJO), 随后再次对它进行编码; 不言而喻, ByteToMessageCodec 实现了 ChannelInboundHandler 和 ChannelOutboundHandler 接口

| 方法 | 描述 |
| :--- | :--- |
| TODO | - |

##### 抽象类 MessageToMessageCodec
MessageToMessageCodec 类可以在单个类中实现转换的往返过程; MessageToMessageCodec 是一个参数化类, 其定义如下
```
public abstract class MessageToMessageCodec<INBOUND_IN,OUTBOUNT_IN>
```

| 方法 | 描述 |
| :--- | :--- |
| TODO | - |

##### CombinedChannelDuplexHandler 类
结合一个解码器和编码器可能会对可重用性造成影响; Netty 提供了一种方式, 即能够避免这种惩罚, 又不会牺牲一个解码器和一个编码器作为一个单独的单元部署所带来的便利性; CombinedChannelDuplexHandler 的声明如下
```
public class CombinedChannelDuplexHandler<I extends ChannelInboundHandler, O extends ChannelOutboundHandler>
```
这个类充当了 ChannelInboundHandler 和 ChannelOutboundHandler 的容器, 通过提供分别继承解码器类和编码器类的类型, 可以实现一个编解码器, 而又不必直接扩展抽象的编解码器类
```
// 通过该解码器和编码器实现参数化 CombinedByteCharCodec
public class CombinedByteCharCodec extends CombinedChannelDuplexHandler<ByteToCharDecoder, CharToByteEncoder> {

    public CombinedByteCharCodec() {
        // 将委托实例传递给父类
        super(new ByteToCharDecoder(), new CharToByteEncoder());
    }
}
```

#### 小结
TODO
