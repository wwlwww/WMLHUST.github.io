---
title: java.io.FileNotFoundException-再次踩坑windows编码
tags: []
categories: []
comments: true
date: 2016-03-24 16:26:10
updated: 2016-03-24 16:26:10
description:
---

####**问题**

为了方便读取文件，直接从windows文件属性里复制了路径，如图：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzI0MTEzNTAxMTMz?x-oss-process=image/format,png)

然后贴到eclipse里：
	![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzI0MTQ1NTA2ODUw?x-oss-process=image/format,png)
	![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzI0MTUwNDMyNTQ5?x-oss-process=image/format,png)
	
表面上看pathIn和pathIn2似乎没有什么不同，然而，在创建File对象时，总是提示java.io.FileNotFoundException，要报警，上次就碰到这个问题但是没解决。上次碰到的问题是，在windows一个问价夹里竟然可以存在两个同名文件！！
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzI0MTUwNjQ1OTc0?x-oss-process=image/format,png)
####**是时候展现真正的技术了**
我把那些字符串都复制出来，写一个简单的html来测试一下：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzI0MTUwODIyNDAy?x-oss-process=image/format,png)
看下实际效果：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzI0MTUxOTE5ODE4?x-oss-process=image/format,png)
这下明了了，前面的"&#8234 ;"特么是个什么鬼，CSDN markdown也打不出来这个字符串，分号和4之间没有空格。搜一下吧：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzI0MTU1NTE4Nzk1?x-oss-process=image/format,png)
从左到右的植入?
关于双向文本
https://en.wikipedia.org/wiki/Bi-directional_text
推荐：http://www.iamcal.com/understanding-bidirectional-text/
其实是unicode标准里，为了适配某些字符集的规定，比如阿拉伯语，显示的时候是从右向左的（为啥？难道他们写字是从右向左？果然是这样！！刚查了一下）
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzI0MTYxNTIyODkw?x-oss-process=image/format,png)
后来我发现，windows里面其实有个小提示的，看箭头指的地方，有个浅灰色的竖线。然而复制到其他地方就不显示了，notepad里显示全部字符也不显示，但是确实被复制过去了。
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzI0MTYxMjQ4NTIx?x-oss-process=image/format,png)

这个问题网上的暂时还没见到这种解法，对于文件确实存在，但是总提示FileNotFound的，stackoverflow也都是让检查文件名是不是违反了windows的命名规则，希望这篇博文有所帮助。

刚开始简直有在电脑上装Ubuntu的冲动，然而想了想那么多的开发环境（手动再见）