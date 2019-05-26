---
layout: post  
title:  "Retrofit with Rxjava : Schedulers.newThread() vs Schedulers.io()"  
date: 2018-11-17  
description: "在使用Retrofit网络请求时使用 Schedulers.newThread() 和 Schedulers.io() 各有什么好处？"
tag: Android开发
---

> 翻译自 [Retrofit with Rxjava Schedulers.newThread() vs Schedulers.io()](https://stackoverflow.com/questions/33415881/retrofit-with-rxjava-schedulers-newthread-vs-schedulers-io) stackoverflow讨论版

**提问：**
在使用Retrofit网络请求时使用 Schedulers.newThread() 和 Schedulers.io() 各有什么好处？我在许多的项目中见到的是使用 Schedulers.io() ，我想知道为什么呢？
像是这样的例子：
```java
observable.onErrorResumeNext(refreshTokenAndRetry(observable))
    .subscribeOn(Schedulers.newThread())
    .observeOn(AndroidSchedulers.mainThread())...
```
VS
```java
observable.onErrorResumeNext(refreshTokenAndRetry(observable))
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())...
```
似乎有看过这样的一个原因：
newThread() 会为每个实例创建一个新线程，而 io() 使用的是线程池来管理
我想知道这个原因具体对我们的应用程序有怎样的影响，其他方面还有什么原因呢？

**回答：**
这个原因是正确的，确实 Schedulers.io() 使用的好处在于它是使用线程池，而Schedulers.newThread()不是。
你应该考虑使用线程池的主要原因，是它们维护了许多空闲且等待工作的预创建线??程。这意味着当你完成工作时，你不需要经历创建线程的开销。完成工作后，该线程也可以重新用于之后的工作，而不是不断创建和销毁线程。

线程创建起来可能很昂贵，因此最大限度地减少动态创建的线程数会更好一些。

有关线程池的更多信息，我建议：
- [Java中的线程池有什么用？](https://stackoverflow.com/questions/3286626/what-is-the-use-of-a-thread-pool-in-java)
- [什么是线程池？](https://softwareengineering.stackexchange.com/questions/173575/what-is-a-thread-pool)
- [线程池模式（维基百科）](https://en.wikipedia.org/wiki/Thread_pool_pattern)

另外，如果你有很多并发工作要使用 Schedulers.io() ，那么你可能会遇到OS i / o限制（比如说，打开文件的最大数量，tcp连接的最大数量，为了保证可靠性，即使在处置后也可能保持打开一段时间） 。每个新线程还需要一个最小的但不可忽视的RAM（> 512K，但工作在1M），因此你可能会用完RAM。