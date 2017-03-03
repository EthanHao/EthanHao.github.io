---
layout: post
title:  "Reactor epoll in multithreaded enviroment "
date:   2017-03-1 10:45:20 -0600
categories: C++ 11,Linux,MemoryPool,ThreadPool,Reactor,
---
Usually the reactor pattern use one process and one thread. The main goal of Reactor pattern is to seperate the application sepecific code. But some time IO is the 
bottleneck of performance. so I try to apply the Reactor pattern in a multithreaded enviroment. The detailed code is
[**here**!](https://github.com/EthanHao/EpollServer) .


###  1. Simply introduce the Reactor pattern.   

![reactor](/img/epoll/reactor.png)

### 2. Reactor pattern with threadpool

  ![ReactorWithPool](/img/epoll/reactorWithPool.png)

### 3.The detail of worker

  ![Open](/img/epoll/ReactorMoreDetail.png)


  