---
title: 说说CGI与FastCGI
date: 2017-11-28 23:47:27
categories: "Web开发常识" 
---
## CGI简介
CGI全称是“通用网管接口”（Common Gateway Interface),是Web服务器（如Appache，Nginx）与外部应用程序通讯的一种接口标准。CGI程序可以用任何一种语言编写，只要这种语言具有标准输入、输出和环境变量。如php,perl等。  

<!-- more -->

CGI规范允许Web服务器执行外部程序，并将它们的输出发送给Web浏览器，将Web的一组简单的静态文档，变成一个完整的新的交互式文档。  

CGI的具体工作过程如下：

1. 客户端访问某个URL，通过HTTP协议向Web服务器发起请求。
2. 服务器端的守护进程启动一个子进程，将HTTP请求里的信息，通过标准输入stdin 传递给URL指定的CGI程序，并启动程序进行处理。
3. CGI程序处理完成后，把结果通过标准输出stdout 返回给服务器的守护进程。
4. 服务器守护进程通过HTTP协议返回给浏览器

![CGI工作过程——摘自深入理解PHP内核](https://wx4.sinaimg.cn/mw690/857afa84ly1fm056ayx48j21fq0sin49.jpg)

## FastCGI简介

FastCGI是Web服务器和处理程序之间通信的协议，是CGI的一种改进方案。与CGI没接受一次请求就Fork一个进程不同。FastCGI像是一个常驻型的CGI，它可以一直执行，在请求到来时，不需要花费时间去fork一个进程来处理请求，从而大大提升性能。  

FastCGI与语言无关，将CGI解析器进程保存在内存中，以此保证较高的性能，FastCGI的工作流程如下：

1. FastCGI进程管理器自身初始化，启动多个CGI解析器进程，等待来自Web Server的连接。
2. Web服务器与FastCGI进程管理器进行Socket通信，通过FastCGI协议发送CGI环境变量和标准输入数据给CGI解释器。
3. CGI解释器进程完成处理后，将标准输出和错误信息，从同一个连接返回给Web服务器。
4. CGI解释器进程接着等待并处理来自Web服务器的下一个连接。

![FastCGI工作流程——摘自深入理解PHP内核](https://wx2.sinaimg.cn/mw690/857afa84ly1fm05nhkw0mj21cu0yuk3p.jpg)

## 参考资料
[FastCGI——深入理解PHP内核](http://www.php-internals.com/book/?p=chapt02/02-02-03-fastcgi "FastCGI")
[CGI——百度百科](https://baike.baidu.com/item/CGI/607810?fr=aladdin)  
[FastCGI——百度百科](https://baike.baidu.com/item/fastcgi)