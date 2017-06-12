---
layout: post
title:  "C++ Performance: Using *=,+=,/=,-= rather than *,+,/,- to remove temporaries"
date:   2017-06-12 15:45:20 -0600
date:   2017-06-12 15:45:20 -0600
categories: C++,Remove temporaries
---
Suppose we defined a Class named V which overloaded the operator *,*=, then which way is faster if you want to do a multiplication of V.  
first way V c = a * b;   
second way V c = a;  c *= b;  
Obviously the result is the same, but the thing we are curious is the performance. Let us do a experiment.  
```cpp

class V {
private:
	int x;
	int y;
public:
	V() :x(0), y(0) {}
	~V() { }
	V(int nx, int ny) :x(nx), y(ny) {  }	

	V operator *(const V& rh) {	
		V lret;
		lret.x = x*rh.x;
		lret.y = y*rh.y;
		return lret;
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
#### * time : update:0.989562 ms  *= :0.462220 ms  diff:0.527342

As you can see, the result shows  *= is twice faster than *;  
Let look the function deeply to find out the reason. let change the code to trace the calling procedure.

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
		V lret;
		lret.x = x*rh.x;
		lret.y = y*rh.y;
		return lret;
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
Operator *             (c = a * b)  
V Default Constructor  (lret)  
V Copy Constructor     (copy lret to c)  
V Destructor		   (lret destrctor)  
V Destructor		   (c destructor)  

The calling flow  of operator*= after optimizing by the complier  
V Copy Constructor      (c = a)  
Operator *=				(c *= b)  
V Destructor			(c destrctor)  

as we can see from the comparsion, we save one destrctor and one default constructor in *= operator;
so that is why we get the best performance from *= rather than * .
