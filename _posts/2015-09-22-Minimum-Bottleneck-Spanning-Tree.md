---
layout: post
title: "Minimum Bottleneck Spanning Tree: Camerini's algorithm"
categories: [all, algorithms]
date: 2015-09-22
author: Rui Zhang
---

### Minimum bottleneck spanning tree
A minimum bottleneck spanning tree (MBST) in an undirected graph is a spanning tree in which the most expensive edge is as cheap as possible[^1]. The most expensive edge in a minimum bottleneck spanning tree called **bottleneck edge**.  

### MBST v.s. MST[^2]
**1. A minimum bottleneck spanning tree is not necessarily a [minimum spanning tree](https://en.wikipedia.org/wiki/Minimum_spanning_tree).**   
Here is a counterexample:  

![fig1]  
In this graph, the minimum spanning tree will be:  
![fig2]  
However, the minimum bottleneck spanning tree can be:  
![fig3]  

**2. A minimum spanning tree is necessarily a minimum bottleneck spanning tree.**  
Proof by contradiction:  
Assume e is the bottleneck edge of MST and e dose not appear in the MBST. Add e to the MBST will form a cycle in the MBST, since weight of e is greater than all edges in MBST, e should be the most expensive edge in the cycle. According to the **cycle property** of MST:

> For any cycle C in the graph, if the weight of an edge e of C is larger than the individual weights of all other edges of C, then this edge cannot belong to a MST.

Thus e should not be an edge in MST, which implies a contradiction.

### Camerini's algorithm
Camerini propesed an algorithms to obtain a MBST in 1978.  
It works as following:
Divide the edges of the graph into 2 sets S and L, the weights of edges in S is smaller than all edges in L.:

* If the subgraph form by edges in S is connected, then the MBST of the subgraph is the MBST of the original graph.  
* Otherwise, we must use some of the edges in L. Since all edges in S are smaller than edges in L, we can pick an arbitrary spanning tree(not necessarily minimum) of each connected component in the subgraph and add the edges in the spaning trees to the final result. Then merge all nodes in each connected component into a new big vertex, and compute the MBST for the graph that form by the big vertices and edges in L.   

Repeat until there are only two big vertices left and one edge connects them.  
Let's see an example:  

![fig4]  

**(1)**Firstly, we divide the edges into 2 sets, red edges are in the smaller set:  

![fig5]  
The subgraph only form by red edges is connected, thus the MBST will be in this subgraph, we don't need to consider edges in larger set. 

![fig6]  

**(2)**Again divide the edges into 2 sets, this time the subgraph form by red edges is not connected:  

![fig7]  
Grow spanning forest using only red edges and collect edges in each spanning tree, we got MBST = {AD, DF, CE}  
Merge the {A, D, F} in the left spanning tree into a new big vertex ADF and similarly {C, E} into CE:  

![fig8]![fig11]  

**(3)**Partition the edges in the graph produced in previous iteration, the subgraph is not connected:  

![fig9]![fig12]  
Collect 2 edges in the spanning tree form by red edge above. Now MBST = {AD, DF, CE, BE, EG}  
Combine {B, CE, G} into a big fat vertex BCEG  

![fig10]![fig13]  
Now we have only two big vertices left and one edge connects them. We collect the final edge AB and terminate.  
The MBST of the original graph is {AD, DF, CE, BE, EG, AB}.  

### Reference

[^1]: [Wikipedia: Minimum_bottleneck_spanning_tree](https://en.wikipedia.org/wiki/Minimum_bottleneck_spanning_tree)
[^2]: [Applications of Minimum Spanning Trees](http://courses.cs.vt.edu/cs5114/spring2009/lectures/lecture08-mst-applications.pdf)


[fig1]: /assets/MBST/fig1.jpg

[fig2]: /assets/MBST/fig2.jpg

[fig3]: /assets/MBST/fig3.jpg

[fig4]: /assets/MBST/fig4.png

[fig5]: /assets/MBST/fig5.png

[fig6]: /assets/MBST/fig6.png

[fig7]: /assets/MBST/fig7.png

[fig8]: /assets/MBST/fig8.png

[fig9]: /assets/MBST/fig9.png

[fig10]: /assets/MBST/fig10.png

[fig11]: /assets/MBST/fig11.png

[fig12]: /assets/MBST/fig12.png

[fig13]: /assets/MBST/fig13.png
