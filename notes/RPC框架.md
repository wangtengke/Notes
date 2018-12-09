# RPC框架
- 一、[RPC基础知识](https://github.com/wangtengke/Notes/blob/master/notes/RPC%E6%A1%86%E6%9E%B6.md#一rpc基础知识)
  - [RPC是什么](https://github.com/wangtengke/Notes/blob/master/notes/RPC%E6%A1%86%E6%9E%B6.md#rpc是什么)
  - [为什么要用RPC](https://github.com/wangtengke/Notes/blob/master/notes/RPC%E6%A1%86%E6%9E%B6.md#为什么要用rpc)
  - [RPC原理与框架](https://github.com/wangtengke/Notes/blob/master/notes/RPC%E6%A1%86%E6%9E%B6.md#rpc原理与框架)
- 二、[RPC框架核心技术点](https://github.com/wangtengke/Notes/blob/master/notes/RPC%E6%A1%86%E6%9E%B6.md#二rpc框架核心技术点)
  - [服务暴露](https://github.com/wangtengke/Notes/blob/master/notes/RPC%E6%A1%86%E6%9E%B6.md#服务暴露)
  - [远程代理对象](https://github.com/wangtengke/Notes/blob/master/notes/RPC%E6%A1%86%E6%9E%B6.md#远程代理对象)
  - [通信](https://github.com/wangtengke/Notes/blob/master/notes/RPC%E6%A1%86%E6%9E%B6.md#通信)
  - [序列化](https://github.com/wangtengke/Notes/blob/master/notes/RPC%E6%A1%86%E6%9E%B6.md#序列化)
- 三、[RPC与消息队列的区别](https://github.com/wangtengke/Notes/blob/master/notes/RPC%E6%A1%86%E6%9E%B6.md#三rpc与消息队列的区别)
  - [功能差异](https://github.com/wangtengke/Notes/blob/master/notes/RPC%E6%A1%86%E6%9E%B6.md#功能差异)
  - [使用场景差异](https://github.com/wangtengke/Notes/blob/master/notes/RPC%E6%A1%86%E6%9E%B6.md#使用场景差异)
  - [不适用场景说明](https://github.com/wangtengke/Notes/blob/master/notes/RPC%E6%A1%86%E6%9E%B6.md#不适用场景说明)

# 一、RPC基础知识
## RPC是什么
RPC（Remote Procedure Call Protocol）——远程过程调用协议，它是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。RPC协议假定某些传输协议的存在，如TCP或UDP，为通信程序之间携带信息数据。在OSI网络通信模型中，RPC跨越了传输层和应用层。RPC使得开发包括网络分布式多程序在内的应用程序更加容易。

##  为什么要用RPC
1. 如果我们开发简单的单一应用，逻辑简单、用户不多、流量不大，那我们用不着；

2. 当我们的系统访问量增大、业务增多时，我们会发现一台单机运行此系统已经无法承受。此时，我们可以将业务拆分成几个互不关联的应用，分别部署在各自机器上，以划清逻辑并减小压力。此时，我们也可以不需要RPC，因为应用之间是互不关联的。
3. 当我们的业务越来越多、应用也越来越多时，自然的，我们会发现有些功能已经不能简单划分开来或者划分不出来。此时，可以将公共业务逻辑抽离出来，将之组成独立的服务Service应用 。而原有的、新增的应用都可以与那些独立的Service应用交互，以此来完成完整的业务功能。所以此时，我们急需一种高效的应用程序之间的通讯手段来完成这种需求，所以你看，RPC大显身手的时候来了！

其实3描述的场景也是服务化 、微服务 和分布式系统架构 的基础场景。即RPC框架就是实现以上结构的有力方式。

## RPC原理与框架
如下图所示：

![RPC流程](https://github.com/wangtengke/Notes/blob/master/imgs/rpc%E6%B5%81%E7%A8%8B.png)

RPC 服务方通过 RpcServer 去导出（export）远程接口方法，而客户方通过 RpcClient 去引入（import）远程接口方法。客户方像调用本地方法一样去调用远程接口方法，RPC 框架提供接口的代理实现，实际的调用将委托给代理RpcProxy 。代理封装调用信息并将调用转交给RpcInvoker 去实际执行。在客户端的RpcInvoker 通过连接器RpcConnector 去维持与服务端的通道RpcChannel，并使用RpcProtocol 执行协议编码（encode）并将编码后的请求消息通过通道发送给服务方。
RPC 服务端接收器 RpcAcceptor 接收客户端的调用请求，同样使用RpcProtocol 执行协议解码（decode）。解码后的调用信息传递给RpcProcessor 去控制处理调用过程，最后再委托调用给RpcInvoker 去实际执行并返回调用结果。如下是各个部分的详细职责：
1. **RpcServer**  
   负责导出（export）远程接口  
2. **RpcClient**  
   负责导入（import）远程接口的代理实现  
3. **RpcProxy**  
   远程接口的代理实现  
4. **RpcInvoker**  
   客户方实现：负责编码调用信息和发送调用请求到服务方并等待调用结果返回  
   服务方实现：负责调用服务端接口的具体实现并返回调用结果  
5. **RpcProtocol**  
   负责协议编/解码  
6. **RpcConnector**  
   负责维持客户方和服务方的连接通道和发送数据到服务方  
7. **RpcAcceptor**  
   负责接收客户方请求并返回请求结果  
8. **RpcProcessor**  
   负责在服务方控制调用过程，包括管理调用线程池、超时时间等  
9. **RpcChannel**  
   数据传输通道  

# 二、RPC框架核心技术点
## 服务暴露
远程提供者需要以某种形式提供服务调用相关的信息，包括但不限于服务接口定义、数据结构、或者中间态的服务定义文件。
## 远程代理对象
服务调用者用的服务实际是远程服务的本地代理。说白了就是通过动态代理来实现。

java 里至少提供了两种技术来提供动态代码生成，一种是 jdk 动态代理，另外一种是字节码生成。动态代理相比字节码生成使用起来更方便，但动态代理方式在性能上是要逊色于直接的字节码生成的，而字节码生成在代码可读性上要差很多。两者权衡起来，个人认为牺牲一些性能来获得代码可读性和可维护性显得更重要。
## 通信
RPC框架与具体的协议无关。RPC 可基于 HTTP 或 TCP 协议，Web Service 就是基于 HTTP 协议的 RPC，它具有良好的跨平台性，但其性能却不如基于 TCP 协议的 RPC。

RPC常用的通信框架是Netty，Netty是NIO框架，参考 https://github.com/code4craft/netty-learning
## 序列化
两方面会直接影响 RPC 的性能，一是传输方式，二是序列化。
1. 序列化方式：毕竟是远程通信，需要将对象转化成二进制流进行传输。不同的RPC框架应用的场景不同，在序列化上也会采取不同的技术。 就序列化而言，Java 提供了默认的序列化方式，但在高并发的情况下，这种方式将会带来一些性能上的瓶颈，于是市面上出现了一系列优秀的序列化框架，比如：Protobuf、Kryo、Hessian、Jackson 等，它们可以取代 Java 默认的序列化，从而提供更高效的性能。

2. 编码内容：出于效率考虑，编码的信息越少越好（传输数据少），编码的规则越简单越好（执行效率高）。

# 三、RPC与消息队列的区别
## 功能差异
在架构上，RPC和Message的差异点是，Message有一个中间结点Message Queue，可以把消息存储。

**消息的特点**
1. Message Queue把请求的压力保存一下，逐渐释放出来，让处理者按照自己的节奏来处理。
2. Message Queue引入一下新的结点，系统的可靠性会受Message Queue结点的影响。
3. Message Queue是异步单向的消息。发送消息设计成是不需要等待消息处理的完成。

所以对于有同步返回需求，用Message Queue则变得麻烦了。

**RPC的特点**

1. 同步调用，对于要等待返回结果/处理结果的场景，RPC是可以非常自然直觉的使用方式(RPC也可以是异步调用)。
2. 由于等待结果，Consumer（Client）会有线程消耗。如果以异步RPC的方式使用，Consumer（Client）线程消耗可以去掉。但不能做到像消息一样暂存消息/请求，压力会直接传导到服务Provider。

## 使用场景差异
1. 希望同步得到结果的场合，RPC合适。
2. 希望使用简单，则RPC；RPC操作基于接口，使用简单，使用方式模拟本地调用。异步的方式编程比较复杂。
3. 不希望发送端（RPC Consumer、Message Sender）受限于处理端（RPC Provider、Message Receiver）的速度时，使用Message Queue。
随着业务增长，有的处理端处理量会成为瓶颈，会进行同步调用到异步消息的改造。这样的改造实际上有调整业务的使用方式。比如原来一个操作页面提交后就下一个页面会看到处理结果；改造后异步消息后，下一个页面就会变成“操作已提交，完成后会得到通知”。

## 不适用场景说明
1. RPC同步调用使用Message Queue来传输调用信息。 上面分析可以知道，这样的做法，发送端是在等待，同时占用一个中间点的资源。变得复杂了，但没有对等的收益。
2. 对于返回值是void的调用，可以这样做，因为实际上这个调用业务上往往不需要同步得到处理结果的，只要保证会处理即可。（RPC的方式可以保证调用返回即处理完成，使用消息方式后这一点不能保证了。）
3. 返回值是void的调用，使用消息，效果上是把消息的使用方式Wrap成了服务调用（服务调用使用方式成简单，基于业务接口）。
