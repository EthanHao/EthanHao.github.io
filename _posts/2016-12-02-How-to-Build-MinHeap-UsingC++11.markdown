---
layout: post
title:  "How to build a min heap with a remove function"
date:   2016-12-02 14:45:20 -0600
categories: [C++11,Algorithm]
---
As we know the C++ 11 provided us a lot of great functionalities. It is 
very convinect for us to use these functionalities to address our issues.

when I was trying to review the notations about the heap with an [online algorithm problem](https://www.hackerrank.com/challenges/qheap1);
It turns out I can use the `Priority_queue` container to build a `min heap`. 

As we know, heap doesn't provide a `remove function to delete an arbitary element`. So first mission is to construct an our own priority_queue with a remove function.
But since the elements on heap is nearly ordered, we can not use some sort of binary search function to locate the element which we are going to delete. theoritically It looks
like we can not avoid a linear time to search the element. 

After locating this element, the next step is to delete it and call `std::make_queue` to let the all of elements to be a heap again.But the result is getting some time exceeding on
some test case.

So according the requiement of this problem,the tricky part is we don't need to delete element until it is the first element.



The code looks like this:

{% highlight cpp %}
template<typename T>
class custom_priority_queue :
	public std::priority_queue<T, std::deque<T>, std::greater<T>>
{
private:
	std::set<T> mDeleteingSet;
	void clearBegin(){
		if (this->c.size() > 0) {
			mDeleteingSet.erase(*this->c.begin());
			this->c.erase(this->c.begin());
			std::make_heap(this->c.begin(), this->c.end(), this->comp);
		}

	}
public:


	bool remove(const T& value) {
		if (value == *this->c.begin()){
			clearBegin();
			while (this->c.size() > 0 &&
				mDeleteingSet.find(*this->c.begin()) != mDeleteingSet.end()){
				clearBegin();
			}
		}
		else {
			mDeleteingSet.insert(value);
		}
		return true;
	}

	void push(const T& value){
		auto lIte = mDeleteingSet.find(value);
		if (lIte != mDeleteingSet.end()){
			mDeleteingSet.erase(lIte);
		}
		else {
			std::priority_queue<T, std::deque<T>, std::greater<T>>::push(value);
		}
	}

};
{% endhighlight  %}