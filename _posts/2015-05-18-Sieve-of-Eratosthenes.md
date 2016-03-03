---
layout: post
title: "Sieve of Eratosthenes"
categories: [all, algorithms]
date: 2015-05-18
author: Rui Zhang
---

A prime number is a natural number that has exactly two distinct natural number divisors: 1 and itself. Finding all prime numbers in a given range is an ancient problem in number theory. A naive alogrithm would be for each number *N* in the range, check whether *N* is divisible by one of the numbers from 2 to sqrt(*N*). Following C++ code implements this alogrithm:
{% highlight c++ %}
std::vector<int> generatePrimes(int n) {
    std::vector<int> primes;
    for (int i = 2; i <= n; i++) {
        bool is_prime = true;
        for (int j = 2; j <= sqrt(i); j++) {
            if (i % j == 0) {
                is_prime = false;
                break;
            }
        }
        if (is_prime) {
            primes.push_back(i);
        }
    }
    return primes;
}
{% endhighlight %}
The time complexity of this algorithm is equal to sqrt(2) + sqrt(3) + ... + sqrt(N), according to the formula in [On the sum of the square roots of the first n natural numbers](http://ramanujan.sirinudi.org/Volumes/published/ram09.pdf), T(*N*) = O(*N*sqrt(*N*)). Is there a better solution?

### Prime sieve
A prime sieve is a fast type of algorithm for finding primes. The idea of prime sieve is to remvoe the composite numbers from the range until only prime numbers are left. There are many prime sieve algorihms. I'll explain a simple and ancient one called sieve of Eratosthenes which is named after Greek mathematician Eratosthenes of Cyrene.  
Let's see the algorithm:  
1. Create a list of integer from 2 to n, mark all of them as prime.  
2. Let p equal to the smallest prime 2.  
3. Mark all numbers in range 2p, 3p, 4p, ... and less than or equal to n as composite number.  
4. Set p to the next number in the list that greater than p and marked as prime, repeat step 3 and 4. Stop if no such number.  

The main idea is to start from the smallest prime p, mark all numbers divisible by p in the range as composite, and then move to the next smallest prime p in the list. When the algorithm stops, the remaining numbers in the list are prime numbers.

{% highlight c++ %}
std::vector<int> generatePrimes(int n) {
    std::vector<bool> marks(n+1, true);     
    marks[0] = marks[1] = false; //0 and 1 are not primes
    for (int i = 2; i <= n; i++) {        
        if (marks[i]) {
            for (int j = i * 2; j <= n; j += i) {                
                marks[j] = false;
            }
        }    
    }
    std::vector<int> primes;
    for (int i = 2; i <= n; i++) {
        if (marks[i]) {
            primes.push_back(i);
        }
    }
    return primes;
}
{% endhighlight %}

There are two improvements for sieve of Eratosthenes, firstly in step 2 we can start from p * p since all smaller multiples of p have already been marked in previous loops, and we can stop when p * p is greater than *N*. Secondly we can only check odd numbers and odd multiples, as all even numbers greater than 2 are not primes.
After refinement:

{% highlight c++ %}
std::vector<int> generatePrimes(int n) {
    std::vector<bool> marks(n+1, true);     
    marks[0] = marks[1] = false; 
    for (int i = 3; i <= sqrt(n); i += 2) { 
        if (marks[i]) {
            for (int j = i * i; j <= n; j += i) {                
                marks[j] = false;
            }
        }    
    }
    std::vector<int> primes;
    if (n > 2) {
        primes.push_back(2);
    }
    for (int i = 3; i <= n; i += 2) {
        if (marks[i]) {
            primes.push_back(i);
        }
    }
    return primes;
}
{% endhighlight %}

The loop of the algorithm totally execute (*N*/2) * (1/3 + 1/5 + 1/7 + ... + 1/p) times. This sequence is [Harmonic Series of Primes](http://mathworld.wolfram.com/HarmonicSeriesofPrimes.html), which is bounded by log(log(*N*)). Thus the complexity of Sieve of Eratosthenes is O(*N*log(log(*N*))). Much better than the naive algorithm!

#### Run time 
*Tested on Ubuntu 14.04 machine with 8 cores 32G memory*  

|N | Naive|Simple Sieve of Eratosthene|Improved Sieve of Eratosthene|
|:---:|:------:|:---:|:---:|
|10000    |0.006s|0.004s|0.004s|
|100000   |0.031s|0.021s|0.006s|
|1000000  |0.469s|0.112s|0.019s|
|10000000 |11.806s|1.132s|0.317s|
|100000000|5m14.426s|12.876s|3.479s|
{: class="table table-striped"}
