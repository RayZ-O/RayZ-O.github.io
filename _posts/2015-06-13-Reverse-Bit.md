---
layout: post
title: "Reverse Bits optimization: Piecewise Cache"
categories: [all, algorithms]
date: 2015-06-13
author: Rui Zhang
---

### Reverse a sequence
Reverse a sequence is a common operation in programming. Most languages provide libraray functions to reverse array, string, or other sequence containers/collections. We can also easily code our own reverse function:

{% highlight c++ %}
void reverse(std::vector<int> &v) {
    int i = 0;
    int j = v.size() - 1;
    while (i < j) {
        std::swap(v[i++], v[j--]);
    }
}
{% endhighlight %}
The idea is simple. Swap the first and last, second and second last ... and so on.

### Reverse bits of an integer
This is an easy problem on leetcode: [190. Reverse Bits](https://leetcode.com/problems/reverse-bits/). The idea is straightforward. 

{% highlight c++ %}
uint32_t reverseBits(uint32_t n) {        
    uint32_t res = 0;        
    for (int i = 0; i < 32; i++) {            
        res <<= 1;            
        res |= n >> i & 1;        
    }        
    return res;    
}
{% endhighlight %}
In iteration i, add the *i*-th lowest bit of n to result and left shift result by 1.

### Follow up - call multiple times
How to optimize if this function is called multiple times?  

#### First Try
First idea comes to my mind is caching: use a hashtable to store values we've computed.
{% highlight c++ %}
std::unordered_map<uint32_t, uint32_t> mp;

uint32_t reverseBits(uint32_t n) { 
    if (mp.find(n) != mp.end()) {
        return mp[n];
    }  

    uint32_t res = 0;        
    for (int i = 0; i < 32; i++) {            
        res <<= 1;            
        res |= n >> i & 1;        
    }        

    mp[n] = res;

    return res;    
}
{% endhighlight %}

Assume the input is uniformly distributed in the range of uint32_t, use *uniform_int_distribution* in C++11 standard library to generate input numbers:

{% highlight c++ %}
int main()
{
    unsigned seed = std::chrono::system_clock::now().time_since_epoch().count();
    std::default_random_engine generator (seed);
    std::uniform_int_distribution<unsigned> distribution(0, std::numeric_limits<unsigned>::max());
    for (int i = 0; i < 10000000; ++i) {
        reverseBits(distribution(generator));
    }        
}
{% endhighlight %}

Run this test 10 times for non-cached and cached function, the average runtime shown as following:

||non-cached | cached|
|:---:|:---:|:------:|
|Total|29.160s|3m12.994s|
|Average|2.916s|19.299s|
{: class="table table-striped table-nonfluid"}

 Suprisingly, the cached version is 6x slower than the naive version. Since the input is uniformly distributed in a large range [0, 4294967296). The cache miss rate is very high. The unordered_map spend lots of time allocating small table entries and growing the hashtable but it's almost useless.

Modify the cached version to count the cache hits:
{% highlight c++ %}
std::unordered_map<uint32_t, uint32_t> mp;
int count = 0;

uint32_t reverseBits(uint32_t n) { 
    if (mp.find(n) != mp.end()) {
        count++;
        return mp[n];
    }  

    uint32_t res = 0;        
    for (int i = 0; i < 32; i++) {            
        res <<= 1;            
        res |= n >> i & 1;        
    }        

    mp[n] = res;

    return res;    
}
{% endhighlight %}

Rerun the test 10 times:

||count|
|:---:|:---:|
|1|0|
|2|0|
|3|0|
|4|0|
|5|0|
|6|0|
|7|0|
|8|1|
|9|0|
|10|0|
{: class="table table-striped table-nonfluid"}

Lol, the cache is only hit once. By the way, this proves the uniform distribution generator in standard library is really good.

#### Second Try
How can we increase the cache hit ratio?  
An intuitive solution is to reduce the number of possible values in the hashtable.
Notice that to reverse a sequence, we can divide the sequence into several parts, reverse each part and put it in the right position.
For example, divide the input into 4 parts and build the reverse number in 4 step: 

~~~
Initialization:
input n = 00000010 10010100 00011110 10011100
result res = 00000000 00000000 00000000 00000000

1. Reverse the lowest 8 bits of n and add to the result
res = 00000000 00000000 00000000 00111001

2. Left shift res by 8, reverse the second lowest 8 bits of n and add to the result
res = 00000000 00000000 00111001 01111000

3. Repeat the above process for second highest 8 bits
res = 00000000 00111001 01111000 00101001

4. Get the last 8 bits and terminate
res = 00111001 01111000 00101001 01000000
~~~

Great, if each part is cached seperately in the hashtable, there are only 256 possible values.

{% highlight c++ %}
std::unordered_map<uint8_t, uint32_t> mp;

uint8_t reverse8Bits(uint8_t n) {
    uint8_t res = 0;        
    for (int i = 0; i < 8; i++) {            
        res <<= 1;            
        res |= n >> i & 1;        
    } 
    mp[n] = res;
    return res;
}

uint32_t reverse32Bits(uint32_t n) {   
    uint32_t res = 0;
    for (int i = 0; i < 4; i++) {
        res <<= 8;
        uint8_t part = n & 0xff;
        if (mp.find(part) != mp.end()) {
            count++;
            res |= mp[part];
        } else {
            res |= reverse8Bits(part);
        }
        n >>= 8;
    }
    return res;    
}
{% endhighlight %}

Runtime:

||non-cached | cached|piecewise cached|
|:---:|:---:|:------:|:------:|
|Total|29.160s|3m12.994s|1m52.217s|
|Average|2.916s|19.299s|11.222s|
{: class="table table-striped table-nonfluid"}

The runtime is still 3x slower than the non-cached version. Since reverse a 32-bit integer is not an expensive operation. The run time of the cached version is dominated by memory allocation for table entries.

The good news is the cahce hits increase dramatically:

||count|percentage|
|:---:|:---:|:---:|
|1|39999708|99.99%|
|2|39999205|99.99%|
|3|39999629|99.99%|
|4|39999320|99.99%|
|5|39998840|99.99%|
|6|39999601|99.99%|
|7|39999654|99.99%|
|8|39999708|99.99%|
|9|39999847|99.99%|
|10|39999261|99.99%|
{: class="table table-striped table-nonfluid"}


#### Third Try
In order to avoid the overhead of unordered_map, array can be used to cache the result. Array eliminates dynamic memory allocation and unneccessary meta data.

{% highlight c++ %}
uint32_t cache[256];

void initCache() {
    for (int i = 0; i < 256; i++) {
        cache[i] = -1;
    }
}

uint8_t reverse8Bits(uint8_t n) {
    uint8_t res = 0;        
    for (int i = 0; i < 8; i++) {            
        res <<= 1;            
        res |= n >> i & 1;        
    } 
    cache[n] = res;
    return res;
}

uint32_t reverse32Bits(uint32_t n) {   
    uint32_t res = 0;
    for (int i = 0; i < 4; i++) {
        res <<= 8;
        uint8_t part = n & 0xff;
        if (cache[i] != -1) {
            res |= cache[part];
        } else {
            res |= reverse8Bits(part);
        }
        n >>= 8;
    }
    return res;    
}
{% endhighlight %}
    
Runtime:

||non-cached |cached|piecewise cached|piecewise cached array|
|:---:|:---:|:------:|:------:|:------:|
|Total|29.160s|3m12.994s|1m52.217s|17.685s|
|Average|2.916s|19.299s|11.222s|1.768s|
{: class="table table-striped table-nonfluid"}

Bingo! Using array is 6x faster than unordered_map and 1.6x faster than the non-cached version.

