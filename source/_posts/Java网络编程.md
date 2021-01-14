---
title: Java网络编程
date: 2021-01-14 09:11:58
tags:
---

## 什么是网络编程

packet包和邮件的思想类似

TCP连接

UDP连接

+ 计算机网络：计算机网络是指将地理位置不同的具有独立功能的多台计算机及其外部设备，通过通信线路连接起来，在网络操作系统，网络管理软件及网络通信协议的管理和协调下，实现资源共享和信息传递的计算机系统。（不同地理位置计算机，通过线路连接，在通信软件和协议的协调下，实现资源共享）

+ 网络编程的目的：传播交流信息、数据交换、通信
+ 需要解决的问题：
    + 如何准确的定位到网络上的一台主机，端口，定位到计算机上的某个资源
    + 找到这个主机，如何传输资源。

JavaWeb：网页编程 B/S

网络编程：TCP/IP

## 网络通信的要素

通信双方地址

+ ip
+ 端口号

规则：网络通信协议

+ TCP/IP四层模型
+ OSI七层网络模型

![img](Java网络编程/v2-1578921092d775e024345fa8a531a85e_1440w.jpg)

## IP

ip地址：InetAdress

+ 唯一定位一台网络上的计算机
+ 127.0.0.1：本机localhost，本机回环地址
+ IP地址：
  + IPV4/IPV6
    + IPV4：127.0.0.1 4个字节组成 0-255，42亿个，30亿在北美，亚洲4亿，2011年用尽。
    + IPV6：fe80::489:aab1:a507:2b83%en0，128位，8个无符号整数。
  + 公网（互联网）-私网（局域网）
    + 129.168.xx.xx专门给组织内部使用的
    + ABCDE类地址计算：对半分，可以计算出来，阿里的笔试题中有。
+ 域名：记忆问题引入域名。

## 端口

端口：表示计算机上的每一个程序的进程

+ 不同的进程有不同的端口号。用来区分软件。
+ 端口被规定0-65535
+ TCP端口、UDP端口。65535*2，tcp和udp的端口号不冲突。
+ 端口分类
  + 公有端口0~1023
    + HTTP：80
    + HTTPS：443
    + FTP：21
    + Telent：23
  + 程序注册端口：1024~49151，分配给用户或者程序
    + Tomcat：8080
    + MySql：3306
    + Oracle：1521
  + 动态、私有端口：49512~65535
    + `lsof`  查看端口
    + `leof -i:8080 `查看指定端口被占用情况
    + `kill pid`杀死对应端口进程

## 通信协议

协议：TCP/IP协议

网络通信协议：速率、传输码率、代码结构、传输控制等

TCP/IP协议簇：实际上是一组协议

+ TCP：用户传输协议
+ UDP：用户数据报协议
+ IP：网络互连协议

TCP和UDP对比：

TCP：

+ 连接、文档
+ 三次握手 四次挥手
+ 客户端、服务端
+ 传输完成，释放连接、效率低

UDP：

+ 不连接、不稳定
+ 客户端、服务端，没有明确界限
+ 不管有没有准备好，都可以发送
+ DDoS攻击：网络阻塞，导致端口不可用

## TCP

客户端Client

+ 连接服务器Socket
+ 发送消息

服务器Server

+ 建立服务的端口ServerSocket
+ 等待用户的连接，accept
+ 接收用户的消息

```
服务端：
public class Server {

    public static void main(String[] args) {
        ServerSocket serverSocket = null;
        Socket socket = null;
        InputStream inputStream = null;
        ByteOutputStream byteOutputStream = null;
        try {
            serverSocket = new ServerSocket(12340);
            while (true) {
                socket = serverSocket.accept();
                inputStream = socket.getInputStream();
                byteOutputStream = new ByteOutputStream();
                byte[] buffer = new byte[1024];
                int length = 0;
                while ((length = inputStream.read(buffer)) != -1) {
                    byteOutputStream.write(buffer, 0, length);
                }
                System.out.println(byteOutputStream.toString());
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        } finally {
            if (byteOutputStream != null) {
                byteOutputStream.close();
            }
            if (inputStream != null) {
                try {
                    inputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (socket != null) {
                try {
                    socket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (serverSocket != null) {
                try {
                    serverSocket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

```
客户端：
public class Client {

    public static void main(String[] args) throws IOException {
        InetAddress localHost = InetAddress.getLocalHost();
        Socket socket = new Socket(localHost, 12340);
        OutputStream outputStream = socket.getOutputStream();
        outputStream.write("hello".getBytes());
        outputStream.close();
        socket.close();
    }

}
```

