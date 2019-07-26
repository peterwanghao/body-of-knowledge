# 1 NIO介绍

传统的并发型服务器设计是利用阻塞型网络I/O 以多线程的模式来实现的，然而由于系统常常在进行网络读写时处于阻塞状态，会大大影响系统的性能；自Java1. 4 开始引入了NIO(新I/O) API，通过使用非阻塞型I/O，实现流畅的网络读写操作，为开发高性能并发型服务器程序提供了一个很好的解决方案。这就是java nio。

## 传统阻塞型网络 I/O的不足

Java 平台传统的I/O 系统都是基于Byte（字节）和Stream（数据流）的，相应的I/O操作都是阻塞型的，所以服务器程序也采用阻塞型I/O 进行数据的读、写操作。传统的设计为了实现服务器程序的并发性要求，系统由一个单独的主线程来监听用户发起的连接请求，一直处于阻塞状态；当有用户连接请求到来时，程序都会启一个新的线程来统一处理用户数据的读、写操作。在阻塞的网络编程方式中，针对于每一个单独的网络连接，都必须有一个线程对应的绑定该网络连接，进行网络字节流的处理。看看以前传统写法：

```
try {
   ServerSocket ssc = new ServerSocket(8086);
   while (true) {
    //阻塞
    Socket s = ssc.accept();
    try {
     //每来一个客户端就开启一个读线程
     new ReadThread(s).start();
     //同时每来一个客户端就开启一个写线程
     new WriteThread(s).start();
    } catch (Exception e) {
     e.printStackTrace();
    }
   }
  } catch (IOException e) {
   e.printStackTrace();
}
```

以上就是传统阻塞模式写法，在这段代码中，有三个阻塞的方法，是ServerSocket的accept()方法，InputStream的read()方式以及OutputStream的write()方式。因此我们需要三个的线程（主线程也是一个）分别进行处理，要每一个客户端分配二个线程来处理输入、输出数据。这样如果有大量的连接存在，就存在大量的线程，其线程与客户机的比例近似为1:1，随着线程数量的不断增加，服务器启动了大量的并发线程，会大大加大系统对线程的管理开销，这将成为吞吐量瓶颈的主要原因，而大量的线程又都阻塞在read()或者write()方法，同时CPU又需要来回频繁的在这些线程中间调度和切换，必然带来大量的系统调用和资源竞争。但传统模式的也是有它的优点的，优点就是简单、实用、易管理。

对于并发型服务器，系统用在阻塞型I/O 等待和线程间切换的时间远远多于CPU 在内存中处理数据的时间，因此传统的阻塞型I/O 已经成为制约系统性能的瓶颈。Java1.4 版本后推出的NIO 工具包，提供了非阻塞型I/O 的异步输入输出机制，为提高系统的性能提供了可实现的基础机制。

## NIO包组成

针对传统I/O 工作模式的不足，NIO 工具包提出了基于Buffer（缓冲区）、Channel（通道）、Selector（选择器）的新模式；Selector（选择器）、可选择的Channel（通道）和SelectionKey（选择键）配合起来使用，可以实现并发的非阻塞型I/O 能力。

### **Buffer（缓冲器）**

由于操作系统和应用程序数据通信的原始类型是byte,也是IO数据操作的基本单元，在NIO中，每一个基本的原生类型(boolean除外)都有Buffer的实现：CharBuffer、IntBuffer、DoubleBuffer、ShortBuffer、LongBuffer、FloatBuffer和ByteBuffer，数据缓冲使得在IO操作中能够连续的处理数据流。当前有两种ByteBuffer，一种是Direct ByteBuffer，另外一种是NonDirect ByteBuffer；ByteBuffer是普通的Java对象，遵循Java堆中对象存在的规则；而Direct ByteBuffer是native代码，它内存的分配不在Java的堆栈中，不受Java内存回收的影响，每一个Direct ByteBuffer都是直接分配的一块连续的内存空间，也是NIO提高性能的重要办法之一。另外数据缓冲有一个很重要的特点是，基于一个数据缓冲可以建立一个或者多个逻辑的视图缓冲(View Buffer).比方说，通过View Buffer,可以将一个Byte类型的Buffer换作Int类型的缓冲；或者一个大的缓冲转作很多小的Buffer。之所以称为View Buffer是因为这个转换仅仅是逻辑上，在物理上并没有创建新的Buffer。这为我们操作Buffer带来诸多方便。

NIO和传统IO（一下简称IO）之间第一个最大的区别是，**IO是面向流的，NIO是面向缓冲区的**。 Java IO面向流意味着每次从流中读一个或多个字节，直至读取所有字节，它们没有被缓存在任何地方。此外，它不能前后移动流中的数据。如果需要前后移动从流中读取的数据，需要先将它缓存到一个缓冲区。NIO的缓冲导向方法略有不同。数据读取到一个它稍后处理的缓冲区，需要时可在缓冲区中前后移动。这就增加了处理过程中的灵活性。

### Channel（通道）

包含socket，file和pipe三种管道，都是全双工的通道。Channel是一个与操作系统紧密结合的本地代码较多的对象，是(Buffer)缓冲器和I/O 服务之间的通道，具有双向性，既可以读入也可以写出，可以更高效的传递数据。我们这里主要讨论ServerSocketChannel 和SocketChannel，它们都继承了SelectableChannel，是可选择的通道，分别可以工作在同步和异步两种方式下（这里的可选择不是指可以选择两种工作方式，而是指可以有选择的注册自己感兴趣的事件）。当通道工作在同步方式时，它的功能和编程方法与传统的ServerSocket、Socket 对象相似；当通道工作在异步工作方式时，进行输入输出处理不必等到输入输出完毕才返回，并且可以将其感兴趣的（如：接受操作、连接操作、读出操作、写入操作）事件注册到Selector 对象上，与Selector 对象协同工作可以更有效率的支持和管理并发的网络套接字连接。

Channel和IO中的Stream(流)是差不多一个等级的。只不过Stream是单向的，譬如：InputStream, OutputStream。而Channel是双向的，既可以用来进行读操作，又可以用来进行写操作。

NIO中的Channel的主要实现有：

- FileChannel
- DatagramChannel
- SocketChannel
- ServerSocketChannel

### Selector（选择器）和SelectionKey（选择键）

各类Buffer是数据的容器对象；各类Channel实现在各类Buffer与各类I/O 服务间传输数据。Selector 是实现并发型非阻塞I/O 的核心，各种可选择的通道将其感兴趣的事件注册到Selector对象上，Selector在一个循环中不断轮循监视这各些注册在其上的Socket 通道。SelectionKey类则封装了SelectableChannel对象在Selector 中的注册信息。当Selector 监测到在某个注册的SelectableChannel上发生了感兴趣的事件时，自动激活产生一个SelectionKey对象，在这个对象中记录了哪一个SelectableChannel 上发生了哪种事件，通过对被激活的SelectionKey 的分析，外界可以知道每个SelectableChannel 发生的具体事件类型后，可进行相应的处理。

SelectionKey满足以下三个条件之一， Key 就失效：
- Channel被关闭。
- Selector被关闭。
- 通过调用Key的cancel()方法将Key本身取消。

## NIO工作原理

通过上面的讨论，我们可以看出在并发型服务器程序中使用NIO，实际上是通过网络事件驱动模型实现的。我们应用Select机制，不用为每一个客户端连接新启线程处理，而是将客户端Socket连接注册到特定的Selector对象上，这就可以在单线程中利用Selector对象管理大量并发的网络连接，更好的利用了系统资源；采用非阻塞I/O的通信方式，不要求阻塞等待I/O 操作完成即可返回，从而减少了管理I/O 连接导致的系统开销，大幅度提高了系统性能。

当有读或写等任何注册的事件发生时，可以从Selector中获得相应SelectionKey，从SelectionKey 中可以找到发生的事件和该事件所发生的具体的SelectableChannel，以获得客户端发送过来的数据。由于在非阻塞网络I/O 中采用了事件触发机制，处理程序可以得到系统的主动通知，从而可以实现底层网络I/O无阻塞、流畅地读写，而不像在原来的阻塞模式下处理程序需要不断循环等待。使用NIO，可以编写出性能更好、更易扩展的并发型服务器程序。

应用 NIO 工具包，基于非阻塞网络I/O设计的并发型服务器程序与以往基于阻塞I/O的实现程序有很大不同，在使用非阻塞网络I/O的情况下，程序读取数据和写入数据的时机不是由程序员控制的，而是Selector 决定的。

通过使用NIO 工具包进行并发型服务器程序设计，一个或者很少几个Socket 线程就可以处理成千上万个活动的Socket 连接，大大降低了服务器端程序的开销；同时网络I/O 采取非阻塞模式，线程不再在读或写时阻塞，操作系统可以更流畅的读写数据并可以更有效地向CPU 传递数据进行处理，以便更有效地提高系统的性能。