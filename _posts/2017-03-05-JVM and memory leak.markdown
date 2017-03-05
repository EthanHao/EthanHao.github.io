---
layout: post
title:  "JVM and memory leak"
date:   2017-03-03 10:45:20 -0600
categories: Java,Memory Leak,Performance,
---

What is memory leaking in Java? and what kind of result memory leaking can bring? The first result I can come up with is lowerring performance.
Let's dig into it from the memory model of JVM 

### Mode of JVM  
Speaking of the memory model , we could use some tool like VisualJVM and VisualGC to observer the inside of JVM. 

 ![alt text](/img/JVM/pool.png) 
 
Actually we can begin from the memory pool, from the above picture we can see there are six types of memory pool.  
•	Code cache (NON_HEAP)  
•	Compressed Class Space(NON_HEAP)  
•	Meta Space(NON_HEAP)  
•	PS Eden Space (HEAP)  
•	PS Old Gen (HEAP)  
•	Ps Survivor Space (HEAP)  
As we can see, the first three pools is NON_HEAP type.  So the memory management is totally different from the last three.   
Moreover, let’s look at the memory manager. there are four different memory management way.  
•	CodeCacheManager  
•	MetaSpace Manager   
•	PS MarkSweep (Collector)  
•	PS Scavenge (Collector)  

The first two just apply to NON_HEAP memory pool. So we will just take care of the last two, PS MarkSweep and PS Scavenge.  And through the detailed attributes of these two collectors, we can find out the relationship between the collector and the memory pool.

•	PS MarkSweep (Collector)    
Applied to PS Eden Space  
Applied to PS Old Gen  
Applied to PS Survivor  
•	PS Scavenge (Collector)  
Applied to PS Eden Space  
Applied to PS Survivor  

And conventionally, we call the PS Eden Space and PS Survivor young generation.  And PS Old Gen as old generation. We can see the PS Scavenge collector only take care of young generation. PS MarkSweep need to take care of all of them.  We can use VisualGC to trace the change of these memory pools dynamically during running of applications. Like the following picture, we got one Eden ,two Survivors and one Old.
 
 ![alt text](/img/JVM/gc.png) 

 PS Scavenge is a parallel copy collector, his job is copying the objects in PS Eden Space to Ps Survivor or between two PS Survivors in multithreaded environment. And if it finds some objects have been living so long, it will copy these objects to the old generation pool.

PS MarkSweep is a parallel scavenge mark-sweep collector, it will check the mark of each block of memory and to decide if we can collect it.

Let’s back to our problem, what do these collection numbers mean? As we can see, the PS scavenge collector was triggered 14 times during the lifetime of this application. The triggered reason is allocation failure. That means if you want to new object on heap, but JVM found there is no enough free space for this object on PS Eden Space, so in this condition, JVM will call PS scavenge to collect the free memory in the Eden. So PS scavenge will do his job, free some objects which no one would use it again, copy the survivor objects to the PS Survivors, and copy those objects who have been living so long.

### How Garbage collection impact performance 


Speaking of the overall impact on the throughput performance, we should know one thing, are the application threads still running 
when JVM doing the garbage collection, or just the application threads all pause until the garbage collection getting done?  
As I know, all the application threads would be suspended during garbage collection, so that means it would definitely affect the throughput 
performance of the application, in this example according to the output for the 192ms of life time this application just doing nothing
 but garbage collection. In contrast to some languages not using virtual machine and not using garbage collection, like C++, we should 
 free the memory manually, but the upside of freeing memory manually is there are only effective objects in the memory, all the ineffective 
 objects would be freed in time. So If we use C++ to implement this application, we can save the garbage collection time to do some business stuff, 
 the throughput performance of C++ version will be better than the Java version.

 Back to JVM, they got two ways to trigger garbage collection, one is periodical, the other one is waiting until getting failure to allocate new object.
 If we use first way, that means we get garbage collection more frequently, also means all the threads in application will be suspended 
 frequently in a short time, that will cause increasing context switch and lower the performance. 
 If we use the second way, that means we got a lot of objects to handle, so we need to take more time to collect these objects, 
 and the threads in application will be suspended longer than first way. It’s hard to make a balance.

 But Java is improving his virtual machine, comparing to JVM1.1,  It splits the memory pool into young
 generation and old generation since JVM1.2, and this is a balance of frequency and duration. 

### Memory leaking increase garbage collection

These information means memory leaking existing in this application. What’s memory leaking in Java? 
 Let’s go through the definition of memory leaking.
Definition of memory leaking in Java: objects are no longer being used by the application,
 but Garbage Collector cannot remove them because they are being referenced.
For the report we can see some of leaking classes are collection class like Array, HashMap and TreeMap; 
some of them are very basic classes like Object,Class and String; and two of them are about Reflect.
How could this happen? Especially we use garbage collector, the garbage collector was supposed to collect everything. 
But unfortunately, the truth is cold. Even the garbage collector is very smart, we also got some memory leaking.

Let’s start to dig into it from a simple concept- reference. Actually in code level, there are a few types of reference. 
For simplicity, we just separate into two types of reference, strong reference and weak reference. 
Strong reference means these two object has the same lifetime, otherwise it is a weak reference. 
 Strong reference is simple, we get no chance to get memory leaking with the same lifetime. 
 But weak reference is the problem. Imagine if you get a vector, 
 each element in this vector points to another object, after these objects get a very short lifetime, 
 you will no longer use it, and you anticipate the garbage collector would free memory of these objects
 for you. But unfortunately, the vector get the weak reference to these objects, 
 so garbage collector thinks these objects are alive and not to free them.  If these objects occupy a lot of memory, 
 you can image the JVM will trigger a lot of garbage collection because of out of memory. This would cause the performance of your application so bad.
 
Another situation is you got two objects and they are referring to each other like doubly linked list. 
The garbage collection also cannot free the memory immediately for you.
As we can see, the memory leaking is not just about the leaking, it is also about the performance. 
So when we design and code our application, we should take a lot of care about avoiding memory leaking. 

 The following are some hand-on quick tip to avoid memory leaking.
 
•	Pay attention to collection class like HashMap, Array, etc. and do you best to shorten their lifetime. 
 And try not to define these collection object as static, because static object get a lifetime as long as the whole process.

•	When you use a custom object as the key of map, don’t forget to implement the Equal() and Hashcode() function to make this object immutable.

•	Pay attention to event listener and register, you must unregister it when his lifetime is over.

•	Don’t forget to set null to a member variable which point to another object.

•	Always close connection or file stream in finally block 

•	Don’t mix primitive type with wrapper class when passing parameter, Autobox create object every time.


