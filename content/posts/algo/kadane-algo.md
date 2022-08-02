---
title: "Kadane’s Algorithm"
date: 2022-03-19T20:13:42-04:00
draft: false
tags: ['algorithm','array']
author: Yifei Li
showToc: true
TocOpen: false
math: true
draft: false
hidemeta: false
comments: false
description: "How dynamic programming is used in computing max subarray sum"
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
## Maximum Subarray
> Given an integer array nums, find the contiguous subarray (containing at least one number) which has the largest sum and return its sum.

(from leetcode [53](https://leetcode.com/problems/maximum-subarray/))

#### Explanation
One intuition might be to apply a greedy algorithm and ignore all negative elements because negative elements may drag down the total sum of a subarray. However, we cannot skip every negative entry because there could be another entry that outpass the previous negative entries.

For each new entry $e$ we visit, we either take it as an extension of the current subarr or start a new array at $e$. So we need to find a way to determine whether $e$ is a point that we need to start a new array.

We want to maximize the sum if possible, but we don’t know what the subsequent elements will be. But what we know are the elements and their partial sum we’ve visited. So it’s clear that we don’t want to include the prev subarray if their sum is negative. Because negative prefix sum is a net loss. Let array P record the max subarray sum before index i, then 
$$P[i] = \max \begin{cases}
    A[i] & \text{if prev max sum is negative} \cr
    P[i-1]+A[i] & \text{if prev max sum is nonnegative}
\end{cases}$$

For actual implementation, we can use one variable to store the max result we've seen and another variable for the current max sum. So the time complexity is O(n) and space complexity is O(1).

#### Example
For an array `[3, -5, 4, 2, -3]`, the iteration process are:
```
A:          [3, -5, 4, 2, -3]
max_seen:    3,  3, 4, 6,  6
max_cur:     3, -2, 4, 6,  3
```

#### Python3 implementation
```python3
def find_max_sum_subarr(A:List[int]) -> int:
    # assume A is not empty
    max_seen = max_cur = A[0]

    for e in A[1:]:
        max_cur = max(e, e+max_cur)
        max_seen = max(max_seen, max_cur)

    return max_seen
```


## Maximum Sum Circular Subarray
> Given a circular integer array nums of length n, return the maximum possible sum of a non-empty subarray of nums. A circular array means the end of the array connects to the beginning of the array.

(from leetcode [918](https://leetcode.com/problems/maximum-sum-circular-subarray/))

#### Explanation
There’s a very neat way to compute max subarray sum in a circular array. In a circular array, the max_subarr_sum could span across a non-circular or circular subarr. We know that the total sum of an array $S$ is fix, so ($S$ - max_subarr_sum) = min_subarr_sum. Also, it’s intuitive that if the subarr with max sum is circular then the subarr with min sum is not circular and vice versa. 

If we want to reuse the above function to compute the extreme sum in a non-circular way, we can check on the following two cases:

- case A - when max sum is non-circular: max_subarr_sum
- case B - when max sum is circular (i.e. min sum is non-circular): total_sum $-$ min_subarr_sum

![Example of two conditions](/max_sum_circular.PNG#center)

Before we simply return the max value of the two cases, there's one edge case: all elements are negative. If we simply apply the Kadane algorithm to find the minimum sum of subarray, we will include all elements which will leave us an empty subarr for the circular max subarr. Since we need a non-empty subarray with maximum sum, we will return the non-circular subarray with max value. 

#### Example
```
A:              [-3, -2, -1]
result_max:      -3, -2, -1
cur_max:         -3, -2, -1

result_min:      -3, -5, -6
cur_min:         -3, -5, -6

return:          -1
```

#### Python3 implementation
```python3
def find_circular_max_sum(A:List[int]) -> int:
    # assume A is not empty

    def find_sum(f):
        result = cur = A[0]
        for e in A[1:]:
            cur = f(e, e+cur)
            result = f(result, cur)
        return result
    
    no_cir_max = find_sum(max)
    no_cir_min = find_sum(min)

    # if min_subarr includes all elements in A
    # use the non-circular max result to avoid empty subarray
    if no_cir_min==sum(A):
        return no_cir_max
    return max(no_cir_max, sum(A)-no_cir_min)

```