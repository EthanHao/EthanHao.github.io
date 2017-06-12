---
layout: post
title:  "C++ Performance: Using C++ Proxy Object to remove temporaries"
date:   2017-06-12 10:45:20 -0600
categories: C++,Proxy Object, Type Conversion Operator Overloading
---
Sometimes we want to write our code in a normal way ,like multiple number multiplication. Usually we write it like this f = a*b*c*d*e;
But in this way the complier will generate a lot of hidder temporaries for you.  How to remove this temporaries but not giving up the old way,
and at the same time to improve the performance. This way is crazy , but it is worth to do this sometimes.
In this blod, I am going to show the difference of performance using two different way.

#### Using old way, no proxy object  
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
	V& operator = (const V&r) = default;
		
	V operator *(const V& rh) {		
		return V(x*rh.x, y*rh.y);
	}
	V& operator *=(const V& rh) {	
		x *= rh.x;
		y *= rh.y;
		return *this;
	}
	int GetX() { return x; }
};


int main()
{
	const int count = 100000;
	V a(10, 8);
	V b(9, 22);
	V c(91, 22);
	V d(22, 222);
	V e(21, 2);
	timer ltime,ltime2;
	timer::initTimer();
	ltime.tic();
	V f;
	for (int i = 0; i < count; i++) {
		f = a * b * c * d *e;
		
	}
	ltime.toc();


	float f1 = ltime.timeInSeconds();
	printf("* time : %f ms  %d \n", f1 * 1000.0f, f.GetX());

    return 0;
}

```

#### Using proxy object, Keep the caller not changed.  
```cpp

class V {
private:
	int x;
	int y;
	friend class VMV;
	friend class VMVMV;
	friend class VMVMVMV;
	friend class VMVMVMVMV;
public:
	V():x(0),y(0){ }
	~V(){ }
	V(int nx, int ny) :x(nx), y(ny) {}
	V(const V&r) = default;
	V& operator = (const V&r) = default;

	V& operator *=(const V& rh) {	
		x *= rh.x;
		y *= rh.y;
		return *this;
	}
	int GetX() { return x; }
};
class  VMV
{
public:
	const V& m1;
	const V& m2;
public:
	VMV(const V& n1,const V& n2):m1(n1),m2(n2){}
	operator V() { return V(m1.x*m2.x, m1.y*m2.y); }
	friend VMV operator * (const V& n1, const V&n2);
};
inline VMV operator * (const V& n1, const V&n2) {
	return VMV(n1, n2);
}

class  VMVMV
{
public:
	const V& m1;
	const V& m2;
	const V& m3;
public:
	VMVMV(const VMV & n1, const V& n2) :m1(n1.m1), m2(n1.m2),m3(n2) {}
	operator V() { return V(m1.x*m2.x*m3.x, m1.y*m2.y* m3.y); }
	friend VMVMV operator * (const VMV& n1, const V&n2);
};
inline VMVMV operator * (const VMV& n1, const V&n2) {
	return VMVMV(n1, n2);
}
class  VMVMVMV
{
public:
	const V& m1;
	const V& m2;
	const V& m3;
	const V& m4;
public:
	VMVMVMV(const VMVMV & n1, const V& n2) :m1(n1.m1), m2(n1.m2), m3(n1.m3),m4(n2) {}
	operator V() { return V(m1.x*m2.x*m3.x*m4.x, m1.y*m2.y* m3.y*m4.y); }
	friend VMVMVMV operator * (const VMVMV& n1, const V&n2);
};
inline VMVMVMV operator * (const VMVMV& n1, const V&n2) {
	return VMVMVMV(n1, n2);
}

class  VMVMVMVMV
{
public:
	const V& m1;
	const V& m2;
	const V& m3;
	const V& m4;
	const V& m5;
public:
	VMVMVMVMV(const VMVMVMV & n1, const V& n2) :m1(n1.m1), m2(n1.m2), m3(n1.m3), m4(n1.m4),m5(n2) {}
	operator V() { return V(m1.x*m2.x*m3.x*m4.x*m5.x, m1.y*m2.y* m3.y*m4.y*m5.y); }
	friend VMVMVMVMV operator * (const VMVMVMV& n1, const V&n2);
};
inline VMVMVMVMV operator * (const VMVMVMV& n1, const V&n2) {
	return VMVMVMVMV(n1, n2);
}


int main()
{
	const int count = 100000;
	V a(10, 8);
	V b(9, 22);
	V c(91, 22);
	V d(22, 222);
	V e(21, 2);
	timer ltime,ltime2;
	timer::initTimer();
	ltime.tic();
	V f;
	for (int i = 0; i < count; i++) {
		f = a * b * c * d *e;
		
	}
	ltime.toc();


	float f1 = ltime.timeInSeconds();
	printf("* time : %f ms  %d \n", f1 * 1000.0f, f.GetX());

    return 0;
}

```

#### Performance in release mode
* time : 1.195190 ms  3783780  (first way with no proxy)
* time : 0.506106 ms  3783780  (second way with proxy)

As we can see, we will get a huge improvement in second way.
