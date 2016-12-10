---
layout: post
title:  "Expirence the smart point of share_ptr of C++"
date:   2016-12-10 10:45:20 -0600
categories: C++ SmartPointer
---
As we know, since c++ 11 the standard library serve a few smart pointer class for us. it is really convinent.
I try to use a share_ptr class to implement some my project to share the ownership of a object. since I don't use
share_ptr before, I decide to do some expriment about this sort of smart pointer.

I try to distinguish the difference between passing value and passing reference, and use a member share_ptr of class to share the 
ownership of a raw pointer; It's kind of cool, it is exactly what I expected.

here comes the code.
{% highlight cpp %}
#include "stdafx.h"
#include <iostream>
#include <memory>
class B
{
public:
	B(){
		std::cout << "b constructor" << std::endl;
	}
	~B() {
		std::cout << "b destructor" << std::endl;
	}
};

class C
{
	std::shared_ptr<B> mpB;
public:
	C(std::shared_ptr<B> & nb) :
		mpB(nb)
	{
		std::cout << "nb use count " << nb.use_count() << std::endl;
		std::cout << "C constructor" << std::endl;
		std::cout <<  "mpB use count "<< mpB.use_count() << std::endl;
	}
	~C()
	{
		std::cout << "C destructor" << std::endl;
	}
	void use_value(std::shared_ptr<B>  nb)
	{
		std::cout << "mpB use use count in use_value " << mpB.use_count() << std::endl;
		std::cout << "nb use use count in use_value " << nb.use_count() << std::endl;
	}
	void use_ref(const std::shared_ptr<B>&  nb)
	{
		std::cout << "mpB use use count in use_ref " << mpB.use_count() << std::endl;
		std::cout << "nb use use count in use_ref " << nb.use_count() << std::endl;
	}
};
int main()
{
	std::shared_ptr<B> lpB(new B());
	std::unique_ptr<C> lpC(new C(lpB));
	std::unique_ptr<C> lpC2(new C(lpB));
	lpC->use_value(lpB);
	lpC2->use_ref(lpB);
	std::cout <<"lpB use count " << lpB.use_count() << std::endl;
	
    return 0;
}
{% endhighlight  %}

here comes the result:
{% highlight cpp %}
b constructor  
nb use count 2  
C constructor  
mpB use count 2  
nb use count 3  
C constructor  
mpB use count 3  
lpB use count 3
lpB use count 3
C destructor
C destructor
b destructor
Press any key to continue . . .
{% endhighlight  %}
