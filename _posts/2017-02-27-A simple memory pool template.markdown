---
layout: post
title:  "A simple memory pool template "
date:   2017-02-27 10:45:20 -0600
categories: C++ 11,Linux,MemoryPool
---
If you run into a situation that you need to allocate a specific object very frequently. Obviously the traditional way will cause the memeory fragment because of 
frequent allocation on heap, and you will get more risky for failure of allocation after a certain running period. so this situation the memory pool definately is your
best choice. 

I got this situation on my open source project. A lot of client will connect and disconnect the server occasionally. I need to allocate a chunk of heap memory
for each connecting, and recycle the memeory when disconnecting. so I came up with a very simple and effective memeory pool to solve this problem.Here is the code .



{% highlight cpp %}
 template <typename T > class MemoryPool {
        public  :
            MemoryPool(int nSize) throw (std::bad_alloc&) : mnSize(nSize){
                std::unique_ptr<T[]> lp (new T[mnSize]);
                mArrayBuffer = std::move(lp);
                mnFreeSize = mnSize;
                for(int i = 0; i < mnSize ; i++)
                    mBitSet.push_back(false);
                    
            }
            
            T* alloc()  {
                if(!mArrayBuffer)
                    return nullptr;
                //check a empty slot in the bitset
                for(int i = 0 ; i < mnSize ; i++)
                {
                    if(mBitSet[i] == false)
                    {
                        mnFreeSize--;
                        mBitSet[i] = true;
                        return &mArrayBuffer[i]; 
                    }
                }
                return nullptr;
            }
            void free(T* np) {
                if(!mArrayBuffer)
                    return ;
                if(np != nullptr && 
                        ( np >= &mArrayBuffer[0] && np <= &mArrayBuffer[mnSize-1]) ){
                    int offset = (np- &mArrayBuffer[0]) % sizeof(T);
                    mBitSet[offset] = false;
                    mnFreeSize++;
                } 
            }
            int GetFreeSize() {return mnFreeSize;}
        private:
            const int mnSize;
            int mnFreeSize;
            std::vector<bool> mBitSet;
            std::unique_ptr<T[]> mArrayBuffer;
                   
    };
{% endhighlight  %}


By the way , in the first place I wanted to use the BitSet instead of Vector<Bool>. But bitset in C++ 11 can not support dynamically size changing. And sound like the compiler
will optimize the vector<Bool> as a bitset. so I think the vector<Bool> is the right answer for this class.