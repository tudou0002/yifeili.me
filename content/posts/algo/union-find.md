---
title: "Union Find"
date: 2022-06-03T21:20:33-04:00
draft: false
tags:  ['algorithm','graph']
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
Union-Find algorithm works well in questions related to finding connected components in an undirected graph. Below are several examples:

## Number of Provinces
> There are n cities. Some of them are connected, while some are not. If city a is connected directly with city b, and city b is connected directly with city c, then city a is connected indirectly with city c.
A province is a group of directly or indirectly connected cities and no other cities outside of the group.
You are given an n x n matrix isConnected where isConnected[i][j] = 1 if the ith city and the jth city are directly connected, and isConnected[i][j] = 0 otherwise.
Return the total number of provinces.

(from leetcode [547](https://leetcode.com/problems/number-of-provinces/))
#### Explanation
As the name of the algorithm suggests, there are two modules `union()` and `find()`. 
- `union(u,v)` takes two nodes `u,v` that are connected by an edge as input. If `u` and `v` are in the same component, we just return 0. If `u` and `v` are not in the same component, we merge them and return 1 indicating we performed the merge operation. We can implement union by rank/size of component, which avoids unbalanced situation and speeds up path compression operation. 
- `find(n)` takes one node `n` as input and return the "root" parent of this node in its current component. Basically, we keep tracing upwards untill `parent[n]==n` where n is a node. We can implement path compression, which makes the height of the subset tree won't exceed 2.

We initialize two arrays to hold the parents and rank of each node. Then iterate all the edges in the graph and call `union(u,v)` whenever we inspect on an edge `(u,v)`.

With the path compression and union by rank technique, the union() operation takes O(1) in time. We need O(n) space to store the parent and rank information for each node.

#### Example
![Union-find process for a example graph](/union_find.PNG#center)

#### Python3 Implementation
code are modified from [this video](https://www.youtube.com/watch?v=8f1XPm4WOUc)
```python3
def findCircleNum(isConnected: List[List[int]]) -> int:
    total = len(isConnected)
    par = [i for i in range(total)]
    rank = [1]*total

    def union(n1, n2):
        # return 0 if we did not union i.e. two nodes already in the same province
        # return 1 if we perform union
        p1, p2 = find(n1), find(n2)
        
        if p1==p2:
            return 0
        else:
            if rank[p1]>rank[p2]:
                par[p2] = p1
                rank[p1] += rank[p2]
            else:
                par[p1] = p2
                rank[p2] += rank[p2]
            return 1
    
    def find(n):
        res = n
        
        while res!=par[par[n]]: # res == par[par[n]] iff n is the toppest node
            res = par[par[n]]   # path compression
            par[n] = res    # modify par of n at the same time
            
        return res
    
    cnt = total
    for i in range(total):
        for j in range(i+1, total):
            if isConnected[i][j]:
                cnt -= union(i,j)
    return cnt           
```

## Accounts Merge
> Given a list of accounts where each element accounts[i] is a list of strings, where the first element accounts[i][0] is a name, and the rest of the elements are emails representing emails of the account.
Now, we would like to merge these accounts. Two accounts definitely belong to the same person if there is some common email to both accounts. Note that even if two accounts have the same name, they may belong to different people as people could have the same name. A person can have any number of accounts initially, but all of their accounts definitely have the same name.
After merging the accounts, return the accounts in the following format: the first element of each account is the name, and the rest of the elements are emails in sorted order. The accounts themselves can be returned in any order.

(from leetcode [721](https://leetcode.com/problems/accounts-merge/))

#### Explanation
The tricky part of this question is how to map the given info into a graph. We can treat each user as a node and build an edge bwtween two users if they share some common emails. After doing the union-find process, we want to return the "root" node in each connected component along with all emails attached to all users in this component. Notice that after the common union-find process, `par` doesn't necessarily stores the root parent for each node. We need to run another pass of `find()` on each node to update values stored in `par`.


#### Python3 Implementation
```python3
def accountsMerge(self, accounts: List[List[str]]) -> List[List[str]]:
    def union(n1, n2):
        p1, p2 = find(n1), find(n2)
        if p1 == p2:
            return
        if rank[p1]>rank[p2]:
            par[p2] = p1
            rank[p1] += rank[p2]
        else:
            par[p1] = p2
            rank[p2] += rank[p2]
        return 
    
    def find(n):
        res = n
        while res!=par[par[n]]:
            # res == par[par[n]] iff n is the toppest node
            res = par[par[n]]
            par[n] = res    # modify par of n at the same time

        return res
    
    # build connected component
    email2name = {}
    cnt = len(accounts)
    par = [i for i in range(cnt)]
    rank = [1]*cnt
    for n, account in enumerate(accounts):
        for email in account[1:]:
            if email in email2name:
                # visited email
                union(n, email2name[email])
            email2name[email] = n

    for i in range(cnt):
        # update every node's parent to be the root parent
        find(i)

    unique_acc = collections.defaultdict(set)
    for i,p in enumerate(par):
        unique_acc[p] |= set(accounts[i][1:])
    return [[accounts[k][0]]+sorted(v) for k,v in unique_acc.items()]
```

## Number of Operations to Make Network Connected
> There are n computers numbered from 0 to n - 1 connected by ethernet cables connections forming a network where connections[i] = [a_i, b_i] represents a connection between computers ai and bi. Any computer can reach any other computer directly or indirectly through the network.
You are given an initial computer network connections. You can extract certain cables between two directly connected computers, and place them between any pair of disconnected computers to make them directly connected.
Return the minimum number of times you need to do this in order to make all the computers connected. If it is not possible, return -1.

(from leetcode [1319](https://leetcode.com/problems/number-of-operations-to-make-network-connected/))
#### Explanation
This question is asking for the minimal spanning tree for the computer networks. If the total number of edges is less than n-1, there aren't enough edges to form a tree so we can directly return -1. 

We then run the union-find algorithm to tag the number of edges connect two different components by `union()`. We can use the return value of `union()`. Since the number of edges in a tree with n nodes is n-1, the minimum number of edges we need to change is (n-1-count_connecting_edge). 

#### Python3 Implementation
```python3
def makeConnected(self, n: int, connections: List[List[int]]) -> int:
    # min spanning tree check
    if len(connections)<n-1:
        return -1
    
    par = [i for i in range(n)]
    rank = [1]*n
    
    def union(n1, n2):
        p1, p2 = find(n1), find(n2)
        if p1 == p2:
            return 0
        if rank[p1]>rank[p2]:
            par[p2] = p1
            rank[p1] += rank[p2]
        else: 
            par[p1] = p2
            rank[p2] += rank[p1]
        return 1
    
    def find(n):
        res = par[n]
        while res != par[par[n]]:
            res = par[par[n]]
            par[n] = res
        return res
    
    cnt = 0
    for u, v in connections:
        if union(u,v):
            cnt += 1
    return n-1-cnt
```