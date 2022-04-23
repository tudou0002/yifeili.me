---
title: "Kth Smallest in Two Sorted Arrays"
date: 2022-03-19T19:58:30-04:00
draft: true
tags: ['algorithm'] 
author: Yifei Li
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
math: true
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
A thorough analysis on a more generalized case of the Leetcode problem [*Median of Two Sorted Arrays*](https://leetcode.com/problems/median-of-two-sorted-arrays/).

> You are given two sorted arrays (A and B) and a positive integer k. Design an algorithm for computing the kth smallest element in an array consisting of the elements of the initial two arrays arranged in sorted order. Array elements may be duplicated within and across the input arrays.

(from EPI 24.15)

The brute-force solution is to first combine the two sorted arrays into one sorted array and access the kth element. The time complexity is O(n). 

However, we can think of finding kth smallest in the combined array as finding xth smallest in A and (k-x)th smallest B. "Finding an element in a sorted array" can usually be done by a binary search with time complexity $O(logn)$. The next step is to find a way to adjust the binary search conditions.

I'll explain with a concrete example as follows. Given two arrays A and B, the sorted combined array is C. When k is set to be 7, x is 4 and $k-x$ is 3, correspond to 8 and 7 (noted with red underline). 
![arrays](/kth_smallest_1.PNG#center)
To make sure 7 and 8 are the (k-1)th and kth smallest elements in the combined array, we need to make sure all the elements come after 8 in C are larger than or equal to 7&8. We don't have to compare 7 with 10&11 since they come from the same array. Same thing with 8 and 15. However, we do need to check if $8<=10$ and $7<=15$, i.e. we need to check whether
$$A[x-1]<=B[k-x]$$
$$B[k-x-1]<=A[x]$$
![equations](/kth_smallest_2.PNG#center)

Now we have the condition checks, we still need to figure out what are the left bound and right bound we should start with to find x. By definition, we can search in the interval [0, k]. However, we also need to make sure all the elements we check during the binary search are within the bound of A and B respectively. Let the length of A be m and length of B be n, we want to make sure:
- $0<=k-x<=n$. When $x==0$ and $k>n$, this requirement is broken. Therefore, we want to set the lower bound of x to be $k-n$ if $k-n>0$ so that $0<=k-(k-n)<=n$. Therefore the lower bound of x should be: `max(0, k-len(B))`
- $0<=x<=m$. When $k=x$ and $k>m$, this requirement is broken. Therefore, we want to set the upper bound of x to be $m$ if $k>m$. Therefore the upper bound of x should be: `min(len(A), k)`.

During our binary search, we still need to take care of the cases when index is out of bound. We can treat the index smaller than 0 as negative infinity and index larger than the last elment as positive infinity. This setting will not change our condition check but can simplify the implementation. 

The time complexity is $O(logk)$ and space complexity is O(1). Note that the problem Median in two sorted array can use the same idea with minor change on the return value.

```python3
def find_kth_in_two_sorted_array(A:List[int], B:List[int], k:int) -> int:
    m, n = len(A), len(B)
    if m>n: 
        # make sure A is a smaller array
        A, B, m, n = B, A, n, m
    l, r = max(0, k-n), min(k, m)

    while l<r:
        x = l + (r-l) // 2
        a1 = float('-inf') if x-1<0 else A[x-1]
        a2 = float('inf') if x>=m else A[x]
        b1 = float('-inf') if k-x-1<0 else B[k-x-1]
        b2 = float('inf') if k-x>=n else B[k-x]
        
        if a1<=b2 and b1<=a2:
            return int(max(a1, b1))
        elif a2>b1:
            r = x-1
        elif b1>a2:
            l = x+1 

    # check the boundary value
    a1 = float('-inf') if l-1<0 else A[l-1]
    b1 = float('-inf') if k-l-1<0 else B[k-l-1]
    return int(max(a1, b1))

```
