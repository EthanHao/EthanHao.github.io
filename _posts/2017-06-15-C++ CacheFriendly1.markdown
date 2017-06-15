---
layout: post
title:  "C++ Performance: Cache Friendly(1) Keep your structure as small as possible"
date:   2017-06-15 11:45:20 -0600
categories: C++,RVO
---
In this post, I will try to use a linear searching algorithm based on linked list to show the effect to performance when the size of the node is different.
According to the locality theory (spatial locality), the better locality is, the faster your application will be. And the samller your structure is, the higer cache hit ratio is.
And I use Vtune to measure the cache hit or miss ratio.
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
	char a[36];

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
#### The result of Vtune:  this time the sizeof(Node) is 48   
Elapsed Time:0.524s  
CPU Time: **0.479s**  
Memory Bound:79.8%of Pipeline Slots  
L1 Bound:**30.8%** of Clockticks  
L2 Bound:**14.8%** of Clockticks  
L3 Bound:**36.2%** of Clockticks  
DRAM Bound:0.0%of Clockticks  
Loads:277,208,316
Stores:47,400,711

Okay, After that I am going to change the size of Node. and then running Vtune again.
```cpp
class Node  {
public:
	Node* next;	
	int key;
	Node * pre;		

	Node() :key(0) ,next(nullptr){}
	Node(int k): key(k),next(nullptr){}
};

```
#### The result of Vtune:  this time the sizeof(Node) is 12  
Elapsed Time:0.417s  
CPU Time:**0.374s**  
Memory Bound:72.4%of Pipeline Slots  
L1 Bound:**34.4%**of Clockticks  
L2 Bound:**27.6%**of Clockticks  
L3 Bound:**10.9%**of Clockticks  
DRAM Bound:0.0%of Clockticks  
Loads:332,109,963  
Stores:57,600,864  

you can see this time the L3 Bound decreased so much. as we all know, the L3 cache is the slowest one. so that means we got less CPU time in total . 