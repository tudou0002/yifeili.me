---
title: "Binary Search Advanced"
date: 2022-03-29T23:52:52-04:00
draft: false
tags:  ['algorithm', 'binary search']
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

As mentioned in the previous post: [Binary Search Basic](https://www.yifeili.me/posts/binary-search-basic/), binary search also applies to intervals of numbers. In this post, I'll introduce several use cases of binary search in a more subtle way. 

## Split Array Largest Sum
>Given an array nums which consists of non-negative integers and an integer m, you can split the array into m non-empty **continuous** subarrays.
Write an algorithm to minimize the largest sum among these m subarrays.

(from leetcode [410](https://leetcode.com/problems/split-array-largest-sum/))

#### Explanation

The key to this problem is finding how it can be mapped into a binary search problem. Normally, we learn binary search in a problem setting where we want to find a particular target number. However, **binary search also applies to finding the max/min number in an interval**. We just need to change the condition check a little bit.

The brute-force solution for this problem is to list all the possible ways to split the array `A` into `m` non-empty subarrays and record the minimum largest sum we've met so far. However, we can directly search the possible minimum sum and reversely check if we can meet the requirement. 
- The minimum largest sum is the largest element in the array because when `m==len(A)`, we are minimizing every subarray's sum. The maximum largest sum is the sum of the whole array when `m==1`. 
- For each possible largest sum `s` we want to check: 
    - if it's possible to split the array into `m` parts, we move to the left of the interval because we want to find the minimum. Also, we need to update minimum `result` we've seen so far. 
    - if it's not possible to split the array into `m` parts, suggesting `s` is too small, we move to the right.

#### Python3 Implementation
```python3
def splitArray(self, nums: List[int], m: int) -> int: 

    def validate(s):
        # validate if it's possible to split the array when max_subarr_sum == s
        split = 1
        split_sum = 0
        for n in nums:
            split_sum += n
            if split_sum>s:
                split_sum = n
                split += 1
                if split>m:
                    return False
        return True
    
    l, r = max(nums), sum(nums)
    res = r+1
    while l<=r:
        mid = l + (r-l)//2
        if validate(mid):
            res = min(res, mid)
            r = mid -1
        else:
            l = mid + 1
    return res
```

## Kth Smallest Element in a Sorted Matrix
>Given an n x n matrix where each of the rows and columns is sorted in ascending order, return the kth smallest element in the matrix.
Note that it is the kth smallest element in the sorted order, not the kth distinct element.
You must find a solution with a memory complexity better than O(n^2).

(from leetcode [378](https://leetcode.com/problems/kth-smallest-element-in-a-sorted-matrix/))

#### Explanation
To find the kth element in a collection of elements, a common approach is to use heap. For this problem, since the given matrix has a sorted feature, I'll start from a binary search perspective. 

We want to find the kth element and the matrix is sorted, the search interval is bounded by the top left element and bottom right element. Since they are the smallest and largest elements in the matrix respectively. 

After fixing the search interval, given a number $x$, we want to find a way to decide whether we should move to the left or to the right. For $x$ to be the kth smallest element, there must be $c$ elements that are smaller than or equal to the element x. 
- if c>k: x is too large, we need to move to the left
- if c< k: x is too small, we need to move to the right
- if c=k: there are exactly k elements that are smaller than or equal to the element x. However, we don't know if x is in the matrix. 

We cannot return a value that is not in the matrix. Instead of going through all the matrix and check its existence, we can just return the minimum element that meets the $c>=k$ condition. I'll explain this decision with an example in the next section.

Another similar problem is leetcode 668 [Kth Smallest Number in Multiplication Table](https://leetcode.com/problems/kth-smallest-number-in-multiplication-table/) 

#### Example
Assume we are given a 4*4 matrix as follow and we want to return the 13th element in the matrix, i.e. we want to return 10 as the correct result. 

I'll explain the steps when $x=11, x=10, x=9$ separately. Each case the steps are similar:
- we start from the top right corner and initialize a `counter=0` to keep track of the number of elements that are <= x
- when the current element is <= x, move the pointer downwards and add the number of elements up to the current element to `counter` 
- when the current element is > x, move to left

when x=11
![when x=11](/kth_matrix_10_11.PNG#center)

when x=10
![when x=10](/kth_matrix_10_11.PNG#center)

when x=9
![when x=9](/kth_matrix_9.PNG#center)

We can observe that `counter` result is the same when x=11 and x=10. However, 10 is present in the matrix and 11 is not. Therefore, in the binary search, we can update the final result whenever `counter>=k` to keep track of the minimum number that meets the requirements.


#### Python3 Implementation
```python3
def kthSmallest(matrix: List[List[int]], k: int) -> int:
    n = len(matrix[0])
    l, r = matrix[0][0], matrix[n-1][n-1]
    min_res = r+1
    def count_kth(x):
        counter = 0
        i, j = 0, n-1   # intialize at the top right corner
        while i<n and j>=0:
            if x<matrix[i][j]:
                # move to the left
                j -= 1
            elif x>=matrix[i][j]:
                # move downwards
                i += 1
                counter += (j+1)
        return counter
    
    while l<=r:
        mid = l + (r-l)//2
        if count_kth(mid)>=k:
            min_res = min(mid, min_res)
            # make sure the element exists in the matrix
            # return the smallest possible in the range
            r = mid-1
        else:
            l = mid+1
    return min_res
```

## `bisect` module
Sometimes, the bianry search module is not a major point that the interviewer wants you to focus on. Python provides a `bisect` module for doing a binary search on a sorted list. The common usages are:
- `bisect.bisect_left(a, x, lo=0, hi=len(a), key=None)`: Locate the insertion point for x in a to maintain sorted order. If x is already present in a, the insertion point will be before (to the **left** of) any existing entries. `key` specifies a key function of one argument that is used to extract a comparison key from each input element.
- `bisect.bisect_right(a, x, lo=0, hi=len(a), key=None)`

