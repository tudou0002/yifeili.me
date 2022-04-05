---
title: "Binary Search Basic"
date: 2022-03-29T23:52:26-04:00
draft: false
tags:  ['algorithm', 'binary search']
author: Yifei Li
showToc: true
TocOpen: false
math: true
hidemeta: false
comments: false
description: "Intro to binary search explained with questions"
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

# Intro
Assume we want to search an element from a collection of n elements, we need to inspect every element with a worst case time complexity of O(n). If we know the collection is sorted, each time we can eliminate half of the collection by comparing the middle element with the target element. The time complexity can be improved to O(logn). 

Without loss of generality, given an integer array `A`, sorted in ascending order, we want to find an integer `x` using binary search, an iterative implementation of binary search is as follow:
```python3
def binary_search(A:List[int], x: int) -> int:
    '''
    The function returns the index of x in A and -1 if x is not in A. 
    Assume A is sorted ascendingly.
    '''

    left, right = 0, len(A)-1
    while left <= right:
        mid = left + (right-left)//2    # avoid potential overflow
        if A[mid] > x:
            right = mid - 1
        elif A[mid] < x:
            left = mid + 1
        else:   # A[mid] == x
            return mid
    return -1

```
Above is a basic binary search structure. When the condition changes, we need to adjust some settings to ensure the correct output. In general, my thinking process for a binary search problem is:
1. Identify the pattern: How binary search can fit in to the solution.
2. Identify the interval we want to search from i.e. `lower_bound` and `upper_bound`
3. Determine the condition checks: comparing with what condtion, when to move left or right, etc.
4. Double check corner cases, especially the ending situation

Here are some problems that can be solved with some minor changes to the basic binary search algorithm.

# Algo questions

## Sqrt(x)
> Given a non-negative integer x, compute and return the square root of x.
Since the return type is an integer, the decimal digits are truncated, and only the integer part of the result is returned.

(from leetcode [69](https://leetcode.com/problems/sqrtx/))

#### Explanation
In this problem, the search interval is not explicitly given as a sorted array. A brute-force way is to compute $i^2$ for $i\in{1,2,..,x}$ and return i when $i^2<=x<(i+1)^2$. We can use binary search and halve the candidates each time. The key is to map the continuous numbers from 1 to x to our search space. 
- We can shrink our search interal to `[0, math.ceil(x/2)]`. 
- Each time we square the number and compare with x. If $i^2>x$, indicating i is too large, we move to the left. If $i^2<x$, indicating i is too small, we move to the right. We can directly return $i$ if $i^2=x$.
- When we exit the while loop, left pointer points to the largest integer whose square is less than $x$.


#### Python3 Implementation
```python3
def mySqrt(x: int) -> int:
        # interval [0, math.ceil(x/2)]
        l, r = 0, math.ceil(x/2)
        while l<=r:
            mid = l + (r-l)//2
            p = mid*mid
            if p>x:
                r = mid -1
            elif p == x:
                return mid
            else:
                l = mid + 1
        return r
```

## Single Element in a Sorted Array
>You are given a sorted array consisting of only integers where every element appears exactly twice, except for one element which appears exactly once.
Return the single element that appears only once.
Your solution must run in O(log n) time and O(1) space.

(from leetcode [540](https://leetcode.com/problems/single-element-in-a-sorted-array/))

#### Explanation
A bit manipulation solution is to take the XOR of all elements in the `nums`. The result will be the element that appeared only once. A detailed explanation can be found in this post: [Single Number](https://www.yifeili.me/posts/single-number/#element-appears-twice-except-for-one-appears-once).

However, the problem requires a O(log n) solution and given array is sorted for our convenience. Binary search with a time complexity of O(n) is a natural choice. 
- The search interval is still `[0, len(nums)-1]`
- The way we decide wether a number appears twice is to compare with the adjacent elements. If we cannot find the same element around this position, we can return. Otherwise, we need to check the number of elements on each side, because the side contains the target number will have odd number of elements. ![Illustration of the condition checks](/binary_search_basic.PNG#center) 
- An edge case is that when we want to access the out of index elements. We can assign a value which is not equal to any elements in the given array. I use the infinity in my implementation.


#### Python3 Implementation

```python3
def singleNonDuplicate(nums: List[int]) -> int:
        # change in if-else condition
        n = len(nums)
        l, r = 0, n-1
        while l<= r:
            m = l + (r-l)//2
            ml = nums[m-1] if m>0 else float('inf')
            mr = nums[m+1] if m<n-1 else float('inf')
            if nums[m] == ml:
                if (m-1)%2==1:
                    r = m-2
                else:
                    l = m+1
            elif nums[m] == mr:
                if m%2==1:
                    r = m-1
                else:
                    l = m+2
            else:
                return nums[m]
```

## Search in Rotated Sorted Array 
>There is an integer array nums sorted in ascending order (with **distinct** values).
Prior to being passed to your function, nums is possibly rotated at an unknown pivot index k. 
Given the array nums after the possible rotation and an integer target, return the index of target if it is in nums, or -1 if it is not in nums.
You must write an algorithm with O(log n) runtime complexity.

(from leetcode [33](https://leetcode.com/problems/search-in-rotated-sorted-array/description/))

#### Explanation
When we want to search in a sorted array, naturally we can choose binary search. It's worth noticing that binary search still works when the sorted array is rotated, since partially sorted array still gives us a hint about which part the target number exists. For this variant, we need to twist the condition checks a little bit and the binary search algorithm will work like a charm.
- Interval is still `[0, len(nums)-1]`, the same as the basic binary search
- When we want to decide which direction to move, we need to determine `target` is in which half of the array. Since we can only count on the sorted subarray, we can first decide whether we should move to the sorted half, if not, we can move to the other half. The detailed steps are 1. identify which half is sorted 2. whether `target` is in the sorted half
- When we exit the loop, it means that there's no element matches the `target`. Therefore, we return -1.

#### Python3 Implementation
```python3
def search(nums: List[int], target: int) -> int:
    left, right = 0, len(nums)-1
    
    while left<=right:
        mid = left + (right-left)//2
        if nums[mid]==target:
            return mid
        elif nums[left]<=nums[mid]:
            # left half is sorted
            if nums[left]<= target< nums[mid]:
                # target is in the sorted half
                right = mid-1
            else:
                # target is in the unsorted half
                left = mid + 1
        else:
            # right half is sorted
            if nums[mid]< target<= nums[right]:
                left = mid + 1
            else:
                right = mid -1
                
    return -1
```

Another variation is problem [81](https://leetcode.com/problems/search-in-rotated-sorted-array-ii/), when the elements are not distinctive. In the worst case, this problem cannot be solved less than linear time. However, we can still apply the binary search strategy to shorten the searched elements. We also need to deduplicate the elements before picking a side to search from. 


