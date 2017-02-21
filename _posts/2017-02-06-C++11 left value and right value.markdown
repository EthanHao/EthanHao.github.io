---
layout: post
title:  "The definition of l-value and r-value in C++"
date:   2017-02-01 10:45:20 -0600
categories: C++,C++11
---
Accroding to the MSDN website, the definition of l-value like below:
>Expressions that refer to memory locations are called "l-value" expressions.
 An l-value represents a storage region's "locator" value, or a "left" value, implying that it can appear on the left of the equal sign (=). L-values are often identifiers.

####Any of the following C expressions can be l-value expressions:  
An identifier of integral, floating, pointer, structure, or union type  
A subscript ([ ]) expression that does not evaluate to an array  
A member-selection expression (–> or .)  
A unary-indirection (*) expression that does not refer to an array  
An l-value expression in parentheses  
A const object (a nonmodifiable l-value)  

By contrast to l-value, r-value refer to the expression you can not get the memory-locations. Because of not being able to get the locations, so you cannot
put the r-value at the left of equal sign(=). 