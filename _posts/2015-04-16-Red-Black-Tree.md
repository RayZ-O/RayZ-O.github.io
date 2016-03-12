---
layout: post
title: "Self Balancing Binary Search Tree 2 - Red-Black Tree"
categories: [all, datastructures]
date: 2015-04-16
author: Rui Zhang
---

The red-black tree is invented by Rudolf Bayer in 1972. It's one of the most popular self-balancing binary search tree. TreeMap/TreeSet in Java and std::map/std::set in C++ STL use red-black tree as underlying data structure.

### Property
Red-Black tree has 5 properties[^1].   
1. Every node is either red or black  
2. The root is black  
3. Every leaf is black  
4. There is no two adjacent red nodes  
5. For each node, all simple paths from the node to a leaf contain the same number of black nodes.  

A red-black tree:  
![fig1]  

The property above guarantees the height of a red-black tree is at most 2log(n+1). Therefore red-black tree guarantees common operations in O(log n) time.

|Space|O(n)|
|Search|O(log n)|
|Insert|O(log n)|
|Delete|O(log n)|
|Successor|O(log n)<sup>[1]</sup>|
|Predecessor|O(log n)<sup>[1]</sup>|
{: class="table table-striped table-nonfluid"}
[1]. without extra pointer

### Node Structure
A red-black tree node stores one extra field color for balancing. 
{% highlight c++ %}
enum class Color { RED, BLACK };

template <typename T, typename U>
struct RBTreeNode {
    T key;
    U value;
    Color color;
    RBTreeNode *parent;
    RBTreeNode *left;
    RBTreeNode *right;
    RBTreeNode() = default;
    RBTreeNode(T k, U v, RBTreeNode *l, RBTreeNode *r) : key(k), value(v), color(Color::RED),
                                                         parent(nullptr), left(l), right(r) { }    
};
{% endhighlight %}

An extra sentinel Nil is used instead of nullptr to simplify the implementation. All children of a leaf node point to Nil.

{% highlight c++ %}
template <typename T, typename U>
class RBTree {
private:
    RBTreeNode<T, U> *Nil;
    RBTreeNode<T, U> *root_;
    int size_;

    RBTreeNode<T, U>* GetUncle(RBTreeNode<T, U> *x);

    void LeftRotation(RBTreeNode<T, U> *x);

    void RightRotation(RBTreeNode<T, U> *x);

    void RBInsertFixup(RBTreeNode<T, U> *x);

    void RBTransplant(RBTreeNode<T, U> *u, RBTreeNode<T, U> *v);

    void RBDelete(RBTreeNode<T, U> *z);

    void RBDeleteFixup(RBTreeNode<T, U> *x);

    void Cleanup(RBTreeNode<T, U> *x);
public:
    RBTree() : size_(0) { 
        Nil = new RBTreeNode<T, U>();
        Nil->color = Color::BLACK;
        Nil->left = Nil;
        Nil->right = Nil;
        root_ = Nil;
    }

    ~RBTree() { 
        Cleanup(root_);
        delete Nil;
    }

    int size() const { return size_; }

    RBTreeNode<T, U>* Search(T key);

    void RBInsert(T key, U value);  

    void RBDelete(T key);

    RBTreeNode<T, U>* Successor(RBTreeNode<T, U> *z);

    RBTreeNode<T, U>* Predecessor(RBTreeNode<T, U> *z);
};
{% endhighlight %}

### Search, Successor, Predecessor
Search, Successor and Predecessor of red-black tree are the same as a standard BST.
{% highlight c++ %}
template <typename T, typename U>
RBTreeNode<T, U>* RBTree<T, U>::Search(T key) {
    RBTreeNode<T, U> *x = root_;
    while (x != Nil) {
        if (key == x->key) {
            return x;
        }
        else if (key < x->key) {
            x = x->left;
        }
        else {
            x = x->right;
        }
    }
    return nullptr;  // return nullptr if not found
}

template <typename T, typename U>
RBTreeNode<T, U>* RBTree<T, U>::Successor(RBTreeNode<T, U> *z) {
    RBTreeNode<T, U> *x = z->right;
    if (x != Nil) {
        while (x->left != Nil) {
            x = x->left;
        }
        return x;
    }
    else {
        RBTreeNode<T, U> *y = z->parent;
        while (y != Nil && z == y->right) {
            z = y;
            y = y->parent;
        }
        return y;
    }
}

template <typename T, typename U>
RBTreeNode<T, U>* RBTree<T, U>::Predecessor(RBTreeNode<T, U> *z) {
    RBTreeNode<T, U> *x = z->left;
    if (x != Nil) {
        while (x->right != Nil) {
            x = x->right;
        }
        return x;
    }
    else {
        RBTreeNode<T, U> *y = z->parent;
        while (y != Nil && z == y->left) {
            z = y;
            y = y->parent;
        }
        return y;
    }
}
{% endhighlight %}

### Insertion[^2]
The newly inserted node is always set to red. Similar to other self-balancing trees, to guarantee the red-black tree properties are preserved, auxiliary fix-up is needed after insertion and deletion. There are two kinds of fix-up for red-black tree: **Recoloring** and **Rotation**.

Rotation is discussed in details in the previous post [Self Balancing Binary Search Tree(1) - AVL Tree](http://rayz-o.github.io/blog/2015/04/01/AVL-Tree).  
They are modified to use Nil in if condition.
{% highlight c++ %}
template <typename T, typename U>
void RBTree<T, U>::LeftRotation(RBTreeNode<T, U> *x) {
    RBTreeNode<T, U> *y = x->right;
    x->right = y->left;
    if (y->left != Nil) {
        y->left->parent = x;
    }
    y->parent = x->parent;
    if (x->parent == Nil) {
        root_ = y;
    } else if (x == x->parent->left) {
        x->parent->left = y;
    } else {
        x->parent->right = y;
    }
    y->left = x;
    x->parent = y;
}

template <typename T, typename U>
void RBTree<T, U>::RightRotation(RBTreeNode<T, U> *x) {
    RBTreeNode<T, U> *y = x->left;
    x->left = y->right;
    if (y->right != Nil) {
        y->right->parent = x;
    }
    y->parent = x->parent;
    if (x->parent == Nil) {
        root_ = y;
    } else if (x == x->parent->right) {
        x->parent->right = y;
    } else {
        x->parent->left = y;
    }
    y->right = x;
    x->parent = y;
}
{% endhighlight %}

The function below performs standard binary search tree insertion and then fixes red-black tree property violation.

{% highlight c++ %}
template <typename T, typename U>
void RBTree<T, U>::RBInsert(T key, U value) {
    RBTreeNode<T, U> *y = Nil;
    RBTreeNode<T, U> *x = root_;
    while (x != Nil) {
        y = x;
        if (key == x->key) { 
            x->value = value;
            return;
        }
        else if (key < x->key) {
            x = x->left;
        }
        else {
            x = x->right;
        }
    }
    RBTreeNode<T, U> *z = new RBTreeNode<T, U>(key, value, Nil, Nil);
    z->parent = y;
    if (y == Nil) {
        root_ = z;
    } else if (key < y->key) {
        y->left = z;
    } else {
        y->right = z;
    }
    size_++;
    RBInsertFixup(z);
}
{% endhighlight %}
There are 6 cases in red-black tree insertion fix-up differ in uncle color or subtree structure.

Denote the newly inserted node as *x*.

**Case 1**: *x* is root.
Fix-up: Only need to change x's color to black
{% highlight c++ %}
if (x == root) {
    x->color = Color::BLACK;
}    
{% endhighlight %}
-------------------------------------------------------

In case 2-6 we need to know the color of the uncle node. The function below return the uncle of x. Note that null node is considered black in red-black tree.  
{% highlight c++ %}
template <typename T, typename U>
RBTreeNode<T, U>* RBTree<T, U>::GetUncle(RBTreeNode<T, U> *x) {
    RBTreeNode<T, U>* p = x->parent;
    RBTreeNode<T, U>* g = x->parent->parent;
    return p == g->left ? g->right : g->left; 
}
{% endhighlight %}
-------------------------------------------------------

**Case 2**: x's uncle is Red.   
![fig2]  
**Fix-up**:  
(1) set color of parent and uncle to black  
(2) change color of grandparent to red  
(3) reassign x = x's grandparent and run fix-up for new x.  

![fig3]  

**Implementation**: 
{% highlight c++ %}
if (u != Nil && u->color == Color::RED) {
    p->color = Color::BLACK;
    u->color = Color::BLACK;
    g->color = Color::RED;
    x = g;
}
{% endhighlight %}

-------------------------------------------------------

**Case 3**: x's uncle is Black, parent is left child of grandparent and x is left child of parent(LL).  
![fig4]  
**Fix-up**:  
(1) swap color of g and p  
(2) right rotation on g 

![fig5]  

**Implementation**: 
{% highlight c++ %}
if (u->color == Color::BLACK && x == p->left && p == g->left) { 
    p->color = Color::BLACK;
    g->color = Color::RED;
    RightRotation(g);
}
{% endhighlight %}

-------------------------------------------------------

**Case 4**: x's uncle is Black, parent is left child of grandparent and x is right child of parent(LR).  
![fig6]  
**Fix-up**:  
(1) left rotation on p  
(2) converted to LL case  

![fig7]  

**Implementation**: 
{% highlight c++ %}
if (u->color == Color::BLACK && x == p->right && p == g->left) { 
    LeftRotation(p);
    x->color = Color::BLACK;
    g->color = Color::RED;
    RightRotation(g);
}
{% endhighlight %}
-------------------------------------------------------

**Case 5**: x's uncle is Black, parent is right child of grandparent and x is right child of parent(RR).  
![fig8]  
**Fix-up**:  
(1) swap color of g and p  
(2) left rotation on g  

![fig9]  

**Implementation**: 
{% highlight c++ %}
if (u->color == Color::BLACK && x == p->right && p == g->right) { 
    p->color = Color::BLACK;
    g->color = Color::RED;
    LeftRotation(g);
}
{% endhighlight %}
-------------------------------------------------------

**Case 6**: x's uncle is Black, parent is right child of grandparent and x is left child of parent(RL).  
![fig10]  
**Fix-up**:  
(1) right rotation on p  
(2) converted to RR case  

![fig11]  

**Implementation**: 
{% highlight c++ %}
if (u->color == Color::BLACK && x == p->left && p == g->right) { 
    RightRotation(p);
    x->color = Color::BLACK;
    g->color = Color::RED;
    LeftRotation(g);
}
{% endhighlight %}
-------------------------------------------------------

#### **Complete Insertion Fix-up**

{% highlight c++ %}
template <typename T, typename U>
void RBTree<T, U>::RBInsertFixup(RBTreeNode<T, U> *x) {
    while (x != root_ && x->parent->color == Color::RED) { // x is red at the beginning of each iteration
        RBTreeNode<T, U>* u = GetUncle(x);
        RBTreeNode<T, U>* p = x->parent;
        RBTreeNode<T, U>* g = p->parent;   // g can't be NULL since x->parent->color == Color::RED
        if (u->color == Color::RED) { // uncle is red
            u->color = Color::BLACK;
            x = g;
        }
        else if (p == g->left) {
            if (x == p->right) {             // LR case                
                LeftRotation(p);
                std::swap(x, p);
            }
            RightRotation(g);                // LL case
        }
        else {
            if (x == p->left) {              // RL case                
                RightRotation(p);
                std::swap(x, p);
            }
            LeftRotation(g);                 // RR case
        }
        p->color = Color::BLACK;
        g->color = Color::RED;
    }
    root_->color = Color::BLACK;
}

{% endhighlight %}

### Deletion
Deletion is more complex than Insertion. It consists of a modified binary search tree deletion and a red-black property violation fix-up.

The standard binary search tree deletion is explained in details in the previous post [Self Balancing Binary Search Tree(1) - AVL Tree](http://rayz-o.github.io/blog/2015/04/01/AVL-Tree). 

The **RBDelete** function adds additional code to keep track with the node which may cause red-black property violation.

{% highlight c++ %}
template <typename T, typename U>
void RBTree<T, U>::RBTransplant(RBTreeNode<T, U> *u, RBTreeNode<T, U> *v) {
    RBTreeNode<T, U> *p = u->parent;
    if (p == Nil) {
        root_ = v;
    } else if (u == p->left) {
        p->left = v;
    } else {
        p->right = v;
    }
    v->parent = p;
}

template <typename T, typename U>
void RBTree<T, U>::RBDelete(T key) {
    RBTreeNode<T, U> *x = Search(key);
    if (x) {
        RBDelete(x);
        size_--;
    }
}

template <typename T, typename U>
void RBTree<T, U>::RBDelete(RBTreeNode<T, U> *z) {
    RBTreeNode<T, U> *x = Nil;
    RBTreeNode<T, U> *y = z;
    Color y_original_color = y->color;
    if (z->left == Nil) {
        x = z->right;
        RBTransplant(z, z->right);
    } else if (z->right == Nil) {
        x = z->left;
        RBTransplant(z, z->left);
    }
    else {
        y = z->right;
        while (y->left != Nil) {
            y = y->left;
        }
        y_original_color = y->color;
        x = y->right;
        if (y->parent == z) {
            x->parent = y;
        } else {
            RBTransplant(y, y->right);
            y->right = z->right;
            y->right->parent = y;
        }
        RBTransplant(z, y);
        y->left = z->left;
        y->left->parent = y;
        y->color = z->color;
    }

    z->left = Nil;
    z->right = Nil;
    delete z;

    if (y_original_color == Color::BLACK) {
        RBDeleteFixup(x);
    }
}
{% endhighlight %}
There are 8 cases in red-black tree deletion fix-up. The first four and the last four are symmetric.

**Case 1**: x's sibling w is red.

![fig16]  
**Fix-up**:  
(1) switch color of w and p  
(2) left rotation on p
(3) now x's sibling is a black node, case 1 is converted to case 2, 3 or 4

-------------------------------------------------------

**Case 2**: x's sibling w is black, and both of w's children are black.  

![fig17]  
**Fix-up**:  
(1) set color of w to red    
(2) left rotation on p  
(3) reassign p to x, and continue fix-up on new x

-------------------------------------------------------

**Case 3**: x's sibling w is black, w's left child is red, w's right child is black
![fig18]  
**Fix-up**:  
(1) switch color of w and w's left child  
(2) right rotation w  
(3) now sibling of x has a red right child, case 3 is converted to case 4

-------------------------------------------------------

**Case 4**: x's sibling w is black, and w's right child is red

(1)  w's left child is black  
![fig19]  

(2)  w's left child is red  
![fig20]  

**Fix-up**:  
 (1) switch color of p and w  
 (2) set w's right child to red  
 (3) left rotation on p

-------------------------------------------------------

**Case 5-8** are symmetric, same as Case 1-4 with "right" and "left" exchanged 

-------------------------------------------------------

**Implementation**:  

{% highlight c++ %}

template <typename T, typename U>
void RBTree<T, U>::RBDeleteFixup(RBTreeNode<T, U> *x) {
    while (x != root_ && x->color == Color::BLACK) {
        RBTreeNode<T, U> *p = x->parent;
        if (x == p->left) {
            RBTreeNode<T, U> *w = p->right;
            if (w->color == Color::RED) {
                w->color = Color::BLACK;
                p->color = Color::RED;
                LeftRotation(p);
                w = p->right;
            }
            if (w->left->color == Color::BLACK && w->right->color == Color::BLACK) {
                w->color = Color::RED;
                x = p;
            }
            else {
                if (w->right->color == Color::BLACK) {
                    w->left->color = Color::BLACK;
                    w->color = Color::RED;
                    RightRotation(w);
                    w = p->right;
                }
                w->color = p->color;
                p->color = Color::BLACK;
                w->right->color = Color::BLACK;
                LeftRotation(p);
                x = root_;
            }
        }
        else {
            RBTreeNode<T, U> *w = p->left;
            if (w->color == Color::RED) {
                w->color = Color::BLACK;
                p->color = Color::RED;
                RightRotation(p);
                w = p->left;
            }
            if (w->left->color == Color::BLACK && w->right->color == Color::BLACK) {
                w->color = Color::RED;
                x = p;
            }
            else {
                if (w->left->color == Color::BLACK) {
                    w->right->color = Color::BLACK;
                    w->color = Color::RED;
                    LeftRotation(w);
                    w = p->left;
                }
                w->color = p->color;
                p->color = Color::BLACK;
                w->left->color = Color::BLACK;
                RightRotation(p);
                x = root_;
            }
        }
    }
    x->color = Color::BLACK;
}
{% endhighlight %}
<br>

### **Cleanup:**  
The cleanup function recursively releases allocated nodes.
{% highlight c++ %}
template <typename T, typename U>
void RBTree<T, U>::Cleanup(RBTreeNode<T, U> *x) {
    if (x == Nil) {
        return;
    } 
    Cleanup(x->left);
    Cleanup(x->right);
    delete x;
}
{% endhighlight %}

<br>
**Reference:**

[^1]: [Introduction to Algorithms, 3rd Edition ](http://www.amazon.com/Introduction-Algorithms-Edition-Thomas-Cormen/dp/0262033844)

[^2]: [Geeksforgeeks: Red-Black Tree Set 2 (Insert)](http://www.geeksforgeeks.org/red-black-tree-set-2-insert)

[fig1]: /assets/RBTree/fig1.png

[fig2]: /assets/RBTree/fig2.png

[fig3]: /assets/RBTree/fig3.png

[fig4]: /assets/RBTree/fig4.png

[fig5]: /assets/RBTree/fig5.png

[fig6]: /assets/RBTree/fig6.png

[fig7]: /assets/RBTree/fig7.png

[fig8]: /assets/RBTree/fig8.png

[fig9]: /assets/RBTree/fig9.png

[fig10]: /assets/RBTree/fig10.png

[fig11]: /assets/RBTree/fig11.png

[fig16]: /assets/RBTree/fig16.png

[fig17]: /assets/RBTree/fig17.png

[fig18]: /assets/RBTree/fig18.png

[fig19]: /assets/RBTree/fig19.png

[fig20]: /assets/RBTree/fig20.png
