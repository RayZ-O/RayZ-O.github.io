---
layout: post
title: "Advanced priority queue: Fibonacci Heap"
categories: [all, datastructures]
date: 2015-3-11
author: Rui Zhang
---

### Amortized Complexity
Some data structures such as Fibonacci Heap and Splay Tree, perform really good in practice even the worst-cast time complexity is not attractive. Because for these data structures it is impossible for every operation in a sequence to take the worst-case time. In this case, the amortized analysis is a more clever reasoning way to provide a tighter bound. Amortized complexity concerns with the overall costs of a sequence of operations and guarantees the average performance of each operation in the worst case.

### Fibonacci heap
Fibonacci heap is invented by Michael L. Fredman and Robert E. Tarjan in 1984. It's a fancy data structure that has two advantages over the Binary heap:  

* The Fibonacci heap is a [meragable heap](https://en.wikipedia.org/wiki/Mergeable_heap)  
* The Fibonacci hHeap provides O(1) amortized time insert and decrease key operations, which makes it perfectly suited for applications that uses these operations frequently such as Dijkstra's Algorithm and Prim's Algortihm.

|Operation|Binary heap(worst-case)|Fibonacci Heap(amortized)|
|:---:|:---:|:---:|
|Make-Heap[^1]|O(1)   |O(1)|
|Insert      |O(lg N)|O(1)|
|Get-Min     |O(1)   |O(1)|
|Extract-Min |O(lg N)|O(lg N)|
|Merge       |O(n)   |O(1)|
|Decrease-Key[^2]|O(lg N)|O(1)|
|Delete[^2]  |O(lg N)|O(lg N)|
|Search		 |O(N)   |O(N)|

[^1]: creates a new empty heap.

[^2]: require a pointer to the target node as input because search is expensise in both binary heap and fibonacci heap.
 
From a theoretical perspective, all operations of Fibonacci heap are at least as fast as binary heap, thus fibonacci heap is obviously better than binary heap. However, because of the constant factor and implementation complexity, it's only attractive when the input size is large.

#### Structure overview
![fig1]


#### Node structure

{% highlight c++ %}
template <typename T>
class FibNode	{
	T data;
	int degree;
	bool childCut;
	FibNode *parent;
	FibNode *child;
	FibNode *lsibling;
	FibNode *rsibling;
}
{% endhighlight %}

* data: user data associated with the node
* degree: number of children
* childCut: 
	* true if node has lost a child since it became a child of its current parent. 
	* Set to false by remove min, which is the only operation that makes one node a child of another. 
	* Set ot false for a root nodes.
* parent: point to the parent node
* child: point to any of the children
* lsibling, rsibling: used for circular doubly linked list of siblings


[fig1]: /assets/FibHeap/fig1.png

[fig2]: /assets/FibHeap/fig2.png

[fig3]: /assets/FibHeap/fig3.png

[fig4]: /assets/FibHeap/fig4.png

[fig5]: /assets/FibHeap/fig5.png

[fig6]: /assets/FibHeap/fig6.png

[fig7]: /assets/FibHeap/fig7.png
