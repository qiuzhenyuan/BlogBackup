---
title: PHP中的类型转换
date: 2017-09-18 17:46:47
categories: "语言细节" 
tags: "PHP"
---



>类型转换，是指变量从一种数据类型转变成另一种数据类型，类型转换的方法有两种，一种是自动转换，另一种是强制转换。

<!-- more -->

## 自动类型转换的判别

PHP 在变量定义中不需要（或不支持）明确的类型定义；变量类型是根据使用该变量的上下文所决定的。也就是说，如果把一个 string 值赋给变量 $var，$var 就成了一个 string。如果又把一个integer 赋给 $var，那它就成了一个integer。  
```
	<?php
	    $var=123;
	    var_dump($var);
	    $var='hi';
	    var_dump($var);
	    $var=true;
	    var_dump($var);
	?> 
```

**输出结果如下：**
![这里写图片描述](http://img.blog.csdn.net/20170226181551803?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMzM5OTQ3NDQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

PHP 的自动类型转换的一个例子是加法运算符"+"。如果任何一个操作数是float，则所有的操作数都被当成float，结果也是float。否则操作数会被解释为integer，结果也是integer。注意这并没有改变这些操作数本身的类型；改变的仅是这些操作数如何被求值以及表达式本身的类型。  
```
	<?php
	    //运算自动转换
	    $foo = "0";  // $foo 是字符串 (ASCII 48)
	    var_dump($foo);
	    $foo += 2;   // $foo 现在是一个整数 (2)
	    var_dump($foo);
	    $foo = $foo + 1.3;  // $foo 现在是一个浮点数 (3.3)
	    var_dump($foo);
	    $foo=1;
	    $bar=$foo+1.22;     //$foo还是一个整形，$bar是浮点数
	    var_dump($foo);
	    var_dump($bar);
	?>
```
**运行结果如下：**
![这里写图片描述](http://img.blog.csdn.net/20170226181654026?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMzM5OTQ3NDQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 强制转换  
PHP 中的类型强制转换和 C 中的非常像：在要转换的变量之前加上用括号括起来的目标类型。 
允许的强制转换有： 


- (int), (integer) - 转换为整形 integer 
- (bool), (boolean) - 转换为布尔类型 boolean 
- (float), (double), (real) - 转换为浮点型 float 
- (string) - 转换为字符串 string 
- (array) - 转换为数组 array 
- (object) - 转换为对象 object 
- (unset) - 转换为 NULL (PHP 5) 


>另外，还可以用strval(),intval()，floatval()等函数来进行强制类型转换。

## 各种类型的相互转换

### 转换为布尔型  
当转换为 boolean 时，以下值被认为是 FALSE： 


- 布尔值 FALSE 本身  
- 整型值 0（零）  
- 浮点型值 0.0（零）  
- 空字符串，以及字符串 "0" 
- 不包括任何元素的数组  
- 不包括任何成员变量的对象（仅 PHP 4.0 适用）  
- 从空标记生成的 SimpleXML 对象  


**所有其它值都被认为是 TRUE（包括任何资源）。**  
测试程序如下：
```
	<?php
	    $arr=array();
	    if(0){
			echo '0 is true </br>';
		}else{
			echo '0 is false </br>';
		}
		if(0.0){
			echo '0.0 is true </br>';
		}else{
			echo '0.0 is false </br>';
		}
		if(''){
			echo '空字符串 is true </br>';
		}else{
			echo '空字符串 is false </br>';
		}
		if($arr){
			echo "空数组 is true </br>";
		}else{
			echo "空数组 is false </br>";
		}
		if(null){
			echo 'null is true </br>';
		}else{
			echo 'null is false </br>';
		}
```

**运行结果如下：**

![这里写图片描述](http://img.blog.csdn.net/20170226181830042?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMzM5OTQ3NDQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  


### 转换为string型
一个值可以通过在其前面加上 (string) 或用 strval() 函数来转变成字符串。在一个需要字符串的表达式中，会自动转换为 string。比如在使用函数 echo 或 print 时，或在一个变量和一个 string 进行比较时，就会发生这种转换。类型和类型转换可以更好的解释下面的事情，也可参考函数 settype()。 

一个布尔值 boolean 的 TRUE 被转换成 string 的 "1"。Boolean 的 FALSE 被转换成 ""（空字符串）。这种转换可以在 boolean 和 string 之间相互进行。 

一个整数 integer 或浮点数 float 被转换为数字的字面样式的 string（包括 float 中的指数部分）。使用指数计数法的浮点数（4.1E+6）也可转换。

>注意：数组转化成字符串会变成Array，对象不可以转换成字符串。资源 resource 总会被转变成 "Resource id #1"  这种结构的字符串，其中的 1 是 PHP 在运行时分配给该 resource 的唯一值。不要依赖此结构，可能会有变更。直接把 array，object 或 resource 转换成 string 不会得到除了其类型之外的任何有用信息。可以使用函数 print_r() 和 var_dump() 列出这些类型的内容。 
```
	<?php
		error_reporting(E_ALL&~E_NOTICE);
		$arr=array();
		$object=new stdClass();  
		var_dump(strval(1));
		var_dump(strval(1.01));
		var_dump(strval(true));
		var_dump(strval(false));
		var_dump($arr);
	?>
```
**运行结果如下:**

![这里写图片描述](http://img.blog.csdn.net/20170226182415082?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMzM5OTQ3NDQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  


### 转换为浮点数
如果对字符串进行数值运算，**包含 '.'，'e' 或 'E'** 并且其数字值在整型的范围之内（由 PHP_INT_MAX 所定义），该字符串将被当成 **float**来取值，否则转换为integer。
```
	<?php
		$foo = 1 + "10.5";                // $foo is float (11.5)
		$foo = 1 + "-1.3e3";              // $foo is float (-1299)
		$foo = 1 + "bob-1.3e3";           // $foo is integer (1)
		$foo = 1 + "bob3";                // $foo is integer (1)
		$foo = 1 + "10 Small Pigs";       // $foo is integer (11)
		$foo = 4 + "10.2 Little Piggies"; // $foo is float (14.2)
		$foo = "10.0 pigs " + 1;          // $foo is float (11)
		$foo = "10.0 pigs " + 1.0;        // $foo is float (11)     
	?>   
```


### 转换为整型  
要明确地将一个值转换为 integer，用 (int) 或 (integer) 强制转换。不过大多数情况下都不需要强制转换，因为当运算符，函数或流程控制需要一个 integer 参数时，值会自动转换。还可以通过函数 intval() 来将一个值转换成整型。   

从**布尔型**转换时，FALSE 将产生出 0（零），TRUE 将产生出 1（壹）。

当从**浮点数**转换成整数时，将向下取整。 
>注意：如果浮点数超出了整数范围（32 位平台下通常为 +/- 2.15e+9 = 2^31，64 位平台下通常为 +/- 9.22e+18 = 2^63），则结果为未定义，因为没有足够的精度给出一个确切的整数结果。在此情况下没有警告，甚至没有任何通知！ 

从**字符串**转换时，字符串的首字母若为整型，则转化为该整型，若字符串首字母不是整形，则转化为0.  
```
	<?php
	    $foo=12 + "5fa";        //结果为17
	    var_dump($foo);
	    $bar=12 + "f5a";        //结果为12
	    var_dump($bar);
		?>
```
### 转换为数组  
对于任意 integer，float，string，boolean 和 resource 类型，如果将一个值转换为数组，将得到一个仅有一个元素的数组，其下标为 0，该元素即为此标量的值。换句话说，(array)$scalarValue 与 array($scalarValue) 完全一样。 

如果一个 object 类型转换为 array，则结果为一个数组，其单元为该对象的属性。键名将为成员变量名，不过有几点例外：整数属性不可访问；私有变量前会加上类名作前缀；保护变量前会加上一个 '*' 做前缀。这些前缀的前后都各有一个 NULL 字符。这会导致一些不可预知的行为： 
```
	<?php
		class A {
		    private $A; // This will become '\0A\0A'
		}
		
		class B extends A {
		    private $A; // This will become '\0B\0A'
		    public $AA; // This will become 'AA'
		}
		
		var_dump((array) new B());
	?>   
```
### 转换为对象  
如果将一个对象转换成对象，它将不会有任何变化。如果其它任何类型的值被转换成对象，将会创建一个内置类 stdClass 的实例。如果该值为 NULL，则新的实例为空。数组转换成对象将使键名成为属性名并具有相对应的值。对于任何其它的值，名为 scalar 的成员变量将包含该值。  
```
	<?php
		$obj = (object) 'ciao';
		echo $obj->scalar;  // outputs 'ciao'
	?> 
```
### 转换为null  
使用 (unset) $var 将一个变量转换为 null 将不会删除该变量或 unset 其值。仅是返回 NULL 值而已。


## 参考资料
[PHP - Manual](http://www.php.net/manual/zh/ "PHP - Manual")



