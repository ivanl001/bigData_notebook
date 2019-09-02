## 1, socket之tcp实现

### 1.1, tcp的server
```python
#
# author      : ivanl001
# creator     : 2018-12-08 13:20
# description : tcp-server
#

import socket

# 1, 创建socket对象，socket.SOCK_STREAM代表tcp协议
sk = socket.socket(socket.AF_INET,socket.SOCK_STREAM)

# 2, 绑定端口，设置连接数
sk.bind(("127.0.0.1", 8080))
#  backlog等于5，表示内核已经接到了连接请求，但服务器还没有调用accept进行处理的连接个数最大为5
sk.listen(5)


# 3, 启动监听
buffer = 1024
print("-----启动监听------")
while True:
    (client, addr) = sk.accept()
    print("接收到连接，地址是:" + str(addr))
    # 这里要用接受到的那个client接受数据哈
    data = client.recv(buffer)
    # 上面的data是字节类型，我们这里转成字符串
    dataStr = data.decode("utf-8")
    print(dataStr)

```
### 1.2, tcp的client
```python
#
# author      : ivanl001
# creator     : 2018-12-08 13:31
# description : tcp-client
# 

import socket

# 1, 创建socket对象，socket.SOCK_STREAM代表tcp协议
sk = socket.socket(socket.AF_INET,socket.SOCK_STREAM)

# 2, 绑定端口，设置连接数
sk.connect(("127.0.0.1", 8080))


# 3, 启动监听
sk.send("ivanl001 is the king of world! ".encode("utf-8"))
sk.close()
```

## 2, udp实现

### 2.1, udp之receiver
```python
#
# author      : ivanl001
# creator     : 2018-12-08 13:47
# description : udp
# 

import socket

# 1, 创建socket对象，socket.SOCK_DGRAM代表udp协议
receiver = socket.socket(socket.AF_INET,socket.SOCK_DGRAM)

# 2, 绑定端口，设置连接数
receiver.bind(("127.0.0.1", 8080))

# 3, 启动监听
buffer = 1024
print("-----启动监听------")
while True:
    (data, addr) = receiver.recvfrom(buffer)
    print("接收到连接，地址是:" + str(addr))
    dataStr = bytes.decode(data, "utf-8")
    print("接收到udp数据:" + dataStr)
```
### 2.2, udp之sender
```python
#
# author      : ivanl001
# creator     : 2018-12-08 13:51
# description : 
# 

import socket

# 1, 创建socket对象，socket.SOCK_DGRAM代表udp协议
sender = socket.socket(socket.AF_INET,socket.SOCK_DGRAM)

# 2, 绑定端口，设置连接数
sender.bind(("127.0.0.1", 8081))
print("启动发送者-----------------------")

# 3, 启动监听
buffer = 1024
print("发送数据-----------------------")
sender.sendto("ivanl001 is the king of world!".encode("utf-8"), ("127.0.0.1", 8080))
print("发送数据完成-----------------------")
```