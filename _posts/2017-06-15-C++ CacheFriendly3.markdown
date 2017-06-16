---
layout: post
title:  "C++ Performance: Cache Friendly(3) Using row first when iterating 2D array "
date:   2017-06-16 12:45:20 -0600
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

LLC Miss Count:The LLC (last-level cache) is the last, and longest-latency, level in the memory hierarchy before main memory (DRAM). Any memory requests missing here must be serviced by local or remote DRAM, with significant latency. The LLC Miss Count metric shows total number of demand loads which missed LLC. Misses due to HW prefetcher are not included.  

```cpp

const int rowsize = 10000;
const int colsize = 10000;
int global[rowsize][colsize];

void InitRowMajor() {
	for (int i = 0; i < rowsize; i++) {
		for (int j = 0; j < colsize; j++) {
			global[i][j] = i + j;
		}
	}
}
void InitColMajor() {
	for (int i = 0; i < colsize; i++) {
		for (int j = 0; j < rowsize; j++) {
			global[j][i] = i + j;
		}
	}
}
int main()
{

	

	timer ltime,ltime2;
	timer::initTimer();
	ltime.tic();
	//InitRowMajor();
	InitColMajor();
	ltime.toc();

	

	float f1 = ltime.timeInSeconds();
	printf("* time : %f ms   \n", f1 * 1000.0f);

    return 0;
}
```
#### The result of Vtune: based on row major

CPU Time:**0.335s**  
Memory Bound:17.0%of Pipeline Slots  
L1 Bound:**8.7%**of Clockticks  
L2 Bound:**0.0%**of Clockticks  
L3 Bound:**0.0%**of Clockticks  
DRAM Bound:10.5%of Clockticks  
Loads:613,518,405  
Stores:220,803,312  
LLC Miss Count:**450,027**  
Average Latency (cycles):24  

#### The result of Vtune:  based on column major  
lapsed Time:**1.440s**  
CPU Time:**1.364s**  
Memory Bound:43.7%of Pipeline Slots  
L1 Bound:**0.0%**of Clockticks  
L2 Bound:**9.4%**of Clockticks  
L3 Bound:**2.5%**of Clockticks  
DRAM Bound:14.7%of Clockticks  
Loads:678,620,358  
Stores:242,403,636  
LLC Miss Count:**750,045**  
Average Latency (cycles):26    

you can compare these two result, row major iterating is almost 5 times faster than col major iterating in total.