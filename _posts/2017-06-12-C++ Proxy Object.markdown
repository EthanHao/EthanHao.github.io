---
layout: post
title:  "C++ Performance: Using C++ Proxy Object to avoid unnecessary caculation"
date:   2017-06-12 10:45:20 -0600
categories: C++,Proxy Object, Type Conversion Operator Overloading
---
Sometimes we can using C++ cool features to archive our goal of avoid unnecessary caculation, at the same time we don't change the caller code at all. Like the example below,
The GetLength function returns the square root of x, typically it is a very low efficient caculation.Sometimes it is not necessary to do this when do the comparsion.
so when we use this GetLength to do a comparsion, we can using a series of C++ features to avoid it, the mainly features includes
C++ Proxy object, Type Conversion Operator Overloading ,etc.  

```cpp
class Test {
private:
	int x;
public:
	Test(int nx) :x(nx) {};
	~Test() {};
	Test(const Test& rh) :
		x(rh.x) {};
	Test& operator=(const Test&rh) {
		if (this != &rh) {
			x = rh.x;
		}
		return *this;
	}

public:
	int GetLenth() {
		std::cout << "GetLength "<<  x << std::endl;
		return sqrt(x);
	}
};

int main()
{
	Test A(10);
	Test B(8);
	Test C(18);
	if (A.GetLenth() > B.GetLenth()) //this comparsion we must avoid to call sqrt
	{
		int clen = C.GetLenth(); //this one call the getLength 
		std::cout << "CLen is " << clen << std::endl;
	}
    return 0;
}

```
1, Introduce the ProxyLength class  
2, Modify the GetLength function of Test, let return a instance of ProxyLength  
3, In ProxyLength, we overload the operator >  
4, In ProxyLength, We overload the type conversion int  
the final code like below  
```cpp
class ProxyLength;
class Test {
private:
	int x;
public:
	friend class ProxyLength;

	Test(int nx) :x(nx) {};
	~Test() {};
	Test(const Test& rh) :
		x(rh.x) {};
	Test& operator=(const Test&rh) {
		if (this != &rh) {
			x = rh.x;
		}
		return *this;
	}
	ProxyLength GetLenth();

};

class ProxyLength {
private:
	const Test& mTest;
public:
	
	~ProxyLength() {}
	ProxyLength(const Test& nTest): mTest(nTest){}

	bool operator > (const ProxyLength& rh) {
		std::cout << "Operator >" << std::endl;
		return mTest.x > rh.mTest.x;
	}
	operator int() { 
		std::cout << "Operator int" << std::endl; 
		return sqrt(mTest.x); }
};


ProxyLength Test::GetLenth() {
	std::cout << "ProxyGetLength" << std::endl;
	return ProxyLength(*this);
}


int main()
{
	Test A(10);
	Test B(8);
	Test C(18);
	if (A.GetLenth() > B.GetLenth()) {
		int clen = C.GetLenth();
		std::cout << "CLen is " << clen << std::endl;
	}
    return 0;
}

```


