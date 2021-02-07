---
layout: post
comments: true
title: Yet Another Union Find Implementation
---

In this post I would like to show an alternative implementation of [Union Find (aka Disjoint Set)](https://en.wikipedia.org/wiki/Disjoint-set_data_structure) that uses variable reference, which is an interesting concept explained vividly in this [SO post](https://stackoverflow.com/questions/986006/how-do-i-pass-a-variable-by-reference). The idea is to represent the subset representative as a dictionary, so that during a merge when we update the subset representative of any member in the subset, all the other members in that subset sync with the update. One advantage of this implementation is that we don't need a Find function to trace the root of the subset representative, because the members in the same subset always have the same representative. The setup of the graph consists two classes: Node and Graph. A graph is made of node, and it also keeps track of all the edges. The graph is undirected and does not contain self loops. A node needs to keep track of two things: its subset representative and whether it has been merged into a subset or not. The subset representative of a node is initialized as itself. 

```python
class Node():
    def __init__(self,node_name):
        self.subset = {'root': node_name}
        self.merged = False
        
class Graph():
    def __init__(self):
        self.nodes = {}
        self.edges = set()
    
    def add_node(self, node_name):
        if node_name not in self.nodes:
            self.nodes[node_name] = Node(node_name)
            
    def add_edge(self,u,v):
        self.add_node(u)
        self.add_node(v)
        self.edges.add((u,v))
            
    def merge(self, x, y):
        if x.merged:
            y.subset['root'] = x.subset['root']
            if not y.merged:
                y.subset = x.subset
                y.merged = True
        else:
            x.subset = y.subset
            x.merged = True
            y.merged = True
            
    def is_cyclic(self):
        for edge in self.edges:
            x,y = edge
            #print (x,y)
            node_x = self.nodes[x]
            node_y = self.nodes[y]
            if node_x.subset['root'] == node_y.subset['root']:
                return True
            self.merge(node_x,node_y)
        return False
```

When checking a graph is cyclic or not, we loop through all the edges in the graph. We check the two vertices of the edges and merge one of them into the other's subset. When merging, it's necessary to check whether the node we are about to merge has already been merged into a subset or not. See the merge function taken from above:
```python
    def merge(self, x, y):
        if x.merged:
            y.subset['root'] = x.subset['root']
            if not y.merged:
                y.subset = x.subset
                y.merged = True
        else:
            x.subset = y.subset
            x.merged = True
            y.merged = True
```

When the node has been merged, it means that there are other nodes that sync with the subset representative of this node. So, we need to change the 'root' value of its subset representative to that of the other node. This corresponds to `y.subset['root'] = x.subset['root']` in the merge function. But if the node has never not been merged, we need to reassign its subset representative altogether to that of the other node, because we need to establish the sync. This corresponds to `y.subset = x.subset`. It's the key part of the implementation to determine when to change the "root" value of the subset representative and when to reassign the subset representative. Afterwards, we need to update the merged status of an unmerged node to "True", and that's the most of it. To test the implementation on a graph:
```python
A = Graph()
A.add_edge(1,2)
A.add_edge(2,3) 
A.add_edge(2,4)
A.add_edge(4,5)
A.add_edge(5,6) 
A.add_edge(5,7)
A.add_edge(3,6)

print (A.is_cyclic())

# True
```
