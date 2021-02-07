---
layout: post
title: Alternative Union Find Implementation
---

In this post I would like to show an alternative implementation of [Union Find](https://en.wikipedia.org/wiki/Disjoint-set_data_structure) that uses veriable reference, which is an interesting concept explained vividly in this [SO post](https://stackoverflow.com/questions/986006/how-do-i-pass-a-variable-by-reference#:~:text=Reference%20values%20are%20hidden%20in,references%20to%20the%20target%20objects.). The idea is to represent the subset representative as a dictionary, so that when we change the subset representative of any member in the subset, all the other members "follow" the change automatcially and simultaneously. One advantage of this implementation is that we don't need a Find function to trace the root of the subset representative, because the members in the same subset always have the same representative. The setup of the graph consists two classes: Node and Graph. A graph is made of nodes, but it also keeps track of all the edges. A node needs to keep track of two things: its subset representative and whether it has been merged or not. 

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
