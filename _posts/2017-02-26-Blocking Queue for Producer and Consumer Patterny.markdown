---
layout: post
title:  "Blocking Queue for Producer and consumer pattern"
date:   2017-02-22 10:45:20 -0600
categories: C++ 11,Linux,Producer and Consumer,Condition Variable
---
As we knew, The queue is the core component of the producer and consumer pattern. and the another important one is how to wake up one of the consumer.
Because Most of time, we get more than one comsumer. 
Fortunately we can combile the queue and condition variable to achieve this goal using C++ 11.


{% highlight cpp %}
template<class T>
    class ProducerConsumerQueue {
    private:
        std::queue<T> mQueue;
        std::mutex mMutex;
        std::condition_variable mCondition;
    public:
        
        void enqueue(const T& nValue) {
            std::unique_lock<std::mutex> lock(mMutex);
            mQueue.push(nValue);
            //Notify one thread to handle this item
            mCondition.notify_one();
        }

        T dequeue() {
            std::unique_lock<std::mutex> lock(mMutex);
            //Avoid spurious waken
            mCondition.wait(lock,[this]{return !mQueue.empty();} );
           
            T lFrontItem = mQueue.front();
            mQueue.pop();
            return lFrontItem;
        }


    };
{% endhighlight  %}


By the way , the condition variable only can be working with the unique_lock ,not the lock_guard. We must take care of that. The resaon is the wait function of the condition_variable
will unlock the mutex. actually lock_guard does't expost the manually unlock function to the outside.