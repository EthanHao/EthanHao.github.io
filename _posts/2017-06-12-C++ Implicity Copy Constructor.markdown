---
layout: post
title:  "C++ Performance: Using the Default copy constructor rather than the explicit one"
date:   2017-06-12 10:45:20 -0600
categories: C++,Copy Constructor
---
Shocking truth about the performance between default copy constructor and the one you defined. 
In this experiment ,we do it with two steps. first with the default or implicity copy constructor,
the second one we use an explicit copy constructor;

### The First Step
```cpp

class V {
private:
	int x;
	int y;
public:
	V():x(0),y(0){ }
	~V(){ }
	V(int nx, int ny) :x(nx), y(ny) {}
	V(const V&r) = default;
	//V(const V&r) :x(r.x), y(r.y) { 
	//	//std::cout << "V Copy Constructor" << std::endl; 
	//}
	V& operator = (const V&r) {		
		if (this != &r) {
			x = r.x;
			y = r.y;
		}
		return *this;
	}	
	
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


### The Second Step, Modify the class V, give it an explicit copy constructor.
```cpp

class V {
private:
	int x;
	int y;
public:
	V():x(0),y(0){ }
	~V(){ }
	V(int nx, int ny) :x(nx), y(ny) {}
	//comment the default one 
	//V(const V&r) = default;
	
	//The explicity copy constructor
	V(const V&r) :x(r.x), y(r.y) { }
	V& operator = (const V&r) {		
		if (this != &r) {
			x = r.x;
			y = r.y;
		}
		return *this;
	}	
	
	V operator *(const V& rh) {		
		return V(x*rh.x, y*rh.y);
	}
	V& operator *=(const V& rh) {	
		x *= rh.x;
		y *= rh.y;
		return *this;
	}
};



```

The Result
#### * time : update:0.720582 ms   *= :0.471422 ms  diff:0.249160 //the result of old version    
#### * time : update:0.723414 ms   *= :0.700763 ms diff:0.022651 //the result of new version     
This time we need compare the *= time, because in *= operator function the copy constructor would be called.  
you can see the result, the default on is 30% faster. I am very curious about that truth. so I just dig into the disassembly code.   

###The disassembly code with implicit or default copy constructor
```cpp
V c = a;
009A5D6B  mov         eax,dword ptr [a]  
009A5D6E  mov         dword ptr [ebp-0C0h],eax  
009A5D74  mov         ecx,dword ptr [ebp-24h]  
009A5D77  mov         dword ptr [ebp-0BCh],ecx  
009A5D7D  mov         byte ptr [ebp-4],2  
```

###The disassembly code with explicit copy constructor 
```cpp
		V c = a;
009A1C3B  lea         eax,[a]  
009A1C3E  push        eax  
009A1C3F  lea         ecx,[ebp-0C0h]  
009A1C45  call        V::V (09A13F7h)  
009A1C4A  mov         byte ptr [ebp-4],2  
```

you can see the disassembly code with implicit or default copy constructor, they just move member variables one by one, do not to call the default copy constructor.
that means We do not need to push something to the stack ,and call some functions. 
