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


### No optimization  

As we all knew, the compiler will do some great job to improve the performance and reduce the size. But sometimes some optimization we want to avoid. 
{% highlight cpp %}  
>int add(int na) { 
	return na + 1; 
}

int main()
{
	int a = 1;
	int b ;
	a = add(a);
	b = a;
	return b;
#### 00C51546  push        2
00C51548  pop         eax  
}
{% endhighlight  %}

the result looks creazy, the compiler just generate one line assemble code for use , and this line get nothing about the variable a and b.
what if we declare the variable a and b as a volatile variable.
 
{% highlight cpp %} 
int main()
{
0086150C  push        ebp  
0086150D  mov         ebp,esp  
0086150F  push        ecx  
	volatile int a = 1;
00861510  mov         dword ptr [ebp-4],1  
	volatile int b ;
	a = add(a);
00861517  mov         eax,dword ptr [a]  
0086151A  inc         eax  
0086151B  mov         dword ptr [a],eax  
	b = a;
0086151E  mov         eax,dword ptr [a]  
00861521  mov         dword ptr [a],eax  
	return b;
00861524  mov         eax,dword ptr [b]  
}
{% endhighlight  %}

### Execution Order
Sometimes the assemble code excution order is not the same as the order you c++ code. that's is a big problem for muitithreaded programming. Assume we gotta share two variables between
two threads. one thread do things like the following piece of code.
{% highlight cpp %} 
a = 1;
b = 1
{% endhighlight  %}
and another thread do thing like this 
{% highlight cpp %} 
if(b == 1)
{
	assert(a == 1);
}
{% endhighlight  %}
that could be a risk if you turn on code optimization. sometimes the compiler will generate different order of code. even though you declare b as a volatile variable.


