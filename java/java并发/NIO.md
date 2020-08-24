## NIO

### NIO和IO的区别

| IO         | NIO        |
| ---------- | ---------- |
| 面向Stream | 面向Buffer |
| 阻塞IO     | 非阻塞IO   |
|            | Selectors  |

**面向Stream和面向Buffer**

Java NIO和IO之间最大的区别是IO是面向流（Stream）的，NIO是面向块（buffer）的，所以，这意味着什么？

面向流意味着从流中一次可以读取一个或多个字节，拿到读取的这些做什么你说了算，这里没有任何缓存（这里指的是使用流没有任何缓存，接收或者发送的数据是缓存到操作系统中的，流就像一根水管从操作系统的缓存中读取数据）而且只能顺序从流中读取数据，如果需要跳过一些字节或者再读取已经读过的字节，你必须将从流中读取的数据先缓存起来**。**

面向块的处理方式有些不同，数据是先被 读/写到buffer中的，根据需要你可以控制读取什么位置的数据。这在处理的过程中给用户多了一些灵活性**，**然而，你需要额外做的工作是检查你需要的数据是否已经全部到了buffer中，你还需要保证当有更多的数据进入buffer中时，buffer中未处理的数据不会被覆盖

**阻塞IO和非阻塞IO**

所有的Java IO流都是阻塞的，这意味着，当一条线程执行read()或者write()方法时，这条线程会一直阻塞知道读取到了一些数据或者要写出去的数据已经全部写出，在这期间这条线程不能做任何其他的事情

java NIO的非阻塞模式(Java NIO有阻塞模式和非阻塞模式，阻塞模式的NIO除了使用Buffer存储数据外和IO基本没有区别)允许一条线程从channel中读取数据，通过返回值来判断buffer中是否有数据，如果没有数据，NIO不会阻塞，因为不阻塞这条线程就可以去做其他的事情，过一段时间再回来判断一下有没有数据

NIO的写也是一样的，一条线程将buffer中的数据写入channel，它不会等待数据全部写完才会返回，而是调用完write()方法就会继续向下执行

下面是测试的代码：

(1)首先是IO

```java
public class QQServer {
    static byte[] bytes = new byte[1024];

    public static void main(String[] args) {
        try {
            //服务器的类似于监听的socket
            ServerSocket serverSocket = new ServerSocket();
            serverSocket.bind(new InetSocketAddress(8080));
            while(true){
                System.out.println("wait conn");
                //阻塞
                Socket socket = serverSocket.accept();
                System.out.println("conn success");
                System.out.println("wait data");
                //阻塞  read读了多少字节
                int read = socket.getInputStream().read(bytes);
                System.out.println("data success");
                String content = new String(bytes);
                System.out.println(content);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

解释如下：

- 当代码读到Socket socket = serverSocket.accept()时，代码处于阻塞的状态，即下面的代码不会执行下去了，等到有连接了，才会执行，假设有10000个请求，就需要有10000个连接，而且都是阻塞在这里的，性能低下
- 当代码读到int read = socket.getInputStream().read(bytes)的时候，也是处于阻塞状态，等到读到字节了才会继续走下去，不然的话什么都做不了



(2)然后是NIO

```java
public class QQServer_NIO {
    static List<SocketChannel> list = new ArrayList<>();
    static ByteBuffer byteBuffer = ByteBuffer.allocate(512);
    public static void main(String[] args) throws IOException, InterruptedException {
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        serverSocketChannel.bind(new InetSocketAddress(8080));
        serverSocketChannel.configureBlocking(false);

        while(true){
            SocketChannel socketChannel = serverSocketChannel.accept();
            if(socketChannel == null){
                Thread.sleep(1000);
                System.out.println("no conn");
                for (SocketChannel client: list) {
                    int k = client.read(byteBuffer);
                    if(k >0){
                        byteBuffer.flip();
                        System.out.println(byteBuffer.toString());
                    }
                }
            }else {
                System.out.println("有人连接了");
                socketChannel.configureBlocking(false);
                list.add(socketChannel);
                for (SocketChannel client: list) {
                    int k = client.read(byteBuffer);
                    if(k >0){
                        byteBuffer.flip();
                        System.out.println(byteBuffer.toString());
                    }
                }
            }
        }
    }
}

```

解释如下：

- 当代码读到 SocketChannel socketChannel = serverSocketChannel.accept()时候，代码不会阻塞，回执行下面的代码
- 当代码读到int k = client.read(byteBuffer)不会阻塞

**Selectors**

Java NIO的selectors允许一条线程去监控多个channels的输入，你可以向一个selector上注册多个channel，然后调用selector的select()方法判断是否有新的连接进来或者已经在selector上注册时channel是否有数据进入。selector的机制让一个线程管理多个channel变得简单。

**NIO和IO对应用的设计有何影响**

选择使用NIO还是IO做你的IO工具对应用主要有以下几个方面的影响

1、使用IO和NIO的API是不同的（废话）

2、处理数据的方式

3、处理数据所用到的线程数

**处理数据的方式**

在IO的设计里，要一个字节一个字节从InputStream 或者Reader中读取数据，想象你正在处理一个向下面的基于行分割的流



```java
Name:AnnaAge: 25Email: anna@mailserver.comPhone:1234567890
```

处理文本行的流的代码应该向下面这样



```java
InputStream input = ... ; // get the InputStream from the client socket BufferedReader reader = new BufferedReader(new InputStreamReader(input)); String nameLine   = reader.readLine();String ageLine    = reader.readLine();String emailLine  = reader.readLine();String phoneLine  = reader.readLine();
```

注意，一旦reader.readLine()方法返回，你就可以确定整行已经被读取，readLine()阻塞知道一整行都被读取

![img](https://img-blog.csdn.net/20160624201306581?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

NIO的实现会有一些不同，下面是一个简单的例子



```java
ByteBuffer buffer = ByteBuffer.allocate(48); int bytesRead = inChannel.read(buffer);
```

注意第二行从channel中读取数据到ByteBuffer，当这个方法返回你不知道是否你需要的所有数据都被读到buffer了，你所知道的一切就是有一些数据被读到了buffer中，但是你并不知道具体有多少数据，这使程序的处理变得稍微有些困难

想象一下，调用了read(buffer)方法后，只有半行数据被读进了buffer，例如：“Name: An”，你能现在就处理数据吗？当然不能。你需要等待直到至少一整行数据被读到buffer中，在这之前确保程序不要处理buffer中的数据

你如何知道buffer中是否有足够的数据可以被处理呢？你不知道，唯一的方法就是检查buffer中的数据。可能你会进行几次无效的检查（检查了几次数据都不够进行处理），这会令程序设计变得比较混乱复杂



```java
ByteBuffer buffer = ByteBuffer.allocate(48); int bytesRead = inChannel.read(buffer); while(! bufferFull(bytesRead) ) {    bytesRead = inChannel.read(buffer);}
```

bufferFull方法负责检查有多少数据被读到了buffer中，根据返回值是true还是false来判断数据是否够进行处理。bufferFull方法扫描buffer但不能改变buffer的内部状态

is-data-in-buffer-ready 循环柱状图如下

![img](https://img-blog.csdn.net/20160624202700773?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

**总结**

**NIO允许你用一个单独的线程或几个线程管理很多个channels（网络的或者文件的），代价是程序的处理和处理IO相比更加复杂**

**如果你需要同时管理成千上万的连接，但是每个连接只发送少量数据，例如一个聊天服务器，用NIO实现会更好一些，相似的，如果你需要保持很多个到其他电脑的连接，例如P2P网络，用一个单独的线程来管理所有出口连接是比较合适的**



**![img](https://img-blog.csdn.net/20160624203325908?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)**

**如果你只有少量的连接但是每个连接都占有很高的带宽，同时发送很多数据，传统的IO会更适合**

**![img](https://img-blog.csdn.net/20160624203459503?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)**


  

