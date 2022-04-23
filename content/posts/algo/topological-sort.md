---
title: "Topological Sort"
date: 2022-04-13T22:09:58-04:00
draft: false
tags:  ['algorithm', 'graph']
author: Yifei Li
showToc: true
TocOpen: false
math: true
hidemeta: false
comments: false
description: ""
disableHLJS: true # to disable highlightjs
disableShare: true
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/tudou0002/yifeili.me/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

Topological sort is a graph traversal in which each node *v* is visited only after all its dependencies are visited*. (from wikipedia)*

Since topological sort requires the graph to be DAG(directed acyclic graph), it’s common to see topological sort combined with cycle detection. If we can build a directed graph given the requirements or there’s a dependent/ordered relationship in the problem, we can consider topological sort as a possible approach.

Normally we can use modified DFS or BFS to traverse the whole graph. Depending on the problem, additional data structures may be required to help with the node visiting. All the example problems listed below are solved by DFS.

## Course Schedule
(Detect if cycle exists in the graph, return True/False)
>There are a total of `numCourses` courses you have to take, labeled from `0` to `numCourses - 1`. You are given an array prerequisites where prerequisites[i] = [$a_i$, $b_i$] indicates that you must take course $b_i$ first if you want to take course $a_i$.
Return true if you can finish all courses. Otherwise, return false.

(from leetcode [207](https://leetcode.com/problems/course-schedule/))

#### Explanation
Given a list of prerequisites, the first thing we need to do is to construct a graph in which each edges points to the prerequisite course. The situation when we cannot finish all courses is that there exists a loop in the graph. For example, three nodes like `a->b->c->a` cannot be finished. Then, we just need to traverse the graph and do the cycle detection. The basic idea is to check if we will run into a node we've visited before. 
I also used another set called `can_take` to cache all the nodes we've inspected because we know the following traversal is doable. The time complexity is the same as the DFS, i.e. O(n+m) where n is `numCourses` and m is the number of relationships in `prerequisites`. We also need additional O(n+m) spaces to store the edges dictionary.  

#### Python3 Implementation
```python3
def canFinish(numCourses: int, prerequisites: List[List[int]]) -> bool:
        edges = {v :[] for v in range(numCourses)}
        for e in prerequisites:
            edges[e[0]].append(e[1])
                
        visited = set()
        can_take = set()
        
        def dfs(cur):
            if cur in can_take:
                return True
            if cur in visited:
                return False
            
            visited.add(cur)
            for nbr in edges[cur]:
                if not dfs(nbr):
                    return False
            can_take.add(cur)
            return True
        
        # iterate all vertices in case the graph is not connected
        for vx in edges:
            if not dfs(vx): return False
            
        return True
```

## Course Schedule II
(Return the topological ordering of a graph if possible)
>There are a total of numCourses courses you have to take, labeled from `0` to `numCourses - 1`. You are given an array prerequisites where prerequisites[i] = [$a_i$, $b_i$] indicates that you must take course $b_i$ first if you want to take course $a_i$.
Return the ordering of courses you should take to finish all courses. If there are many valid answers, return any of them. If it is impossible to finish all courses, return an empty array.

(from leetcode [210](https://leetcode.com/problems/course-schedule-ii/))

#### Explanation
In addtion to cycle detection, we also want to return the ordered courses that meet the requirements. Instead of using two sets to record the states, I initialized three states for each node to record its state. Each node is `UNVISIT` as default state. Every time we inspected a node, we change its state to `VISITED`. If there's no cycle in its prerequisites, we label its state as `TAKABLE` because all its descendents nodes in the graph can be taken. Implementation is similar to the above example. 

#### Python3 Implementation
```python3
def findOrder(numCourses: int, prerequisites: List[List[int]]) -> List[int]:

        edges = {c:[] for c in range(numCourses)}
        for c1,  c2 in prerequisites:
            edges[c1].append(c2)
            
        # states
        UNVISIT, VISITED, TAKABLE = range(3)
        states = [UNVISIT]*numCourses

        order = []

        def dfs(node):
            if states[node] == TAKABLE:
                return True
            if states[node] == VISITED:
                return False
            
            states[node] = VISITED
            for nbr in edges[node]:
                if not dfs(nbr):
                    return False
            states[node] = TAKABLE
            order.append(node)
            return True
        
        for course in range(numCourses):
            if not dfs(course):
                return []
            
        return order
```

## Find Eventual Safe States
(Return the nodes that are not in a cycle)
>There is a directed graph of `n` nodes with each node labeled from `0` to `n - 1`. The graph is represented by a 0-indexed 2D integer array graph where graph[i] is an integer array of nodes adjacent to node i, meaning there is an edge from node i to each node in graph[i].
A node is a terminal node if there are no outgoing edges. A node is a safe node if every possible path starting from that node leads to a terminal node.
Return an array containing all the safe nodes of the graph. The answer should be sorted in ascending order.

(from leetcode [802](https://leetcode.com/problems/find-eventual-safe-states/))

#### Explanation
This problem is an extension of the above two: instead of instant return after detecting a cycle, we skip it and only record the nodes that are not in a cycle. We still use three states to tag nodes in the graph: `UNVISIT`, `SAFE`, and `UNSAFE`. `SAFE` denotes a node is not in a cycle. Either a terminal node or a node whose outgoing path are `SAFE` is defined as `SAFE`.

#### Python3 Implementation
```python3
def eventualSafeNodes(graph: List[List[int]]) -> List[int]:
        # use UNVISIT, TOVISIT, VISITED in the dfs solution
        UNVISIT, SAFE, UNSAFE = range(3)
        result = []
        n = len(graph)
        states = [UNVISIT]*n
        
        def dfs(node):
            # run dfs from this node, 
            if states[node] == UNSAFE:
                return False
            elif states[node] == SAFE:
                return True 
            # if node is a terminal node
            if graph[node] == []:
                states[node] = SAFE
            else:
                # else, set to UNSAFE and check in the next step
                states[node] = UNSAFE
            if all(dfs(next_n) for next_n in graph[node]):
                # if all nodes on the outgoing paths are safe
                states[node] = SAFE
                return True
            return False
        
        for i in range(n):
            if dfs(i):
                result.append(i)
        return result
```


## All Ancestors of a Node in a DAG
(Record the ancestors of a given node. )
>You are given a positive integer n representing the number of nodes of a Directed Acyclic Graph (DAG). The nodes are numbered from `0` to `n - 1` (inclusive).
You are also given a 2D integer array edges, where edges[i] = [$from_i$, $to_i$] denotes that there is a unidirectional edge from $from_i$ to $to_i$ in the graph.
Return a list answer, where `answer[i]` is the list of ancestors of the ith node, sorted in ascending order.
A node u is an ancestor of another node v if u can reach v via a set of edges.

(from leetcode [2192](https://leetcode.com/problems/all-ancestors-of-a-node-in-a-directed-acyclic-graph/))

#### Explanation
A trick is to reverse all the edges’ direction and run DFS to record the children (ancestors in the original graph). Cache node's children for each node to reduce time complexity so we don't have to run DFS every single time.

#### Python3 Implementation
```python3
def getAncestors(n: int, edges: List[List[int]]) -> List[List[int]]:
        # reverse the edges
        # find all descendents
        reverse_edges = collections.defaultdict(list)
        for u, v in edges:
            reverse_edges[v].append(u)
            
        results = [[]]*n
        
        def dfs(node):
            if results[node]:
                # if visted
                return 
            descendents = []
            for nei in reverse_edges[node]:
                dfs(nei)
                descendents.extend(results[nei]+[nei])
            results[node] = sorted(list(set(descendents)))
            return
            
            
        for i in range(n):
            dfs(i)
            
        return results
```