#### NIO 详解

主要方法：
```
flip()：反转缓冲区，将limit的值设为position的值， 然后position的值设为0。为从缓冲区读取字节做准备。
rewind()：从头再读或再写，limit不变，position设置为0。
mark()：标记当前的position值，和reset()配合使用。
reset()：将当前position设为mark标记的值。
hasRemaining()：position和limit之前是否还有元素。
clear()：清空整个缓冲区（没有擦除）。 position的值设为0， limit的值设为capacity，mark的值被丢弃。为把字节写到缓冲区做准备。
compact()：只清空已经读过的数据（没有擦除）。未读数据复制到缓冲区的起始处，position设到最后一个未读数据后。
```

写数据到Buffer 有两种方式：
```
从 Channel 写到 Buffer。
通过 Buffer 的 put() 方法写到 Buffer 里。
```
从Buffer中读取数据 有两种方式：
```
从Buffer读取数据到Channel。
使用get()方法从Buffer中读取数据。
```

equals()，当满足下列条件时，表示两个Buffer相等：
```
有相同的类型（byte、char、int等）。
Buffer中剩余的byte、char等的个数相等。
Buffer中所有剩余的byte、char等都相同。
```

compareTo()，比较两个同类型Buffer的剩余元素， 如果满足下列条件，则认为一个Buffer“小于”另一个Buffer：
```
第一个不相等的元素小于另一个Buffer中对应的元素 。
所有元素都相等，但第一个Buffer比另一个先耗尽(第一个Buffer的元素个数比另一个少)。
``````

> FileChannel

无法直接打开一个FileChannel，需要通过 InputStream、OutputStream 或 RandomAccessFile 的 getChannel 方法来获取一个 FileChannel 实例。

FileChannel 常用方法：
```
read(ByteBuffer)，从文件通道读取数据到ByteBuffer，-1 表示已经到达输入的末尾。
write(ByteBuffer)，从字节缓存区写到文件通道。
close()，关闭。
transferFrom()，将数据从源通道传输到FileChannel中 toChannel.transferFrom(position, count, fromChannel);
transferTo()，将数据从FileChannel传输到其他的channel中 fromChannel.transferTo(position, count, toChannel);

position() 获取当前位置
position(int) 设置当前位置
size() 返回该实例所关联文件的大小
truncate(int) 截取文件的前int个字节
force(boolean) 将通道里尚未写入磁盘的内存缓存数据强制写到磁盘上，boolean指明是否同时将文件元数据（权限信息等）写到磁盘上
```

Scattering Reads，数据从一个channel读取到多个buffer中。不适用于动态消息（消息大小不固定）。
```
    ByteBuffer header = ByteBuffer.allocate(128);
    ByteBuffer body   = ByteBuffer.allocate(1024);
    ByteBuffer[] bufferArray = { header, body };
    channel.read(bufferArray);
```
Gathering Writes 是指数据从多个buffer写入到同一个channel。能较好的处理动态消息。
```
    ByteBuffer header = ByteBuffer.allocate(128);
    ByteBuffer body   = ByteBuffer.allocate(1024);
    ByteBuffer[] bufferArray = { header, body };
    channel.write(bufferArray);
```

复制文件的应用

说了这么多，让我们在代码中感受一下 FileChannel 的使用：

我们要实现一个复制文件的 API，现在有两种方式。第一种采用传统的 IO 流加装饰缓存数组来实现，第二种采用 NIO 流中 FileChannel 的 transferTo 函数来实现通道对通道的传输。用函数运行时间进行测试发现，小文件（不超过十几M时）拷贝时，用流更快，大文件用通道更快。

因为通道的 transferTo 不经过用户态，直接在内核态传输数据，减少上下文切换和额外IO操作，不仅仅在文件拷贝时可以这么用，在读取磁盘文件然后进行 Socket 发送时也可以如此。使用流读写时，进行了多次上下文切换，比如应用读取数据时，首先在内核态将数据从磁盘读取到内核缓存，再切换到用户态将数据从内核缓存读取到用户缓存。

```
public static void copyFileByStream(File source, File dest) throws FileNotFoundException, IOException {
    try (InputStream is = new FileInputStream(source);
    OutputStream os = new FileOutputStream(dest);) {
        byte[] buffer = new byte[1024];
        int len;
        while ((len = is.read(buffer)) != -1) {
            os.write(buffer, 0, len);
        }
        os.flush();
    }
}

@SuppressWarnings("resource")
public static void copyFileByChannel(File source, File dest) throws FileNotFoundException, IOException {
    try (FileChannel srcChannel = new FileInputStream(source).getChannel();
            FileChannel destChannel = new FileOutputStream(dest).getChannel();) {
        for (long len = srcChannel.size(); len > 0;) {
            long l = srcChannel.transferTo(srcChannel.position(), len, destChannel);
            len -= l;
        }
    }
}
```

> 内存映射文件

通过 FileChannel 的 map 方法，允许我们创建和修改那些因为太大而不能放入内存的文件，我们可以假定整个文件都放在内存中（实际上只是一部分），可以把它当做非常大的数组来访问，可以很容易的修改。但是，它的缺点是，创建映射文件的花费大于以常规方式读写几十 MB 的数据，所以只在操作较大文件是才推荐这么做。

MappedByteBuffer 本质上是一种 Direct Buffer。Direct Buffer 生命周期内内存地址都不会发生改变，进而内核可以安全地对其进行访问，很多 IO 操作会很高效，同时它保存在堆外，减少了堆内对象存储的可能额外维护工作。但是，它的创建和销毁过程，都会比一般的堆类 BUffer 增加部分开销。通常都建议用于长期使用、数据量较大的场景。
```
FileChannel fc = new RandomAccessFile(new File("bigfile.txt"), "rw").getChannel();
MappedByteBuffer mbb = fc.map(FileChannel.MapMode.READ_WRITE, 0, fc.size());
...
fc.close();
```
> 文件锁

通过文件加锁来同步访问作为共享资源的文件，文件锁对其他的操作系统进程时可见的，因为 Java 的文件加锁直接映射到了本地操作系统的加锁工具。

通过 FileChannel 对象的 lock 或 tryLock 方法可以对文件加锁并返回 FileLock 对象，通过 FileLock 对象的 release 方法可以释放锁。

> SocketChannel
连接到TCP网络套接字的通道。

创建方式

1.打开一个SocketChannel并连接到互联网上的某台服务器。
```
    SocketChannel socketChannel = SocketChannel.open();
    socketChannel.connect(new InetSocketAddress("http://jenkov.com", 80));
```
2.一个新连接到达ServerSocketChannel时，会创建一个SocketChannel。
```
close() 关闭。
read(ByteBuffer) 读取数据
write(ByteBuffer) 写入数据 while(buf.hasRemaining()) {channel.write(buf);}
```
非阻塞模式

**configureBlocking(false)设置套接字通道为非阻塞模式。**

此时：
```
connect 方法可能在连接建立之前就返回了。为了确定连接是否建立，可以调用 finishConnect() 的方法。
write 方法在尚未写出任何内容时可能就返回了。所以需要在循环中调用。
read 方法在尚未读取到任何数据时可能就返回了。所以需要关注它的int返回值。
c. ServerSocketChannel
可以监听新进来的TCP连接。

打开：ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
关闭：serverSocketChannel.close();
```

监听新进来的连接，会一直阻塞到有新连接到达：
```
    while(true){
        SocketChannel socketChannel = serverSocketChannel.accept();
        //do something with socketChannel...
    }
```
非阻塞模式

非阻塞模式下，accept() 方法会立刻返回，如果还没有新进来的连接,返回的将是 null
```
    serverSocketChannel.configureBlocking(false);
    while(true){
        SocketChannel socketChannel =serverSocketChannel.accept();
        if(socketChannel != null){//do something with socketChannel...}
    }
```

> DatagramChannel

能收发UDP包的通道。因为UDP是无连接的网络协议，所以不能像其它通道那样读取和写入。它发送和接收的是数据包。

打开：
```
   DatagramChannel channel = DatagramChannel.open();
   channel.socket().bind(new InetSocketAddress(9999));
```
接收数据：
- receive(ByteBuffer) 会将接收到的数据包内容复制到指定的Buffer. 如果Buffer容不下收到的数据，多出的数据将被丢弃。

发送数据：
```
   // 发送一串字符到”jenkov.com”服务器的UDP端口80
   int bytesSent = channel.send(buf, new InetSocketAddress("jenkov.com", 80));
```

连接到特定的地址：

由于UDP是无连接的，连接到特定地址并不会像TCP通道那样创建一个真正的连接。而是锁住DatagramChannel ，让其只能从特定地址收发数据。
```
channel.connect(new InetSocketAddress("jenkov.com", 80));
```
连接后，也可以使用read()和write()方法，就像在用传统的通道一样。只是在数据传送方面没有任何保证
```
   int bytesRead = channel.read(buf);
   int bytesWritten = channel.write(but);
```

> Pipe

管道是**2个线程之间的单向数据连接**(线程间通信)。Pipe 有一个 source 通道和一个 sink 通道。数据会被写到 sink 通道，从source通道读取。
```
创建 Pipe pipe = Pipe.open();
```
写数据到 sink 通道
```
    Pipe.SinkChannel sinkChannel = pipe.sink();
    while(buf.hasRemaining()) {sinkChannel.write(buf);}
```
从source通道读数据
```
    Pipe.SourceChannel sourceChannel = pipe.source();
    int bytesRead = sourceChannel.read(buf);
```

> Selector

选择器，能够检测一到多个 NIO 通道，并能够知晓通道是否为诸如读写事件做好准备。一个单独的线程可以管理多个 Channel，从而管理多个网络连接。

非阻塞 IO 的核心在于使用一个 Selector 来管理多个通道，可以是 SocketChannel，也可以是 ServerSocketChannel，将各个通道注册到 Selector 上，指定监听的事件。

之后可以只用一个线程来轮询这个 Selector，看看上面是否有通道是准备好的，当通道准备好可读或可写，然后才去开始真正的读写，这样速度就很快了(因为阻塞时大部分时间都是在等待确认是否可以真正读写)。我们就完全没有必要给每个通道都起一个线程。

Server：
```
/*
    作者：见参考资料 1
*/
public class SelectorServer {

    public static void main(String[] args) throws IOException {
        Selector selector = Selector.open();

        ServerSocketChannel server = ServerSocketChannel.open();
        server.socket().bind(new InetSocketAddress(8080));

        // 将 server socket 通道注册到 Selector 中，监听 OP_ACCEPT 事件
        server.configureBlocking(false); // 非阻塞
        server.register(selector, SelectionKey.OP_ACCEPT);

        while (true) {
            // 需要不断地去调用 select() 方法获取最新的准备好的通道
            // select 方法时阻塞的，直到有通道就绪才返回
            int readyChannels = selector.select();
            if (readyChannels == 0) {
                continue;
            }
            Set<SelectionKey> readyKeys = selector.selectedKeys();
            // 遍历
            Iterator<SelectionKey> iterator = readyKeys.iterator();
            while (iterator.hasNext()) {
                SelectionKey key = iterator.next();
                iterator.remove(); // 手动从 selected-key set 中移除

                // 当“连接就绪”时，服务器通道建立新的客户端连接
                if (key.isAcceptable()) {
                    // 有已经接受的新的到服务端的连接
                    SocketChannel socketChannel = server.accept();

                    // 有新的连接并不代表这个通道就有数据，
                    // 这里将这个新的代表客户端连接的 SocketChannel 注册到 Selector，监听 OP_READ 事件，等待数据
                    socketChannel.configureBlocking(false);
                    socketChannel.register(selector, SelectionKey.OP_READ);
                }
                // 剩下的条件，过滤的是客户端通道的就绪状态
                else if (key.isReadable()) {
                    // 有数据可读
                    // 上面一个 if 分支中注册了监听 OP_READ 事件的 SocketChannel
                    SocketChannel socketChannel = (SocketChannel) key.channel();
                    ByteBuffer readBuffer = ByteBuffer.allocate(1024);
                    int num = socketChannel.read(readBuffer);
                    if (num > 0) {
                        // 处理进来的数据...
                        System.out.println("收到数据：" + new String(readBuffer.array()).trim());
                        socketChannel.register(selector, SelectionKey.OP_WRITE);
                    } else if (num == -1) {
                        // -1 代表连接已经关闭
                        socketChannel.close();
                    }
                }
                else if (key.isWritable()) {
                    // 通道可写
                    // 给用户返回数据的通道可以进行写操作了
                    SocketChannel socketChannel = (SocketChannel) key.channel();
                    ByteBuffer buffer = ByteBuffer.wrap("返回给客户端的数据...".getBytes());
                    socketChannel.write(buffer);

                    // 重新注册这个通道，监听 OP_READ 事件，客户端还可以继续发送内容过来
                    socketChannel.register(selector, SelectionKey.OP_READ);
                }
            }
        }
    }
}
```
Client:
```
class NIOClient{
    public static void main(String[] args) {
        try (SocketChannel socketChannel = SocketChannel.open();) {
            socketChannel.connect(new InetSocketAddress(InetAddress.getLocalHost(), 8080));
            socketChannel.write(ByteBuffer.wrap("发送给服务器的数据...".getBytes()));
            ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
            socketChannel.read(byteBuffer);
            byteBuffer.flip();
            while (byteBuffer.hasRemaining()) {
                System.out.print(Charset.defaultCharset().decode(byteBuffer));
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

> 向Selector注册通道:

通过SelectableChannel.register()方法来实现。

**Channel 必须处于非阻塞模式下**。

**这意味着不能将FileChannel与Selector一起使用，因为 FileChannel 不能切换到非阻塞模式。而套接字通道都可以。**
```
    channel.configureBlocking(false);
    SelectionKey key = channel.register(selector, Selectionkey.OP_READ);
```

> 通道的关注点（interest set）

Channel 注册时，第二个 int 参数可以表明它关注的操作状态（如连接、接受、读或写）。**可以用|来监听多个事件：**
```
int 型的操作位	描述
OP_CONNECT	   某个 channel 成功连接到另一个服务器称为“连接就绪”
OP_ACCEPT	    一个 server socket channel 准备好接收新进入的连接称为“接收就绪”
OP_READ	      一个有数据可读的通道可以说是“读就绪”
OP_WRITE	     等待写数据的通道可以说是“写就绪”

```

> SelectionKey

操作键（SelectionKey）代表着 Selector 上注册过的一个 SelectableChannel 对象。

一个 Selector 对象有三个 SelectionKey 的集合：

- key set，代表所有注册过的通道，用 keys 方法返回。

  当通道通过 register() 方法向 Selector 注册时，会向 Selector 对象的 keys set 中添加一个 SelectionKey 对象。在 selection 操作时被取消的键会从 key set 中删除。

- selected-key set，代表符合 Selector 关注点（多个关注点时至少符合一个）已就绪的通道，用 selectedKeys 方法返回。

  selection 操作时会往 selected-key set 中添加键，这些键只能通过该集合或集合迭代器的 remove 方法来移除。

- cancelled-key set，代表已经取消但还没有解注册的通道，无法直接访问。

  当一个键被取消时（关闭通道，或调用 cancel 方法），它会被加入 cancelled-key set 中。下一次 selection 操作时，被取消的键会被解注册并从 cancelled-key set 中移除。

SelectionKey 代表着一个 SelecteableChannel 对象，它两个标记状态的操作集：

- 关注点集（interest set），标记着下一次 Selector 调用 selection 操作时，该键的哪些操作状态会被检查。interestOps() 方法可以返回代表关注点的 int 值。

- 就绪集（ready-operation set），标记着键对应的通道在哪种操作状态上处于就绪状态，在 selection 操作时会被更新。readyOps() 方法返回代表该集合的 int 值。可以用 isAcceptable()、isConnectable()、isReadable()、isWritable() 这4个方法来过滤就绪状态。

常用方法还有：
```
channel() SelectionKey 对应的通道对象
selector() 通道注册的选择器对象
attach(Object) 和attachment()方法配合，将对象附着到SelectionKey上。也可以在注册时就附着
attachment() 取出附着对象
选择操作（selection）
返回监听事件已经就绪的那些通道的键集。

select() 阻塞到至少有一个通道在注册的事件上就绪了。
select(long) 和select()一样，除了最长会阻塞long毫秒。
selectNow() 不会阻塞，不管什么通道就绪都立刻返回，没有通道变成可选择的，则此方法直接返回 null。
```
选择操作有三个步骤：

- 清空 cancelled-key set 中的所有键，解注册对应的通道。

- 向底层操作系统查询，剩余的通道中哪些在自己的关注点上已就绪。对于至少在一个关注点上已经就绪的通道：
  1. 如果通道的键过去不在 selected-key set 中，那么现在添加到 selected-key set，它的就绪状态会被更新，过去存在它的就绪集上的信息会被丢弃。
  2. 如果这个键过去就在 selected-key set 中了，将操作系统的返回结果更新到键的就绪集。
- 如果在步骤（2）正在进行时将任何键添加到 cancelled-key set 中，则按照步骤（1）处理它们。
选择之后，返回就绪的键集合，遍历这个已选择的键集合来访问就绪的通道
```
    Set<SelectionKey> selectedKeys = selector.selectedKeys();
    Iterator keyIterator = selectedKeys.iterator();
    while(keyIterator.hasNext()) {
        SelectionKey key = keyIterator.next();
        if(key.isAcceptable()) {
            // a connection was accepted by a ServerSocketChannel.
        } else if (key.isConnectable()) {
            // a connection was established with a remote server.
        } else if (key.isReadable()) {
            // a channel is ready for reading
        } else if (key.isWritable()) {
            // a channel is ready for writing
        }
        // 需要手动把这个键从就绪键集中移除
        keyIterator.remove();
    }
```
