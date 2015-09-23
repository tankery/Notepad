---
layout: post
title: RxJava 的那些坑
description: "RxJava 是 ReactiveX，响应函数式编程库的一个平台扩展，本文结合自己的使用，对其进行介绍"
headline: "RxJava 是 ReactiveX，响应函数式编程库的一个平台扩展，本文结合自己的使用，对其进行介绍"
categories: development
tags:
  - ReactiveX
  - RxJava
  - 函数式编程
comments: true
mathjax: null
featured: true
published: true
---

RxJava 是 ReactiveX，响应函数式编程库的一个平台分支，本文结合自己的使用，对其进行介绍。

项目中使用的异步任务库 [Groundy](https://github.com/telly/groundy) 停止维护了。
它推荐我们使用 [RxJava](https://github.com/ReactiveX/RxJava)，[ReactiveX](http://reactivex.io/) 的Java平台扩展。
提供函数式编程，方便异步、事件驱动编程的库。

这个库其实不大，但由于其基于一套抽象的编程范式，使得其学习曲线非常陡峭。
我花了一周的上下班和其他零碎时间，才自信到可以将此编程模式引入目前的项目中。

但是，也正因为在学习过程中，不断的发现其过人之处，才能兴致勃勃的不断了解。
并且，随着学习的不断深入，我越来越发现，ReactiveX 为我打开了一扇通向全新世界的门。

<!--break-->

## RxJava 入门

这篇文章，我不打算进行入门性的介绍。
因为网络上的文章实在是很多：

最直观、最权威的介绍来自官网，它用简单的语言，说清楚了RxJava到底是什么，有什么优势：

 - [ReactiveX](http://reactivex.io/)


想直接上手使用，请看这四篇介绍文章：

1. [深入浅出RxJava（一：基础篇）](http://blog.csdn.net/lzyzsd/article/details/41833541)
2. [深入浅出RxJava(二：操作符)] (http://blog.csdn.net/lzyzsd/article/details/44094895)
3. [深入浅出RxJava三--响应式的好处](http://blog.csdn.net/lzyzsd/article/details/44891933)
4. [深入浅出RxJava四-在Android中使用响应式编程](http://blog.csdn.net/lzyzsd/article/details/45033611)


而如果你想知道，我们为什么需要一个RxJava (一步步抽象，最终发现我们需要的，就是 RxJava），请看这篇文章：

 - [《NotRxJava懒人专用指南》](http://www.devtf.cn/?p=323)


总的来说，我认为 ReactiveX 是一个能较好解决异步调用模块组合问题的库。

怎么说呢。
同步的函数可以这么组合：

``` Java
A a = getA();
B b = getBFromA(a);
C c = getCFromB(b);
```

而异步函数呢，如果使用回调，将会陷入一个叫做回调地狱的窘境中，有多囧，见图：

![Callback Hell](http://seajones.co.uk/content/images/2014/12/callback-hell.png)

而 ReactiveX 扩展了观察者模式，构建了一套利用 Observable 将异步函数进行组合的系统。

Observable 有些类似于 Android 的 Future，将异步返回值，包装到了 Observable 中。
并在这之上，提供了大量的操作符，对 Observable 进行操作，从而解决了异步函数的组合问题。

举个栗子，ReactiveX 可以这么组合函数：

``` Java
Observable<A> a = getAsyncA();
Observable<B> b = a.flatMap(va -> getAsyncB(va));
Observable<C> c = b.flatMap(vb -> getAsyncC(vb));
```

栗子中使用了 Java 8 的 lambda 函数来简化程序。

由于其返回值是连续的，我们甚至可以将其组合成链式结构：

``` Java
Observable<C> c = getAsyncA()
    .flatMap(a -> getAsyncB(a))
    .flatMap(b -> getAsyncC(b));
```

最后，一系列转换完成后，才使用一个 Observer 来订阅最终的值：

``` Java
c.subscribe(vc -> doWithC(vc));
```

可以看到，ReactiveX 实际上是利用了一个 Observable，异步返回值，来对应原来函数的直接返回值。
并利用一些操作符，对这个异步的返回值进行操作，以获得最终的结果。
这样，将复杂的嵌套结构，扁平化成了一个链式结构。
大大增加了程序的可读性。


