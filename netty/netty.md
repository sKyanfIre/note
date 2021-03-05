### netty

* 传输方式
  * OIO 低延迟，支持并发量小
  * NIO 支持并发量大，几万~几十万并发量
  * Local vm 
  * Embedded

* bootstrap

  * `Bootstrap `

    client/`DatagramChannel`

    1个`EventLoopGroup`

  * `ServerBootstrap`

    server

    2个`EventLoopGroup` 因为server有两组`Channel`,一组处理io的Channel，一组接受连接的Channel

* EventLoopGroup

* Channel

* ChannelFuture

* ChannelPipeline

* ChannelHandlerContext

* ChannelConfig

* ByteBufHolder

* ByteBuf

  * 双指针 write index/ read index
  * 零拷贝
  * 支持自定义Buffer
  * 不需要调用 flip()
  * 引用计数

* ByteBufProcessor

* ByteBufPool

* MessageBuf

* ByteBufAllocator

  获取实例的二种方式

  * Channel
  * ChannelHandlerContext

* Unpooled

* ByteBufUntil `hexdump()`

* ReferenceCountUtil `release()`

* Decoder Encoder Codec
  * ByteToMessageDecoder
  * MessageToByteEncoder
  * ByteToMessageCodec/MessageToMessageCodec
  * CombinedChannelDuplexHandler
  
* Serialization

  * ProtoBuf
  
* test 

  ​	EmbeddedChannel

  * writeInbound()
  * readInbound()
  * writeOutbound()
  * readOutbound()
  * finsh()

  ```java
  EmbeddedChannel test = new EmbeddedChannel(new CustomChannelHandler());
  try{
      test.writeInbound(byteBuf);
  }catch(Exception e) {
      // handler中抛出的异常可以直接捕获
  }
  ```

  

