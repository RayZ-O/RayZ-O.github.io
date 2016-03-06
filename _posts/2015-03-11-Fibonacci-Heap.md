---
layout: post
title: "The power of amortization: Fibonacci Heap"
categories: [all, datastructures]
date: 2015-3-11
author: Rui Zhang
---

### Amortized Complexity
For some data structures such as Fibonacci Heap and Splay Tree, it is impossible for every operation in a sequence to take the worst-case time. In this case, the amortized analysis is a more clever reasoning way to provide a tighter bound. Amortized complexity concerns with the overall costs of a sequence of operations and guarantees the average performance of each operation in the worst case.

### Fibonacci heap
Fibonacci heap is invented by Michael L. Fredman and Robert E. Tarjan in 1984. It's a fancy data structure that has two advantages over the Binary heap:  

* The Fibonacci heap is a [meragable heap](https://en.wikipedia.org/wiki/Mergeable_heap)  
* The Fibonacci hHeap provides O(1) amortized time insert and decrease key operations, which makes it perfectly suited for applications that uses these operations frequently such as Dijkstra's Algorithm and Prim's Algortihm.

|Operation|Binary heap(worst-case)|Fibonacci Heap(amortized)|
|:---|:---:|:---:|
|*Make-Heap*<sup>1</sup>|O(1)   |O(1)|
|*Insert*      |O(lg N)|O(1)|
|*Get-Min*     |O(1)   |O(1)|
|*Extract-Min* |O(lg N)|O(lg N)|
|*Merge*       |O(n)   |O(1)|
|*Decrease-Key*<sup>2</sup>|O(lg N)|O(1)|
|*Delete*<sup>2</sup>  |O(lg N)|O(lg N)|
{: class="table table-striped table-nonfluid"}

1. creates a new empty heap.  
2. require a pointer to the target node as input because search is expensise in both binary heap and fibonacci heap.
 
From a theoretical perspective, all operations of Fibonacci heap are at least as fast as binary heap, thus fibonacci heap is obviously better than binary heap. However, because of the constant factor and implementation complexity, it's only attractive when the input size is large[^1].

#### Structure overview
A Fibonacci heap is a forest that the key of a tree node is greater than or equal to its parent. Each node has a pointer to its parent, a pointer to one of its children and pointers to left sibling and right sibling. Children of a node are linked together in a cicular, doubly linked list. Roots of all trees in the forest are linked in the same manner. An extra pointer is maintained for accessing the minimum node in the heap.  
![fig1]  

#### Node structure

{% highlight c++ %}
template <typename T>
struct FibNode {
    T key;
    int degree;
    bool childCut;
    FibNode *parent;
    FibNode *child;
    FibNode *lsibling;
    FibNode *rsibling;
    FibNode(T d) : key(d), degree(0), childCut(false), parent(nullptr), 
                   child(nullptr), lsibling(this), rsibling(this) {}
};
{% endhighlight %}

* key: key associated with the node
* degree: number of children
* childCut: 
    * true if node has lost a child since it became a child of its current parent. 
    * Set to false by remove min, which is the only operation that makes one node a child of another. 
    * Set ot false for a root nodes.
* parent: point to the parent node
* child: point to any of the children
* lsibling, rsibling: used for circular doubly linked list of siblings

#### Potential Function
Use the potential method to analyze the performance of Fibonacci heap.
Define the potential of Fibonacci heap H by:  
Φ(H) = t(H) + 2m(H).

* t(H): number of trees in H
* m(H): number of nodes whose childCut equal to true in H

#### *Make-Heap*
Make an empty heap. Since t(H) = 0 and m(H) = 0, the amortized cost of *Make-Heap* is equal to its actual cost O(1).
{% highlight c++ %}
template <typename T>
class FibHeap   {
    FibNode<T>* min_;
    int size_;
public:
    FibHeap(): min_(nullptr), size_(0) {  }
    int Size() const { return size_; } 
};
{% endhighlight %}

#### *Insert*
Insert a node to the top level doubly linked list. It takes O(1) to insert a node in doubly linked list.  
After inserting a node to H, t(H') = t(H) + 1, m(H') = m(H). The  increase in potential is:  
Φ(H') - Φ(H) = (t(H) + 1) + 2m(H) - (t(H) + 2m(H)) = 1  
Thus the amortized cost of insertion is O(1) + 1 = O(1).

{% highlight c++ %}
template <typename T>
void FibHeap<T>::Insert(T key) {
    FibNode<T> node = new FibNode(key);
    if (!min_) {
        min_ = node;
    } else {
        insert_list_node(min_, node);
        if (key < min_->key) {
            min_ = node;
        }
    }
    size_++;
}

template <typename T> // helper function to insert a node into doubly linked list
void FibHeap<T>::insert_list_node(FibNode<T>* &li, FibNode<T>* node) { 
    if (!li) {
        li = node;
    } else {
        li->rsibling->lsibling = node;
        node->rsibling = li->rsibling;
        li->rsibling = node;
        node->lsibling = li;
    }    
}
{% endhighlight %}

#### *Get-Min*
Since we have a pointer pointing to the minimum node in the heap, we can get min in O(1). Because the potential doesn't change, the amortized cost is equal to the actual cost O(1).
{% highlight c++ %}
template <typename T>
FibNode<T> FibHeap<T>::GetMin() {
    return min_;
}
{% endhighlight %}

#### *Merge* 
Simply merge two top level linked lists and find the new minimum node.
The change in potential is:  
Φ(H) - (Φ(H1)+Φ(H2))= t(H) + 2m(H) - (t(H1) + 2m(H1) + t(H2) + 2m(H2)) = 0  
Therefore the amortized cost is equal to the actual cost O(1). 
{% highlight c++ %}
template <typename T>
FibHeap<T> FibHeap<T>::Merge(FibHeap<T> h1, FibHeap<T> h2) {
    if (h1.Size() == 0) {
        reutrn h2;
    } else if (h2.Size() == 0) {
        reutrn h1;
    } else { // merge linked lists
        h1->min_->lsibling->rsibling = h2->min_->lsibling;
        h2->min_->lsibling->rsibling = h1->min_->lsibling;
        h2->min_->lsibling = h1->min_;
        h1->min_->lsibling = h2->min_;
        if (h2->min_->key < h1->min_->key) { // find the new minimum node
            h1->min_ = h2->min_;
        }
        h1->size_ += h2.size_;
        return h1;
    }
}
{% endhighlight %}

#### *Extract-Min*
Extract min works by first inserting all children of the minimum node to the root list and removing the minimum node from the root list. Then pairwise combines roots of equal degree until at most one root in each degree.  

1. Insert all children of the minimum node to the root list and remove the minimum node from the root list.  
![fig2]  

2. Pairwise combines roots of equal degree until at most one root in each degree.  
![fig3]  
![fig4]  
![fig5]  

3. Adjust the min pointer.  
![fig6]  

To analyze the amortized cost of *Extract-Min*, we need to know the maximum degree of any node in a n-node  Fibonacci heap. Denote the maximum degree as D(n). D(n) = O(lg n)[^2].  
The size of the root list is at most D(n) + t(H) - 1 after inserting all children of min node to root list and removing min node from root list. The actual cost of *Extract-Min* is O(D(n) + t(H)).  
After *Pairwise-Combine*, the number of trees in the Fibonacci heap is at most D(n) + 1, since the maximum degree of any root in the heap is D(n) and at most one root in each degree.  
The amortized cost is at most  
O(D(n) + t(H)) + (D(n) + 1 + 2m(H)) - (t(H) + 2m(H)) = O(D(n)) + O(t(H)) - t(H) = O(D(n)) = O(lg n)  
Since in *Extract-Min*, the constant factor in O(t(H)) is 1.

{% highlight c++ %}
template <typename T>
FibNode<T> FibHeap<T>::ExtractMin() {
    FibNode<T>* z = min_;
    if (!z) {
        if (z->child) {
            FibNode<T> *c = z->child;
            do {
                insert_list(z, c);    // inserts children to root list
            } while (c != z->child);            
        } 
        remove_list_node(z);          // removes the minimum node
        if (size_ == 1) {
            min_ = nullptr;
        } else {
            min_ = z->rsibling;
            PairwiseCombine();        // pairwise combines roots of equal degree
        }
        size_--;
    }
    return z;
}

template <typename T> // helper function to delete a node into doubly linked list
void FibHeap<T>::remove_list_node(FibNode<T>* node) { 
    node->rsibling->lsibling = node->lsibling;
    node->lsibling->rsibling = node->rsibling;
}

template <typename T>
void FibHeap<T>::PairwiseCombine() {
    std::vector<FibNode<T>*> roots(log2(size_)+1, nullptr);
    FibNode<T>* x = min_;
    do {
        FibNode<T>* next = x->rsibling;
        int d = x->degree;
        while (roots[d]) {
            FibNode<T>* y = roots[d];
            if (x->key > y->key) {
                std::swap(x, y);
            }
            Link(x, y);
            roots[d] = nullptr;
            d++;
        }
        roots[d] = x;
        x = next;
    } while (x != min_);
    min_ = nullptr;
    for (auto r : roots) {
        if (!r) {
            continue;
        }
        if (!min_) {
            min_ = r;
        } else {
            insert_list_node(min_, r);
            if (r->key < min_->key) {
                min_ = r;
            }
        }        
    }
}

template <typename T>
void FibHeap<T>::Link(FibNode<T>* x, FibNode<T>* y) {
    remove_list_node(y);
    insert_list_node(x->child, y);
    x->degree++;
    y->childCut = false;
}
{% endhighlight %}

#### *Decrease-Key*
If the key of a node is less than its parent after *Decrease-Key*, cut the node and insert to the root list. Follow path from parent of the node to the root, encountered nodes with childCut = true are cut from their sibling lists and inserted into root list. Stop at first node with childCut = false, if it's not root, set childCut = true for this node.  
Here is an example(Sibling pointers is not shown, red nodes have childCut = true):  
![fig7]  

1. Decrease 12 to 5 and cut the subtree rooted at the decreased node, insert it into root list.
![fig8]  

2. Follow path from parent of the node to the root, if encounter nodes with childCut = true, cut the subtree and insert to root list.  
![fig9]  
![fig10]  

3. Stop at first node with childCut = false and update childCut.  
![fig11]  

Denote the number of *Cascading-Cut* invocation as c, the actual cost of *Decrease-Key* is O(1) + O(c).  
During the process of *Decrease-Key*, at most c - 1 nodes with childCut = true change to false and the last call changes a node to true, the change of potential is:  
(t(H) + c) + 2(m(H) - (c - 1) + 1) - (t(H) + 2m(H)) = 4 - c  

Thus, the amortized cost of *Decrease-Key* is O(c) + 4 - c = O(1).

{% highlight c++ %}
template <typename T>
void FibHeap<T>::DecreaseKey(FibNode<T>* node, T key) {
    if (key > node->key) {
        return;
    }
    node->key = key;
    FibNode<T>* p = node->parent;
    if (p && node->key < p->key) {
        Cut(p, node);
        CascadingCut(p); // cascading cut from parent to root
    }
    if (node->key < min_->key) {
        min_ = node;
    }
}

template <typename T>
void FibHeap<T>::Cut(FibNode<T>* parent, FibNode<T>* node) {
    remove_list_node(node);
    parent->degree--;
    insert_list_node(min_, node);
    node->childCut = false;
}

template <typename T>
void FibHeap<T>::CascadingCut(FibNode<T>* node) {
    FibNode<T>* p = node->parent;
    if (p) {
        if (!node->childCut) { 
            node->childCut = true;
        } else {
            Cut(p, node);
            CascadingCut(p);
        }
    }
}
{% endhighlight %}

#### *Delete*
*Decrease-Key* and *Extract-Min* can be used to implement *Delete*.
{% highlight c++ %}
template <typename T>
void FibHeap<T>::Delete(FibNode<T>* node) {
    DecreaseKey(node, min_->key);
    ExtractMin();
}
{% endhighlight %}

**Reference**  

[^1]: [Introduction to Algorithms, 3rd Edition](http://www.amazon.com/Introduction-Algorithms-Edition-Thomas-Cormen/dp/0262033844)

[^2]: [University of Florida COP 5536 lecture notes: Analysis of Fibonacci heaps](http://www.cise.ufl.edu/~sahni/cop5536/powerpoint/all.zip)

[fig1]: /assets/FibHeap/fig1.png

[fig2]: /assets/FibHeap/fig2.png

[fig3]: /assets/FibHeap/fig3.png

[fig4]: /assets/FibHeap/fig4.png

[fig5]: /assets/FibHeap/fig5.png

[fig6]: /assets/FibHeap/fig6.png

[fig7]: /assets/FibHeap/fig7.png

[fig8]: /assets/FibHeap/fig8.png

[fig9]: /assets/FibHeap/fig9.png

[fig10]: /assets/FibHeap/fig10.png

[fig11]: /assets/FibHeap/fig11.png
