---
layout: post
title: "Strongly Connected Components: Tarjan's algorithm"
categories: [all, algorithms]
date: 2015-09-02
author: Rui Zhang
---

### Strongly Connected Components
A strongly connected component of a directed graph is a maximal subset of vertices C, such that for every pair of vertices u and v in C, vertices u and v are reachable from each other[^1].  
For example, in the following graph, the strongly connected components are {A} {B, F, G} {D, E} {H, I} {J}  

![fig1]

There are two classic algorithms solve strongly connected component problem in linear time. Both of them are [DFS](https://en.wikipedia.org/wiki/Depth-first_search)-based algorithms. Kosaraju's algorthm uses two passes of DFS while Tarjan's algorthm runs one pass of DFS and maintians a stack for nodes that have been visited but not yet assigned to any component.

### Tarjan's algorthm
Tarjan's algorthm is published by Robert Tarjan in 1972. He's also the co-inventor of both [splay trees](https://en.wikipedia.org/wiki/Splay_tree) and [Fibonacci heaps](https://en.wikipedia.org/wiki/Fibonacci_heap).

#### Tree Edge & Back Edge
During the DFS, there are three possible states for a vertex: seen, unseen, finished. Initally all vertice are marked as unseen, when an unseen vertex is first visited, its state changes to seen. After a vertex traverse all its neighbours, it becomes finished. Here comes the C++ code:  

{% highlight c++ %}
enum class State {SEEN, UNSEEN, FINISHED}; // since C++11

void dfs(std::vector<std::vector<int> &graph, int u) {
    graph[u].state = State::SEEN;
    for (int v : graph[u].adjs) {
        if (graph[v].state == State::UNSEEN) {
            dfs(graph, v);
        }
    }    
    graph[u].state = State::FINISHED;
}
{% endhighlight %}

**Tree edge:** When a vertex u is traversing its adjacency list, if a neighbours v is UNSEEN, then Edge (u, v) is a tree edge.  
**Back edge:** When a vertex u is traversing its neighbours, if a neighbours v is already mark as SEEN, then Edge (u, v) is a back edge.

The type of edges provide important information about a graph:

* For a connected graph, tree edges form a spanning tree of the graph  
* A directed graph is acyclic if and only if it has no back edges.

#### Algorithm details
As mentioned above, Tarjan's algorthm maintain a stack to store the nodes that not yet assiged to a component. Moreover, we need to add 2 metadata field to each vertex: discovered time and "low value".

* discovered time stores when this vertex is first discovered
* The low value is tricky. In the DFS-tree form by all Tree Edges, low of a vertex u is the dicovered time of the topmost reachable ancestor via the subtree of u. For a vertex u, the low value records the minimum value among: 
	* u's own discovered time
	* low value of a neighbour v where (u, v) is a tree edge
	* the discovered time of a neighbour v where (u, v) is a back edge



The following code shows how to update discover(discovered time) and low(low value) during DFS.
{% highlight c++ %}
enum class State {SEEN, UNSEEN, FINISHED}; // since C++11
int time = 0;                              // global timestamp

void dfs(std::vector<std::vector<int> &graph, int u) {
    time++;
    graph[u].discover = time;
    graph[u].low = time;    
    graph[u].state = State::SEEN;

    for (int v : graph[u].adjs) {
        if (graph[v].state == State::UNSEEN) { // Tree Edges
            dfs(graph, v);
            graph[u].low = min(graph[u].low, graph[v].low);
        } else if (graph[v].state == State::SEEN) {   // Back Edges
            graph[u].low = min(graph[u].low, graph[v].discover);
        }
    }    
    graph[v].state = State::FINISHED;
}
{% endhighlight %}




Reference:

[^1]: [Introduction to Algorithms, 3rd Edition](http://www.amazon.com/Introduction-Algorithms-Edition-Thomas-Cormen/dp/0262033844)

[fig1]: /assets/SCC/fig1.png

[fig2]: /assets/SCC/fig2.png

[fig3]: /assets/SCC/fig3.png

[fig4]: /assets/SCC/fig4.png

[fig5]: /assets/SCC/fig5.png

[fig6]: /assets/SCC/fig6.png

[fig7]: /assets/SCC/fig7.png
