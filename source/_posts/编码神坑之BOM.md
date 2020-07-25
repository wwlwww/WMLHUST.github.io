---
title: 编码神坑之BOM
tags: []
categories: []
comments: true
date: 2015-10-29 21:43:29
updated: 2015-10-29 21:43:29
description:
---

今天在做项目时，发现了一个超奇葩的问题。

背景条件：

1. 客户端 java

2. 后台php

问题：

php后台使用"echo $result"返回了一个字符串“success”，但是这个字符串和java本地的"success"字符串用equals比较时竟然返回结果是false。

java端的代码：   
![](/images/bom-1.png)

过程：

哎哟喂，这个可把我给难住了，百思不得其解。直接打印返回的刚开始根据网上查到equals函数实现，自己写个函数替代equals，如下：



然后就单步跟踪调试了，发现281行的循环根本进不去，然后看了下变量各个字符的值，表面的问题终于找到了



这是返回字符串变量的各个位的字符。   
![](/images/bom-2.png)


这是本地用String succString = "success"生成的一个字符串。   
![](/images/bom-3.png)


可以发现，虽然第一行显示字符串的值都是“success”，然而，count和value一项都不一样。。

这简直醉了，难道是java网络数据读取的问题？（我为什么会怀疑到这个点。。）

继续找直接找到了post请求的返回数据，从里面一层一层也可以找到这个字符串的所在：

![](/images/bom-4.png)

再继续展开value：   
![](/images/bom-5.png)


但是，为什么里面是这样子的，我还没想通，看起来也不像是因为字节序的问题，wireshark抓包结果   
![](/images/bom-6.png)

最后的7个数字确实跟success的ASCII码是一样的。

可以确认，之所以字符串前面有空白字符，是因为后台传返回数据即是这样。

为了查看后台返回的数据，我又写了个表单，用来直接模拟post请求：   
![](/images/bom-7.png)
![](/images/bom-7.2.png)

输入数据，返回结果。表面没问题   
![](/images/bom-8.png)

审查元素：   
![](/images/bom-9.png)


可以看到，实际前面多了一串莫名其妙的字符 &#65279

这一串数字可以直接google到，它其实是UTF-8编码的一个叫BOM的头。

BOM 全称 bom order mark，"EF BB BF" 这三个字节就叫BOM。

(这个为什么这么叫，似乎应该是固定好的标志，我试着用unicode解码并没有解出来”BOM“)。

在utf8文件中常用BOM来表明这个文件是UTF-8文件，而BOM的本意是在utf16中用。

utf-8文件在php中输出的时候bom是会被输出的，所以要在php中使用utf-8，必须要是使用不带bom头的utf-8文件。

wiki的解释非常不错，在里面也找到了各种编码的BOM标志。

https://zh.wikipedia.org/wiki/%E4%BD%8D%E5%85%83%E7%B5%84%E9%A0%86%E5%BA%8F%E8%A8%98%E8%99%9F



其次，windows的记事本保存的文本都是有BOM的，notepad++等文本编辑器可以设置无bom编码格式。

参考这个知乎问题：http://www.zhihu.com/question/20167122



嗯，修改编码格式后，一切正常。

