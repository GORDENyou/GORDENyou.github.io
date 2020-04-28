---
title: "TCP和UDP"
author: "Gordenyou"
date: 2018-02-20T15:09:01+08:00
categories: ["技术之美"]
tags: ["网络编程"]
lastmod: 
draft: true
summary: "老生常谈。不和你多BB，开撸"
---

Android开发中涉及到的网络层协议不多，像TCP、UDP、HTTPS是接触的最多也是最主要的！

## User Datagram Protocol 

其实记住了这个名字你就已经了解了大半。

**用户数据报协议，位于传输层，非连接协议。**



#### 为什么不可靠？

1. 很懒，向网络层发送数据后不进行备份。
2. UDP在IP数据报的头部仅仅加入了复用和数据校验。
3. 发送端生产数据，同时需要接收端抓取数据，如果时间不一致，可能导致通信失败。（没有标准的客户端和服务端）



#### 有什么特点？

结构简单，无校验，速度快，容易丢包，可广播



#### UDP能做什么？

1. DNS、TFTP、 SNMP这些网络协议就是使用UDP通信
2. 传递视频、音频、普通视频（无关紧要的数据）



#### UDP数据报中数据所占用的最大长度？

![](../index.assets/udp_header.png)

从图中我们可以得知，UDP数据报的长度最长为 2^16 = 65536 ， 其中要减去1位不存放信息，而且还需要减去UDP的头部信息，所以最终的最长数据长度为 65536 - 1 - 8 =  65527，但是由于IP首部也要占用20个字节，最终的结果为 65507字节。



#### 核心API（Java）

`DatagramSocket.java`   //用于接收和发送UDP的类

```java
DatagramSocket() //创建简单实例，不指定端口和IP

DatagramSocket(int port) //创建监听固定端口实例

DatagramSocket(int port, InetAddress localAddr) //创建固定指定IP的实例（自己的电脑可能有多张网卡）
    
receive(DatagramPacket d) //接收
    
send(DatagramPacket d) //发送
    
setSoTimeout(int timeout) //设置超时，毫秒
    
close() //释放资源
```



`DatagramPacket.java`  // 将`byte`数组、目标地址、目标端口等数据包装成报文或者将报文拆解成`byte`数组

```
DatagramPacket(byte[] buf, int offset, int length, InetAddress address, int port) //前三个参数指定buf使用的区间，后两个参数指定目标机器地址和端口

DatagramPacket(byte[] buf, int offset, int length, SocketAddress address) //SocketAddress用来指定Socket端口

setData()....其它的看API吧。
```

这里需要注意，我们发送数据的时候需要地址和端口，接收数据的时候只需要监听一个接收端口。（废话好多。）



#### UDP的单播、多播（组播）、广播

上图：

![](../index.assets/udp_broadcast.png)

这里需要注意： 多播是路由器实现的。不需要我们重复的发送信息。

说道这里，我们不得不提我们的`IP`地址的类别，不多说，上图：

![](../index.assets/ip_address.png)

可以看出，我们家庭用的`IP`地址一般是C类地址，D类地址为多播所预留。



#### 如何得到广播地址？

- 255.255.255.255 为受限广播地址，只有我们自己的局域网内所连接的设备能收到广播，而且需要设备进行监听。

- 广播地址是通过子网掩码运算出来的，那么它末尾一定是255吗？不一定！来看两个例子

  ![](../index.assets/udp_example1.png)

  ![](../index.assets/udp_example2.png)

  广播地址是所在网段中的最后一个地址。