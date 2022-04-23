---
title: "Shortest Path"
date: 2022-03-20T22:14:38-04:00
draft: false
tags: ['algorithm']  
author: Yifei Li
showToc: true
TocOpen: false
math: true
hidemeta: false
comments: false
description: "Several algorithms for computing the shortest path in a graph."
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
There are several alorithms designed for solving the **Shortest Path Problem**. In this post, I collected several questions that apply BFS, Floyd-Warshall, and Bellman-Ford in their solutions.

## Word Ladder
>A transformation sequence from word beginWord to word endWord using a dictionary wordList is a sequence of words `beginWord -> s1 -> s2 -> ... -> sk` such that:
>Every adjacent pair of words differs by a single letter.
Every si for $1 <= i <= k$ is in wordList. Note that beginWord does not need to be in wordList.
`s[k] == endWord`
> Given two words, beginWord and endWord, and a dictionary wordList, return the number of words in the shortest transformation sequence from beginWord to endWord, or 0 if no such sequence exists.  

(From leetcode [127](https://leetcode.com/problems/word-ladder/))

#### Explanation
Naturally we can map the string question into a graph question. Each node is a string and an available edge is when two nodes are different by one character. After building a connected unweighted graph, we can traverse the graph using BFS from `start_word` to `end_word` and get the shortest distance. The implementation can be broken down into two steps:
- Build edges: For each word $w$ in the word list, we want to check if there are other words in the list that are different from $w$ by one character. We can use a nested loop and two pointers to compare each letter. Assume there are n words in the wordlist and d is the length of each word. This approach has a time complexity of $O(n^2d)$. A more time-efficient way to do this is to check if any of the possible one-letter-off variant is in the `set(wordlist)`. This way, we can improve the time complexity to $O(n*24d)=O(nd)$.
- BFS find shortest path: Since the graph is unweighted, and we want to record how many words(nodes) are in the shortest path. We can use a hashmap to count the number of words $c$ along the path. 

#### Example
```
Input:  beginWord = "hit", endWord = "cog", 
        wordList = ["hot","dot","dog","lot","log","cog"]
Output: 5
```
The graphic representation and its word count $c$ are present as follows:
![word ladder example](/word_ladder.PNG#center)

#### Python3 implementation
```python3
class Graph:
    def __init__(self, nodes):
        self.nodes = nodes
        self.edges = {node:[] for node in self.nodes}
    
    def add_edge(self, u, v):
        # undirected edge
        self.edges[u].append(v)
        # self.edges[v].append(u)
        
    def find_min_distance(self, s, t):
        # sanity check
        if t not in self.nodes:
            return 0
        
        # BFS since the graph is unweighted
        # BFS traversing, return count when the visiting node is t
        # else return 0 after visit all the nodes
        distances = {n:float('inf') for n in self.nodes}
        visited = set()
        q = collections.deque([s])
        visited.add(s)
        distances[s] = 1
        while q:
            cur = q.popleft()
            if cur == t:
                return distances[cur]
            for nei in self.edges[cur]:
                if nei not in visited:
                    q.append(nei)
                    visited.add(nei)
                    distances[nei] = min(distances[nei], distances[cur]+1)
        return 0
        


def ladderLength(self, beginWord: str, endWord: str, wordList: List[str]) -> int:
    # sanity check
    if endWord not in wordList:
        return 0
    
    def check_connection(s1, s2):
        # return true if two str inputs diff by 1 character
        count = 0
        for c1, c2 in zip(s1, s2):
            if c1!=c2:
                count+=1
            if count>1:
                return False
        return count<=1
    
    # build graph
    nodes = [beginWord]+wordList
    g = Graph(nodes)
    n = len(nodes)
    nodes = set(nodes)
    d = len(beginWord)
    for node in nodes:
        for i in range(d):
            for c in string.ascii_lowercase:
                cand = node[:i]+c+node[i+1:]
                if cand in nodes:
                    g.add_edge(node, cand)   
    
    # compute min_distance
    return g.find_min_distance(beginWord, endWord)
```


## Optimize road network
> Write a program that takes the a road network graph and proposals for new road edges, and returns the proposed road edge which leads to the most improvement in the total distance. The total distance is defined to be the sum of the shortest path distances between all pairs of cities. All road edges allow for bi-directional traffic, and the original traffic network is connected.

(from EPI 24.34)

#### Explanation
The solution is straightforward: compute the shortest path, check the new proposal one by one, update the network if it can be shortened. The problem left is to choose which algorithm to compute the shortest path in a graph. For this question, we will choose the **Floyd-Warshall algorithm** because how it’s defined. Floyd-Warshall algorithm uses dynamic programming to solve the shortest path between all pairs of nodes. [Notice that Dijkstra is usually used for solving the shortest path problem with specific source and target with a time complexity of O(n^2). ]

More detailed steps are as follows:

- Compute the shortest path using Floyd-Warshall algorithm and record the results in a 2 array
- For each proposal between (u,v), we just need to compare among the 3 scenarios: original network, original network using new proposal (u, v), original network using new proposal (v, u) [since all sections are bi-directional]
- update the smallest route weight sum if needed
#### Example

#### Python3 implementation


## Check if arbitrage is possible
>Design an algorithm to determine whether there exists an arbitrage - a way to start with a single unit of some currency C and convert it back to more than one unit of C through a sequence of exchanges.

(from EPI 24.35)

#### Explanation
Bellman-Ford is the only shortest path algorithm that can be used in a negative-weighted graph with cycles. We can map the currency types into graph nodes and the exchange rates as directed edges with weights. Then finding an arbitrage becomes finding a cycle that the production of all weights is larger than 1. This requirement can be generalized into finding a cycle in a graph with special condition. One of the feature of Bellman-Ford algorithm is that it can detect a negative cycle in a graph. If we can convert our production==1 requirement into sum<0, we can directly apply the Bellman-ford algo. 

We can use the log rule log(a*b) = loga + logb. Actually multiplying -1 on each side is more useful for this problem:  -log(a*b) = (-loga) + -(logb). If a*b>1, LHS<0 which suggest RHS also <0. In other words, if we apply a weight conversion function w(e) = -log(w(e)) on every edge, we can convert our production==1 requirement into sum<0.

The way that Bellman-ford can detect whether a graph contains a negative cycle is that after we relax |v|-1 times, we relax one more time. If the distance of any node changes, we know there’s a negative cycle because a negative cycle in a graph will cause Bellman-ford to reduce the path infinitely and we cannot decide a fixed answer.

#### Example

#### Python3 implementation