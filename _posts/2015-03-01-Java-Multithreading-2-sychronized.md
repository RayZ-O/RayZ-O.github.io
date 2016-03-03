---
layout: post
title: "Java Multithreading 2: sychronized"
categories: [all, multithreading, java]
date: 2015-03-01
author: Rui Zhang
---

### Monitor
Monitor is a synchronization construct that control access of shared data. A monitor consist of a mutex and condition variable. Only one thread can execute any monitor procedure at any time. If second thread arrives, it will be blocked until the first thread is done.

### Monitor in Java 
Java supports monitor on language level by providing two basic synchronization idioms: synchronized methods and synchronized statement. Synchronization code added by compiler to guarantee at each point in time, at most one thread may be executing critical code.

### Intrinsic lock of Object
In Java, every object has an intrinsic lock associated with it. To access an object, a thread must acquire the object's intrinsic lock. The intrinsic lock is a reentrant lock, which means the a thread can acquire the lock it already owns multiple times. Thus synchronized method can be called recursivelly.
