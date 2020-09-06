# IO

---

## 1. 字节流

设备上的所有数据都是以二进制存储的，而二进制最终都是以一个 8bit 的数据单元进行体现，所以计算机中的最小数据单元就是字节。Java 中字节流处理数据的基本单位就是字节。

关于 read()：

* InputStream 是一个抽象类，存在一个抽象方法 read()，这个方法会返回一个 [0, 255] 的整数，因为一个字节有 8bit，共计 256 种状态，read() 会将读取到的 8bit 数据转换为 int 返回，对应 0 ~ 255
* 这个方法会一直阻塞，直到读取结束返回 -1，所以 read() 一般放在循环中，不用担心循环导致 CPU 空转

## 2. 编码

编解码就是通过码表来将字符和字节数据对应的过程。

码表：

* GBK：英文 1 字节，中文 2 字节
* UTF-8：英文 1 字节，中文 3 字节
* Unicode（UTF-16BE）：每个字符 2 字节。Unicode 码元就是一个 Unicode 代码单元，范围是 0x0000 ~ 0xFFFF，在这个范围内的每个值都与一个字符对应。**Java 中的 String 和 char 就默认把字符用 Unicode 规则编码**

## 3. 字符流

由于存在多种编码方式，所以根据编码方式不同，字符流基于字节流在读取字节时查找指定码表，从而读取不同的字节数来直接映射成字符。

所以相应的字符流的 read() 的返回值范围就是 [-1, 0xFFFF]，即返回的字符范围为 0 ~ 65535。

## 4. 打印流

打印流用于输出各种类型的数据，如

```java
public void print(int i){
    write(String.valueOf(i));
}
```

print() 有各种数据类型的重载，大部分都是调用 String.valueOf()，而相应的 valueOf() 也有针对各种数据类型的重载。

## 5. 标准流

System 中有三个静态属性：

* in：InputStream
* out / err：PringStream

in 是标准输入流，通过 InputStream 中的 read() 默认读取标准输入（键盘）的数据，out 则是借助打印流将内容打印到标准输出（控制台），err 与 out 的区别是打印的结果是红色：

```java
int num = System.in.read();
System.out.print(num);
// 输入 1
// 输出 49
```

## 5. BIO

Java 中阻塞 IO 的连接、读、写都会阻塞线程：

* 连接导致阻塞：希望的是有连接到达时再进行 accept，而不是一直等待着连接到达
  
  ```java
  while(true){
      // accept() 会阻塞住执行，直到有客户端连接进来
      Socket socket= serverSocket.accept();
  }
  ```

* 读（写）导致阻塞：服务端准备开始读（写）数据但数据没准备好，而我们希望的是数据准备再进行读写操作

使用多线程：在进行阻塞 IO 时，无从得知连接是否到达、能不能写、能不能读，只能被动等待导致线程阻塞，所以除了开多线程，没有好的办法利用CPU。但 IO 过程也会导致线程阻塞从而造成频繁的上下文切换，这就是阻塞 IO 的弊端。

## 6. NIO

NIO 引入了 Channel 和 Buffer 的概念：

* Channel：表示对文件、Socket 等能够进行 IO 的设备的连接
* Buffer：对内存中一块区域的包装，方便对其中的数据进行操作

数据总是由 Channel 读到 Buffer，或由 Buffer 写入 Channel。BIO 是基于流进行操作，数据读取过程中没有缓存，而 NIO 由于引入了 Buffer，通过 Buffer 的封装可以更灵活的操作数据。

以上是 NIO 在文件读写的应用，但由于 UNIX 不支持文件的非阻塞 IO，所以在 Java 中文件 NIO 不支持非阻塞模式。

### 6.1 非阻塞 IO

NIO 中的 Socket NIO 是支持非阻塞模式的，举例来说，ServerSocketChannel 中的 accept() 在非阻塞模式下、没有连接可被 accept 时会返回 null 而不是阻塞住。

### 6.2 如何利用非阻塞特性

由于 NIO 的连接、读、写都可以立刻返回，如果使用轮询的方式直接调用就成了 Unix 的非阻塞 IO，浪费 CPU 资源，所以需要一种机制来得知连接到达或数据可读或可写，所以 NIO 还引入了 Selector：

* Selector 是 Channel 的多路复用器，Java 中的 Selector 是对操作系统 select / poll / epoll 的封装，是阻塞的，它会让出 CPU 资源，所以一般放在 while(true) 中使用而不用担心导致 CPU 空转

将连接到达、可读可写事件注册到 Selector 上，当有事件触发时 select() 就会退出阻塞，然后查看具体触发的事件种类（SelectionKey）进行相应的操作即可。这种事件注册的形式正是 Java NIO 对多路复用 IO 模型的体现。

也就是说，相比与 BIO， **NIO 将线程对 IO 操作的粒度变小了**：BIO 中一个线程将一次连接中所有的事情都做了，而 NIO 将这种情况进行了拆分。

需要注意的是，由于 NIO 中读写、accept 都是非阻塞的，所以理论上在 select 解除阻塞以后（也就是事件到达），后续的操作一个线程就可以完成，因为几乎都是纯 CPU 操作，效率很高。而这种模型就可以看作是单线程 Reactor 的朴素形式，只不过没有抽象出各个角色而已。

### 6.3 NIO 的粘包断包

BIO 通过流来传输数据，在传输完毕之前会阻塞，但 NIO 的数据是从 channel 写入 buffer 的，根据 buffer 定义的大小不同，一次读入 buffer 的数据可能不完整或一次读入了两个及两个以上的数据。数据之间没有明确的界限，就会出现粘包断包。

解决：调整 buffer 为要读取数据的大小

* 发送端发送数据前发送一个 int 类型的数字，表示数据长度
* 接收端读到这个数字后，创建对应大小的 buffer
* channel.read() 返回实际读取的大小，一次 OP_READ 中循环调用 channel.read()，当返回值累计达到要读取数据段大小时表示读取完毕

### 6.4 使用场景

NIO 适用于大连接数的长时间 IO，但不会频繁发送大量数据，偏向于使用较少资源维护大量的连接数。

而 BIO 则更适合传输大文件，但是连接数不能过大。