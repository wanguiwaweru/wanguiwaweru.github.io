---
title: "Graphs"
date: "2022-02-21"
draft: false
tags: ["python", "dsa"]
---

# Graphs

Graphs are data structures represented using nodes and edges.

Nodes are the data while the edges are the connections between nodes.

Neighbor nodes are nodes accessible through an edge.

Edges in directed graphs follow a given direction.

In undirected graphs, the edges between nodes are bidirectional.

## Creating adjacency lists

Adjacency lists are useful in representing graphs in code. The lists are similar to dictionaries with a key-value pair where the key is a node and the values are the neighbor nodes.

The image below demonstrates how we can represent an adjacency list as a graph.

![Adjacency](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdxrbfnelm4r513vbrxi2.jpg)

- Note that every node appears in the adjacency list including those without neighbor nodes.

Given edges, we can create an adjacency list as follows.

Every pair in the edges list represents a connection between nodes, therefore in the adjacency list we include both elements as keys with each other as values.

We update the adjacency list based on the edges provided. We can use the function below to create an adjacency list given edges in an undirected graph.

```python
def createGraph(edges):
    graph = {}
    for edge in edges:
        x,y = edge
        if x not in graph:
            graph[x] = []
        if y not in graph:
            graph[y] = []
        graph[x].append(y)
        graph[y].append(x)

    return graph

edges = [['i','j'],['k','i'],['m','k'],['k','l'],['o','n']]

print(createGraph(edges))
```

In this post we will go through two main concepts that are central to solving graph interview questions.

## Depth-First Search

In this approach we follow a node as far as we can before moving to the other neighbor. We explore the nodes following the edges as far as possible before changing directions.

Depth-first search can be achieved using iteration or recursion. It uses the `stack` data structure which follows the `Last In First Out(LIFO)` principle.

### Iterative approach to print all nodes.

```python
def depthFirst(graph,source):
    stack = [source]
    while stack:
        current = stack.pop()
        print(current)
        for neighbor in graph[current]:
            stack.append(neighbor)

graph = {
    'a' : ['c','b'],
    'b': ['d'],
    'c': [ 'e'],
    'd': ['f'],
    'e': [],
    'f': []
}

print(depthFirst(graph,'a'))
```

Elements are removed from the top of the stack and appended to the top of the stack, thus last in is first out.

### Recursion

```python
def depthFirst(graph,source):
    print(source)
    for neighbor in graph[source]:
        depthFirst(graph, neighbor)

graph = {
    'a' : ['c','b'],
    'b': ['d'],
    'c': [ 'e'],
    'd': ['f'],
    'e': [],
    'f': []
}

print(depthFirst(graph,'a'))
```

## Breadth-First Search

In this approach we explore all the immediate neighbors of a given node and keep following this process.

Breadth-first search explores all the sides evenly without following one direction entirely. It can be achieved through iteration. It uses the  `queue` data structure which follows the `First In First Out(FIFO)` principle.

```python
def breadthFirst(graph,source):
    queue = [source]
    while queue:
       current = queue.pop(0)
       print(current)
       for neighbor in graph[current]:
           queue.append(neighbor)

graph = {
    'a': ['c', 'b'],
    'b': ['d'],
    'c': ['e'],
    'd': ['f'],
    'e': [],
    'f': []
}

print(breadthFirst(graph, 'a'))
```
Elements are removed from the front of the queue and appended to the back of the queue, thus first in is first out.

## Other graph problems
- [Find Paths in Graphs](https://dev.to/wanguiwaweru/find-paths-in-graphs-1a10)
- [Number of Connected Components in an Undirected Graph](https://dev.to/wanguiwaweru/number-of-connected-components-in-an-undirected-graph-5gk0)