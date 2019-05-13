# 一、RPC的实现原理

RPC主要解决两个问题：

- 解决分布式系统中，服务之间的调用问题
- 远程调用时，就像调用本地方法一样，让调用者感知不到远程调用的逻辑

以计算器Calculator为例，如果实现类CalculatorImpl是放在本地的，那么直接调用即可：

![](http://mycsdnblog.work/201919141437-0.png)

现在系统变成分布式了，CalculatorImpl和调用方不在同一个地址空间，那么就必须要进行远程过程调用：

![](http://mycsdnblog.work/201919141440-A.png)

那么如何实现远程过程调用，也就是RPC呢，一个完整的RPC流程，可以用下面这张图来描述：

![](http://mycsdnblog.work/201919141440-L.png)

其中左边的Client，对应的就是前面的Service A，而右边的Server，对应的则是Service B。

1. Service A的应用层代码中，调用了Calculator的一个实现类的add方法，希望执行一个加法运算；
2. 这个Calculator实现类，内部并不是直接实现计算器的加减乘除逻辑，而是通过远程调用Service B的RPC接口，来获取运算结果，因此称之为**Stub**；
3. Stub怎么和Service B建立远程通讯呢？这时候就要用到**远程通讯工具**了，也就是图中的**Run-time Library**，这个工具将帮你实现远程通讯的功能，比如Java的**Socket**，就是这样一个库，当然，你也可以用基于Http协议的**HttpClient**，或者其他通讯工具类，都可以，**RPC并没有规定说你要用何种协议进行通讯**；
4. Stub通过调用通讯工具提供的方法，和Service B建立起了通讯，然后将请求数据发给Service B。需要注意的是，由于底层的网络通讯是基于**二进制格式**的，因此这里Stub传给通讯工具类的数据也必须是二进制，比如calculator.add(1,2)，你必须把参数值1和2放到一个Request对象里头（这个Request对象当然不只这些信息，还包括要调用哪个服务的哪个RPC接口等其他信息），然后**序列化**为二进制，再传给通讯工具类，这一点也将在下面的代码实现中体现；
5. 二进制的数据传到Service B这一边了，Service B当然也有自己的通讯工具，通过这个通讯工具接收二进制的请求；
6. 既然数据是二进制的，那么自然要进行**反序列化**了，将二进制的数据反序列化为请求对象，然后将这个请求对象交给Service B的Stub处理；
7. 和之前的Service A的Stub一样，这里的Stub也同样是个“假玩意”，它所负责的，只是去解析请求对象，知道调用方要调的是哪个RPC接口，传进来的参数又是什么，然后再把这些参数传给对应的RPC接口，也就是Calculator的实际实现类去执行。很明显，如果是Java，那这里肯定用到了**反射**。
8. RPC接口执行完毕，返回执行结果，现在轮到Service B要把数据发给Service A了，怎么发？一样的道理，一样的流程，只是现在Service B变成了Client，Service A变成了Server而已：Service B反序列化执行结果->传输给Service A->Service A反序列化执行结果 -> 将结果返回给Application，完毕。

# 二、实现

![](http://mycsdnblog.work/201919141606-9.png)

## 2.1 公共模块

```java
package com.example.rpc.common;

/**
 * @Author: 98050
 * @Time: 2019-03-14 15:05
 * @Feature:
 */
public interface HelloService {

    /**
     * 接收一条消息
     * @param content
     * @return
     */
    String sayHello(String content);

    /**
     * 接收一个用户
     * @param user
     * @return
     */
    String saveUser(User user);
}
```

```java
package com.example.rpc.common;

import java.io.Serializable;

/**
 * @Author: 98050
 * @Time: 2019-03-14 15:21
 * @Feature:
 */
public class RPCRequest implements Serializable {

    private static final long serialVersionUID = -6956922122610203456L;
    private String classNmae;
    private String methodName;
    private Object[] parameters;

    public String getClassNmae() {
        return classNmae;
    }

    public void setClassNmae(String classNmae) {
        this.classNmae = classNmae;
    }

    public String getMethodName() {
        return methodName;
    }

    public void setMethodName(String methodName) {
        this.methodName = methodName;
    }

    public Object[] getParameters() {
        return parameters;
    }

    public void setParameters(Object[] parameters) {
        this.parameters = parameters;
    }
}
```

```java
package com.example.rpc.common;

import java.io.Serializable;

/**
 * @Author: 98050
 * @Time: 2019-03-14 15:02
 * @Feature:
 */
public class User implements Serializable {
    /**
     * 当前类的标识
     */
    private static final long serialVersionUID = -1971211899635907185L;

    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                '}';
    }
}
```

一共有三个公共类，调用的方法接口、pojo，用来封装请求的实体。

## 2.2 服务端

服务端要做的事情：实现接口，接收请求，处理请求，返回结果。

### 2.2.1 实现接口

```java
package com.example.rpc.server;

import com.example.rpc.common.HelloService;
import com.example.rpc.common.User;

/**
 * @Author: 98050
 * @Time: 2019-03-14 15:06
 * @Feature:
 */
public class HelloServiceImpl implements HelloService {
    @Override
    public String sayHello(String content) {
        return "hello world :" + content;
    }

    @Override
    public String saveUser(User user) {
        System.out.println(user);
        return "success";
    }
}
```

### 2.2.2 接收请求

采用BIO模式，一个连接就创建一个线程，使用线程池

```java
package com.example.rpc.server;

import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.*;

/**
 * @Author: 98050
 * @Time: 2019-03-14 15:07
 * @Feature:
 */
public class RpcServerProxy {

    private ExecutorService executorService = new ThreadPoolExecutor(0, Integer.MAX_VALUE,
            60L, TimeUnit.SECONDS,
            new SynchronousQueue<Runnable>());

    public void publish(Object service,int port) throws IOException {
        ServerSocket serverSocket = null;
        try{
            serverSocket = new ServerSocket(port);
            while (true){
                /**
                 * 接收一个请求(BIO)
                 */
                Socket socket = serverSocket.accept();
                executorService.execute(new ProcessorHandler(socket, service));
            }
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if (serverSocket != null){
                serverSocket.close();
            }
        }
    }
}
```

具体业务操作在handler中进行：

```java
package com.example.rpc.server;

import com.example.rpc.common.RPCRequest;

import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.net.Socket;

/**
 * @Author: 98050
 * @Time: 2019-03-14 15:14
 * @Feature:
 */
public class ProcessorHandler implements Runnable{

    private Socket socket;
    private Object service;


    public ProcessorHandler(Socket socket, Object service) {
        this.socket = socket;
        this.service = service;
    }

    @Override
    public void run() {
        System.out.println("开始处理客户端请求");
        ObjectInputStream objectInputStream = null;
        try {
            objectInputStream = new ObjectInputStream(socket.getInputStream());
            /**
             * 反序列化
             */
            RPCRequest rpcRequest = (RPCRequest) objectInputStream.readObject();
            Object result = invoke(rpcRequest);
            ObjectOutputStream outputStream = new ObjectOutputStream(socket.getOutputStream());
            outputStream.writeObject(result);
            outputStream.flush();
        } catch (IOException | ClassNotFoundException | NoSuchMethodException | IllegalAccessException | InvocationTargetException e) {
            e.printStackTrace();
        } finally {
            try {
                assert objectInputStream != null;
                objectInputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    private Object invoke(RPCRequest rpcRequest) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        Object[] parameters = rpcRequest.getParameters();
        Class<?>[] type = new Class[parameters.length];
        for (int i = 0; i < parameters.length; i++) {
            type[i] = parameters[i].getClass();
        }
        Method method = service.getClass().getMethod(rpcRequest.getMethodName(), type);
        return method.invoke(service, parameters);
    }
}
```

接收到请求，将请求对象进行反序列化，获取调用的方法名称和方法参数，在invoke方法里面使用Java反射技术得到方法的运行结果；拿到结果后，将结果返回给客户端。

### 2.2.3 启动服务端

```java
package com.example.rpc;

import com.example.rpc.server.RpcServerProxy;
import com.example.rpc.common.HelloService;
import com.example.rpc.server.HelloServiceImpl;

import java.io.IOException;

/**
 * @Author: 98050
 * @Time: 2019-03-14 15:01
 * @Feature:
 */
public class Server {

    public static void main(String[] args) throws IOException {
        HelloService helloService = new HelloServiceImpl();
        RpcServerProxy rpcServerProxy = new RpcServerProxy();
        rpcServerProxy.publish(helloService, 8080);
    }
}
```

开始提供服务，端口是8080

## 2.3 客户端

客户端的要做的就是获取接口对象，然后调用内部的方法，得到返回结果。

但是客户端没有接口的实现，所以如何获取接口对象呢？通过动态代理，但是在代理的过程中我们只是把被代理方法的名字和参数发送到服务器端，然后获取返回结果。

### 2.3.1 获取接口对象

```java
package com.example.rpc.client;

import java.lang.reflect.Proxy;

/**
 * @Author: 98050
 * @Time: 2019-03-14 15:44
 * @Feature:
 */
public class RpcClientProxy {
    public <T> T clientProxy(Class<T> interfaces,String host,int port){
        return (T) Proxy.newProxyInstance(interfaces.getClassLoader(), new Class<?>[]{interfaces},new RemoteInvocationHandler(host,port));
    }
}
```

### 2.3.2 实现Handler

```java 
package com.example.rpc.client;

import com.example.rpc.common.RPCRequest;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

/**
 * @Author: 98050
 * @Time: 2019-03-14 15:49
 * @Feature:
 */
public class RemoteInvocationHandler implements InvocationHandler {
    private String host;
    private int port;

    public RemoteInvocationHandler(String host, int port) {
        this.host = host;
        this.port = port;
    }

    /**
     * 返回调用的结果
     * @param proxy
     * @param method
     * @param args
     * @return
     * @throws Throwable
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        RPCRequest request = new RPCRequest();
        request.setMethodName(method.getName());
        request.setParameters(args);
        RpcNetTransport rpcNetTransport = new RpcNetTransport(host, port);
        return rpcNetTransport.sendRequest(request);
    }
}
```

通过动态代理，拦截到被代理的方法，获取方法名称和参数，封装到请求实体中，然后发送给服务器端，并且获取返回结果，最终返回给调用者。

具体的请求发送以及结果返回，封装在RpcNetTransport中，即消息处理

```java
package com.example.rpc.client;

import com.example.rpc.common.RPCRequest;

import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.net.Socket;

/**
 * @Author: 98050
 * @Time: 2019-03-14 15:52
 * @Feature:
 */
public class RpcNetTransport {

    private String host;
    private int port;

    public RpcNetTransport(String host, int port) {
        this.host = host;
        this.port = port;
    }

    private Socket newSocket(){
        System.out.println("创建一个新的socket连接");
        Socket socket;
        try{
            socket = new Socket(host,port);
        } catch (IOException e) {
            throw  new RuntimeException("连接建立失败");
        }
        return socket;
    }

    public Object sendRequest(RPCRequest rpcRequest) throws IOException {
        Socket socket = null;
        try{
            socket = newSocket();
            ObjectOutputStream outputStream = new ObjectOutputStream(socket.getOutputStream());
            outputStream.writeObject(rpcRequest);
            outputStream.flush();

            ObjectInputStream objectInputStream = new ObjectInputStream(socket.getInputStream());
            Object object = objectInputStream.readObject();

            objectInputStream.close();
            outputStream.close();

            return object;
        } catch (IOException | ClassNotFoundException e) {
            throw new RuntimeException("发送数据异常" + e);
        }finally {
            if (socket != null){
                socket.close();
            }
        }
    }
}
```

### 2.3.3 服务调用

```java
package com.example.rpc;

import com.example.rpc.client.RpcClientProxy;
import com.example.rpc.common.User;
import com.example.rpc.common.HelloService;

/**
 * @Author: 98050
 * @Time: 2019-03-14 15:39
 * @Feature:
 */
public class Client {

    public static void main(String[] args) {
        RpcClientProxy proxy = new RpcClientProxy();
        HelloService helloService = proxy.clientProxy(HelloService.class, "localhost", 8080);
        User user = new User();
        user.setName("李四");
        System.out.println(helloService.saveUser(user));
        //System.out.println(helloService.sayHello("123123123123123"));
    }
}
```

### 2.3.4 测试

开始服务端：

![](http://mycsdnblog.work/201919141741-c.png)

客户端调用saveUser方法：

![](http://mycsdnblog.work/201919141742-C.png)

服务端接收：

![](http://mycsdnblog.work/201919141742-Q.png)

客户端调用sayHello方法：

![](http://mycsdnblog.work/201919141743-Y.png)

服务端：

![](http://mycsdnblog.work/201919141743-6.png)