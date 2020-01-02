---
title: hashlib模块使用
date: 2019-12-29 22:17:00
tags:
- python
- hashlib
categories:
- python
---
> &#160; &#160; &#160; &#160;md5在爬虫队列去重、接口加密方面都能用到，是比较基础和常见的摘要算法，本文记录:
> 1、基本概念&#160; &#160;2、hashlib模块的使用&#160; &#160;3、hmac模块的使用
<!--more-->
### 一、基本概念
&#160; &#160; &#160; &#160;Python的hashlib提供了常见的摘要算法，主要提供 如MD5，SHA1, SHA224, SHA256, SHA384, SHA512 算法等。
&#160; &#160; &#160; &#160;摘要算法又称哈希算法、散列算法。它通过一个函数，把任意长度的数据转换为一个长度固定的数据串（通常用16进制的字符串表示）。

### 二、如何产生hash值
```Python
import hashlib
md5 = hashlib.md5()
md5.update('this is md5'.encode('utf-8'))
print(md5.digest())  #digest:摘要（二进制）
print(md5.hexdigest()) #hex:十六进制
print(md5.)
```
输出： 

![](/python-hashlib/1.png)
注意：

- md5可以替换为其他算法，不同算法得到的长度是不同的。
- 无论传入多大文本，同一种算法得到的长度都是相同的。
- `md5 = hashlib.md5()`括号内也可以传值
- 文本过大可以使用`md5.update()`多次传值，其结果与一次传值是一样的。

### 三、hmac模块
&#160; &#160; &#160; &#160;使用hmac模块时，若要使用update()，初始值必须一致，否则结果会不同。
```Python
import hmac
h1=hmac.new(b'tom')          #初始值必须保证一致，最终得到的结果就会不一样
h1.update(b'hello')
h1.update(b'world')
print(h1.hexdigest())

h2=hmac.new(b'tom')         #初始值必须保证一致，最终得到的结果就会不一样
h2.update(b'helloworld')
print(h2.hexdigest())

h3=hmac.new(b'tomhelloworld')   #初始值不一样，所以与上面两种的结果不一样
print(h3.hexdigest())
```
输出：

![](/python-hashlib/1.png)

