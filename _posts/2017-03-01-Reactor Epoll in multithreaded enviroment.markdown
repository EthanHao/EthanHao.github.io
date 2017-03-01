---
layout: post
title:  "Reactor epoll in multithreaded enviroment "
date:   2017-03-1 10:45:20 -0600
categories: C++ 11,Linux,MemoryPool,ThreadPool,Reactor,
---
Instruction  
Usually the reactor pattern use one process and one thread. The main goal of Reactor pattern is to seperate the application sepecific code. 


###  1. Simply introduce the Reactor pattern.   

![reactor](../img/reactor.png)

### 2. Reactor pattern with threadpool

  ![Open Device](./img/reactorWithPool.png)

### 3.The detail of worker

  ![Open](./img/ReactorMoreDetail.png)


  