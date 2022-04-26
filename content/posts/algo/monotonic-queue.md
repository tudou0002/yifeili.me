---
title: "Monotonic Queue"
date: 2022-04-22T22:29:10-04:00
draft: false
tags:  ['algorithm', 'queue']
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

## Intro
When we want to efficiently find the maximum/minimum element in a queue, we can use a data structure called **deque** to simplify the process. Assume we have two elements $a$, $b$ and $b>a$. Given that $a$ is enqueued earlier than $b$, we know $a$ doesn't matter anymore for the max element result once $b$ is enqueued. Since queue is FIFO, $a$ will be dequeued earlier than $b$ and while $a$ is still in queue, we know $b$ is for sure larger than $a$. 
This deque can also be called monotonic queue,because all the elements in this queue is monotonically incresing/decresing. WLOG, the operations for a monotonic decresing queue are:
- dequeue: pop the left most element, which is the maximum element.
- enqueue: iteratively pop from the rightmost side of the deque until the rightmost element is greater than or equal to the element being enqueued, then add the new element to the right.

Now let's dive into some of the applications of the deque we just learnt. You may need some time to accept why we can get rid of some elements, but once you understand the logic, these questions will reduce to a single pattern. 


## Sliding Window Maximum
When there's a window size constraint.
>You are given an array of integers nums, there is a sliding window of size k which is moving from the very left of the array to the very right. You can only see the k numbers in the window. Each time the sliding window moves right by one position.
Return the max sliding window.

(from leetcode [239](https://leetcode.com/problems/sliding-window-maximum/))

#### Explanation
In this problem, apart from finding the maximum entry, we are given a constraint of finding the maximum in a fixed size window. The brute force solution is to inspect every possible window and find the maximum iteratively. However, since we are moving the window, each time only the boundary elements are changed, i.e. left most element is evicted and the next right element is added to the window. These operations are similar to what we have for a queue. Therefore we can modify the operations for a monotonic decreasing queue:
- Item stored: `(index, num)` because we need to check if all elements in the queue are within the window size
- dequeue: iteratively pop the entries from left if it falls outside of the window
- enqueue: iteratively pop from the rightmost side of the deque until the rightmost element is greater than or equal to the element being enqueued, then add the new element to the right.

This way, the leftmost entry will always hold the maximum elements in the current window. Both time and space complexity are O(n).

#### Example 
```
nums:     [3,2,1,5,3,4]
k:                   3
results:      [3,5,5,5]
```
![Step-by-step example for finding sliding window maximum using mono-decreasing queue](/mono_queue.PNG#center)

#### Python3 Implementation
```python3
def maxSlidingWindow(nums: List[int], k: int) -> List[int]:
    Item = collections.namedtuple('Item', ('idx', 'num'))
    results = []
    deque = collections.deque()
    
    for i, num in enumerate(nums):
        # dequeue 
        while deque and (i - deque[0].idx)>=k:
            deque.popleft()
        # enqueue
        while deque and deque[-1].num<num:
            deque.pop()
        deque.append(Item(i, num))
        if i>=k-1:
            results.append(deque[0].num)
        
    return results
                
```

## Shortest Subarray with Sum at Least K
When the optimization goal is a function (subarray sum) instead of the single value stored in the deque
> Given an integer array nums and an integer k, return the length of the shortest non-empty subarray of nums with a sum of at least k. If there is no such subarray, return -1.
A subarray is a contiguous part of an array.

(from leetcode [862](https://leetcode.com/problems/shortest-subarray-with-sum-at-least-k/))

#### Explanation
We can first generate the prefix sum array `P` of the given array `A` where `P[i] = sum(A[:i])`. The sum of a subarray `A[j:i+1]` (subarray from j to i inclusively) can be easily compute by `P[i]-P[j-1]`. For each i, we need to compare all `j<i` to check the subarray sum and update minimum length if applicable. The time complexity is then $O(n^2)$.

Can we improve this? The answer is yes, by using monotonic queue. The intuition is that we don't want to inspect every `j`, we want to stop once we inspect some `P[j]`. Let's take a minute to analyze the objective function: `P[i]-P[j]>=k`, if `P[j]`'s (for all `j<i`) are monotonically increasing, we can stop once we found a `P[j]` that `P[i]-P[j]<k`. 

The operations for a mono-increasing queue are:
- Item stored: `(index, element)` because we need to compute the subarray length to find the shortest subarray that meets the requirement.
- dequeue: iteratively pop the entries from left if it meets our objective function, update shortest subarray length.
- enqueue: iteratively pop from the rightmost side of the deque until the rightmost element is smaller than the element being enqueued, then add the new element to the right.


#### Python3 Implementation
```python3
def shortestSubarray(nums: List[int], k: int) -> int:

    n = len(nums)
    Item = collections.namedtuple('Item', ('idx', 'prefix_sum'))
    # minimum element is in the first place
    deque = collections.deque([Item(0, 0)])
    result = n+1
    cur_prefix = 0
    for i in range(n):
        cur_prefix += nums[i]
        # dequeue
        while deque and cur_prefix-deque[0].prefix_sum>=k:
            item = deque.popleft()
            result = min(result, i+1-item.idx)
        # enqueue 
        while deque and deque[-1].prefix_sum>=cur_prefix:
            deque.pop()
        deque.append(Item(i+1, cur_prefix))
            
    return result if result<n+1 else -1
```

## Max Value of Equation
When the value in deque item is not explicit. 
> You are given an array points containing the coordinates of points on a 2D plane, sorted by the x-values, where points[i] = [xi, yi] such that $x_i < x_j$ for all 1 <= i < j <= points.length. You are also given an integer k.
Return the maximum value of the equation $y_i + y_j + |x_i - x_j|$ where $|x_i - x_j| <= k$ and 1 <= i < j <= points.length.
It is guaranteed that there exists at least one pair of points that satisfy the constraint $|x_i - x_j| <= k$.


(from leetcode [1499](https://leetcode.com/problems/max-value-of-equation/))
#### Explanation
The brute force solution uses two nested for loops, compares each pair of points and applies the equation with a time complexity of $O(n^2)$. The optimization idea is similar to the one above, we want to implement a monotonic queue and make use of the sorted feature. First, let's find the objective function and then figure out the corresponding operations. Since $x_i <> x_j$, equation $y_i + y_j + |x_i - x_j|$ can be transformed into $y_i + y_j + x_j - x_i = (x_j+y_j)+(y_i - x_i)$. $x_j, y_j$ are points we want to inspect in the outer loop, and $x_i, y_i $ are the points stored in the queue. 

The operations for a mono-decreasing queue are:
- Item stored: `(x, y-x)` x checks if items should be dequeue from left by checking if $|x_i - x_j| <= k$. The objective function $(x_j+y_j)+(y_i - x_i)$ depends on the single value of $y-x$.
- dequeue: iteratively pop the entries from left if it meets $|x_i - x_j| <= k$
- enqueue: iteratively pop from the rightmost side of the deque until the rightmost element is larger than or equal to the element being enqueued, then add the new element to the right.

#### Example 


#### Python3 Implementation
```python3
def findMaxValueOfEquation(self, points: List[List[int]], k: int) -> int:
    
    deque = collections.deque()
    result = float('-inf')
    for x, y in points:
        # x, y are x_j and y_j in our analysis
        # dequeue
        while deque and abs(deque[0][0]-x)>k:
            deque.popleft()
        # update result
        if deque:
            result = max(result, deque[0][1]+y+x)
        # enqueue
        while deque and y-x>deque[-1][1]:
            deque.pop()
        deque.append([x,y-x])
        
    return result
```

## Pattern 
For a monotonic queue question, here are some of the key points to figure out before you start coding:
- **Objective funtion**: it can be as simple as the value of element or more complexed like an equation using values from several elements. 
- **Item**: items are usually consist of two entries. One controls whether we need to dequeue the current element from left (e.g. index of an element). The other can control the objective funtion result.
- **dequeue**: evict elements from left if they meets some kind of condition. 
- **enqueue**: evict elements from right and append the new element. This is how we control the queue to be monotonically increasing/decreasing.