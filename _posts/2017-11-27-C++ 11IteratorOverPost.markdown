---
layout: post
title:  "C++ Performance: Prefer C++11 auto and for loop to old-fasion iterator loop  "
date:   2017-11-27 9:45:20 -0600
categories: C++,Performance
---
In this post, I will try to figure out what is the perfomance difference between auto-for loop introduced since C++11 and old-fasion iterator.  

By the way this is an experiment, so I did not care about the memory leaking.

#### Terms  

```cpp
TEST_F(ServerTest, TestIteratorPost){
  std::map<int,int> lmap;
  int n = 100000;
  for(int i = 0; i < n; i++)
    lmap[i]=i;
  auto t1 = std::chrono::high_resolution_clock::now();
  for(int i = 0; i < 10; i++)
  {
    for(auto lite = lmap.begin(); lite !=lmap.end(); lite++){
      lite->second +=1;
    }
  }
  auto t2 = std::chrono::high_resolution_clock::now();
  int64_t llong = std::chrono::duration_cast<std::chrono::microseconds>(t2 - t1).count() ;
  std::cout << "Delta t2-t1: " 
              << llong << " microseconds"<< std::endl;
}
TEST_F(ServerTest, TestIteratorPre){
  std::map<int,int> lmap;
  int n = 100000;
  for(int i = 0; i < n; i++)
    lmap[i]=i;
  auto t1 = std::chrono::high_resolution_clock::now();
  for(int i = 0; i < 10; i++)
  {
    for(auto lite = lmap.begin(); lite !=lmap.end(); ++lite){
      lite->second +=1;
    }
  }
  auto t2 = std::chrono::high_resolution_clock::now();
  int64_t llong = std::chrono::duration_cast<std::chrono::microseconds>(t2 - t1).count() ;
  std::cout << "Delta t2-t1: " 
              << llong << " microseconds"<< std::endl;
}
TEST_F(ServerTest, Test11Iterator){
  std::map<int,int> lmap;
  int n = 100000;
  for(int i = 0; i < n; i++)
    lmap[i]=i;
  auto t1 = std::chrono::high_resolution_clock::now();
  for(int i = 0; i < 10; i++)
  {
    for(auto& val : lmap){
        val.second +=1;
    }
  }
  auto t2 = std::chrono::high_resolution_clock::now();
  int64_t llong = std::chrono::duration_cast<std::chrono::microseconds>(t2 - t1).count() ;
  std::cout << "Delta t2-t1: " 
              << llong << " microseconds"<< std::endl;
}
```
#### The result of these two test cases.


1: [ RUN      ] ServerTest.TestIteratorPost  
1: Delta t2-t1: 34660 microseconds  
1: [       OK ] ServerTest.TestIteratorPost (137 ms)  
1: [ RUN      ] ServerTest.TestIteratorPre  
1: Delta t2-t1: 33279 microseconds  
1: [       OK ] ServerTest.TestIteratorPre (133 ms)  
1: [ RUN      ] ServerTest.Test11Iterator  
1: Delta t2-t1: 19951 microseconds  
1: [       OK ] ServerTest.Test11Iterator (119 ms)  


you can compare these result, auto-for loop > pre-iterator > post-iterator.
so highly recommend , using auto-for loop as much as possible. it is way faster than old way.



