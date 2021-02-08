---
layout: post
comments: true
excerpt_separator: <!--more-->
title: Yet Another Union Find Implementation
---

In this post I would like to show an alternative implementation of [Union Find (aka Disjoint Set)](https://en.wikipedia.org/wiki/Disjoint-set_data_structure) that uses variable reference, which is an interesting concept explained vividly in this [SO post](https://stackoverflow.com/questions/986006/how-do-i-pass-a-variable-by-reference). The idea is to keep track of the subset of a node as a list, like so:
<!--more-->
```python
class Node():
    def __init__(self,node_name):
        self.subset = [node_name]
```
When merging two nodes x and y, suppose x is being merged into y, then we first add the subset of x to the subset of y, and then reassign the subset of x to the subset of y. Whichever node has a smaller subset should be merged, which is equivalent to [Union by Size](https://en.wikipedia.org/wiki/Disjoint-set_data_structure#Merging_two_sets). This part can be implemented like this: 
```python
def merge(x, y):
    if len(x.subset) < len(y.subset):
        x.subset.extend(y.subset)
        for node in y.subset:
	    node.subset = x.subset
    else:
        y.subset.extend(x.subset)
        for node in x.subset:
	    node.subset = y.subset
```
The key part of merge is to modify the subset of a node **in place** if it has a larger subset, so that all the other nodes in the same subset will sync with the change because their subset refer to the same variable. On the other hand, we need to **reassign** the subset of a node if it is in the smaller subset, as well as reassign the subset of all the other nodes in the same subset. One advantage of this implementation is that we don't need a Find function to trace the subset representative, because the members in the same subset always sync with other's subset representative. Talking about subset representative, We can just make the first node in the subset list as the subset representative. The complete implementation is the following:
```python
class Node():
    def __init__(self,node_name):
        self.subset = [node_name]
        
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
        if len(x.subset) < len(y.subset):
            x.subset.extend(y.subset)
            for node_name in y.subset:
                self.nodes[node_name].subset = x.subset
        else:
            y.subset.extend(x.subset)
            for node_name in x.subset:
                self.nodes[node_name].subset = y.subset
            
    def is_cyclic(self):
        for edge in self.edges:
            x,y = edge
            #print (x,y)
            node_x = self.nodes[x]
            node_y = self.nodes[y]
            if node_x.subset[0] == node_y.subset[0]:
                return True
            self.merge(node_x,node_y)
        return False
```
The setup of the graph consists two classes: Node and Graph. A graph is made of nodes, and it also keeps track of all the edges. When checking a graph is cyclic or not, first we loop through all the edges in the graph. For each edge, we check whether its two vertices already have the same subset representative or not. If not, we merge them. Otherwise, we found a cycle. To run some tests:
```python
A = Graph()
A.add_edge(6,7)
A.add_edge(2,8) 
A.add_edge(5,6)
A.add_edge(0,1)
A.add_edge(2,5) 
A.add_edge(2,3)
A.add_edge(0,7)
A.add_edge(1,2)
print (A.is_cyclic())

# True
```
