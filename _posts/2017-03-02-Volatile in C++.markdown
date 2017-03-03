---
layout: post
title:  "Volatile Keyword in　C++ "
date:   2017-03-02 10:45:20 -0600
categories: C++, Volatile, MultiThread,
---
Instruction  
I think Volatile keyword in C++ is not obvious and direct as other keyword like const,mutable etc. The main reason is we don't know how Volatile works. 
We are told this keyword usually would be used to qulify the shared value in muitithreaded envoriment. But even though we need to take a lot of care to use this keyword.
And The direct way to learn Volatile is through the assemble code.

### Prerequiste  
* I use VS2015 to do this expirement  
* Turn on the optimization  

### Avoid from assigning register to the variable 
* Normal variable without Volatile
{% highlight cpp %}  
>int add(int na) { 
	return na + 1; 
}
int a = 1;
int b = 0;
int main()
{
	a = add(a);
	b = a;
	return b;
}

>a = add(a);
00901850  mov         eax,dword ptr [a (090B008h)]  
00901855  inc         eax  
00901856  mov         dword ptr [a (090B008h)],eax  
	b = a;
#### 0090185B  mov         dword ptr [b (090B148h)],eax  
{% endhighlight  %}

We can see the compiler just do some optimization to move the eax to b, not from the real memory. Assume before b=a, a will be changed in another thread, but b can not get updated value.
so this is the problem, that's not what we want. So volatile can help us out. (by the way, if you application only get one thread, forget about the volatile).

{% highlight cpp %}  
>int add(int na) { 
	return na + 1; 
}
volatile int a = 1;
int b = 0;
int main()
{
	a = add(a);
	b = a;
	return b;
}
a = add(a);
00F51530  mov         eax,dword ptr [a (0F57000h)]  
00F51535  inc         eax  
00F51536  mov         dword ptr [a (0F57000h)],eax  
	b = a;
#### 00F5153B  mov         eax,dword ptr [a (0F57000h)]  
#### 00F51540  mov         dword ptr [b (0F57128h)],eax  
	return b;
{% endhighlight  %}

We can see after a was declared as a volatile variable, the compiler will not force to get the a value from the memory.
