---
layout: post
title:  "Memory Leak tracker in C++"
date:   2017-03-06 10:45:20 -0600
categories: C++,Memory Leak,
---
Sometimes if you don't get any memory leak detecting tool, it's hard to find which memory leaked. even though you get the address of leaking memory, it's difficult to
map this address to a specific class or specific code line where you allocated this memory.
Usually we can hook the operator new to track the memory. so based on this theory , we can develop some very simple class to help us to track the memory.
So I just implemented a helper class to track the memory. please see the code from [here](https://github.com/EthanHao/MemLeakTracker.git)


# Add header file
include "MemHeader.h"

# Before exiting, dump the unfreed memory on heap  
 MemoryLeaking::MemoryLinkList::Dump();
 
# Result  
memory leaking at  Address 0x1bc05b0 Size 8 File welcome.cc Line 53  
memory leaking at  Address 0x1bc04f0 Size 128 File  Line 0  
memory leaking at  Address 0x1bc04b0 Size 8 File welcome.cc Line 53  
memory leaking at  Address 0x1bc0470 Size 8 File welcome.cc Line 53  
memory leaking at  Address 0x1bc0430 Size 8 File welcome.cc Line 53  
memory leaking at  Address 0x1bc03f0 Size 8 File welcome.cc Line 53  
memory leaking at  Address 0x1bc0330 Size 8 File welcome.cc Line 53  
memory leaking at  Address 0x1bc02f0 Size 8 File welcome.cc Line 53  
memory leaking at  Address 0x1bc0250 Size 8 File welcome.cc Line 53  
memory leaking at  Address 0x1bc01c0 Size 8 File welcome.cc Line 53  
memory leaking at  Address 0x1bc0140 Size 8 File welcome.cc Line 53  
memory leaking at  Address 0x1bc0060 Size 4 File welcome.cc Line 46  


# Priciple

Prepend a memory node struct including information about file ,line , size, previous node and nexe node. So we used doubly linked list
to keep track of all the memory we allocated .  

* Allocate, when you allocate a memory, just insert this node to the head of linked list  
* Free, when you free, firstly remove this node from the linked list, then free the real memory.   

# Hook  
* Override the operator new ,new [], delete and delete[]  

```cpp
void *operator new(size_t size) throw (std::bad_alloc&){
    //std::cout << "new" << std::endl;
    void *p = MemoryLeaking::MemoryLinkList::Allocte(size);
    if (p == nullptr) throw std::bad_alloc();
    return p;
}
void operator delete(void *p) {
    MemoryLeaking::MemoryLinkList::free(p);
}
void *operator new[](size_t size) throw (std::bad_alloc&) {
    //std::cout << "new[]" << std::endl;
    void *p = MemoryLeaking::MemoryLinkList::Allocte(size);
    if (p == nullptr) throw std::bad_alloc();
    return p;
}
void operator delete[](void *p) {
    MemoryLeaking::MemoryLinkList::free(p);
}
```
* Define new operator  

```cpp
define new HeapStamp(__FILE__, __LINE__) + new

//define HeapStamp class and overload operator+


class HeapStamp {
public:

    HeapStamp(const char* nsFile, int nLine) : msPath(nsFile), mLine(nLine) {
    }
public:
    const char * const msPath;
    const int mLine;
};

template <class T> inline T *operator+(const HeapStamp &stamp, T *p) {
 
    std::cout << "Allocated a block of memory of " << typeid (T).name() <<
            " Address " << p << " Size " << sizeof (T) << " File " << 
            stamp.msPath << " Line " << stamp.mLine << std::endl;

    MemoryLeaking::MemoryLinkList::SetFileAndLineOfNode(p, stamp.msPath, stamp.mLine);
    return p;
}
```


