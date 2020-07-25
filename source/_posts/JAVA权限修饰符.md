---
title: JAVA权限修饰符
tags: [java]
categories: []
comments: true
date:  2015-10-16 11:23:07
updated:  2015-10-16 11:23:07
description:
---

为了避免看了忘，忘了看，还是用文章记录下来吧。

应该说有四种权限修饰情况，public，protected，private，default

1. public   
可以修饰类、成员变量、成员方法。几乎没有任何限制。

2. private   
可以修饰成员变量、成员方法、内部类。正好和public两个极端   
成员变量：本类内部访问，本类实例都不行。   
成员方法：同 成员变量 
内部类：同 成员变量

3. protected   
修饰成员变量、成员方法、内部类。   
成员变量：该变量可以在本类，本包内其他类，其他包的子类中访问   
成员方法：同 成员变量   
内部类：这个有点复杂，给个链接在这里吧：http://m.blog.csdn.net/blog/clarkdu/334925

4. default   
就是什么关键词都不写，修饰类、成员变量、成员方法   
default又称 包存取权限，对于default的类，只有同包中的类能够调用这个类的成员变量或方法。   
在类的修饰符与类成员修饰符相冲突时，比如类是default，而成员变量a为public，则实际上，a的存取权限也是default。


参考：http://blog.csdn.net/yan8024/article/details/6426451