---
layout: post
title:  "C++ Performance: Cache Friendly(2) Compact the discrete data as much as possible "
date:   2017-06-15 12:45:20 -0600
categories: C++,CPU Cache
---
In this post, I will try to use a linear searching algorithm based on linked list to show the effect to performance when data layout is different.
First experiment I will use to create a linked list in a normal way which creates each node using new keyword. Second one I will create a linked list based on
a consectutive memory space.  and the size of Node is the same.

By the way this is an experiment, so I did not care about the memory leaking.

#### Terms  
L1 Bound： This metric shows how often machine was stalled without missing the L1 data cache. The L1 cache typically has the shortest latency. However, in certain cases like loads blocked on older stores, a load might suffer a high latency even though it is being satisfied by the L1.

L2 Bound: This metric shows how often machine was stalled on L2 cache. Avoiding cache misses (L1 misses/L2 hits) will improve the latency and increase performance.

L3 Bound: This metric shows how often CPU was stalled on L3 cache, or contended with a sibling Core. Avoiding cache misses (L2 misses/L3 hits) improves the latency and increases performance.

```cpp
class Node  {
public:
	Node* next;	
	int key;
	Node * pre;		

	Node() :key(0) ,next(nullptr){}
	Node(int k): key(k),next(nullptr){}
};
Node * head = nullptr;
Node * searchKey(const int key) {
	Node * p = head;
	while (p) {
		if (p->key == key)
			return p;
		p = p->next;
	}
	return nullptr;
}
void CreateDiscrete(const int count) {

	Node * p = nullptr;
	for (int i = 0; i < count - 1; i++) {
		Node * newp = new Node(i);
		if (p == nullptr)
		{
			head = newp;
			p = head;
		}
		else {
			p->next = newp;
			p = newp;
		}
	}
	
}
int main()
{
	const int count = 10000;	
	CreateDiscrete(count);

	timer ltime,ltime2;
	timer::initTimer();
	ltime.tic();
	for (int i = 0; i < count; i++) {
		Node * p = searchKey(i);		
	}
	ltime.toc();


	float f1 = ltime.timeInSeconds();
	printf("* time : %f ms  %d \n", f1 * 1000.0f, sizeof(Node));

    return 0;
}
```
#### The result of Vtune:  the sizeof(Node) is 12  ,based on a discrete memory layout
Elapsed Time:0.417s  
CPU Time:**0.374s**  
Memory Bound:72.4%of Pipeline Slots  
L1 Bound:**34.4%**of Clockticks  
L2 Bound:**27.6%**of Clockticks  
L3 Bound:**10.9%**of Clockticks  
DRAM Bound:0.0%of Clockticks  
Loads:332,109,963  
Stores:57,600,864  


```cpp
class Node  {
public:
	Node* next;	
	int key;
	Node * pre;		

	Node() :key(0) ,next(nullptr){}
	Node(int k): key(k),next(nullptr){}
};
Node * head = nullptr;
Node * searchKey(const int key) {
	Node * p = head;
	while (p) {
		if (p->key == key)
			return p;
		p = p->next;
	}
	return nullptr;
}
void Create(const int count) {
	
	Node* pArray = (Node*)malloc(sizeof(Node)*count);
	Node * p = pArray;
	for (int i = 0; i < count-1; i++) {
		p->key = i;
		p->next = p + 1;
		p += 1;	
		p->key = i+1;
	}
	head = pArray;
}
int main()
{
	const int count = 10000;	
	Create(count);

	timer ltime,ltime2;
	timer::initTimer();
	ltime.tic();
	for (int i = 0; i < count; i++) {
		Node * p = searchKey(i);		
	}
	ltime.toc();


	float f1 = ltime.timeInSeconds();
	printf("* time : %f ms  %d \n", f1 * 1000.0f, sizeof(Node));

    return 0;
}

```
#### The result of Vtune:  this time the sizeof(Node) is 12 ,based on a consectutive memory layout 
CPU Time:0.218s
Memory Bound:52.9%of Pipeline Slots
L1 Bound:36.2%of Clockticks
L2 Bound:2.9%of Clockticks
L3 Bound:1.9%of Clockticks
DRAM Bound:0.0%of Clockticks
Loads:325,209,756
Stores:55,200,828 

you can compare these two result, we can conclude the consectutive memory layout is really helpful to remove the case miss ratio and to improve the performance hugely.
