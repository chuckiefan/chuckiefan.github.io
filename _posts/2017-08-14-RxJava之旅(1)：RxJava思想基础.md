---
layout: post
title: 'RxJava之旅(1)：RxJava思想基础'
date: 2017-04-18
author: Chuckiefan
tags: RxJava Android
---
# 0. 前言

这两年来RxJava越来越流行，但是在对RxJava的态度上呈现两极分化的趋势，一部分人非常喜欢RxJava，觉得非常好用；也有很多人觉得RxJava很难用。在我最初学习RxJava的时候属于后者，在我个人看来，其原因是RxJava的入门曲线比较高，在度过了最初的学习阶段以后才能真正体会到RxJava的好处。
因此，我决定写下这个系列文章，把RxJava的基础到进阶的内容进行讲解，如果能够帮助到和我一样在入门阶段感到困惑的人，我将感到非常荣幸。
本系列文章将会从基础入门开始介绍，其目的是尽快让读者掌握RxJava的基本使用，后续的文章将会对RxJava的特性进行逐渐深入的介绍。但是本系列文章不会提供任何可以直接在实际开发中复制使用的样例代码，其侧重点更多的是对RxJava的理解，我相信对于真正要学习技术的人来说，从文章里复制代码直接使用只是快速应付工作而已，并不能对该技术有任何真正的了解。
本人在学习RxJava时，主要通过扔物线，大头鬼等大牛的文章以及RxJava的源代码。以下是基础入门时所阅读的文章地址，希望能够对各位有所帮助：
[《给Android开发者的RxJava详解》 by:扔物线](http://gank.io/post/560e15be2dca930e00da1083)
[《深入浅出RxJava》 by:大头鬼](http://blog.csdn.net/lzyzsd/article/details/41833541)
[《谜之RxJava》 by:Gemini](https://segmentfault.com/a/1190000004049490)

# 1. 什么是RxJava

在Github官方项目上对ReactiveX的介绍原文如下：

> Reactive Extensions for Async Programming

也就是说，ReactiveX是异步编程的响应式扩展。
而RxJava的介绍如下：

> RxJava – Reactive Extensions for the JVM – a library for composing asynchronous and event-based programs using observable sequences for the Java VM.

也就是说，RxJava是在Java上的响应式扩展，通过使用可观察序列，用于组成异步和基于事件编程的类库。
官方的介绍过于正式和全面，非常不易于初学者的理解。从易于理解的角度来说，RxJava是使用Java语言，以响应式编程思维来进行编程的Java类库。

# 2. 响应式编程思维

## 2.1 学习技术的关键

我个人始终认为，掌握一项技术的关键是理解该技术的思想，而不是技术细节。能否理解思想是“会不会”的问题，对技术细节的熟悉程是“好不好”的问题。对于初学者来说其实不存在“好不好”的问题，因为完全不会这一门技术的情况下根本谈不到对技术运用的娴熟度。
在大学刚开始学习编程语言时，很多人感到非常痛苦，原因就在于过于关注语法和函数运用而忽略了该编程语言背后的编程思维/思想。例如，学习C语言时，重点应该是面向过程编程和结构化程序设计(忽略goto语句)；在C++的学习中，重点应该是面向对象的编程思想。在理解编程思想的基础之上，才能真正理解该编程语言的特性，如C++中的继承，多态等等。

## 2.2 面向过程与面向对象简介

让我们首先回顾一下面向对象和面向过程思想。编程思想决定了编程语言的编程方式，而编程思想的不同是建模角度的不同，说通俗一些就是对世界万物理解方式的不同。
我们都知道，计算机编程在现实生活中的运用是通过对具体问题的关键特性进行抽象（建模），从而利用计算机进行解决。在面向过程的思想下，*一切皆过程*，所有的问题都通过对事件过程进行抽象去解决。而在面向对象编程中，*一切皆对象*，世界万物都是一个个单独离散的个体（对象），所有的动作行为都是个体间的交互。

> 注：之所以如此繁琐的重复面向对象和面向过程的大概内容，是为了能够和响应式编程进行类比，从而尽快理解响应式编程的概念。

## 2.3 响应式编程简介

响应式编程是面向数据流的编程思想，在响应式编程思想下，*一切皆数据流*。响应式编程所侧重的是数据流的流动。
为了更好的理解响应式编程思维，这里举一个网络请求的例子，在不同的编程思想下其解决方式是不同的。

1. 在面向过程的编程思想下，我们所关注的是过程步骤的次序。在该例中，我们所关注的是网络请求的过程：发起请求-\>接收请求-\>处理请求。
2. 在面向对象的编程思想下，我们所关注的是对象建模和对象间的交互，如构建请求网络的类对象，该类对象的构造方法、成员变量以及这个网络请求类对象所需要的成员方法等等。
3. 在响应式编程的思想下，我们所关心的是数据流的流动。如网络请求这一数据流发起后，数据流流向服务器接口，经过服务器接口处理后流出返回的数据流，返回的数据流再经过若干步的处理，最后流出结果为我们所需要的数据流。

*所以，学习RxJava首先要了解什么是响应式编程，而理解响应式编程首先要养成一切皆数据流的编程思维。*

> 限于篇幅和限于个人能力，这里对面向过程、面向对象的概念介绍是不完整和不严谨的（例如，面向对象和面向过程并不是完全独立的两种编程思想，完整地介绍类似的概念过于庞大），但这并不会影响到对RxJava的理解。

## 2.4 响应式编程与RxJava

2.3节对响应式编程进行了简单的介绍，重点突出了一切皆数据流的编程思维方式，但是这还不能完全理解官方github上对RP(React Programming)和RxJava的定义，因此需要进行补充。

### 2.4.1 ReactiveX补充(异步编程)

再来回顾一下上文所引用的ReactiveX定义：

> Reactive Extensions for Async Programming

ReactiveX是对异步编程的响应式扩展，在2.3我们简单介绍了响应式编程，而这一小节重点突出的是，ReactiveX不仅仅是响应式编程，还是支持异步编程的。

### 2.4.2 RxJava补充(观察者模式)

回顾一下RxJava的定义：

> Reactive Extensions for the JVM – a library for composing asynchronous and event-based programs using observable sequences for the Java VM.

这里需要突出的是可观察序列(observable sequences)和基于事件(event-based)。

> 基础事件的概念不需要多述，如果是刚刚学习编程的读者可以通过私信和评论单独询问。

让我们再来看一下官方的详细介绍：

> It extends the observer pattern to support sequences of data/events and adds operators that allow you to compose sequences together declaratively while abstracting away concerns about things like low-level threading, synchronization, thread-safety and concurrent data structures.

大意如下：
> RxJava扩展了观察者模式，以支持数据/事件序列，并且添加了操作符，允许你组成序列，而不用过于关心线程和同步异步的问题。

之所以只翻译了大意，是因为这里要突出的重点是观察者模式(Observer Pattern)，其他内容会在后面的文章中介绍。
观察者模式是软件设计模式中的一种，在软件开发中被大量运用。观察者(Observer)对被观察者(Observable)发出的事件进行响应处理。观察者将自己注册到被观察者中，当被观察者发生变化时会通知观察者，从而观察者对这次变化响应，进行处理。
相信大部分人对于观察者模式都有一定的了解，如果不清楚相关概念请参考软件设计模式相关的书籍和资料，或者通过私信和评论等方式私下询问，这里不再赘述。
观察者模式的类图如下所示：

![观察者设计模式类图](http://upload-images.jianshu.io/upload_images/1226129-65b5ab5fed7d537d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在RxJava中，根据响应式编程的概念，被观察者实际上是数据流，*数据流的流动其实就是观察者对被观察者变化的响应*，这一点对于后面的学习非常关键。

# 3. 总结与预告

这一篇文章并没有涉及到任何的代码细节和RxJava的技术细节，仅仅介绍了RxJava相关的思想基础。但是这些思想基础非常关键，只有在理解这些思想的前提下，才能够更容易理解RxJava的具体内容。
下一篇文章将会开始介绍RxJava的入门内容，我会尽快写好发布出来。在阅读文章的过程中，如果有任何不妥和疑惑以及没有介绍清楚的地方欢迎指出交流，谢谢阅读。