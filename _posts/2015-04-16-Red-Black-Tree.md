---
layout: post
title: "Self Balanced Search Tree(1) - Red-Black Tree"
categories: [all, datastructures]
date: 2015-04-16
author: Rui Zhang
---

Red-Black tree is invented by Rudolf Bayer in 1972. It's one of the most famous self-balancing binary search tree. TreeMap/TreeSet in Java and std::map/std::set in C++ STL use red-black tree as underlying data struture. Other popular self-balancing search trees include 2-3 tree, AA tree, AVL tree, Splay tree and Treap[^1].

### Property
Red-Black tree has 5 properties[^2].   
1. Every node is either red or black  
2. The root is black  
3. Every leaf is black  
4. No two adjacent red nodes  
5. For each node, all simple paths from the node to a leaf contain the same number of black nodes.  


**Reference:**

[^1]: [Wikipedia: Self-balancing binary search tree](https://en.wikipedia.org/wiki/Self-balancing_binary_search_tree)

[^2]: [Introduction to Algorithms, 3rd Edition](http://www.amazon.com/Introduction-Algorithms-Edition-Thomas-Cormen/dp/0262033844)
