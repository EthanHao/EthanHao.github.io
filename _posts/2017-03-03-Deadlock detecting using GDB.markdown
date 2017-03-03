---
layout: post
title:  "DeakLock detecting using GDB "
date:   2017-03-02 10:45:20 -0600
categories: C++, GDB,  MultiThread,
---
Instruction  
I think Volatile keyword in C++ is not obvious and direct as other keyword like const,mutable etc. The main reason is we don't know how Volatile works. 
We are told this keyword usually would be used to qulify the shared value in muitithreaded envoriment. But even though we need to take a lot of care to use this keyword.
And The direct way to learn Volatile is through the assemble code.

### Prerequiste  
* g++, gdb
* C++11
* gdb has the permission to attach a process

### Write a typical code to generate deadlock

as we know, the mutex is not allow to reenter. if you reenter ,it will cause the deadlock

{% highlight cpp %}  
#include <iostream>
#include <map>
#include <string>
#include <chrono>
#include <thread>
#include <mutex>

std::mutex gMutex;

int Reenter(){
   std::lock_guard<std::mutex> lLock(gMutex);
   return 10;	
}

int Callback()
{
   std::lock_guard<std::mutex> lLock(gMutex); 
   return Reenter();
}
int main(int argc, char**argv) {
    // Prints hello message...
    std::cout << "Hello CMake World!" << std::endl;
    std::thread lThread(Callback);
    lThread.join();
    std::cout << "sub thread is gone!" << std::endl;
    return 0;
}

{% endhighlight  %}

#### compile with gdb information  
> g++ test.cpp -ggdb -lpthread -std=c++11 -o test  
#### run it  
> ./test   
#### check the result,it deadlocked as we expected  
>ethan@ubuntu:~/Desktop$ ./test
Hello CMake World!


### Analyze using gdb  
#### Get the pid  
> ethan@ubuntu:~$ ps -a | grep test
  4094 pts/2    00:00:00 test

#### Attach the pid using another terminal  
>ethan@ubuntu:~/Desktop$ sudo gdb -q test 4094
[sudo] password for ethan: 
Reading symbols from test...done.
Attaching to program: /home/ethan/Desktop/test, process 4094
[New LWP 4095]
[Thread debugging using libthread_db enabled]

#### Backtrace  
>(gdb) bt
#0  0x00007f65c7d3e9cd in pthread_join (threadid=140075107923712, 
    thread_return=0x0) at pthread_join.c:90
#1  0x00007f65c7a6cb97 in std::thread::join() ()
   from /usr/lib/x86_64-linux-gnu/libstdc++.so.6
#2  0x00000000004011ea in main (argc=1, argv=0x7fff7a26af28) at test.cpp:27

#### Info threads  
>(gdb) info threads
  Id   Target Id         Frame 
* 1    Thread 0x7f65c815a740 (LWP 4094) "test" 0x00007f65c7d3e9cd in pthread_join (threadid=140075107923712, thread_return=0x0) at pthread_join.c:90
  2    Thread 0x7f65c70cb700 (LWP 4095) "test" __lll_lock_wait ()
    at ../sysdeps/unix/sysv/linux/x86_64/lowlevellock.S:135
(gdb) thread 2

#### Info of mutex  
(gdb) p gMutex
$1 = {<std::__mutex_base> = {_M_mutex = {__data = {__lock = 2, __count = 0, 
        ***__owner = 4095,*** __nusers = 1, __kind = 0, __spins = 0, __elision = 0, 
        __list = {__prev = 0x0, __next = 0x0}}, 
      __size = "\002\000\000\000\000\000\000\000\377\017\000\000\001", '\000' <repeats 26 times>, __align = 2}}, <No data fields>}

#### Switch to the thread  
>(gdb) thread 2
[Switching to thread 2 (Thread 0x7f65c70cb700 (LWP 4095))]
#0  __lll_lock_wait () at ../sysdeps/unix/sysv/linux/x86_64/lowlevellock.S:135
135	../sysdeps/unix/sysv/linux/x86_64/lowlevellock.S: No such file or directory.

#### Check the stack of that thread
>(gdb) bt
#0  __lll_lock_wait () at ../sysdeps/unix/sysv/linux/x86_64/lowlevellock.S:135
#1  0x00007f65c7d3fdfd in __GI___pthread_mutex_lock (mutex=0x6052e0 <gMutex>)
    at ../nptl/pthread_mutex_lock.c:80
#2  0x0000000000401007 in __gthread_mutex_lock (__mutex=0x6052e0 <gMutex>)
    at /usr/include/x86_64-linux-gnu/c++/5/bits/gthr-default.h:748
#3  0x0000000000401498 in std::mutex::lock (this=0x6052e0 <gMutex>)
    at /usr/include/c++/5/mutex:135
#4  0x000000000040151e in std::lock_guard<std::mutex>::lock_guard (
    this=0x7f65c70cae20, __m=...) at /usr/include/c++/5/mutex:386
#5  0x00000000004010ef in Reenter () at test.cpp:11
#6  0x000000000040114b in Callback () at test.cpp:18
#7  0x0000000000402801 in std::_Bind_simple<int (*())()>::_M_invoke<>(std::_Index_tuple<>) (this=0x2085058) at /usr/include/c++/5/functional:1531
#8  0x000000000040275a in std::_Bind_simple<int (*())()>::operator()() (
    this=0x2085058) at /usr/include/c++/5/functional:1520
#9  0x00000000004026ea in std::thread::_Impl<std::_Bind_simple<int (*())()> >::_M_run() (this=0x2085040) at /usr/include/c++/5/thread:115
#10 0x00007f65c7a6cc80 in ?? () from /usr/lib/x86_64-linux-gnu/libstdc++.so.6
#11 0x00007f65c7d3d6fa in start_thread (arg=0x7f65c70cb700)
    at pthread_create.c:333
#12 0x00007f65c74dbb5d in clone ()
    at ../sysdeps/unix/sysv/linux/x86_64/clone.S:109

