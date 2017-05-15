---
layout: post
title:  "Four different ways using SSE intrinsics to implemented Matrix * Vector"
date:   2017-05-15 10:45:20 -0600
categories: C++,Performance,SSE
---
SSE Intrinsic let the CPU caculate four float numbers or two double numbers in the same time, it is very useful to improve the performance like Image Processing , Matrix caculation,etc.
But even you use SSE, there is many different ways to archieve your goal, and each way with different performance. so this chapter I will show some ways to archieve Matrix*Vector function, which 
is a very typical and common operation you would run into so often.


# Prerequisite
Vect4D_SIMD is a vector class reprensent a vector inclulding x,y,z,w four float numbser .  
Matrix_SIMD is a matrix class inclulding four Vect4D_SIMD member, v0,v1,v2,v3 respectively.  
In this application we use Col Major to implement the mulitiplication.  

# Rough code about Stress test
```cpp
for (int i = 0; i < 1000; i++)
	{
		vout = M * A;
		vout = M * B;
		vout = M * C;
	}
```	
# First way using _mm_dp_ps
```cpp
 Vect4D_SIMD Matrix_SIMD::operator * (const Vect4D_SIMD &v) const
{
	//using mm_dp_ps
	__m128 lv0 = _mm_dp_ps(v0._m, v._m, 0xFF);
	__m128 lv1 = _mm_dp_ps(v1._m, v._m, 0xFF);
	__m128 lv2 = _mm_dp_ps(v2._m, v._m, 0xFF);
	__m128 lv3 = _mm_dp_ps(v3._m, v._m, 0xFF);
	return Vect4D_SIMD(lv0.m128_f32[0], lv1.m128_f32[0], lv2.m128_f32[0], lv3.m128_f32[0]);
	
}
```
##### Running Time about stress test in release mode. Matrix*Vect_SIMD: 1.657408s
 
# Second way using _mm_dp_ps and _mm_shuffle_ps
```cpp
Vect4D_SIMD Matrix_SIMD::operator * (const Vect4D_SIMD &v) const
{

	return Vect4D_SIMD(_mm_shuffle_ps(_mm_shuffle_ps(_mm_dp_ps(v0._m, v._m, 0xFF), _mm_dp_ps(v1._m, v._m, 0xFF), _MM_SHUFFLE(0, 1, 0, 0)),
									 _mm_shuffle_ps(_mm_dp_ps(v2._m, v._m, 0xFF), _mm_dp_ps(v3._m, v._m, 0xFF), _MM_SHUFFLE(0, 3, 0, 2)),
										_MM_SHUFFLE(2, 0, 2, 0)));
	
}

```
##### Running Time about stress test in release mode.   Matrix*Vect_SIMD: 1.618239
  
# Third way using _mm_add_ps combining with _mm_add_ps
```cpp
Vect4D_SIMD Matrix_SIMD::operator * (const Vect4D_SIMD &v) const
{

	__m128 result = _mm_dp_ps(v0._m, v._m, 0xF1);
	result = _mm_add_ps(result, _mm_dp_ps(v1._m, v._m, 0xF2));
	result = _mm_add_ps(result, _mm_dp_ps(v2._m, v._m, 0xF4));
	result = _mm_add_ps(result,_mm_dp_ps(v3._m, v._m, 0xF8));

	return Vect4D_SIMD(result);
	
}

```
##### Running Time about stress test in release mode.  Matrix*Vect_SIMD: 1.586541
 
# Fourth way using _mm_hadd_ps and _mm_mul_ps
```cpp
Vect4D_SIMD Matrix_SIMD::operator * (const Vect4D_SIMD &v) const
{

	return Vect4D_SIMD( _mm_hadd_ps( _mm_hadd_ps( _mm_mul_ps(v0._m, v._m), _mm_mul_ps(v1._m, v._m) ),
								 	_mm_hadd_ps (_mm_mul_ps(v2._m, v._m), _mm_mul_ps(v3._m, v._m) ) ) );
	
}

```
##### Running Time about stress test in release mode.   Matrix*Vect_SIMD: 1.334228

