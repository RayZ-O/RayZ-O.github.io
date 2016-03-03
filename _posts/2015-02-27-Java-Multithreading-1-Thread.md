---
layout: post
title: "Java Multithreading 1: Thread"
categories: [all, multithreading, java]
date: 2015-02-27
author: Rui Zhang
---

### Process vs. Thread
Both process and thread are central concepts in concurrent programming. Process is an abstraction of a running program. A thread is a component of a process. Threads are lighter weight than processes, they are easier and faster to create and destroy than processes. Multiple threads in the same process share many resources. As shown in the following table:

|Per-Process item|Per-Thread item|
|:---:|:------:|
|Address space |[Program counter](https://en.wikipedia.org/wiki/Program_counter)|
|Global variables|Registers|
|Open files |Stack|
|Child processes |State|
|Signals and signal handlers||
{: class="table table-striped table-nonfluid"}
The items in the first column are shared by all threads in a process, which means if a thread open a file, the file is visible to all other threads in the same process.

Another difference is that a thread can easily communicate with other thread (of the same process) since they share resources. However, A process communicate with other process by using [inter-process communication](https://en.wikipedia.org/wiki/Inter-process_communication).

### Create Threads
Most implementations of the JVM run as a single process so concurrent programming is mostly concerned with multithreading. There are two ways to create a thread in Java:

#### 1. Provide a **Runnable** object
The *Runnable* interface defines a single method *run()*, the run() method will be called when the thread starts. The *Runnable* object is pass to the constructor of Thread.

{% highlight java %}
class MyRunnable implements Runnable {
    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            System.out.println(i);
        }
    }
}

public class CreateThread {
    public static void main(String[] args){
        Thread t = new Thread(new MyRunnable());
    }
}
{% endhighlight %}

We can also use anonymous class in place:  

{% highlight java %}
public class CreateThread {
    public static void main(String[] args){
        Thread t = new Thread(new Runnable(){
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    System.out.println(i);
                }
            }
        });
    }
}
{% endhighlight %}
In Java 8, lambda expression is even better:  

{% highlight java %}
public class CreateThread {
    public static void main(String[] args){
        Thread t = new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                System.out.println(i);
            }
        });
    }
}
{% endhighlight %}

#### 2. Extends **Thread** class
The *Thread* class implements *Runnable*, we can override *run()* in the subclass:

{% highlight java %}
public class MyThread extends Thread{
    public static void main(String[] args){
        Thread t = new MyThread();
    }

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            System.out.println(i);
        }
    }
}
{% endhighlight %}

### Start and Join
The *start()* method should be called in order to start the new thread.  
The *join()* method block the calling thread until a thread terminates or specified number of miliseconds.

{% highlight java %}
public class StartThread {
    public static void main(String[] args){
        Thread t = new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                System.out.println(i);
            }
        });
        t.start();
        try {
            t.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
{% endhighlight %}

### Sleep
*Thread* class has a static method *sleep()*, which pause the current thread for the specified number of miliseconds. The thread scheduler selects another thread to run. It's useful for pacing or waiting for other threads that have time requirements.  Sleep times are not guaranteed to be precise because it's rely on the underlying OS and the scheduler. 

{% highlight java %}
public class SleepThread {
    public static void main(String[] args){
        Thread t = new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(i);
            }
        });
    }
}
{% endhighlight %}

### Interrupt
As you can see, the *join()* and *sleep()* method throws an InterruptedException which means the waiting period can be terminated by interrupt. Calling *interrupt()* method of a thread object stops this thread. The programmer can define what to do when a thread is interrupted. Commonly it's used for terminating a thread. 
Internally each thread uses a boolean flag to store the interrupt status, invoking interrupt() sets this flag. Any method that terminated by InterruptedException clears the interrupt status.

{% highlight java %}
public class CreateThread {
    public static void main(String[] args){
        Thread t = new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    System.out.println("I am interrupted!");
                    break;
                }
                System.out.println(i);
            }
        });
        t.start();
        t.interrupt();
    }
}
{% endhighlight %}
