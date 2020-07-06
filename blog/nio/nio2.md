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
