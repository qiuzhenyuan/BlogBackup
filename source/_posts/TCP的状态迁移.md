---
title: TCP的状态迁移
date: 2018-01-28 20:29:30
categories: "网络协议" 
tags: [TCP,网络协议]
---

## 状态迁移
### TCP连接的11种状态

一条TCP连接在其生存期内会经历一系列状态。TCP连接包含11个状态。分别是：LISTEN、SYN_SENT、SYN_RECEIVED、ESTABLISHED、FIN_WAIT_1、FIN_WAIT_2、CLOSE_WAIT、CLOSING、LAST_ACK、TIME_WAIT以及最终状态CLOSED。状态转换图如下所示：  

<!-- more -->  

![TCP状态迁移图](https://wx1.sinaimg.cn/mw690/857afa84gy1fmif7bahqij20vv0jq0uf.jpg)  



简单分析下状态含义以及转换的过程：  

**CLOSED** ：是一个虚拟的状态，它代表一条连接结束时的状态。  
**LISTEN** ：表示服务端正在等待来自TCP客户端的连接请求。服务端为此要调用bind()以及listen()函数。    
**SYN_SENT** : 客户端发起连接，发送SYN给服务端后，进入此状态。    
**SYN_RECEIVED** : 服务端接收到客户端的SYN请求，服务端由LISTEN状态进入SYN_RECEIVED状态，同时发送一个ACK和一个SYN给客户端。**另一种情况** 是客户端发送SYN的同时接收到服务器的SYN请求，客户端就会由SYN_SENT状态进入SYN_RECEIVED状态。    
**ESTABLISHED** : 表示经过三次握手，连接已经建立，可以在两个方向上传输数据。这是一条连接在数据传输阶段的常见状态。  
**FIN_WAIT_1** : 主动关闭的一方发送FIN，由ESTABLISHED状态进入FIN_WAIT_1状态。  
**FIN_WAIT_2** : 主动关闭的一方，接收到对方的FIN ACK,由FIN_WAIT_1 状态，进入FIN_WAIT_2 状态。  
**CLOSE_WAIT** : 被动关闭的一方，在接收到FIN后，由ESTABLISHED状态进入此状态。  
**CLOSING** :  这种状态表示此时双方刚好可能都在关闭连接，即客户端向服务器发送FIN报文，进入FIN_WAIT_1状态后，没有收到服务器发来的ACK报文，反而受到服务器发来的FIN报文，说明此时客户端和服务器同时发起关闭连接，随后，客户端进入CLOSING状态。  
**LAST_ACK** :  被动关闭的一方，发起关闭请求，由CLOSE_WAIT状态，进入此状态。在接收到ACK后，会进入CLOSED状态。
**TIME_WAIT** :  有3个状态可以转化为TIME_WAIT状态。  

1. 由FIN_WAIT_2进入此状态：在双方不同时发起FIN的情况下，主动关闭的一方在完成自身发起的关闭请求后，接收到被动关闭一方的FIN后进入的状态。   
2. 由CLOSING状态进入:双方同时发起关闭，都做了发起FIN的请求，同时接收到了FIN并做了ACK的情况下，由CLOSING状态进入。
3. 由FIN_WAIT_1状态进入：同时接受到FIN（对方发起），ACK（本身发起的FIN回应），与2的区别在于本身发起的FIN回应的ACK先于对方的FIN请求到达，而2是FIN先到达。这种情况概率最小。

**CLOSED** :  在超时或者连接关闭时候进入此状态。

### 关闭连接的三种方式

在结束一条连接的时候任一方都需要独立地关闭属于自己的一般连接。因此，TCP的一端可能通过下列三种状态迁移方式，将连接从ESTABLISHED状态，转换到CLOSED状态。  
一端先关闭：  
ESTABLISHED -> FIN_WAIT_1 -> FIN_WAIT_2 -> TIME_WAIT -> CLOSED   
另一端先关闭：  
ESTABLISHED -> CLOSED_WAIT -> LAST_ACK -> CLOSED   
两端同时关闭：  
ESTABLISHED -> FIN_WAIT_1 -> CLOSING -> TIME_WAIT -> CLOSED   

## TIME_WAIT状态分析
为什么关闭连接后，不直接进入CLOSED状态，而是先进入TIME_WAIT状态，再进入CLOSED状态呢？  

当连接出于TIME_WAIT 状态时，要等待2MSL的时间，才能迁移到CLOSED状态。  

一种情况，因为TCP连接的本地端，需要发送一个ACK报文以确认另一端的FIN报文，然而它并不确定这个ACK报文是否已成功交付。另一端，如果没有收到ACK报文，就会重传FIN报文。在这种情况下，TIME_WAIT状态的作用就是等待，以在有需要时重传ACK。

假设目前连接的通信双方都已经调用了close()，双方同时进入CLOSED的终结状态，而没有走TIME_WAIT状态。会出现如下问题，现在有一个新的连接被建立起来，使用的IP地址与端口与先前的完全相同，后建立的连接是原先连接的一个完全复用。还假定原先的连接中有数据报残存于网络之中，这样新的连接收到的数据报中有可能是先前连接的数据报。为了防止这一点，TCP不允许新连接复用TIME_WAIT状态下的socket。处于TIME_WAIT状态的socket在等待两倍的MSL时间以后，将会转变为CLOSED状态。这就意味着，一个成功建立的连接，必然使得先前网络中残余的数据报都丢失了。

>MSL就是maximum segment lifetime(最大分节生命期），这是一个IP数据包能在互联网上生存的最长时间，超过这个时间IP数据包将在网络中消失 。MSL在RFC 1122上建议是2分钟，而源自berkeley的TCP实现传统上使用30秒。


## 参考资料
《计算机网络基础教程》    
[TCP/IP详解--TCP连接中TIME_WAIT状态过多](http://blog.csdn.net/yusiguyuan/article/details/21445883 "TCP/IP详解--TCP连接中TIME_WAIT状态过多")  
[TCP/IP TIME_WAIT状态理](http://elf8848.iteye.com/blog/1739571 "TCP/IP TIME_WAIT状态原理")  
