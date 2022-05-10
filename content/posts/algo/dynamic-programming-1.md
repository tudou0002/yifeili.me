---
title: "Dynamic Programming Tips"
date: 2022-05-05T03:19:34-04:00
draft: false
tags:  ['algorithm', 'dynamic programming']
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
Similar to divide and conquer, dynamic programming breaks a problem into smaller problems. However, dynamic programming fits better when the subproblems are overlapped (one can reuse the result from other subproblems), subproblems depend on the results of other subproblems. DP is normally meant to solve optimization problems - finding min/max value under certain constraints. 

### :blossom: 4 steps for solving a DP problem
According to CLRS[1], there are four steps when developing a dynamic programming algorithm:
1. Characterize the structure of an optimal solution
2. Recursively define the value of an optimal solution
3. Compute the value of an optimal solution , typically in a bottom-up fashion
4. Construct an optimal solution from computed information

### :cherry_blossom: Runtime analysis

The running time of DP depends on the product of two factors: the number of subproblems overall (sometimes can be the space needed for memorization) and how many choices we look at for each subproblem.

### :sunflower: Comparison between top-down and bottom-up
For a dynamic programming algorithm, we can either choose bottom-up strategy or top-down plus memoization strategy. 
> Memoization offers the efficiency of the bottom-up dynamic programming approach while maintain a top-down strategy

In general practice, if all subproblems must be solved at least once, bottom-up usually outperforms the TP by a constant factor, because bottom-up has no overhead for recursion and less overhead for maintaining the table. Moreover, for some problems we can exploits the regular pattern of table accesses in the DP to recude time or space requirements even further.

Alternatively, if some subproblems in the subproblem space need not be solved at all, the memorized solution has the advantages of solving only those subproblems that are definitely required.

### :hibiscus: Implementation trick in Python

If you don’t want to implementing the memoization structure, use `@functools.lru_cache(None)` in python to the recursive function so that it can automatically cache the subproblems' results.


**Reference**  
[1] Cormen, T. H., Leiserson, C. E., Rivest, R. L., &amp; Stein, C. (2009). Dynamic Programming. In Introduction to algorithms. The MIT Press. 