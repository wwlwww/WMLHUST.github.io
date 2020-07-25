---
title: PathEscape与QueryEscape
tags: []
categories: []
comments: true
date: 2020-05-29 12:56:52
updated: 2020-05-29 12:56:52
description: 搞懂PathEscape与QeuryEscape区别
---
在给client种cookie时，发现个问题，种进去的加密cookie，解密时总报错。   
原因是：golang中，对一个字符串做url转义有两个方法，`url.PathEscape()`和`url.QueryEscape`。但是两个方法的行为有些区别。两者混用导致，编码和解码后，和原始字符串不一致。

### 1. 举个例子，直接对比下效果   

以 + 和 空格 这两个字符为例。   

| 待转义字符 | PathEscape | QueryEscape | PathUnEscape | QueryUnEscape |
| :-------: | :--------: | :---: |:---: |:---: |
| +    | +    | %2B |  + |  空格 |
| 空格  | %20  | + | 空格 |  空格 |

### 2. 具体功能
#### 2.1 PathEscape
对特殊字符串进行转义，以便其可以作为url路径的一部分。就是URL地址两个 / 之间的部分

#### 2.2 QueryEscape
对特殊字符串进行转义，以便其可以作为url query的参数，也就是 ？后面那一串kv。

#### 2.3 对比
1. 两者的共同点在于：都会将一些特殊字符，转义为`%AB`的形式。特殊字符的定义为，除` a-z，A-Z，0-9，- _ ~ · , / ; ?` 的字符。

2. 不同点在于：对于一些特殊字符，转义行为不同。

| 字符 | PathEscpae | QueryEscape | 
| :-------: | :--------: | :---: |
| $ | Y | N |
| & | Y | N |
| + | Y | N |
| : | Y | N |
| = | Y | N |
| @ | Y | N |

具体的可参考RFC文档（URI、URL的两篇）和Golang的源码。（Golang源码更简单直接）

### 3. 其他
其他语言似乎没分得那么清，具体实现上也有一些区别，比如python/javascript，encode行为就和golang的不一致。总之，同一语言，如golang，QueryEscape编码后，一定要配合QueryUnEscape使用。

一些细节也不同，比如JS里的encodeURI和encodeURIComponent。可以理解为，encodeURI，是把参数当做一个完整的URI在编码，而encodeURIComponent是把参数当做URI的一个segment。
```javascript
// JS里
encodeURI("12+34 56")
output: "12+34%2056"

encodeURIComponent("12+34 56")
output: "12%2B34%2056"

var a = "http://www.ruanyifeng.com/blog/2010/02/url_encoding.html"
encodeURI(a)
output: "http://www.ruanyifeng.com/blog/2010/02/url_encoding.html"

encodeURIComponent(a)
output: "http%3A%2F%2Fwww.ruanyifeng.com%2Fblog%2F2010%2F02%2Furl_encoding.html"
```

### 参考：
1. http://www.ruanyifeng.com/blog/2010/02/url_encoding.html
2. https://tools.ietf.org/html/rfc1738
3. https://tools.ietf.org/html/rfc3986

