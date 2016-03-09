---
layout: post
title: "Java Multithreading 2: sychronized"
categories: [all, multithreading, java]
date: 2015-03-01
author: Rui Zhang
---

### Monitor
Monitor is a synchronization construct that control access of shared data. A monitor consist of a mutex and condition variable[^1]. Only one thread can execute any monitor procedure at any time. If second thread arrives, it will be blocked until the first thread is done. 

### Monitor in Java
Java supports monitor on language level by providing two basic synchronization idioms: *synchronized* methods and *synchronized* statements[^2]. Synchronization code added by compiler to guarantee at each point in time, at most one thread may be executing critical code. In Java, every object implicitly has a condition variable tied to its lock(mutex). The intrinsic lock is a reentrant lock, which means the a thread can acquire the lock it already owns multiple times. Thus *synchronized* method can be called recursivelly.

*synchronized* also prevents the compiler from reordering the code statement inside the moniter, which may cause race condition in the multithreading environment. Moreover, before entering the *synchronized* methods or statements the program reads data from memory other than cache, and writes modified data to memory before it release the lock to avoid memory inconsistency[^3].

### Why is synchronization needed?
Consider the following count class, what will the program print?:

{% highlight java %}
public class Counter {
    private static int count = 0;

    public static void increment() { 
        count++;
    }

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                increment();
            }
        });
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                increment();
            }
        });
        t1.start();
        t2.start();

        try {
            t1.join();
            t2.join();
            System.out.println(count);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
{% endhighlight %}

The answer is undefined. It may print the expected count 2000, but most of time it prints a random number less than 2000. Because the *count++;* statement consist of three steps:  
(1) read the value of count  
(2) add 1 to it  
(3) write it back  
The count value of a thread may overwrite the count value of another thread. The minimum possible value of count at the end of the program is 2.

~~~
1. thread 1 and thread 2 read count when it's 0.
2. thread 1 iterates 999 times and write 999 back to count.
3. thread 2 finishes the first iteration, overwrites count to 1.
4. thread 1 read the value of count, which is 1 now.
5. thread 2 read the count value, finishes all remaining loops and write 1000 back to count.
6. thread 1 continues the last loop, increment count by 1 and overwrite count to 2. 
~~~

The output of the program is dependent on the sequence of increment operation. This undefined behaviour is called [Race condition](https://en.wikipedia.org/wiki/Race_condition). Synchronization is needed to get rid of race condition. We can use the *synchronized* keyword to protect the data shared between multiple threads. 

### *synchronized* methods 
*synchronized* keyword can be used in front of both static and non-static methods. We can modify the increment method and make it thread safe.  

(1) static synchronized method

{% highlight java %}
public static synchronized void increment() {
    count++;
} 
{% endhighlight %}

(2) non-static synchronized method

{% highlight java %}
public synchronized void increment() {
    count++;
}
{% endhighlight %}

(3) Static synchronized method locks on class instance and non-static synchronized method locks on current instance. Since static and non-static method use different locks, they can run in parallel. The output of the program below is undefined.
{% highlight java %}
public class Counter {
    static int count = 0;

    public static synchronized void staticIncrement() { 
        count++;
    }

    public synchronized void nonstaticIncrement() {
        count++;
    }

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                staticIncrement();
            }
        });
        Thread t2 = new Thread(() -> {
            Counter counter = new Counter();
            for (int i = 0; i < 1000; i++) {
                counter.nonstaticIncrement();
            }
        });
        t1.start();
        t2.start();

        try {
            t1.join();
            t2.join();
            System.out.println(count);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
{% endhighlight %}

(4) *synchronized* can **NOT** be applied to constructor
{% highlight java %}
public class Counter {
    private int count;

    synchronized Counter() { 
        this.count = 0;
    }
}
{% endhighlight %}

Console:  

~~~
Error: modifier synchronized not allowed here
~~~

### *synchronized* statements
The synchronzied block is preferred over synchronized method in Java. Since with synchronized block, we can only lock critical region.   

(1) synchronized(*<ClassName>*.class) locks on class instance and guarantees there is exactly one thread in the block. 
{% highlight java %}
public static void increment() {
    synchronized(Counter.class) {
        count++;
    }    
}
{% endhighlight %}

(2) synchronized(this) locks on current instance and ensures that there is exactly one thread per instance. 
{% highlight java %}
public void increment() {
    synchronized(this) {
        count++;
    }    
}
{% endhighlight %}

(3) synchronized can also lock on other objects since every object in Java has a intrinsic lock. This is useful for improving concurrency with fine-grained synchronization. Notice that non-final field should not be synchronized on synchronized block, because if the reference of the non-final field is changed, the program may no longer be thread-safe.
{% highlight java %}
private int count1 = 0;
private int count2 = 0;
private final Object lock1 = new Object();
private final Object lock2 = new Object();

public void increment1() {
    synchronized(lock1) {
        count1++;
    }    
}

public void increment2() {
    synchronized(lock2) {
        count2++;
    }    
}
{% endhighlight %}
count1 and count2 in the above program are irrelevant. Therefore, increment1() and increment2() can run in parallel.

(4) If locks on null, **NullPointerException** will be thrown.
{% highlight java %}
private int count1 = 0;
private Object lock1 = null; 

public void increment1() {    
    synchronized(lock1) { 
        count1++;
    }    
}
{% endhighlight %}

Console:  

~~~
Exception in thread "Thread-1" java.lang.NullPointerException
~~~

### Limitation of *synchronized*[^3]

* Doesn't allow concurrent read.
* Can only be used to protect shared data within the same JVM.
* May lead to performance degrade.  


**Reference:**

[^1]: [Wikipedia: Monitor](https://en.wikipedia.org/wiki/Monitor_(synchronization))

[^2]: [The Java™ Tutorials: Synchronization](https://docs.oracle.com/javase/tutorial/essential/concurrency/sync.html)

[^3]: [Java Synchronization Tutorial: What, How and Why](http://javarevisited.blogspot.com/2011/04/synchronization-in-java-synchronized.html)


