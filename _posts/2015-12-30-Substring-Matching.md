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
void Substr(std::string T, std::string P) {
    if (P.empty()) {
        return;
    }  
    int n = T.size(), m = P.size();
    for (int i = 0; i < n - m + 1; i++) {        
        if (equal(T, P, i)) {
            std::cout << "Pattern occurs at" << i << std::endl;
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
Rabin–Karp Algorithm uses Hash function to find the pattern in text. It can be easily extended to multiple patterns string matching. Every substring of length m in T can be seen as a number in base B (B >= |Σ|)[^3]. Thus hash(T[i..i+m)) = T[i] * B<sup>(m-1)</sup> + T[i+1] * B<sup>(m-2)</sup> + ... + T[i+m-1]. For example,  

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

At the first glance we need to iterate through T, at each position i between 0 and n - m + 1 calculate hash value for the substring of length m. However, the hash function we use is a [rolling hash](https://en.wikipedia.org/wiki/Rolling_hash) function. The hash value at position i+1 can be quickly calculated given the hash value at position i using the following formula.  

> hash<sub>i+1</sub> = (hash<sub>i</sub> – T[i] * B<sup>(m-1)</sup> ) * B + T[i + m]

The following code fragment implements these ideas.

{% highlight c++ %}
void RabinKarp(std::string T, std::string P, int base) {
    if (P.empty()) {
        return;
    }  
    int n = T.size(), m = P.size();
    unsigned long long hp = 0;   // hash value of pattern
    unsigned long long ht = 0;   // hash value of substring in T
    for (int i = 0; i < m; i++) {
        hp = hp * base + P[i];
        ht = ht * base + T[i];
    }
    for (int i = 0; i < n - m + 1; i++) {
        if (hs == ht) {
            std::cout << "Pattern occurs at" << i << std::endl;
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
void RabinKarp(std::string T, std::string P, int base, int q) {
    if (P.empty()) {
        return;
    }   
    int n = T.size(), m = P.size();
    unsigned long long hp = 0;   // hash value of pattern
    unsigned long long ht = 0;   // hash value of substring in T
    for (int i = 0; i < m; i++) {
        hp = (hp * base + P[i]) % q;
        ht = (ht * base + T[i]) % q;
    }
    for (int i = 0; i < n - m + 1; i++) {
        if (hs == ht && equal(T, P, i)) {  // needs additional check
            std::cout << "Pattern occurs at" << i << std::endl;
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

Rabin–Karp Algorithm takes O(m) preprocessing time. Because of the extra checking, its worst-case matching time is O((n-m+1)m), which is the same as the Naive algorithm. However, in many applications, we expect few occurrences of P in T, the expected complexity becomes O((n-m+1)+cm) where c is a constant. Since m <= n, the run time is O(n) if the number of matching is low. 

### Finite-state automaton
A finite-state automaton(machine) is a machine that can be in one of a finite number of states[^4]. It can change from one state to another when initiated by a triggering event or condition, this is called a transition[^5].  
Some string matching algorithms build a finite state automaton. The string-matching automata are very efficient. After building the finite automaton, they examine each character in T only once.
Let's see an example, the Exercises 32.3-1 on CLRS[^2]. Given pattern P = "aabab":  
(1) The **state-transition diagram** shown below.   
![fig1]  

(2) The **transition function** of the string-matching automata. It reveals the condition of transition, namely changing from one state to another. For example, when the string-matching automaton is in state 0, if it receive an 'a', it will become state 1. Similarly, when it is in state 4, it will transit to state 2 if the input is 'a' or transit to state 5 if the input is 'b'. Building the transition function takes at least O(m\|Σ\|) time since the size of the table is m\|Σ\|.

|state|input a|input b|P|
|:-:|:-:|:-:|:-:|
|0|1|0|a|
|1|2|0|a|
|2|2|3|b|
|3|4|0|a|
|4|2|5|b|
|5|1|0| |
{: class="table table-striped table-nonfluid table-bordered"}

(3) The matching process of the automaton on T = "aaababaabaababaab"

|i    |-|0|1|2|3|4|5|6|7|8|9|10|11|12|13|14|15|16|
|T[i] |-|a|a|a|b|a|b|a|a|b|a|a |b |a |b |a |a |b |
|state|0|1|2|2|3|4|5|1|2|3|4|2 |3 |4 |5 |1 |2 |3 |
{: class="table table-striped table-nonfluid table-bordered"}

The automaton starts from the initial state 0 and receives input characters from T one by one. It reaches the final state only if there is valid matching. Two occurrences of P are found, ending in position 5 and position 13 where the state = 5.
In summary, finite-state automaton takes O(n) for matching which is the fastest we can do. But the preprocessing time is O(m|Σ|) which can be large if |Σ| is large.

### Knuth–Morris–Pratt(KMP) Algorithm
The KMP algorithm was invented by Donald Knuth and Vaughan Pratt in 1970, and independently by James H. Morris. The KMP algorithm uses an auxiliary array called prefix function[^2] which can be precomputed in O(m) time. This array allows us to compute the transition function efficiently on the fly. 
The main idea of the KMP algorithm is to utilize the partial matching result to determine the new start position when encounters a matching failure. 
For example, let T = "abababacabacaba" and P = "abacaba".

(1) Start from position 0 of both T and P, we found mismatch at position 3.  
![fig2]

(2) Since T[1] = 'b' is not a prefix of P, the matching substring can't start here. T[2] = 'a' is a prefix of P, we can resume comparing at position 3 of T and position 1 of P. The matching fails when T[5] != P[3].  
![fig3]

(3) Similarly T[4] = 'a' is a prefix of P, continue comparing T[5..n) and P[1..m). A valid matching is found.  
![fig4]

(4) We can resume matching at T[11] and P[3] because the suffix T[8..10] = "aba" of the matching in previous step is also the prefix of P.  
![fig5]

The prefix function helps us to determine where to resume comparing in T and P when the matching fails, it's also called failure function. It stores the length of longest common prefix at each position between:   
(a) the prefix of P  
(b) the [proper suffix](https://en.wikipedia.org/wiki/Substring#Suffix) of (a) at this position.

The prefix function of P = "abacaba".

|position|prefix|common prefix|prefix function|
|:-:|:---------|:-|:-:|
|0|a|*no proper suffix* |0|
|1|ab             |-|0|
|2|**a**b**a**    |a|1|
|3|abac           |-|0|
|4|**a**bac**a**  |a|1|
|5|**ab**ac**ab** |ab|2|
|6|**aba**c**aba**|aba|3|
{: class="table table-striped table-nonfluid table-bordered"}

The following code computes prefix function in O(m).  

{% highlight c++ %}
std::vector<int> PrefixFunction(std::string& P) {        
    int m = P.size();
    std::vector<int> prefunc(m, 0); 
    for (int i = 1; i < m; i++) {
        int j = prefunc[i-1];
        while (j > 0 && P[i] != P[j]) {   // while P[i] doesn't expand the best partial match
            j = prefunc[j-1];             // go to the previous best partial match
        }       
        if (P[i] == P[j]) {
            j++;                          // P[i] expands one of the partial match
        } 
        prefunc[i] = j;                   // if P[i] doesn't expand any partial match, prefunc[i] = j = 0    
    }
    return prefunc;
}
{% endhighlight %}

The *KMP* matching function is similar because both of them match a string against P. *KMP* match T against P while *PrefixFunction* match P against itself.

{% highlight c++ %}
void KMP(std::string T, std::string P) {  
    if (P.empty()) {
        return;
    }      
    int n = T.size(), m = P.size();
    std::vector<int> prefunc = PrefixFunction(P);
    int j = 0;
    for (int i = 0; i < n; i++) {        
        while (j > 0 && T[i] != P[j]) {   
            j = prefunc[j-1];             
        }       
        if (T[i] == P[j]) {
            j++;                         
        } 
        if (j == m) {
            std::cout << "Pattern occurs at" << i - m + 1 << std::endl;
            j = prefunc[j-1];
        }                      
    }
}
{% endhighlight %}

The prefix function precomputed by *PrefixFunction*:

|i              |0|1|2|3|4|5|6|
|P[i]           |a|b|a|c|a|b|a|
|prefix function|0|0|1|0|1|2|3|
{: class="table table-striped table-nonfluid table-bordered"}

The string matching process is similar to the finite automaton, j can be treated as the state of the automaton. When j reaches the final state 7 which is the length of P, there is a valid match starting from i - m + 1.

|i   |-|0|1|2|3|4|5|6|7|8|9|10|11|12|13|14|
|T[i]|-|a|b|a|b|a|b|a|c|a|b|a |c |a |b |a |
|j   |0|1|2|3|2|3|2|3|4|5|6|7 |4 |5 |6 |7 |
{: class="table table-striped table-nonfluid table-bordered"}

The running time of KMP algorithm is O(n) because of the following facts[^2]:  
(1) j is increased at most once per iteration, there are at most n - 1 increase.  
(2) each iteration of the inner while loop decrease j, since the prefix can not longer than the original string which means prefunc[j] <= j  
(3) j is never negative  
Therefore, the total number of decrease is at most n - 1, the KMP algorithm runs in O(n).

### Running time Comparison

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

[^4]: [Finite State Automata](http://galaxy.eti.pg.gda.pl/katedry/kiw/pracownicy/Jan.Daciuk/personal/thesis/node12.html)

[^5]: [Wikipedia: Finite-state machine](https://en.wikipedia.org/wiki/Finite-state_machine)

[^6]: [Wikipedia: Knuth–Morris–Pratt algorithm](https://en.wikipedia.org/wiki/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm)

[fig1]: /assets/Substr/fig1.png

[fig2]: /assets/Substr/fig2.png

[fig3]: /assets/Substr/fig3.png

[fig4]: /assets/Substr/fig4.png

[fig5]: /assets/Substr/fig5.png
