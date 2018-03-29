---
title: PHP那些无所不能的数组函数（长期更新）
date: 2017-07-04 00:23:55
tags: [长期更新,PHP]
---


## 基本函数

### 数组元素个数

```php
	int count ( mixed $array_or_countable [, int $mode = COUNT_NORMAL ] )
```

<!-- more -->  
array_or_countable：可以是数组或者 Countable 对象。

mode：如果可选的 mode 参数设为 COUNT_RECURSIVE（或 1），count() 将递归地对数组计数。对计算多维数组的所有单元尤其有用。

## 字符串相关

#### 字符串转数组

```php
array explode ( string $delimiter , string $string [, int $limit ] )
```

`delimiter` 是边界上的分隔字符。

`string` 是输入的字符串。

#### 数组转字符串

```php
string implode ( string $glue , array $pieces )
```

输入字符串，以及数组。输出字符串。



## 数据结构相关

### 1. 栈：array_push与array_pop  


```php
	int array_push ( array &$array , mixed $value1 [, mixed $... ] );    
	mixed array_pop ( array &$array );
```


array_push()将 $array 当成一个栈，并将`$value`的值变量压入 $array 的末尾。  
array_pop()弹出并返回 array 数组的最后一个单元，并将数组 array 的长度减一。


### 2. 队列： array_shift与array_unshift()

```php
	mixed array_shift ( array &$array )
```

array_shift()将 array 的第一个单元移出并作为结果返回，将 array 的长度减一并将所有其它单元向前移动一位。所有的数字键名将改为从零开始计数，文字键名将不变。

array_unshift()将传入的单元插入到 array数组的开头。注意单元是作为整体被插入的，因此传入单元将保持同样的顺序。所有的数值键名将修改为从零开始重新计数，所有的文字键名保持不变。

### 3. 切片： array_slice()  

```php
	array array_slice ( array $array , int $offset [, int $length = NULL [, bool $preserve_keys = false ]] )  
```

array_slice() 返回根据 offset 和 length 参数所指定的 array 数组中的一段序列。  

## 参考资料
[PHP - Manual](http://www.php.net/manual/zh/ "PHP - Manual")