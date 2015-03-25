title: "NIO Buffer和Channel"
date: 2015-03-25 22:57:08
categories: [java]
tags: [NIO,Buffer,Channel]
---

所有语言运行时系统提供执行I/O较高级别的工具。

在java编程中，标准低版本IO使用流的方式完成I/O操作，所有的I/O都被视为单个的字节流动，称为一个Stream的对象一次移动一个字节。

NIO是在JDK1.4之后出现的一种新的IO，sun官方标榜的nio有如下特性：

>* 为所有的原始类型提供（Buffer）缓存支持
* 字符集编码解决方案（Charset）
*  Channel : 一个新的原始I/O抽象
* 支持锁和内存映射文件的文件访问接口
* 提供多路（non-bloking）非阻塞式的高伸缩性网路I/O

NIO包（java.nio.*）引入了四个关键的抽象数据类型，它们共同解决传统的I/O类中的一些问题。
1.  Buffer：它是包含数据且用于读写的线形表结构。其中还提供了一个特殊类用于内存映射文件的I/O操作。
2． Charset：它提供Unicode字符串影射到字节序列以及逆影射的操作。
3． Channels：包含socket，file和pipe三种管道，它实际上是双向交流的通道。
4． Selector：它将多元异步I/O操作集中到一个或多个线程中。

---

本文先介绍chanel和buffer(通道和缓冲)

`缓冲区和通道是NIO中的核心对象`，通道Channel是对原IO中流的模拟，所有数据都要通过通道进行传输；Buffer实质上是一个容器对象，发送给通道的所有对象都必须首先放到一个缓冲区中。

###Buffer是什么？

Bufer是一个对象，它包含要写入或者刚读出的数据。这是NIO与IO的一个重要区别，在面向流的I/O中您将数据直接写入或者将数据直接读到stream中。
在 NIO 库中，`所有数据都是用缓冲区处理的`。在读取数据时，它是直接读到缓冲区中的。在写入数据时，它是写入到缓冲区中的。任何时候访问NIO中的数据，您都是将它放到缓冲区中。

`缓冲区实质上是一个数组`。通常它是一个字节数组，但是也可以使用其他种类的数组。**但是一个缓冲区不仅仅是一个数组。缓冲区提供了对数据的结构化访问，而且还可以跟踪系统的读/写进程**。

>简单的说Buffer是：一块连续的内存块，是NIO数据读或写的中转地。

###Buffer的类图结构
![Alt Buffer类图](/img/buffer.gif)

Buffer在JDK中是如何实现的？

>JDK源码可以知道，Buffer类是一个抽象类，其中有五个属性，分别是mark、position、limit、capacity、address。并且可以看到这样一行注释：
`//Invariants:mark<=position<=limit<=capacity`

一个 buffer 主要由 position、limit、capacity 三个变量来控制读写的过程。这三个变量在读和写时分别代表的含义如下：
`position	当前写入/读取的单位数据的数量`
`limit	代表最多能写入/读取多少单位的数据量，默认和capacity一致`
`capacity	Buffer的容量`

Buffer抽象类并没有指定Buffer的实现方式，看其子类可以发现，比如ByteBuffer中多出几个属性
`其中有个final byte[]类型的属性，可知Buffer其实是用数组实现的。`

###Buffer中的一些方法？

最基本的对应属性操作的方法，在JDK中不是使用set和get方法，查看源码知道要得到当前Buffer的limit值使用
`public final int limit()`方法，
设定limit的值使用
`public final Buffer limit(int)`方法，其它属性有对应的方法。

```
//用于将写模式转换成读模式
public final Buffer flip() {


    limit = position;    //将limit设置为刚才写入的位置
    position = 0;         //将position设置为0从头开始读
    mark = -1;
    return this;
}
//用于清空缓冲区(并不是真正的清除里面的数据)
//准备再次被写入，limit设置为capacity，position设置为0
public final Buffer clear() : 

//源码实现为position=0,mark=-1。目的是为了重复读。
public final Buffer rewind() : 

public final int remaining() : //一句代码return limit – position;

public final int hasRemaining() : //一句代码return limit > position;
```

###ByteBuffer类

首先可以看到在ByteBuffer类中多了三个属性，一个byte数组型的，一个int型的offset，还有一个boolean型的isReadOnly，两个构造函数均是Package-private型的。

可以使用一下方法产生一个ByteBuffer对象：

* (新建一个buffer)：ByteBuffer bbuf = ByteBuffer.allocate(1024);

查看源码知道allocate执行这样一句话：
` return new HeapByteBuffer(int capacity, int capacity);`

而HeapByteBuffer又是ByteBuffer的子类，并且在HeapByteBuffer的构造方法中执行的是这样一个语句：
`super(-1, 0, lim, cap, new byte[cap], 0);`

也就是说调用的还是ByteBuffer中的构造方法，包范围内使用。这个方法做了如下工作，首先调用Buffer的构造方法，依次初始化mark、position、limit、capacity，然后初始化ByteBuffer的属性byte数组，接着初始offset，这样使用allocate方法就可以构造出一个ByteBuffer对象了。

* (对一个存在的buffer进行wrap)：ByteBuffer bbuf = ByteBuffer.wrap(new Byte[1024] array, 0, 1024);

这个方法比较好用的一点是当这个Byte数组已经存在的话，直接传入这个Byte数组，然后传入起始值和结束值即可。默认wrap实现是初始值传入为0，结束值传入为Byte数组的长度array.length。

---

ByteBuffer类中其它重要方法：

取出:(将数据读入byte数组中)get(byte[] dst) 或者 get(byte[] dst, int offset, int len)
（该方法是用来获取当前ByteBuffer中的指定位置的数据并赋值给dst，最终返回当前对象本身。方法实现时第一步检查参数是否合法，调用的是checkBounds静态包范围私有方法。然后检查len是否大于remaining，接着对dst数组循环赋值，最终返回该对象。）

放入(将数组中的内容放在buffer中)put(byte[] src) 或者 put(byte[] src, int offerset, int len)

（该方法和上面的一对get方法类似，功能是将已有的byte数组从0位置开始放入当前的ByteBuffer中，最终返回ByteBuffer本身。）
put(ByteBuffer src)

（该方法将src的remaining逐个放入当前ByteBuffer中，最终返回当前ByteBuffer。）
除此之外还有类型化的get方法，例如getInt(), getFloat(), getShort()等。

###Buffer的更多内容？

缓冲区分片：slice()方法根据现有的缓冲区创建一种子缓冲区，新的缓冲区与原缓冲区共享部分数据。

只读缓冲区：可以通过调用缓冲区的 asReadOnlyBuffer() 方法，将任何常规缓冲区转换为只读缓冲区，这个方法返回一个与原缓冲区完全相同的缓冲区(并与其共享数据)，只不过它是只读的。只读缓冲区对于保护数据很有用。没有办法将只读缓冲区改变为可写的。

下面例子对缓冲区进行分片，并操作数据：
```
//产生一个ByteBuffer实例
ByteBuffer buffer = ByteBuffer.allocate( 10 );

//对该ByteBuffer实例进行初始化
for (int i=0; i<buffer.capacity(); ++i) {
buffer.put( (byte)i );
}

//修改buffer的position（起点）和limit（终点）

buffer.position( 3 );

buffer.limit( 7 );

//对缓冲区进行分片

ByteBuffer slice = buffer.slice();

//对分片的数据进行操作

for (int i=0; i<slice.capacity(); ++i) {

byte b = slice.get( i );

b *= 11;

slice.put( i, b );

}

//重新定位并输出结果

buffer.position( 0 );

buffer.limit( buffer.capacity() );

while (buffer.remaining()>0) {

System.out.println( buffer.get() );

}
```

直接或者间接缓冲区：直接缓冲区可以加快I/O的读写速度，使用allocateDirect(int capacity)产生一个直接缓冲区。

内存映射文件(将文件中的部分内容映射到内存中,加快读写)：下面代码将一个 FileChannel (它的全部或者部分)映射到内存

中。将文件的前1024个字节映射到内存中：

`MappedByteBuffer mbb = fc.map( FileChannel.MapMode.READ_WRITE, 0, 1024 );`

###Channel是什么？
![Alt Buffer类图](/img/Channels.gif)
Channel 是一个对象，可以通过它读取和写入数据(直接和底层的IO支持抽象)。拿 NIO 与原来的 I/O 做个比较，通道就像是流。

正如前面提到的，所有数据都通过 Buffer 对象来处理。您永远不会将字节直接写入通道中，相反，您是将数据写入包含一个或者多个字节的缓冲区。同样，您不会直接从通道中读取字节，而是将数据从通道读入缓冲区，再从缓冲区获取这个字节。

简单的说Channel是：数据的源头或者数据的目的地(写入就是目的地,读取就是源头,直接和底层IO交道)，用于向buffer提供数据或者读取buffer数据，并且对I/O提供异步支持。


###Channel的类图结构？



java.nio.channels.Channel是一个公共的接口，所有子Channel均实现了该接口
在java.nio.channels包中还实现了Channels、FileLock、SelectionKey、Selector、Pipe等比较好用的类。

`channel包含三种分别是包含socket，file和pipe三种管道，它实际上是双向交流的通道`。

Channel在JDK中是如何实现的？

在Channel接口中共定义了两个方法

`public boolean isOpen();   //Tells whether or not this channel is open`

`public void close() throws IOException();     //Close this channel`

####FileChannel 
使用以下三个方法可以得到一个FileChannel的实例
>FileInputStream.getChannel()
FileOutputStream.getChannel()
RandomAccessFile.getChannel()

上面提到Channel是数据的源头或者数据的目的地，用于向buffer提供数据或者从buffer读取数据。那么在实现了该接口的子类中应该有相应的read和write方法。

在FileChannel中有以下方法可以使用：

>public long read(ByteBuffer[] dsts)
//Reads a sequence of bytes from this channel into the given buffers.
public long write(ByteBuffer[] srcs)
//Writes a sequence of bytes to this channel from the given buffers.

####文件锁定
FileChannel提供两种方法获得FileLock
>FileLock lock();
FileLock lock(long position, long size, boolean size);

要获取文件的一部分上的锁，您要调用一个打开的 FileChannel 上的 lock() 方法。`注意，如果要获取一个排它锁，您必须以写方式打开文件`。
```
RandomAccessFile raf = new RandomAccessFile( “filelocks.txt”, “rw” );

FileChannel fc = raf.getChannel();

FileLock lock = fc.lock( start, end, false );

//原始Stream获取channel,然后获取channel的FileLock
//然后操作FileLock的Lock和release
//在拥有锁之后，您可以执行需要的任何敏感操作，然后再释放锁：

lock.release();
```
####SocketChannel  
使用以下两个方法得到一个SocketChannel的实例
>SocketChannel.open();      //打开一个socket channel
SocketChannel.open(SocketAddress remote);

//调用上面的方法，并connect(remote)
```
InetSocketAddress socketAddress = newbInetSocketAddress(“www.baidu.com”,80);

SocketChannel sc = SocketChannel.open(socketAddress);

sc.read(buffer);

buffer.flip();

buffer.clear();

sc.write(bufer);
```

####DatagramChannel 
与其它的Channel有相同或者相似的方法。

---

##参考资料

[Java nio入门教程 IBM开发者园地](https://www.ibm.com/developerworks/java/tutorials/j-nio/)
