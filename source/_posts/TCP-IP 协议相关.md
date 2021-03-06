---
title: TCP-IP 协议相关
date: 2016-04-20 13:08:24
category: 网络
tags: [TCP/IP,网络]
---


之前一直没太注意“协议栈”这个词，以为没什么特殊含义，现在了解到其描述的是协议中数据传输的过程，发送端从上层协议到底层协议，接收端从底层协议到上层协议，虽说并不具有“后入先出”的特点，但用“栈”来描述的确很形象。 
对于TCP/IP，之前也学了不少，但很多东西久了也难免会忘，要用的时候再来搜索总不是一个好的选择，在这里做个总结，可以不时的复习一下。


----------

# 协议栈概述

TCP/IP协议不是TCP和IP这两个协议的合称，而是指因特网整个TCP/IP协议族。TCP/IP协议栈依其功能不同，被分别归属到这四个层次结构之中，一般可以认为与七层OSI模型有如下对应关系（这中间有不严格的对应关系，下面会讲）：

![OSI与TCP/IP 分层对应关系](http://img.blog.csdn.net/20160418165128245)

要知道每一层里对应了哪些数据，可以从底层开始分析，

## 网络接口层
严格的讲，网络接口层并不是TCP/IP的一部分，因为IP协议是按照将它下面的网络当作一个黑盒子的思想设计的，只关心如何使得数据能够跨越本地网络边界的问题，而不关心如何利用传输媒体。网络接口层对应了OSI参考模型的物理层和数据链路层。

 - 物理层 
物理层规定通信设备的机械的、电气的、功能的和过程的特性，以及如何利用信号进行传输，在这一层，数据的单位称为比特（bit）。

 - 数据链路层 
数据链路层可进一步细分为较低的介质访问控制(MAC)子层和较高的逻辑链路控制(LLC)子层。MAC子层的存在屏蔽了不同物理链路种类的差异性，各种传输介质（铜缆、光线，无线等）的物理层对应到相应的MAC层，物理寻址在此处被定义。LLC子层负责向其上层提供服务，进行数据包的分段与重组等工作。一般的，MAC层由网卡控制，操作系统驱动来控制LLC。

## 网络互联层

这一层定义了最为重要的IP协议，IP协议实现了通过IP地址来进行数据传输，但只有这个是不行的，还需要一些其它协议为数据传输提供必要的基础服务，主要有以下几个协议： 

- ARP (Address Resolution Protocol) 
在以太网协议中规定，同一局域网中的一台主机要和另一台主机进行直接通信，必须要知道目标主机的MAC地址。而在TCP/IP协议中，网络层和传输层只关心目标主机的IP地址。于是需要一种方法，根据目的主机的IP地址，获得其MAC地址。这就是ARP协议要做的事情：主机在发送帧前将目标IP地址转换成目标MAC地址的过程。

- RARP (Reverse Address Resolution)
RARP使用与ARP相同的报头结构，作用与ARP相反。RARP用于将MAC地址转换为IP地址。其因为较限于IP地址的运用以及其他的一些缺点，因此渐为更新的BOOTP或DHCP所替换

- ICMP (Internet Control Message Protocol) 
IP 协议是不可靠协议，不能保证 IP 数据报能够成功的到达目的主机，无法进行差错控制，而 ICMP 协议能够协助 IP 协议完成这些功能。ICMP报文包含在IP数据报中，IP头部的Protocol值为1就说明这是一个ICMP报文，ICMP头部中的类型（Type）域用于说明ICMP报文的作用及格式，此外还有一个代码（Code）域用于详细说明某种ICMP报文的类型，报文大致可分为两类：差错报文（如目标不可到达，超时）、查询报文（如回送消息，地址掩码消息，时间戳消息）。

其中IP、ARP、RARP封装成以太网帧时，会填充以太帧头部的的“帧类型”字段，像这样的协议还有PPPoE、Loopback等。而ICMP、IGMP、TCP、UDP、IGRP、OSPF这些是基于IP协议的，发送时会填充IP数据报的Protocol字段，而TCP和UDP一般认为属于传输层协议，所以可以看出与OSI模型并不是严格对应的。

## 传输层
这一层实现我们最熟悉的TCP和UDP协议，功能主要是提供应用程序间的通信。端口（Port）也是在传输层定义的。

## 应用层
传输层协议实现了数据传输，但如果只使用这些来开发某些复杂的应用显然是不够的，于是就有应用层协议，如HTTP、SSL、Telnet等。


----------


# IP地址
目前使用的IPv4的IP地址使用一个32位的地址，这个地址由子网掩码划分为网络地址和主机地址，子网掩码一般根据主机的数目来计算，这种区分在IP网络里的路由中使用。在早期使用ABCDE来对IP进行分类，如下表这样：
| 分类 | 前缀码 | 开始地址 | 结束地址 | 对应CIDR修饰 |	默认子网掩码
| - |
| A类地址	| 0 |	0.0.0.0  |	127.255.255.255|/8|	255.0.0.0|
|B类地址|	10  |	128.0.0.0|	191.255.255.255|/16|	255.255.0.0|
|C类地址|	110	|   192.0.0.0 |	223.255.255.255|/24|	255.255.255.0|
|D类地址 （群播）|	1110|	224.0.0.0|	239.255.255.255|	/4|	未定义|
|E类地址 （保留）|	1111	|240.0.0.0	|255.255.255.255|	/4	|未定义|

可以看出，这种分类网络的主机数目都可以从地址的最高位得出，因为分类路由协议不指定子网的掩码或前缀长度，路由器必须使用路由通告中的地址类别去得出子网掩码以创建路由表。随着网络的发展，分类网络显得可扩充性不足，于是就有CIDR，目的就是使网络号的位数不仅限于8位、16位、24位，这样可以根据主机数的需求，分配更合适的网络号，能够充分的利用IP地址资源。

需要注意的是，子网的第一个和最后一个地址分别被作为网络识别码和广播地址，任何其它地址都可以被分配给其上的主机。比如192.168.0.0/16中，广播地址是192.168.255.255，在这种情况下，尽管可能带来误解，但192.168.1.255、192.168.2.255等地址可以被分配给主机。同样的，广播地址不一定总是以255结尾。比如，子网203.0.113.16/28的广播地址是203.0.113.31。

私有IP无法直接连接互联网，需要公网IP转发，私有IP地址也不会在Internet中被分配。包括以下几段：

- 10.0.0.0       – 10.255.255.255
- 172.16.0.0   – 172.31.255.255
- 192.168.0.0 – 192.168.255.255


----------


# TCP连接
TCP是面向连接的可靠传输协议，两个进程互发数据之前需要建立连接，这里的连接只是终端中维护一个缓存和状态变量。
## 建立连接（三次握手）
首先，客户端发送一个特殊的TCP报文段(SYN)；
其次，服务器收到请求后，发送一个SYN + ACK，其中ACK是对客户端SYN的回应；
最后，客户端对服务器发来的SYN进行回应，即发送ACK。
![连接过程](http://img.blog.csdn.net/20160419150649825)

## 断开连接时（四次挥手）
首先，当客户端不再需要发送数据时，发送FIN来断开连接。
然后，服务端回复确认，此时，连接处于**半关闭状态**，但此时服务端还可以继续向客户端发送数据，发送完成后再发送FIN来关闭连接，这也是为什么不像握手时那样将FIN和ACK直接回复的原因。
最后，客户端回复确认，连接完全断开。
![断开连接过程](http://img.blog.csdn.net/20160419150713451)

## MTU 与 MSS
**MTU**（Maximum Transmission Unit，最大传输单元），表示数据链路层一次能传输数据的最大长度。常见的值如下表：（windows下可以用ping -f -l 1465 192.168.1.1测试）
| 网络 | MTU（字节）|
|---|
|Ethernet/IEEE 802.11  | 1500|
|IEEE 802.3/802.2      | 1492|
|PPPoE/ADSL            | 1492|
|Dial Up/Modem         | 576 |

> 注：unix网络编程第一卷里说 ipv4协议规定ip层的最小重组缓冲区大小为576，上表的576和这个应该有关系

**MSS**（Maximum Segment Size，最大报文长度），MSS就是一个TCP数据包每次能够传输的最大数据长度。为了达到最佳的传输效能TCP协议在建立连 接的时候通常要协商双方的MSS值，根据双方提供的MSS值得最小值确定为这次连接的最大MSS值。这个值一般为MTU减去IP数据报头的大小20Bytes和TCP数据段的包头 20Bytes，所以往往MSS为(MTU - 40)。另外，UDP数据报的首部为8Bytes，所以UDP数据报的数据区最大长度为(MTU - 28)。


----------


# socket
socket是在应用层和传输层或网络层之间的一个抽象层，它把TCP/IP层复杂的操作抽象为几个简单的接口供应用层调用。socket起源于UNIX，在Unix一切皆文件哲学的思想下，socket是一种"打开—读/写—关闭"模式的实现，服务器和客户端各自维护一个"文件"，在建立连接打开后，可以向自己文件写入内容供对方读取或者读取对方内容，通讯结束时关闭文件。
创建socket的方法：
```
int socket(int domain, int type, int protocol);
```
**domain** 为创建的套接字指定协议集。 例如：

- AF_INET 表示IPv4网络协议
- AF_INET6 表示IPv6
- AF_UNIX 表示本地套接字（使用一个文件）

**type** 如下：

- SOCK_STREAM （可靠的面向流服务或流套接字，TCP）
- SOCK_DGRAM （数据报文服务或者数据报文套接字，UDP）
- SOCK_RAW (在网络层之上的原始协议)
- SOCK_PACKET  （在网络接口层上的原始协议，可以写ARP请求）
- SOCK_SEQPACKET （可靠的连续数据包服务，不属于IP协议族）

**protocol** 指定实际使用的传输协议。 最常见的就是IPPROTO_TCP、IPPROTO_UDP、IPPROTO\_DCCP、IPPROTO\_SCTP。

> 注意：
> 
1. type和protocol不可以随意组合，如SOCK_STREAM不可以跟IPPROTO_UDP组合。当第三个参数为0   		   时，会自动选择第二个参数类型对应的默认协议。
2. WindowsSocket下protocol参数中不存在IPPROTO_STCP


# 参考链接

[几种报文格式及arp分类](http://wenku.baidu.com/link?url=YQ30FITF453NIAXXIDp37XGh_lgr_ZbSQh77tDZqdGunyMv5jT31G8iovrgdYbrcu7Lx3gFPgtDYudsolh_s6z51GD07xi5FSjO1YPxhJp7)

[维基百科-ARP](https://zh.wikipedia.org/wiki/%E5%9C%B0%E5%9D%80%E8%A7%A3%E6%9E%90%E5%8D%8F%E8%AE%AE)

[使用wireshark分析TCP/IP协议中TCP包头的格式](http://www.codeceo.com/article/wireshark-tcp-header.html)

[TCP Message (Segment) Format ](http://www.tcpipguide.com/free/t_TCPMessageSegmentFormat-3.htm)

[维基百科-socket](https://zh.wikipedia.org/wiki/Berkeley%E5%A5%97%E6%8E%A5%E5%AD%97)