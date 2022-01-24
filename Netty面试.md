## Netty面试

### 1、Netty是什么

1）Netty是一个异步事件驱动的网络应用程序框架，可以快速开发可维护、高性能协议的服务器和客户端。

Netty是基于nio的，它封装了jdk的NIO，使用起来更加简单。（一点就够了）

2）简化并优化了TCP和UDP套接字服务器等网络编程
![image-20220123141841076](https://user-images.githubusercontent.com/59955759/150748630-edeb735a-997f-4986-931b-5bd7e60c85f4.png)



### Netty的特点是什么？

高并发：Netty是基于NIO的网络通信框架，相比BIO，他的并发性能得到很多提高

传输快：Netty的传输依赖于零拷贝特性，尽量减少不必要的内存拷贝，实现了更高效率传输

封装好：封装了NIO的很多细节，提供易于使用的调用接口。

### 2、为什么要用Netty（优势）

使用简单：封装了NIO的很多细节

功能强大：有很多种编解码器，支持多种主流协议

定制能力强：可以使用ChannelHandler对通信框架进行灵活的扩展

性能高：与其他NIO框架对比，Netty的性能最优

稳定：修复了已经发现的所有NIO的BUG

社区活跃：Netty是活跃的开源项目

### 3、Netty的应用场景

说法一

1）互联网行业

分布式系统中，节点间的通信一般使用的是RPC框架。Netty作为通信组件，被这些RPC框架所调用

像dubbo的RPC框架

2）游戏行业

3）大数据领域

说法二

很多开源项目比如我们常用的 Dubbo、RocketMQ、Elasticsearch、gRPC 等等都用到了 Netty。



### Java BIO

同步并阻塞，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不作任何事情会造成不必要的线程开销。

![BIO](https://segmentfault.com/img/remote/1460000037714809)

### Java NIO

同步非阻塞，服务器实现模式为一个线程处理多个请求(连接)，即客户端发送的连接请求会被注册到多路复用器上，多路复用器轮询到有 I/O 请求就会进行处理

![NIO](https://segmentfault.com/img/remote/1460000037714808)

### NIO三大核心组件

Buffer、Channel、Selector

### 4、Netty 核心组件有哪些？分别有什么作用？

Channel：

Channel接口是Netty网络操作的抽象类，它除了包括基本的 I/O 操作，如 bind、connect、read、write 之外，还包括了 Netty 框架相关的一些功能，如获取该 Channel的 EventLoop。

比较常用的**Channel**接口实现类是**NioServerSocketChannel**（服务端）和**NioSocketChannel**（客户端），这两个 **Channel** 可以和 **BIO** 编程模型中的**ServerSocket**以及**Socket**两个概念对应上。**Netty** 的 **Channel** 接口所提供的 **API**，大大地降低了直接使用 **Socket** 类的复杂性。



EventLoop（事件循环）：

“EventLoop 定义了 Netty 的核心抽象，用于处理连接的生命周期中所发生的事件”。----《Netty实战》

说白了，**EventLoop 的主要作用就是负责监听网络事件并调用事件处理器进行相关 I/O 操作。**



每当有一个连接到来，Netty就会创建一个Channel，然后从EvenLoopGroup中分配一个EvenLoop来给这个Channel绑定上，在该Channel的整个生命周期中都是由这个绑定的EvenLoop来服务的

ChannelFuture

**Netty** 是异步非阻塞的，所有的 I/O 操作都为异步的。我们可以通过ChannelFuture的addListener()注册一个ChannelFutureListener监听事件，当操作成功或者失败，监听就会自动触发结果。



ChannelHandler

ChannelHandler是消息的具体处理器。负责处理读写操作、客户端连接等事件。



ChannelPipeline

ChannelPipeline为ChannelHandler链提供了容器。当Channel被创建时，它会自动分配到它专属的ChannelPipeline

![image-20220123210556289](https://user-images.githubusercontent.com/59955759/150748866-7d636283-ac36-4350-8700-fa7be00a8254.png)
![image-20220123210628022](https://user-images.githubusercontent.com/59955759/150748915-03874e71-a5b3-474f-b2a2-a082501e9c03.png)


### 5、EventloopGroup 了解么?和 EventLoop 啥关系?

NioEventLoopGroup，主要管理eventLoop的生命周期，可以理解为一个线程池，内部维护了一组线程，每个线程（NioEventLoop）复制处理多个Channel上的事件，而一个Channel只对应于一个线程



### 6、Netty的线程模型？

单Reactor单线程：所有的I/O操作都由一个线程完成，需要执行处理所有的 accept、read、decode、process、encode、send 事件。对于高负载、高并发，并且对性能要求比较高的场景不适用。
![image-20220124133544796](https://user-images.githubusercontent.com/59955759/150749022-a9276b2a-3ae9-4921-bb3a-f20ba8f11a4c.png)


单Reactor多线程：一个NIO线程（ Acceptor 线程）只负责处理所有事务的监听和响应，一个 NIO 线程池负责具体处理：accept、read、decode、process、encode、send 事件。满足绝大部分应用场景，并发连接量不大的时候没啥问题，但是遇到并发连接大的时候就可能会出现问题，成为性能瓶颈。

![image-20220124133610855](https://user-images.githubusercontent.com/59955759/150749057-9aaa8f5a-fbee-4c75-9754-780bae30a82a.png)

主从Reactor多线程

（MainReactor）主线程只负责处理连接，并将连接分配给子线程

（SubReactor）子线程（有多个）复制处理请求，并进行转发

Worker线程池负责具体 的业务处理

![image-20220124134203527](https://user-images.githubusercontent.com/59955759/150749114-bd1572bc-48f6-4ce3-8084-eae8ad5ba592.png)

3中模式用生活案例来理解

单Reactor单线程：前台接待员和服务员是同一个人，全程为顾客服务

单Reactor多线程，一个前台接待员，多个服务员，接待员只负责接待

主从Reactor多线程，多个前台接待员，多个服务生

### 7、TCP 粘包/拆包的原因及解决方法？

TCP是面向数据流的。发送端将多次间隔小且数据量小的数据，合并成一个大的数据块，然后进行封包。这样做虽然提高了效率，但是接收端就难于分辨出完整的数据包了，因为面向流的通信无消息保护边界的。

由于TCP无消息保护边界，需要在接收端处理消息边界问题，也就是我们所说的拆包、粘包问题

解决方案：

使用自定义协议+编解码器来解决

### 



### 8、Netty 长连接、心跳机制了解么？

#### TCP 长连接和短连接了解么？

TCP在进行读写时，client和server之间必须建立一个连接。建立连接需要三次握手，释放/关闭连接时需要四次挥手。这个过程比较消耗网络资源，并且有时间延迟。

短连接：client和server建立连接之后，读写完成就关闭连接。如果下次要发送信息，就要从新建立连接。有点：管理和实现比较简单。缺点：每次建立连接都会消耗大量的网络资源，连接建立还需要耗费时间

长连接：读写完成之后不主动关闭连接，后续读写操作仍使用这个连接。长连接减少了对网络资源的依赖节省时间。对于频繁请求资源的用户，非常适合长连接。

#### 9、为什么需要心跳机制？Netty中心跳机制了解吗？

在TCP保持长连接的过程中，可能会出现断网等网络异常，异常发生时，client与server如果没有交互的话，他们是无法感知对方是否已经掉线的。为了解决这个问题，就引入了心跳机制。

心跳机制的工作原理：在clent与server之间如果一定时间内没有数据交互时，即除于idle状态时，客户端或服务端就会发送一个特殊的数据包（ping）给对方，当接收方收到这个数据报文后，也立即发送一个特殊的数据报文（pong），回应发送方，此即一个PING-PONG交互。所以，当某一端收到心跳消息后，就知道了对方仍然在线。这就确保了TCP连接的有效性。

TCP实际上了就自带长连接选项SO_KEEYALIVE。但是TCP协议层面的长连接灵活性不够。所以，一般情况下我们都是在应用层协议上实现自定义的心跳机制，也就是Netty层面通过编码实现。通过Netty实现心跳机制的核心类是IdleStateHandler

### 10、什么是 Netty 的零拷贝？

在 OS 层面上的 Zero-copy 通常指避免在 用户态(User-space) 与 内核态(Kernel-space) 之间来回拷贝数据。而在 Netty 层面 ，零拷贝主要体现在对于数据操作的优化。

**Netty 中的零拷贝体现在以下几个方面：**

使用Netty提供的CompositeByteBuf类，可以将多个ByteBuf合并为一个逻辑上的ByteBuf。用户可以直接操作合并成的那个ByteBuf

ByteBuf 支持 slice 操作, 因此可以将 ByteBuf 分解为多个共享同一个存储区域的 ByteBuf, 避免了内存的拷贝（只是用一个共享内存，不用每个ByteBuf都拷贝一份共享内存）

通过 FileRegion 包装的FileChannel.tranferTo 实现文件传输, 可以直接将文件缓冲区的数据发送到目标 Channel, 避免了传统通过循环 write 方式导致的内存拷贝问题.

### Netty 服务端和客户端的启动过程了解么？

服务端

两个事件循环组

启动器 

设置Channel类型

设置参数

添加工作线程的handler，通过socketChannel的pipenane

bing（host，port）

客户端

一个事件循环组

启动器

Channel类型

设置参数

设置handler

connect（）

#### NioEventLoopGroup 默认的构造函数会起多少线程？

核数 * 2

## Bootstrap 和 ServerBootstrap 了解么？



### Netty 支持哪些心跳类型设置？

IdleStateHandler

