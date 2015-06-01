---
layout: post
title: "Spring AOP"
date: 2014-08-16 17:46:32 +0800
comments: true
categories:  [study, AOP]
---

对Spring AOP的实现机制一直是只知其然，而不知其所以然，平时工作中也仅限于会配置会用而已。前几周Debug时，发现BeanCopier这个类，进而了解到
cglib (code generate library)这个jar,后来顺藤摸瓜地发现，cglib是实现Spring AOP动态代理的基础，就想着写对Spring AOP的理解梳理一下，并参照网上的例子，在本地搭建maven工程调试了下。查看到是如何通过Enhancer来生成代理类，了解其运行原理。

## AOP存在的价值
AOP 专门用于处理系统中分布于各个模块（不同方法）中的交叉关注点的问题，在 Java EE 应用中，常常通过 AOP 来处理一些具有横切性质的系统级服务，如事务管理、安全检查、缓存、对象池管理等，AOP 已经成为一种非常常用的解决方案。
解决了一些OOP无法解决的问题。是OOP很好的补充，在保证原有实现类和方法不变的情况下，添加额外的一些操作，从另一种行式上实现类的[开闭原则](http://zh.wikipedia.org/wiki/开闭原则)。

## 应用场景
* 切面上实现事务，当要在某些/个方法上实现事务时，将方法定义为切面，使用Spring AOP来实现事务
* 切面上记录日志，当某个Action被人操作或Service调用时，可以将时间(操作或调用时间)、人物(操作人或调用方)记录下来，方便问题定位。
* 权限控制，在某个类上加上权限控制，在程序运行过程中，通过CGLib动态生成带有权限或没有权限的类，由此来实现权限控制。


## Spring AOP 支持的4种类型advice
* before advice 在方法执行前执行
* after returning advice 在方法执行后返回一个结果后执行
* after throwing advice 在方法执行过程中抛出异常的时候执行
* Around advice 在方法执行前后和抛出异常时执行，相当于综合了以上三种通知。但在所定义的AroundMethod中要显示地去调用主体方法函数。


## AOP代理
代理分为静态和动态两种。静态是指使用AOP框架提供的命令进行编译，从而在编译阶段就可生成AOP代理类，因此也被称为编译时增强，可使用AspectJ在编译时增强进行AOP；
而动态代理则在运行时借助于JDK动态代理或CGLIB<sup>[1]</sup>等在内存中“临时”生成AOP动态代理类，因此也被称为运行时增强。
一般来说，编译时增强的AOP框架在性能上更有优势，因为运行时动态增强的AOP框架需要每次运行时进行动态增强。

Spring AOP 采用运行时动态地、在内存中临时生成代理类的方式来生成AOP代理。Spring AOP框架对AOP代理类的处理原则是：如果目标对象的实现类实现了接口，Spring AOP将会采用JDK动态代理来生成AOP代理类；如果目标对象的实现类没有实现接口，Spring AOP将会采用CGLIB来生成AOP代理类--不过这个选择过程对开发者完全透明，开发者也无需关心。当Spring AOP选择使用JDK动态代理时，直接使用JDK提供的Proxy和InvocationHandler来生成AOP代理即可。

简而言之：AOP原理的奥妙就在于动态地生成了代理类，这个代理类实现了对原有类方法进行的额外的操作。


##Reference 
> 1. [Spring AOP 实现原理与 CGLIB 应用](http://www.ibm.com/developerworks/cn/java/j-lo-springaopcglib/#icomments)
> 2. [Spring AOP官方文档](http://docs.spring.io/spring/docs/2.5.4/reference/aop.html) 
> 3. [深入浅出 java动态代理](http://blog.csdn.net/cjl5678/article/details/10586645)


[1]CGLIB(Code Generation Library)简单来说，就是一个代码生成类库，它可以在运行时候动态地生成某个类的子类。
