> Socket = IP + TCP/UDP + port
> socket就是套接字，套接字到作用就是建立通信

## 1，OSI七层网络模型(Open System Interconnect)

1. 物理层
2. 链路层
3. 传输层
    * TCP协议，面向连接到协议
    * UDP协议，无连接协议
    * 
4. 网络层
    * ip协议
5. 会话层
6. 表示层
7. 应用层
    * http，https协议
    * ftp：file transfer prot

![IMAGE](assets/1AFC5EC106629F0593C0830F45088C71.jpg)
![IMAGE](assets/A31BE366412D8CD6CE23C7E9B8A4BD7E.jpg)

## 2，UDP案例：user datagram protocal

*DatagramSocket: udp套接字*

* 不保证消息会被收到
* 没有固定路由，先发送的消息可能会后收到
* 所以也就没有服务器的概念，只有发送方和接收方的概念
* 发送的包叫做DatagramPack：内部需要指定接收方的ip+port

## 3, TCP协议三次握手机制
1. A发送sync信号(序号是x)给B，B接受这个同步信号
2. B接受后再给A发送sync信号(序号是y,确认号x+1), A接受这个同步信号
3. A再给B发送ACK信号(序号是y+1)，B接受到之后正式建立连接，开始通信

## 4，UDP的屏幕广播案例

### 4.1, Capturer
`屏幕录屏的服务端,通过robot录屏，然后根据协议是：8位时间戳 + 1位分片数 + 1位编号 + 4位的数据长度 + 数据长度的数据进行转换成字节流通过UDP发送出去`

```java
package im.ivanl001.bigData.java_01.Java.IM001_ScreenBroadcast;

import javax.swing.*;

/**
 * #author      : ivanl001
 * #creator     : 2018-10-14 10:53
 * #description :
 **/
public class App extends JFrame{

    private JLabel lblIcon ;

    public App(){
        init();
    }

    private void init() {
        this.setTitle("屏幕分享");
        this.setBounds(0, 0, 1366, 768);
        this.setLayout(null);

        lblIcon = new JLabel();
        lblIcon.setBounds(0, 0, 1366, 768);

        ImageIcon icon = new ImageIcon("D:/Koala.jpg");
        lblIcon.setIcon(icon);
        this.add(lblIcon);
        this.setVisible(true);
    }

    public void updateIcon(byte[] dataBytes){
        ImageIcon icon = new ImageIcon(dataBytes);
        lblIcon.setIcon(icon);
    }
}
```

### 4.2, Receiver

`真正的客户端程序，通过调用UI类，并从接受类获取数据`

```java
package im.ivanl001.bigData.java_01.Java.IM001_ScreenBroadcast;

/**
 * #author      : ivanl001
 * #creator     : 2018-10-13 20:46
 * #description : 接收方
 **/
public class Receiver {

    public static void main(String[] args) {

        App app = new App();

        IMReceive imReceive = new IMReceive(app);
        imReceive.start();
    }
}
```

### 4.2，客户端的ui界面
`用来显示界面的`

```java
package im.ivanl001.bigData.java_01.Java.IM001_ScreenBroadcast;

import im.ivanl001.bigData.java_01.IMUtils.IMByteArr0NumUtils;
import im.ivanl001.bigData.java_01.IMUtils.IMCaptureUtils;
import im.ivanl001.bigData.java_01.IMUtils.IMUDPUtils;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.util.zip.ZipEntry;
import java.util.zip.ZipOutputStream;

/**
 * #author      : ivanl001
 * #creator     : 2018-10-13 19:05
 * #description : 截屏方
 **/
public class Capturer {

    public static void main(String[] args) {

        int i = 0;
        //循环的截图并发送
        while (true) {
            i += 1;
            captureOneAndSend();
            System.out.println("--------------------------发送图片：" + i);
            //break;//循环暂且关掉
        }
    }

    public static void captureOneAndSend() {

        int maxSize = 60 * 1024;

        //首先截屏，然后转换成字节数组，这个过程应该一直操作,开始的时候先用静态
        byte[] picBytes = IMCaptureUtils.captureAndGetByteArr();
        //记录时间戳，作为本帧图片的标记
        Long currentTimeStamp = System.currentTimeMillis();

        //然后处理图片，如果图片大于60K，把图片进行切分处理
        int dataLen = picBytes.length;
        int count = 0;
        int lastSliceLen = 0;//最后一个切片的长度

        // 如果能整除，ok，那刚刚好
        if (dataLen % (maxSize) == 0) {
            count = dataLen/(maxSize);
            lastSliceLen = maxSize;
        }else{
            count = (dataLen/(maxSize))+1;
            lastSliceLen = dataLen % (maxSize);
        }

        //这个是每个切片的长度
//        int sliceLen =


        for (int i=0;i<count;i++) {

            //在这里把数组组装起来，发送
            byte[] sliceData = null;
            int sliceLen = 0;
            if (i == count - 1) {
                sliceData = new byte[lastSliceLen];
                sliceLen = lastSliceLen;
            }else {
                sliceData = new byte[maxSize];
                sliceLen = maxSize;
            }

            //计算中长度：协议是：8位时间戳 + 1位分片数 + 1位编号 + 4位的数据长度 + 数据长度的数据
            int totalLen = 8 + 1 + 1 + 4 + sliceLen;

            //初始化字节数组，然后把所有需要的信息都写入内，方便后续进行udp发送
            byte[] frameUnitBytes = new byte[totalLen];

            //1，处理时间戳，这个时间戳是截图的时间戳，这样子的话每个frameunit的时间戳才会相同
            byte[] timeBytes = IMByteArr0NumUtils.long2byteArr(currentTimeStamp);
            System.arraycopy(timeBytes, 0, frameUnitBytes, 0, 8);

            //2,总分片数,因为切片个数不大可能大于一个字节8位，所以直接转成byte类型即可
            byte countBytes = (byte)count;
            //System.arraycopy(countBytes, 0, frameUnitBytes, 8, 1);
            //直接赋值吗
            frameUnitBytes[8] = countBytes;

            //3,序号
            byte no = (byte)i;
            //System.arraycopy(no, 0, frameUnitBytes, 9, 1);
            frameUnitBytes[9] = no;

            //4,这个是切片的长度，不是整个图片的长度哈
            byte[] length = IMByteArr0NumUtils.int2byteArr(sliceLen);
            System.arraycopy(length, 0, frameUnitBytes, 10, 4);

            //5, 数据
            //picBytes;;

            System.arraycopy(picBytes, i*60*1024, sliceData, 0, sliceLen);

            System.out.println(picBytes.length + "-------");

            //System.arraycopy(picBytes, 0, frameUnitBytes, 14, dataLen);

            System.arraycopy(picBytes, i*maxSize, frameUnitBytes, 14, sliceLen);

            System.out.println(frameUnitBytes.length);

            // 发送之前把字节数组压缩以下
            ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
            ZipOutputStream zipOutputStream = new ZipOutputStream(byteArrayOutputStream);
            try {
                zipOutputStream.putNextEntry(new ZipEntry("frameUnitZip"));
                zipOutputStream.write(frameUnitBytes);

            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                try {
                    zipOutputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                } finally {
                    try {
                        byteArrayOutputStream.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
            byteArrayOutputStream.toByteArray();

            //发送
            IMUDPUtils.UdpSendBytes(byteArrayOutputStream.toByteArray());

            System.out.println("发送切片：" + i);

        }
    }
}

```

### 4.4， IMReceive
`真正用来接受数据的类，通过发送的数据的协议进行反解出来`

```java
package im.ivanl001.bigData.java_01.Java.IM001_ScreenBroadcast;

import im.ivanl001.bigData.java_01.IMUtils.IMByteArr0NumUtils;
import im.ivanl001.bigData.java_01.Model.FrameUnit;

import javax.imageio.ImageIO;
import java.awt.image.BufferedImage;
import java.io.*;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.SocketException;
import java.util.HashMap;
import java.util.Map;
import java.util.zip.ZipEntry;
import java.util.zip.ZipInputStream;

/**
 * #author      : ivanl001
 * #creator     : 2018-10-13 21:00
 * #description : 用来接受udp消息的一个类，继承Thread，重开线程
 **/
public class IMReceive extends Thread {

    private static int SB_MAX_SIZE = 60 * 1024;
    private DatagramSocket datagramSocket = null;
    private DatagramPacket datagramPacket = null;

    private byte[] buffer = new byte[SB_MAX_SIZE + 14];

    private App app;

    //这个有点不太好用，该用map，通过标号做key，到时候可以直接通过编号进行排序，因为标号是连续的
//    private List<FrameUnit> frame = new ArrayList<>();
    Map<Integer, FrameUnit> frame = new HashMap<>();


    IMReceive(App app){
        try {
            datagramSocket = new DatagramSocket(8888);
            this.app = app;
        } catch (SocketException e) {
            e.printStackTrace();
        }
        datagramPacket = new DatagramPacket(buffer, buffer.length);
    }

    @Override
    public void run() {
        try {

            long lastTimeStamp = 0;

            //一直循环的进行收取
            while (true) {

                datagramSocket.receive(datagramPacket);
                int sliceLen = datagramPacket.getLength();//注意：这个长度不一定等于SB_MAX_SIZE, 因为最后一片有可能小于这个值


                byte[] buffer01 = new byte[1024];

                //在这里解析这个值，然后显示即可
                //buffer，这个就是接受到的数据了
                //注意：这里需要先进行解压缩buffer
                ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(buffer);
                ZipInputStream zipInputStream = new ZipInputStream(byteArrayInputStream);

                ByteArrayOutputStream byteArrayOutputStream01 = new ByteArrayOutputStream();
                ZipEntry zipEntry = null;
                int len = 0;
                while ((zipEntry = zipInputStream.getNextEntry()) != null) {
                    while ((len = zipInputStream.read(buffer01)) != -1) {
                        byteArrayOutputStream01.write(buffer01, 0, len);
                    }
                }

                byteArrayOutputStream01.close();
                zipInputStream.close();
                byteArrayInputStream.close();

                byte[] theFrameUnitData = byteArrayOutputStream01.toByteArray();

                FrameUnit frameUnit = convertBytesToFrameUnit(theFrameUnitData, theFrameUnitData.length);

                //已经得到了frameUnit了， 判断frameUnit，如果属于同一帧，按照序号进行拼接，如果同一帧没有集齐，下一帧到了，那么直接舍弃没有集齐的那一帧
                long currentTimestamp = frameUnit.getFrameId();

                if (lastTimeStamp == currentTimestamp) {
                    //时间戳相同，进行拼合，并更新lastTimeStamp
                    frame.put(frameUnit.getNo(), frameUnit);
                    lastTimeStamp = currentTimestamp;

                } else {
                    //时间戳不同，先清空图片数组，然后把当前的数据放入图片切片数组，并更新lastTimeStamp
                    frame.clear();
                    frame.put(frameUnit.getNo(), frameUnit);
                    lastTimeStamp = currentTimestamp;
                }

                if (frameUnit.getCount() == frame.size()) {

                    ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();

                    //说明已经集齐，那么把图片拼接起来就ok了
                    for (int i=0;i<frame.size();i++) {
                        FrameUnit frameUnit1 = frame.get(i);
                        int sliceLen01 = frameUnit1.getDataLen();
                        byte[] bytes = frameUnit1.getData();
                        byteArrayOutputStream.write(bytes);
                    }

                    //在这里说明所有本帧的切片信息都已经被写入到byteArrayOutputStream了
                    BufferedImage bufferedImage = ImageIO.read(new ByteArrayInputStream(byteArrayOutputStream.toByteArray()));

                    ImageIO.write(bufferedImage, "jpg", new FileOutputStream("/Users/ivanl001/Desktop/to/capture/" + frameUnit.getFrameId()+ ".jpg"));

                    app.updateIcon(byteArrayOutputStream.toByteArray());

                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }


    }

    private FrameUnit convertBytesToFrameUnit(byte[] buffer, int sliceLen) {

        //首先取出有效数据，因为buffer中很有可能有几个字节内容是无用内容
        byte[] data = new byte[sliceLen];
        System.arraycopy(buffer, 0, data, 0, sliceLen);
        //这个时候data就是正确的需要的数据了,解析data到FrameUnit中即可


        FrameUnit frameUnit = new FrameUnit();
        //1，首先解析时间戳
        frameUnit.setFrameId(IMByteArr0NumUtils.byteArr2long(data));//这里刚好是前八个是时间戳，所以不需要任何处理，把整个data放进去也可以处理

        //2,解析切片个数，这里直接强转是没问题的，小转大不会丢失精度
        frameUnit.setCount(data[8]);//这里会自动从字节转成int类型，从小往大转不会丢精度

        //3,解析切片序号
        frameUnit.setNo(data[9]);

        //4,解析数据长度，4个字节数组转int
        int theDataLen = IMByteArr0NumUtils.byteArr2Int(data, 10);
        System.out.println("数据的长度是：" + theDataLen);
        frameUnit.setDataLen(theDataLen);

        //5,解析data
        byte[] theData = new byte[theDataLen];

        System.arraycopy(buffer, 14, theData, 0, theDataLen);
        frameUnit.setData(theData);

        return frameUnit;
    }
}
```

## 5，TCP的一个小程序

### 5.1，ServerApp服务器端01

```java
package im.ivanl001.bigData.java_01.Java.IM002_TCP;


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

### 5.2, IM_TCP_Server_Thread,服务器端02
`01中的主程序每次接收到客户端的连接请求，都会通过这个类进行创建新的线程，然后进行发送数据或者接受数据`

```java
package im.ivanl001.bigData.java_01.Java.IM002_TCP;

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

### 5.3，ClientApp客户端01

```java
package im.ivanl001.bigData.java_01.Java.IM002_TCP;

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

### 5.4, 客户端02
`这个也是每次接受到服务器的不同socket创建线程用的`

```java
package im.ivanl001.bigData.java_01.Java.IM002_TCP;

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
                Thread.sleep(1000);
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

### 5.5, 封装的IMSocketUtils工具类，上面可能会需要用的

```java
package im.ivanl001.bigData.java_01.Java.IM002_TCP;

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