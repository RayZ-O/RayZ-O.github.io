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
Tarjan's algorthm is published by Robert Tarjan in 1972. He's also the co-inventor of both [splay trees](https://en.wikipedia.org/wiki/Splay_tree) and [Fibonacci heaps](https://en.wikipedia.org/wiki/Fibonacci_heap)[^2].

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
When a vertex u is traversing its neighbours  
**1.Tree edge:** if a neighbours v is UNSEEN, then Edge (u, v) is a tree edge.  
**2.Back edge:** if a neighbours v is already mark as SEEN but not FINISH, then Edge (u, v) is a back edge.

![fig2]  
This is the DFS tree of the above graph, green edges are tree edges and red edges are back edges. Tree edges form a spanning tree of the graph. Back edges point to an ancestor in the DFS tree.

Back edges provide another important information about a graph: A directed graph is acyclic if and only if it has no back edges.

#### Algorithm details
As mentioned above, Tarjan's algorthm maintain a stack to store the nodes that not yet assiged to a component. Moreover, we need to add 2 metadata field to each vertex: discovered time and "low value".

* discovered time stores when this vertex is first discovered
* The low value is tricky. In the DFS-tree form by all Tree Edges, low of a vertex u is the dicovered time of the topmost reachable ancestor via the subtree of u[^3]. For a vertex u, the low value records the minimum value among: 
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
Notice that when (u, v) is a tree edge, the comparison between low of u and low of u's neighbour v is made after DFS vertex v, because low of v is undefined before visiting v.

It's somewhat complex. Let's see a live example:  

1. Start from vertex A, A.discover = A.low = 1.  
![fig3]  

2. Visit B, G, F, update discover and low.  
![fig4] ![fig5] ![fig6]  

3. Encounter back edge (F, B), update F.low = B.discover.  
![fig7]  

4. All edges of F are visited, back track to G, F.low is smaller than G.low, update G.low = F.low.  
![fig8]  

5. Visit H, I.  
![fig9] ![fig10]

6. Find back edge (I, H), update I.low = H.low.  
![fig11]  

7. Visit J, J has no out edge, back track to B.  
![fig12]  

8. Visit D, E.  
![fig13] ![fig14]  

9. See J again, (E, J) is not tree edge or back edge, nothing to do.  
![fig15]  

10. Update E.low = D.discover.  
![fig16]  

11. Back track to A, (A, F) is not tree edge or back edge.  
![fig17]  

The magic happens. All vertices in the same component have the same low value.
To track the strongly connected components, we can use a stack to store the vertices during DFS. For a vertex u, if its discover time is equal to low value after visiting all its neighbours, it is the root of a strongly connected component. Pop vertices from the stack till we get u, they form a strongly connected component rooted at u.

{% highlight c++ %}
enum class State {SEEN, UNSEEN, FINISHED}; // since C++11
int time = 0;                              // global timestamp
std::vector<std::vector<int>> SCC;

void dfs(std::vector<std::vector<int> &graph, int u, stack<int>& st) {
    time++;
    graph[u].discover = time;
    graph[u].low = time;    
    graph[u].state = State::SEEN;
    st.push(u);
    for (int v : graph[u].adjs) {
        if (graph[v].state == State::UNSEEN) { // Tree Edges
            dfs(graph, v);
            graph[u].low = min(graph[u].low, graph[v].low);
        } else if (graph[v].state == State::SEEN) {   // Back Edges
            graph[u].low = min(graph[u].low, graph[v].discover);
        }
    }    
    graph[v].state = State::FINISHED;

    if (graph[u].discover == graph[u].low) {
        vector<int> component;
        while (st.top() != u) {
            component.push_back(st.top());
            st.pop();
        }
        component.push_back(st.top());
        st.pop();
        SCC.push_back(component);
    }
}
{% endhighlight %}

**Reference:**

[^1]: [Introduction to Algorithms, 3rd Edition](http://www.amazon.com/Introduction-Algorithms-Edition-Thomas-Cormen/dp/0262033844)

[^2]: [Wikipedia: Strongly connected component](https://en.wikipedia.org/wiki/Strongly_connected_component)

[^3]: [GeeksforGeeks: Tarjan’s Algorithm to find Strongly Connected Components](http://www.geeksforgeeks.org/tarjan-algorithm-find-strongly-connected-components/)

[fig1]: /assets/SCC/fig1.png

[fig2]: /assets/SCC/fig2.png

[fig3]: /assets/SCC/fig3.png

[fig4]: /assets/SCC/fig4.png

[fig5]: /assets/SCC/fig5.png

[fig6]: /assets/SCC/fig6.png

[fig7]: /assets/SCC/fig7.png

[fig8]: /assets/SCC/fig8.png

[fig9]: /assets/SCC/fig9.png

[fig10]: /assets/SCC/fig10.png

[fig11]: /assets/SCC/fig11.png

[fig12]: /assets/SCC/fig12.png

[fig13]: /assets/SCC/fig13.png

[fig14]: /assets/SCC/fig14.png

[fig15]: /assets/SCC/fig15.png

[fig16]: /assets/SCC/fig16.png

[fig17]: /assets/SCC/fig17.png
