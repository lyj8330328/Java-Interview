# 一、IO的方式

 IO的方式通常分为几种，同步阻塞的BIO、同步非阻塞的NIO、异步非阻塞的AIO。

## 1.1 BIO

BIO 就是传统的 [java.io](http://java.io/) 包，它是基于流模型实现的，交互的方式是同步、阻塞方式，也就是说在读入输入流或者输出流时，在读写动作完成之前，线程会一直阻塞在那里，它们之间的调用时可靠的线性顺序。它的有点就是代码比较简单、直观；缺点就是 IO 的效率和扩展性很低，容易成为应用性能瓶颈。

### 1.1.1 客户端

```java
package com.example.exe1;


import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.Socket;

/**
 * @Author: 98050
 * @Time: 2019-08-31 14:44
 * @Feature: 客户端
 */
public class TcpClient {

    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("127.0.0.1",6666);
        OutputStream outputStream = socket.getOutputStream();
        outputStream.write("你好服务器,我是客户端1".getBytes());

        InputStream inputStream = socket.getInputStream();
        byte[] bytes = new byte[1024];
        int len = inputStream.read(bytes);
        System.out.println(new String(bytes,0,len));
        inputStream.close();
        socket.close();
    }
}
```

### 1.1.2 服务器端

```java
package com.example.exe1;

import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;

/**
 * @Author: 98050
 * @Time: 2019-08-31 14:50
 * @Feature:
 */
public class TcpServer {

    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(6666);
        Socket socket;
        while (true) {
            socket = serverSocket.accept();
            ServerThread thread = new ServerThread(socket);
            thread.start();
        }
    }
}
```

```java
package com.example.exe1;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.Socket;

/**
 * @Author: 98050
 * @Time: 2019-08-31 15:22
 * @Feature:
 */
public class ServerThread extends Thread{

    Socket socket;

    public ServerThread(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        InputStream inputStream = null;
        try {
            inputStream = socket.getInputStream();
            byte[] bytes = new byte[1024];
            int len = inputStream.read(bytes);
            System.out.println(socket.getInetAddress() + new String(bytes, 0, len));
            OutputStream outputStream = socket.getOutputStream();
            outputStream.write("收到谢谢".getBytes());
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            try {
                socket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## 1.2 NIO

NIO基于Reactor，当socket有流可读或可写入socket时，操作系统会相应的通知引用程序进行处理，应用再将流读取到缓冲区或写入操作系统。  也就是说，这个时候，已经不是一个连接就要对应一个处理线程了，而是有效的请求，对应一个线程，当连接没有数据时，是没有工作线程来处理的。

 BIO与NIO一个比较重要的不同，是我们使用BIO的时候往往会引入多线程，每个连接一个单独的线程；而NIO则是使用单线程或者只使用少量的多线程，每个连接共用一个线程。

**NIO的最重要的地方是当一个连接创建后，不需要对应一个线程，这个连接会被注册到多路复用器上面，所以所有的连接只需要一个线程就可以搞定，当这个线程中的多路复用器进行轮询的时候，发现连接上有请求的话，才开启一个线程进行处理，也就是一个请求一个线程模式。**

HTTP/1.1出现后，有了Http长连接，这样除了超时和指明特定关闭的http header外，这个链接是一直打开的状态的，这样在NIO处理中可以进一步的进化，在后端资源中可以实现资源池或者队列，当请求来的话，开启的线程把请求和请求数据传送给后端资源池或者队列里面就返回，并且在全局的地方保持住这个现场(哪个连接的哪个请求等)，这样前面的线程还是可以去接受其他的请求，而后端的应用的处理只需要执行队列里面的就可以了，这样请求处理和后端应用是异步的.当后端处理完，到全局地方得到现场，产生响应，这个就实现了异步处理。

![](http://mycsdnblog.work/201919142024-w.png)

首先，Requester方通过Selector.open()创建了一个Selector准备好了调度角色。

创建了SocketChannel(ServerSocketChannel) 并注册到Selector中，通过设置key（SelectionKey）告诉调度者所应该关注的连接请求。

阻塞，Selector阻塞在select操作中，如果发现有Channel发生连接请求，就会唤醒处理请求。

## 1.3 AIO

异步非阻塞I/O，服务器实现模式为一个有效请求一个线程，**客户端的IO请求都是由操作系统先完成了再通知服务器用其启动线程进行处理**。AIO方式适用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分调用OS参与并发操作，编程比较复杂，jdk1.7开始支持。

## 1.4 总结



## 1.5 Java对BIO、NIO、AIO的支持

- Java BIO ： 同步并阻塞，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，当然可以通过线程池机制改善。
- Java NIO ： 同步非阻塞，服务器实现模式为一个请求一个线程，即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有I/O请求时才启动一个线程进行处理。
- Java AIO(NIO.2) ： 异步非阻塞，服务器实现模式为一个有效请求一个线程，客户端的I/O请求都是由OS先完成了再通知服务器应用去启动线程进行处理，

# 二、一个端口能建立几个TCP连接

为什么一个端口能建立多个TCP连接，同一个端口也就是说 server ip和server port 是不变的。那么只要[client ip 和 client port]不相同就可以了。能保证接唯一标识[server ip, server port, client ip, client port]的唯一性。

一个端口能建立多个UDP连接么？

UPD本身就是无连接的。所以不存在什么多个UDP连接。只是，服务端接收UDP数据需要bind一个端口。一个SOCKET只能绑定到一个端口。

# 三、多个线程可以监听同一个端口吗?

多个线程可以监听同一个端口。

但是，我们通常不这样做，因为，这样做了，不光多个线程能够共用这个端口，多个进程也能共用这个端口，我们无法感应那个进程在用这个端口。

最恐怖的事情是，共用相同端口的进线程，它们不会得到相同的数据，也就是说，一个客户端连接上来，发送数据，这些数据，会混乱的被多个线程处理。  

