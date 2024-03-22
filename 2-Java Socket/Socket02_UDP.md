# Java UDP通信

* 特点：无连接、不可靠通信
* Java提供了一个”java.net.DatagramSocket类“来实现UDP通信

## 一、DatagramSocket类

* 用于创建客户端、服务器

| 构造器                          | 说明                                                 |
| ------------------------------- | ---------------------------------------------------- |
| public DatagramSocket()         | 创建【客户端】的Socket对象，系统会随机分配一个端口号 |
| public DatagramSocket(int port) | 创建服务端的socket对象，并指定端口号                 |

| 方法                                  | 说明               |
| ------------------------------------- | ------------------ |
| public void send(DatagramPacket dp)   | 发送数据包         |
| public void receive(DatagramPacket p) | 使用数据包接收数据 |

## 二、DatagramPacket类

* 用于创建数据包

| 构造器                                                                       | 说明                       |
| ---------------------------------------------------------------------------- | -------------------------- |
| public DatagramPacket(byte[] buf, int length, InetAddress address, int port) | 创建发出去的数据包对象     |
| public DatagramPacket(byte[] buf, int length)                                | 创建用来接收数据包的数据包 |

| 方法                   | 说明                             |
| ---------------------- | -------------------------------- |
| public int getLenth() | 获取数据包，实际接收到的字节个数 |
