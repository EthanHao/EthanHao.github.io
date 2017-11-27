---
layout: post
title:  "C++ Performance: Prefer post-increment to pre-increment "
date:   2017-11-27 9:45:20 -0600
categories: C++,Performance
---
In this post, I will try to figure out what is the perfomance difference between pre-increment and post-increment.  

By the way this is an experiment, so I did not care about the memory leaking.

#### Terms  

```cpp
TEST_F(ServerTest, TestIteratorPostInteger){
  
  int n = 1000000;  
  auto t1 = std::chrono::high_resolution_clock::now();
  for(int i = 0; i < n; i++)
  {
    
  }
  auto t2 = std::chrono::high_resolution_clock::now();
  int64_t llong = std::chrono::duration_cast<std::chrono::microseconds>(t2 - t1).count() ;
  std::cout << "Delta t2-t1: " 
              << llong << " microseconds"<< std::endl;
}
TEST_F(ServerTest, TestIteratorPreInteger){
  
  int n = 1000000;  
  auto t1 = std::chrono::high_resolution_clock::now();
  for(int i = 0; i < n; ++i)
  {
    
  }
  auto t2 = std::chrono::high_resolution_clock::now();
  int64_t llong = std::chrono::duration_cast<std::chrono::microseconds>(t2 - t1).count() ;
  std::cout << "Delta t2-t1: " 
              << llong << " microseconds"<< std::endl;
}
```
#### The result of these two test cases.

1: [ RUN      ] ServerTest.TestIteratorPostInteger  
1: Delta t2-t1: 4720 microseconds  
1: [       OK ] ServerTest.TestIteratorPostInteger (5 ms)  
1: [ RUN      ] ServerTest.TestIteratorPreInteger  
1: Delta t2-t1: 3910 microseconds  
1: [       OK ] ServerTest.TestIteratorPreInteger (4 ms)  

you can compare these two result, Pre-increment is obviously faster than post-increment in total.

#### Reason  
The reason is the way to implement these two incrment. post-increment need to return the old-value, so it will more complex than pre-increment. let's see a
canonical way of implementing the post-increment
```cpp
T T::operator++(int)() {  
    auto old = *this; // remember our original value  
    ++*this;          // always implement postincr in terms of preincr  
    return old;       // return our original value  
}
```

you can see more detail from [Herb Sutter's website](https://herbsutter.com/2013/05/13/gotw-2-solution-temporary-objects/).


