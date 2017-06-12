---
layout: post
title:  "C++ Performance: Utilize the Return Value Optimiztion of Complier (RVO)"
date:   2017-06-12 10:45:20 -0600
categories: C++,RVO
---
This post is the following up of last post(using *= in place of *). In the last post, we implemented the operator* function in a very normal way(define a temporary variable),
some complier can optimize your code in this way, they call it Named Return Value Optimiztion. But some complier do not. we can not count on that. The only thing we can count on 
is the nameless return value optimiztion which is supported by almost all of the complier of C++.
```cpp

class V {
private:
	int x;
	int y;
public:
	V() :x(0), y(0) {}
	~V() { }
	V(int nx, int ny) :x(nx), y(ny) {  }	

	/* the old version
	V operator *(const V& rh) {	
		V lret;
		lret.x = x*rh.x;
		lret.y = y*rh.y;
		return lret;
	}*/
	
	//The new version which can be optimized by the complier.
	V operator *(const V& rh) {				
		return V(x*rh.x, y*rh.y);
	}
	V& operator *=(const V& rh) {
		x *= rh.x;
		y *= rh.y;
		return *this;
	}
};

int main()
{
	const int count = 10000;
	V a(10, 8);
	V b(9, 22);
	timer ltime,ltime2;
	timer::initTimer();
	ltime.tic();
	for (int i = 0; i < count; i++) {
		V c = a * b;
	}
	ltime.toc();

	ltime2.tic();
	for (int i = 0; i < count; i++) {
		V c = a;
		c *= b;
	}
	ltime2.toc();

	float f1 = ltime.timeInSeconds();
	float f2 = ltime2.timeInSeconds();
	printf("* time : update:%f ms  *= :%f ms  diff:%f\n", f1 * 1000.0f, f2 * 1000.0f, (f1 - f2) *1000.0f);

    return 0;
}

```
The Result
#### * time : update:0.989562 ms  *= :0.462220 ms  diff:0.527342  //the result of old version   
#### * time : update:0.702886 ms  *= :0.467175 ms  diff:0.235711  //the result of new version   
As you can see, we get a more than 20% percent of performance improvment by using the RVO.   
We can add the log code to keep track of the calling procedure.   
```cpp

class V {
private:
	int x;
	int y;
public:
	V():x(0),y(0){ std::cout << "V Default Constructor" << std::endl; }
	~V(){ std::cout << "V Destructor" << std::endl; }
	V(int nx, int ny) :x(nx), y(ny) { std::cout << "V Customized Constructor" << std::endl; }
	V(const V&r) :x(r.x), y(r.y) { std::cout << "V Copy Constructor" << std::endl; }
	V& operator = (const V&r) {
		std::cout << "V operator=" << std::endl;
		if (this != &r) {
			x = r.x;
			y = r.y;
		}
		return *this;
	}
	
	V operator *(const V& rh) {
		std::cout << "Operator *" << std::endl;
		return V(x*rh.x, y*rh.y);
	}
	
	V& operator *=(const V& rh) {
		std::cout << "Operator *=" << std::endl;
		x *= rh.x;
		y *= rh.y;
		return *this;
	}
};

```
The calling flow  of operator* after optimizing by the complier  
Operator *                (c = a * b)  
V Customized Constructor  (c(x*rh.x, y*rh.y) customized constructor)
V Destructor              (c destrctor)

The calling flow  of operator*= after optimizing by the complier  
V Copy Constructor      (c = a)  
Operator *=				(c *= b)  
V Destructor			(c destrctor)  

But even though we used the RVO to optimize your code, but we can see using *= operator is still faster than the new version of *operator.