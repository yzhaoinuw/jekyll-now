---
layout: post
comments: true
excerpt_separator: <!--more-->
title: Tracing Depth First Search
---

Here's a variation of the [Depth First Search](https://en.wikipedia.org/wiki/Depth-first_search) (DFS) that traces the path of the search.
<!--more-->
```python
class Node():
    def __init__(self,node_name):
        self.outward = set()
        self.inward = set()
        
class Graph():
    def __init__(self):
        self.nodes = {}
    
    def add_node(self, node_name):
        if node_name not in self.nodes:
            self.nodes[node_name] = Node(node_name)
            
    def add_edge(self,u,v):
        self.add_node(u)
        self.add_node(v)
        self.nodes[u].outward.add(v)
        self.nodes[v].inward.add(u)
        
    def search_path(self, node_name, path, tracks):
        dead_end = True
        path.append(node_name)  
        #print (path)
        outs = self.nodes[node_name].outward
        for out in outs:
            if out not in path:
                self.search_path(out, list(path), tracks)
                dead_end = False
        if dead_end:
           tracks.append(path)
            
    def show_DFS_tracks(self,start_point):
        path = []
        tracks = []
        self.search_path(start_point, path, tracks)
        return tracks
```
When you run it on a directed graph with a given start point, it returns all the paths DFS has searched. Each path terminates at a node which has no neighbors that have not been in the path. For example, starting from node 2 of the following graph, we get
```python
A = Graph()
A.add_edge(0,1)
A.add_edge(0,4) 
A.add_edge(1,5) 
A.add_edge(1,2)
A.add_edge(2,0)
A.add_edge(2,1)
A.add_edge(2,6)
A.add_edge(3,1) 
A.add_edge(3,5) 
A.add_edge(4,6)
A.add_edge(4,7)
A.add_edge(5,1)
A.add_edge(5,6) 
A.add_edge(6,4) 
A.add_edge(6,5)
A.add_edge(7,5)

print (A.show_DFS_tracks(2))

# [[2, 0, 1, 5, 6, 4, 7], [2, 0, 4, 6, 5, 1], [2, 0, 4, 7, 5, 1], [2, 0, 4, 7, 5, 6], [2, 1, 5, 6, 4, 7], [2, 6, 4, 7, 5, 1], [2, 6, 5, 1]]
```
The key difference between this variation and the DFS is that the search paths in DFS share one record of the nodes visited, whereas in this variation each search path has a record of the nodes visited on itself. To do this, we need to make each search path track itself, and not affected by other paths. In addition, we need to let each search path know when to terminate, namely, when it runs into a node that has no neighbors that are not in the path. This happens either when the node in question is an end node in a graph, or when all its outgoing edges lead back to a node that has already been visited on this search path. The key part of this implementation is in the function `search_path`. When it calls itself in the recursion, we need to create a copy of the current path by calling `search_path(out, list(path), tracks)`, notice the "list()" around "path". This way, a separate copy of the current path is created and passed on for subsequent search. It is crucial to create a separate copy because, otherwise, all the search path still share the same path in effect. Other ways of creating a separated copy of a list are discussed in this [SO post](https://stackoverflow.com/questions/2612802/list-changes-unexpectedly-after-assignment-how-do-i-clone-or-copy-it-to-prevent).

