---
layout: post
title: "Single pattern string matching algorithms"
categories: [all, algorithms]
date: 2015-12-30
author: Rui Zhang
---

### Overview
String matching is frequently used in the text editor and search engine. Can you imagine using a text editor without find/replace? Efficient string matching algorithms can greatly improve many applications. The fundamental string matching algorithms are the single pattern string matching algorithms which will be discussed in this post such as Rabin–Karp algorithm, Finite-state automaton, and Knuth–Morris–Pratt algorithm. There are also algorithms for a finite set of patterns such as Aho–Corasick algorithm and Commentz-Walter algorithm, or an infinite number of patterns which are usually represented by regular expression[^1]. 

The single pattern string matching problem is defined as follows. Given a text string T of size n and a pattern string P of size m where m <= n, find all occurrences of P in T[^2]. 

In the following discussion, I assume all characters in T and P are selected from an alphabet Σ. 

### Naive
The naive algorithm is straightforward. Check P[0, m) = T[i, i + m) for each position i in T from 0 to n - m + 1.
{% highlight c++ %}
void Substr(std::string& T, std::string& P) {
    int n = T.size(), m = P.size();
    for (int i = 0; i < n - m + 1; i++) {        
        if (equal(T, P, i)) {
            cout << "Pattern occurs at" << i << endl;
        }
    }
}

bool equal(std::string& T, std::string& P, int i) {
    for (int j = 0; j < m; j++) {
        if (T[i+j] != P[j]) return false;
    }
    return true;
}
{% endhighlight %}
The time complexity is O((n-m+1)m).
O(n-m+1) is used instead of O(n-m) because when n = m, the complexity is O(m), not O(0).

### Rabin–Karp Algorithm
Rabin–Karp Algorithm uses Hash function to find the pattern in text. It can be easily extended to multiple patterns string matching. Every substring of length m in T can be seen as a number in base B (B >= |Σ|). Thus hash(T[i..i+m)) = T[i] * B<sup>(m-1)</sup> + T[i+1] * B<sup>(m-2)</sup> + ... + T[i+m-1]. For example,  

~~~
(1) Σ = {0, 1, ..., 9}, B = |Σ| = 10:
hash("110") = (('1' - '0') * 10 + ('1' - '0')) * 10 + ('0' - '0') 
            = 110 

(2) Σ = {a, b, ..., z}, B = |Σ| = 26:
hash("rui") = (('r' - 'a') * 26 + ('u' - 'a')) * 26 + ('i' - 'a') 
            = (17 * 26 + 20) * 26 + 8 
            = 12020 
~~~

The main idea of Rabin–Karp Algorithm is to pre-compute the hash value for the pattern and compare it with the hash value of every substring of length m in T.

At the first glance we need to iterate through T, at each position i between 0 and n - m + 1 calculate hash value for the substring of length m. However, the hash function we use is a [rolling hash](https://en.wikipedia.org/wiki/Rolling_hash) function. The hash value at position i can be quickly calculated given the hash value at position i - 1 using the following formula.  

> hash<sub>i</sub> = (hash<sub>i–1</sub> – T[i- 1] * B<sup>(m-1)</sup> ) * B + T[i + m - 1]

The following code fragment implements these ideas.

{% highlight c++ %}
void RabinKarp(std::string& T, std::string& P, int base) {
    int n = T.size(), m = P.size();
    unsigned long long hp = 0;   // hash value of pattern
    unsigned long long ht = 0;   // hash value of substring in T
    for (int i = 0; i < m; i++) {
        hp = hp * base + P[i];
        ht = ht * base + T[i];
    }
    for (int i = 0; i < n - m + 1; i++) {
        if (hs == ht) {
            cout << "Pattern occurs at" << i << endl;
        } 
        if (i < n - m) {
            ht = (ht - T[i] * base) * base + T[i + m];
        }        
    }
}
{% endhighlight %}

You may have already found a problem. If the pattern is huge, hp may not fit into unsigned long long. A common solution to handle the overflow is to modulo a sutibale modulus q. The drawback of this solution is that two different strings may map to the same hash value, hash(T[i..i+m)) = hash(P[0..m)) doesn't imply T[i..i+m) = P[0..m). But hash(T[i..i+m)) != hash(P[0..m)) still implies T[i..i+m) != P[0..m). Now we need to check the equality of two string when the hash value is matched.

Here is the new matching function:
{% highlight c++ %}
void RabinKarp(std::string& T, std::string& P, int base, int q) {
    int n = T.size(), m = P.size();
    unsigned long long hp = 0;   // hash value of pattern
    unsigned long long ht = 0;   // hash value of substring in T
    for (int i = 0; i < m; i++) {
        hp = (hp * base + P[i]) % q;
        ht = (ht * base + T[i]) % q;
    }
    for (int i = 0; i < n - m + 1; i++) {
        if (hs == ht && equal(T, P, i)) {  // needs additional check
            cout << "Pattern occurs at" << i << endl;
        } 
        if (i < n - m) {
            ht = ((ht - T[i] * base) * base + T[i + m]) % q;
        }        
    }
}

bool equal(std::string& T, std::string& P, int i) {
    for (int j = 0; j < m; j++) {
        if (T[i+j] != P[j]) return false;
    }
    return true;
}
{% endhighlight %}

Rabin–Karp Algorithm takes O(m) preprocessing time. Because of the extra checking, its worse-case matching time is O((n-m+1)m), which is the same as Naive algorithm. However, in many applications, we expect few occurrences of P in T, the expected complexity becomes O((n-m+1)+cm) where c is a constant. Since m <= n, the run time is O(n) if the number of matching is low.

### Finite-state automaton
(To be continued...)

### Knuth–Morris–Pratt Algorithm
(To be continued...)

### Comparison

|    Algorithm      |Preprocessing| Matching  |
|:------------------|:-----------:|:---------:|
|Naive              |0            |O((n-m+1)m)|
|Rabin–Karp         |O(m)         |O((n-m+1)m)|
|Finite automaton   |O(m\|Σ\|)    |O(n)       |
|Knuth–Morris–Pratt |O(m)         |O(n)       |
{: class="table table-striped table-nonfluid"}

**Reference:**

[^1]: [Wikipedia: String searching algorithm](https://en.wikipedia.org/wiki/String_searching_algorithm)

[^2]: [Introduction to Algorithms, 3rd Edition](http://www.amazon.com/Introduction-Algorithms-Edition-Thomas-Cormen/dp/0262033844)

[^3]: [Topcoder: Introduction to String Searching Algorithms](https://www.topcoder.com/community/data-science/data-science-tutorials/introduction-to-string-searching-algorithms/)
