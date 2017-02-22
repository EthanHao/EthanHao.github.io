---
layout: post
title:  "Five different types IO in Linux"
date:   2017-02-22 10:45:20 -0600
categories: C,Linux,IO
---
### There are five different type IO model under Linux , they are respectively  
*Blocking IO  
*NonBlocking IO  
*IO Multiplexing  
*Signal-Driven  
*Asynchornours IO  
I am going to illustrate the Read function to explain these types of IO model.

### 1.Blocking IO   
![alt text](/img/IO/Blocking.png) 

### 2.Nonblocking IO  
![alt text](/img/IO/NonBlocking.png) 

### 3.IO Multiplexing (select and poll)  
![alt text](/img/IO/IOMultiplexing.png) 

### 4.Signal-Driven IO (SIGIO)  
![alt text](/img/IO/Signal-Driven.png) 
  
### 5.Asynchornours IO (the POSIX aio_functions)  
![alt text](/img/IO/Aio.png) 