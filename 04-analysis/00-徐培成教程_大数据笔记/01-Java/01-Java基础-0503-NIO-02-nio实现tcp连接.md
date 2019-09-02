## 1, nio的tcp连接Server

```java
package im.ivanl001.bigData.java_01.Java.IM004_TCP_NIO;

import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;

/**
 * #author      : ivanl001
 * #creator     : 2018-10-16 20:26
 * #description : nio的服务端的套接字
 **/
public class IMSocket_Server_NIO {

    public static void main(String[] args) throws Exception {

        //1, 首先开启服务器端的socket通道
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        serverSocketChannel.bind(new InetSocketAddress("localhost", 8888));
        serverSocketChannel.configureBlocking(false);


        //2, 然后开启选择器，并为serverSocketChannel注册到选择器中
        Selector selector = Selector.open();


        //3,注册接受消息到选择器中去
        //通道需要到挑选器中进行注册，这里注册accept事件，也就是socket = serverSocket.accept();这个事件
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);


        //4，开始进行选择, 进行轮训
        while (true) {

            //5，挑选符合事件, 先取出key
            selector.select();//这个和accept一样，会堵塞住当前线程
            Iterator<SelectionKey> iterators = selector.selectedKeys().iterator();

            //6，对选择出来的通道进行处理
            while (iterators.hasNext()) {

                SelectionKey selectionKey = iterators.next();

                //捕获异常，以免出现问题的时候移除key
                try {

                    //从注册的accept事件把可读的通道拿出来，方便后面轮训读取
                    if (selectionKey.isAcceptable()) {
                        //说明是服务器端的socket，也就是说有服务器socket因为被监听到accept被添加到选择器中了， 我们需要把这个socket拿出来给它添加监听事件
                        ServerSocketChannel serverSocketChannel1 = (ServerSocketChannel) selectionKey.channel();
                        SocketChannel socketChannel = serverSocketChannel1.accept();

                        socketChannel.configureBlocking(false);
                        socketChannel.register(selector, SelectionKey.OP_READ);//这里监听可读就好了
                    }

                    //这里就是可读的通道，读出来即可
                    if (selectionKey.isReadable()) {
                        //说明可以读取
                        SocketChannel socketChannel = (SocketChannel) selectionKey.channel();

                        byte[] prefixByte = "prefix:".getBytes();
                        byteBuffer.put(prefixByte, 0, prefixByte.length);

                        //TODO: 这里不能等于-1，因为对方通道不关闭，永远不读到-1，这里不是很懂
                        while ((socketChannel.read(byteBuffer)) != 0) {
                            byteBuffer.flip();
                            socketChannel.write(byteBuffer);
                            System.out.println("长度:" + byteBuffer.toString());
                            byteBuffer.clear();
                            System.out.println("长度:" + byteBuffer.toString());

                        }
                    }

                } catch (Exception e) {
                    System.out.println(e);
                    selector.keys().remove(selectionKey);
                }
            }

            selector.selectedKeys().clear();//这里需要把处理过的可以都拿掉，如果除了异常，也需要把异常的那个key拿掉，我这里暂时不做处理
        }
    }
}
```

## 2, nio的tcp连接client

```java
package im.ivanl001.bigData.java_01.Java.IM004_TCP_NIO;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.SocketChannel;

/**
 * #author      : ivanl001
 * #creator     : 2018-10-16 21:03
 * #description : nio的socket客户端实现
 **/
public class IMSocket_Client_NIO {


    public static void main(String[] args) throws Exception {

        //1，打开通道并进行监听
        final SocketChannel socketChannel = SocketChannel.open();
        socketChannel.connect(new InetSocketAddress("localhost", 8888));
        socketChannel.configureBlocking(false);

        //2，开启选择器，并注册读事件
        Selector selector = Selector.open();
        socketChannel.register(selector, SelectionKey.OP_READ);

        //3，准备缓冲区，后续轮训会用到
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();


        //4，这里开启一个通道是为了写消息
        new Thread(){

            public void run() {
                int i = 0;
                while (true) {

                    ByteBuffer byteBuffer1 = ByteBuffer.allocate(1024);
                    i += 1;
                    String msgSend = "ivanl00" + i;
                    byteBuffer1.put((msgSend).getBytes());
                    //flip是为了给写出准备
                    byteBuffer1.flip();

                    try {
                        Thread.sleep(1000);
                        socketChannel.write(byteBuffer1);
                        byteBuffer1.clear();
                        System.out.println("发送消息：" + msgSend);
                        System.out.println("发送消息的长度是：" + msgSend.getBytes().length);
                    } catch (IOException e) {
                        e.printStackTrace();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }.start();

        //5，开始轮训选择器
        while (true) {

            selector.select();//开始进行选择，其实只要选择到就是自己，因为在本类中只注册了一个


            while (socketChannel.read(byteBuffer) > 0) {

                byteBuffer.flip();
                //注意：这里一定要注意，write的时候需要指定长度！！！！！！
                byteArrayOutputStream.write(byteBuffer.array(), 0, byteBuffer.limit());
                byteBuffer.clear();

                System.out.println("接受消息:" + new String(byteArrayOutputStream.toByteArray()));
                //请一定要注意：如果byteArrayOutputStream是在死循环外面创建的，一定要重置，否则之前的数据也会保存在里面！！！！！
                byteArrayOutputStream.reset();
            }

            selector.selectedKeys().clear();
        }
    }
}
```



## 方便对比，常规方式的tcp连接实现也贴一下

## 3, io的tcp连接Server

```java
package im.ivanl001.bigData.java_01.Java.IM002_TCP_IO;


import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.HashMap;
import java.util.Map;

/**
 * #author      : ivanl001
 * #creator     : 2018-10-14 20:07
 * #description :
 **/
public class ServerApp {

    //这里需要维护一个连接的用户信息
    private static Map<String, Socket> connectedUser = new HashMap<>();

    public static void main(String[] args) throws IOException {

        ServerSocket serverSocket = new ServerSocket(8888);
        Socket socket =null;
        System.out.println("服务器开始，端口号是：" + serverSocket.getLocalPort());

        while (true) {

            //每一次有一个新的连接，这里就会接受到一次，也就会根据这次到socket创建一个新的线程
            //后面会使用NIO避免这种线程创建的浪费问题
            socket = serverSocket.accept();
            String ipStr = socket.getInetAddress().getHostAddress();
            int port     = socket.getPort();
            String connectedUserKey = ipStr + ":" + port;

            System.out.println("用用户连接，用户的连接key是 ：" + connectedUserKey);
            connectedUser.put(connectedUserKey, socket);

            IM_TCP_Server_Thread serverThread =new IM_TCP_Server_Thread(socket, connectedUser);
            serverThread.start();

            System.out.println("客户端连接的数量："+connectedUser.size());
        }
    }
}
```

## 4, io的tcp连接client

```java
package im.ivanl001.bigData.java_01.Java.IM002_TCP_IO;

import java.io.IOException;
import java.net.Socket;

/**
 * #author      : ivanl001
 * #creator     : 2018-10-14 20:27
 * #description : 客户端入口
 **/
public class ClientApp {

    public static void main(String[] args) throws IOException, InterruptedException {

        //1,设置socket服务器地址和端口，这里的host和port都是服务器的
        Socket socket_client = new Socket("localhost", 8888);

        //2,发送消息
        IM_TCP_Client_Thread client_thread = new IM_TCP_Client_Thread(socket_client);
        client_thread.start();

        //3, 主线cheng主线程主要交给ui做一些其他方面的事情
        int i = 0;
        while (true) {
            i += 1;
            //System.out.println("更新UI" + i);
            Thread.sleep(10000);
        }
    }
}
```



## 5，3,4中用到的几个类

* IMSocketUtils

```java
package im.ivanl001.bigData.java_01.Java.IM002_TCP_IO;

import im.ivanl001.bigData.java_01.IMUtils.IMByteArr0NumUtils;

import java.io.IOException;
import java.io.InputStream;
import java.net.Socket;

/**
 * #author      : ivanl001
 * #creator     : 2018-10-15 19:23
 * #description :
 **/
public class IMSocketUtils {

    public static void receiveMsg(Socket socket) {
        while (true) {
            //在这里解析客户端发送的消息
            try {
                InputStream inputStream = socket.getInputStream();

                //因为如果不指定协议的话，你是不知道这条数据的长度的，所以就不能进行读取了
                byte[] msgLenBytes = new byte[4];
                inputStream.read(msgLenBytes);
                int msgLen = IMByteArr0NumUtils.byteArr2Int(msgLenBytes);
                System.out.println("消息长度是:" + msgLen);

                byte[] msgBytes = new byte[msgLen];
                inputStream.read(msgBytes);
                String string = new String(msgBytes);

                System.out.println("接收到客户端消息：" + string);

            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    public static void sendMsg(String msg, Socket socket) {

        try {

            //我们是需要指定发送的协议的，数据长度需要包含在发送的数据包中，要不然接收的socket是不知道如何进行解析的
            int msgLen = msg.getBytes().length;
            //把长度转换成四个长度的字节数组，写入到数据中
            byte[] msgLenBytes = IMByteArr0NumUtils.int2byteArr(msgLen);
            byte[] msgBytes = msg.getBytes();
            byte[] msgPack = new byte[4 + msgLen];//消息的长度加上4个字节的长度，这四个字节是用来记录消息的长度信息的
            System.arraycopy(msgLenBytes, 0, msgPack, 0, 4);
            System.arraycopy(msgBytes, 0, msgPack, 4, msgLen);

            socket.getOutputStream().write(msgPack);
            socket.getOutputStream().flush();
            System.out.println("client:" + msg);

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

* IM_TCP_Server_Thread

```java
package im.ivanl001.bigData.java_01.Java.IM002_TCP_IO;

import im.ivanl001.bigData.java_01.IMUtils.IMByteArr0NumUtils;

import java.io.IOException;
import java.io.InputStream;
import java.net.Socket;
import java.util.Map;

/**
 * #author      : ivanl001
 * #creator     : 2018-10-14 20:06
 * #description : 服务器线程，也就是每次收到一个连接就建立一个新的线程
 **/
public class IM_TCP_Server_Thread extends Thread {

    private Socket socket;
    private Map<String, Socket> connectedUser;

    public IM_TCP_Server_Thread(Socket socket, Map<String, Socket> connectedUser) {
        this.socket = socket;
        this.connectedUser = connectedUser;
    }

    @Override
    public void run() {

        System.out.println("-----------------开始接受消息---------------");

        while (true) {
            //在这里解析客户端发送的消息
            try {

                InputStream inputStream = socket.getInputStream();

                //因为如果不指定协议的话，你是不知道这条数据的长度的，所以就不能进行读取了
                byte[] msgLenBytes = new byte[4];
                inputStream.read(msgLenBytes);
                int msgLen = IMByteArr0NumUtils.byteArr2Int(msgLenBytes);
                System.out.println("消息长度是:" + msgLen);

                byte[] msgBytes = new byte[msgLen];
                inputStream.read(msgBytes);
                String string = new String(msgBytes);

                System.out.println("接收到客户端消息：" + string);


                //在这里找到所有的连接的socket，把这个发送其他的socket
                int currentPort = socket.getPort();
                for (Socket socket : connectedUser.values()) {
                    //这里直接把消息转发给所有的客户端，包括自己
                    IMSocketUtils.sendMsg(string, socket);
                }

            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

* IM_TCP_Client_Thread

```java
package im.ivanl001.bigData.java_01.Java.IM002_TCP_IO;

import java.net.Socket;

/**
 * #author      : ivanl001
 * #creator     : 2018-10-14 20:22
 * #description : tcp连接客户端线程
 **/
public class IM_TCP_Client_Thread extends Thread {

    private Socket socket;

    IM_TCP_Client_Thread(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        super.run();

        new Thread(){
            @Override
            public void run() {
                while (true) {
                    //在这里循环接受消息
                    IMSocketUtils.receiveMsg(socket);

                }
            }
        }.start();


        /*//在这里发送消息
        System.out.print("Enter your msg:");
        Scanner scanner = new Scanner(System.in);
        System.out.println("请输入你的姓名：");
        String name = scanner.nextLine();
        System.out.println("输入的内容是：" + name);
        sendMsg(name);*/

        //这里是发消息
        int i = 0;
        while (true) {
            //sendMsg("ivanl00" + i);
            IMSocketUtils.sendMsg("ivanl00" + i, socket);
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            i += 1;
        }
    }

    /*public void sendMsg(String msg) {

        try {

            //我们是需要指定发送的协议的，数据长度需要包含在发送的数据包中，要不然接收的socket是不知道如何进行解析的
            int msgLen = msg.getBytes().length;
            //把长度转换成四个长度的字节数组，写入到数据中
            byte[] msgLenBytes = IMByteArr0NumUtils.int2byteArr(msgLen);
            byte[] msgBytes = msg.getBytes();
            byte[] msgPack = new byte[4 + msgLen];//消息的长度加上4个字节的长度，这四个字节是用来记录消息的长度信息的
            System.arraycopy(msgLenBytes, 0, msgPack, 0, 4);
            System.arraycopy(msgBytes, 0, msgPack, 4, msgLen);

            socket.getOutputStream().write(msgPack);
            socket.getOutputStream().flush();
            System.out.println("client:" + msg);

        } catch (IOException e) {
            e.printStackTrace();
        }
    }*/
}
```

