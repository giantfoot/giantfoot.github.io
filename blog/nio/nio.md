#### NIO

Java IO 即 Java 输入输出。在开发应用软件时，很多时候都需要和各种输入输出相关的媒介打交道。与媒介进行 IO 操作的过程十分复杂，需要考虑众多因素，比如：进行 IO 操作媒介的类型（文件、控制台、网络）、通信方式（顺序、随机、二进制、按字符、按字、按行等等）。

Java 类库提供了相应的类来解决这些难题，这些类就位于 java.io 包中， 在整个 java.io 包中最重要的就是 5 个类和一个接口。5 个类指的是 File、OutputStream、InputStream、Writer、Reader；一个接口指的是 Serializable。

由于老的 Java IO 标准类提供 IO 操作（如 read()，write()）都是同步阻塞的，因此，**IO 通常也被称为阻塞 IO（即 BIO，Blocking I/O）。**

在 JDK1.4 之后，为了提高 Java IO 的效率，Java 又提供了一套 New IO（NIO），原因在于它相对于之前的 IO 类库是新增的。此外，旧的 IO 类库提供的 IO 方法是阻塞的，**New IO 类库则让 Java 可支持非阻塞 IO**，所以，更多的人喜欢称之为非阻塞 IO（Non-blocking IO）。

阻塞与非阻塞，异步与同步 可参考 - [*同步异步和阻塞非阻塞详解*](https://github.com/giantfoot/giantfoot.github.io/blob/master/blog/other/同步异步和阻塞非阻塞.md)

> 同步阻塞 IO：

在此种方式下，用户进程在发起一个 IO 操作以后，必须等待 IO 操作的完成，只有当真正完成了 IO 操作以后，用户进程才能运行。 Java 传统的 IO 模型属于此种方式！

> 同步非阻塞 IO：

在此种方式下，用户进程发起一个 IO 操作以后 便可返回做其它事情，但是用户进程需要时不时的询问 IO 操作是否就绪，这就要求用户进程不停的去询问，从而引入不必要的 CPU 资源浪费。其中目前 Java 的 NIO 就属于同步非阻塞 IO 。

> 异步阻塞 IO：

此种方式下是指应用发起一个 IO 操作以后，不等待内核 IO 操作的完成，等内核完成 IO 操作以后会通知应用程序，这其实就是同步和异步最关键的区别，同步必须等待或者主动的去询问 IO 是否完成，那么为什么说是阻塞的呢？因为此时是通过 select 系统调用来完成的，而 select 函数本身的实现方式是阻塞的，而采用 select 函数有个好处就是它可以同时监听多个文件句柄，从而提高系统的并发性！

> 异步非阻塞 IO：

在此种模式下，用户进程只需要发起一个 IO 操作然后立即返回，等 IO 操作真正的完成以后，应用程序会得到 IO 操作完成的通知，此时用户进程只需要对数据进行处理就好了，不需要进行实际的 IO 读写操作，因为 真正的 IO 读取或者写入操作已经由 内核完成了。**目前 Java 中还没有支持此种 IO 模型。**

> 总结

所有的系统 I/O 都分为两个阶段：**等待就绪和操作**。

举例来说，读函数，分为等待系统可读和真正的读；同理，写函数分为等待网卡可以写和真正的写。Java IO 的各种流是阻塞的。这意味着当线程调用 write() 或 read() 时，线程会被阻塞，直到有一些数据可用于读取或数据被完全写入。

**需要说明的是等待就绪引起的 “阻塞” 是不使用 CPU 的，是在 “空等”；而真正的读写操作引起的“阻塞” 是使用 CPU 的，是真正在”干活”，而且这个过程非常快，属于 memory copy，带宽通常在 1GB/s 级别以上，可以理解为基本不耗时。因此，所谓 “阻塞” 主要是指等待就绪的过程。**

以socket.read()为例子：

传统的阻塞 IO(BIO) 里面 socket.read()，如果接收缓冲区里没有数据，函数会一直阻塞，直到收到数据，返回读到的数据。

而对于非阻塞 IO(NIO)，如果接收缓冲区没有数据，则直接返回 0，而不会阻塞；如果接收缓冲区有数据，就把数据从网卡读到内存，并且返回给用户。

说得接地气一点，BIO 里用户最关心 “我要读”，NIO 里用户最关心” 我可以读了”。NIO 一个重要的特点是：**socket 主要的读、写、注册和接收函数，在等待就绪阶段都是非阻塞的，真正的 I/O 操作是同步阻塞的（消耗 CPU 但性能非常高）。**


**为什么说 NIO 是基于缓冲区的 IO 方式呢？**

因为，当一个链接建立完成后，IO 的数据未必会马上到达，为了当数据到达时能够正确完成 IO操作，在 BIO（阻塞 IO）中，等待 IO 的线程必须被阻塞，以全天候地执行 IO 操作。为了解决这种 IO方式低效的问题，引入了缓冲区的概念，当数据到达时，可以预先被写入缓冲区，再由缓冲区交给线程，因此线程无需阻塞地等待 IO。

> Stream

Java IO 是面向流的 I/O，这意味着我们需要从流中读取一个或多个字节。它使用流来在数据源/槽和 Java 程序之间传输数据。使用此方法的 I/O 操作较慢。下面来看看在 Java 程序中使用输入/输出流的数据流图

![nio](../pic/nio/nio1.jpg)


> Buffer

Buffer 是一个对象，它包含一些要写入或读出的数据。在 NIO 中，数据是放入 Buffer 对象的，而在 IO 中，数据是直接写入或者读到 Stream 对象的。应用程序不能直接对 Channel 进行读写操作，而必须通过 Buffer 来进行，即 Channel 是通过 Buffer 来读写数据的

![nio](../pic/nio/nio2.jpg)

在 NIO 中，所有的数据都是用 Buffer 处理的，它是 NIO 读写数据的中转池。Buffer 实质上是一个数组，通常是一个字节数据，但也可以是其他类型的数组。但一个缓冲区不仅仅是一个数组，重要的是它提供了对数据的结构化访问，而且还可以跟踪系统的读写进程

使用 Buffer 读写数据一般遵循以下四个步骤：
```
1.写入数据到 Buffer；
2.调用 flip() 方法；
3.从 Buffer 中读取数据；
4.调用 clear() 方法或者 compact() 方法。
5.当向 Buffer 写入数据时，Buffer 会记录下写了多少数据。一旦要读取数据，需要通过 .flip() 方法将 Buffer 从写模式切换到读模式。在读模式下，可以读取之前写入到 Buffer 的所有数据。
``````
一旦读完了所有的数据，就需要清空缓冲区，让它可以再次被写入。有两种方式能清空缓冲区：调用 clear() 或 compact() 方法。clear() 方法会清空整个缓冲区。compact() 方法只会清除已经读过的数据。任何未读的数据都被移到缓冲区的起始处，新写入的数据将放到缓冲区未读数据的后面。


Buffer种类：
```
CharBuffer、DoubleBuffer、IntBuffer、LongBuffer、ByteBuffer、ShortBuffer、FloatBuffer
```
上述缓冲区覆盖了我们可以通过 I/O 发送的基本数据类型：
```
characters，double，int，long，byte，short和float
```
Buffer重要成员
```
private int position = 0;
private int limit;
private int capacity;

position 记录当前读取或者写入的位置，写模式下等于当前写入的单位数据数量，从写模式切换到读模式时，置为 0，在读的过程中等于当前读取单位数据的数量；
limit 代表最多能写入或者读取多少单位的数据，写模式下等于最大容量 capacity；从写模式切换到读模式时，等于position，然后再将 position 置为 0，所以，读模式下，limit 表示最大可读取的数据量，这个值与实际写入的数量相等。
capacity 表示 buffer 容量，创建时分配。
```
写/读模式切换时需要调用 flip() 方法
![nio](../pic/nio/nio3.jpg)

Buffer 的选择
通常情况下，操作系统的一次写操作分为两步：
```
将数据从用户空间拷贝到系统空间（即从 JVM 内存拷贝到系统内存）。
从系统空间往网卡写。
```
同理，读操作也分为两步：
```
将数据从网卡拷贝到系统空间；
将数据从系统空间拷贝到用户空间
```
示例
```
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

public class IO_Demo
{
    public static void main(String[] args) throws Exception
    {
        String infile = "D:\\Users\\data.txt";
        String outfile = "D:\\Users\\dataO.txt";
        // 获取源文件和目标文件的输入输出流
        FileInputStream fin = new FileInputStream(infile);
        FileOutputStream fout = new FileOutputStream(outfile);
        // 获取输入输出通道
        FileChannel fileChannelIn = fin.getChannel();
        FileChannel fileChannelOut = fout.getChannel();
        // 创建缓冲区，分配1K堆内存
        ByteBuffer buffer = ByteBuffer.allocate(1024);

        while (true)
        {
            // clear方法重设缓冲区，使它可以接受读入的数据
            buffer.clear();
            // 从输入通道中读取数据数据并写入buffer
            int r = fileChannelIn.read(buffer);
            // read方法返回读取的字节数，可能为零，如果该通道已到达流的末尾，则返回-1
            if (r == -1)
            {
                break;
            }
            // flip方法将 buffer从写模式切换到读模式
            buffer.flip();
            // 从buffer中读取数据然后写入到输出通道中
            fileChannelOut.write(buffer);
        }
        //关闭通道
        fileChannelOut.close();
        fileChannelIn.close();
        fout.close();
        fin.close();
    }
}

```

> Channel

通道是 I/O 传输发生时通过的入口，而缓冲区是这些数据传输的来源或目标。对于离开缓冲区的传输，需要输出的数据被置于一个缓冲区，然后写入通道。对于传回缓冲区的传输，一个通道将数据写入缓冲区中。

例如：
```
有一个服务器通道serverChannel，一个客户端通道 SocketChannel clientChannel；

服务器缓冲区：serverBuffer，客户端缓冲区：clientBuffer。

当服务器想向客户端发送数据时，需要调用 clientChannel.write(serverBuffer)。
当客户端要读时，调用clientChannel.read(clientBuffer) 当客户端想向服务器发送数据时，需要调用serverChannel.write(clientBuffer)。
当服务器要读时，调用serverChannel.read(serverBuffer)
```

Channel 是一个对象，可以通过它读取和写入数据。可以把它看做 IO 中的流。但是它和流相比还有一些不同：
```
Channel 是双向的，既可以读又可以写，而流是单向的（所谓输入/输出流）；
Channel 可以进行异步的读写
对 Channel的读写必须通过 buffer 对象
```

**正如上面提到的，所有数据都通过 Buffer 对象处理，所以，输出操作时不会将字节直接写入到 Channel 中，而是将数据写入到 Buffer 中；同样，输入操作也不会从 Channel 中读取字节，而是将数据从 Channel 读入 Buffer，再从 Buffer 获取这个字节。**

在 Java NIO 中 Channel 主要有如下几种类型：
```
FileChannel：从文件读取数据的
DatagramChannel：读写 UDP 网络协议数据
SocketChannel：读写 TCP 网络协议数据
ServerSocketChannel：可以监听 TCP 连接
```

> Selector

通道和缓冲区的机制，使得 Java NIO 实现了同步非阻塞 IO 模式，在此种方式下，用户进程发起一个 IO 操作以后便可返回做其它事情，而无需阻塞地等待 IO 事件的就绪，但是用户进程需要时不时的询问 IO 操作是否就绪，这就要求用户进程不停的去询问，从而引入不必要的 CPU 资源浪费。

鉴于此，需要有一个机制来监管这些 IO 事件，如果一个 Channel 不能读写（返回 0），我们可以把这件事记下来，然后切换到其它就绪的连接（channel）继续进行读写。在 Java NIO 中，这个工作由 selector 来完成，这就是所谓的同步。

**Selector 是一个对象，它可以接受多个 Channel 注册，监听各个 Channel 上发生的事件，并且能够根据事件情况决定 Channel 读写。这样，通过一个线程可以管理多个 Channel，从而避免为每个 Channel 创建一个线程，节约了系统资源。如果你的应用打开了多个连接（Channel），但每个连接的流量都很低，使用 Selector 就会很方便。**

要使用 Selector，就需要向 Selector 注册 Channel，然后调用它的 select() 方法。这个方法会一直阻塞到某个注册的通道有事件就绪，这就是所说的轮询。一旦这个方法返回，线程就可以处理这些事件。
下面这幅图展示了一个线程处理 3 个 Channel 的情况：

![nio](../pic/nio/nio4.jpg)
