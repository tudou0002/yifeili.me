---
title: "Next Permutation"
date: 2022-07-31T15:10:51-04:00
draft: true
tags:  ['algorithm', 'array']
author: Yifei Li
showToc: true
TocOpen: false
math: true
hidemeta: false
comments: false
description: "A walkthrough of a single algorithm question"
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

## Next Permutation
> A permutation of an array of integers is an arrangement of its members into a sequence or linear order. The next permutation of an array of integers is the next lexicographically greater permutation of its integer. More formally, if all the permutations of the array are sorted in one container according to their lexicographical order, then the next permutation of that array is the permutation that follows it in the sorted container. If such arrangement is not possible, the array must be rearranged as the lowest possible order (i.e., sorted in ascending order). Given an array of integers nums, find the next permutation of nums.
The replacement must be in place and use only constant extra memory.

(from leetcode [31]())

#### Explanation
To find the next permutation of a given array, we need to increase the order of the array as little as we can. Notice that the maximum order form of an array is when it is sorted in a non-ascending order. For example, `[4,3,1]` is the maximum order form of the array `[3,1,4]`. According to the question definition, the next permutation of a maximum order form is the lowest possible order (i.e., the next permutation of `[4,3,1]` is `[1,3,4]`). 

The overall steps for finding the next permutation of an array are:
1. Find the non-descending index in the array reversely. The subarray `nums[non_desc_idx+1:]` is a subarray in its max order form. If input array is already in its max order form (i.e., `non_desc_idx==-1`), go to step 3.
2. Swap `nums[non_desc_idx]` with `nums[swap_idx]` where `nums[swap_idx]` is the smallest element larger than `nums[non_desc_idx]`. Since we want to increment the order as little as possible, we replace the element before subarray `nums[non_desc_idx+1:]` with a slightly larger one we can find in `nums[non_desc_idx+1:]`.
3. Reverse `nums[non_desc_idx+1:]` by swapping. The will change `nums[non_desc_idx+1:]` to its min order form. 

The time complexity of this solution is O(n) and space complexity is O(1).

#### Example
```
input:  nums = [3,4,6,5,2,1]
output: nums = [3,5,1,2,4,6]
```
![Step-by-step example of next permutation](/next_permutation.PNG#center)


#### Python3 Implementation
```python3
def nextPermutation(self, nums: List[int]) -> None:

    def reverse_by_swap(l, r):
        # l, r are idx in array inclusive
        for i in range((r-l+1)//2):
            nums[l+i], nums[r-i] = nums[r-i], nums[l+i]
        return 

    n = len(nums)
    non_desc_idx = n-2
    while non_desc_idx >= 0:
        if nums[non_desc_idx] >= nums[non_desc_idx+1]:
            non_desc_idx -= 1
        else:
            break
    
    # if input array is in its max order form
    if non_desc_idx==-1:
        reverse_by_swap(0,n-1)
        return
    
    for swap_idx in range(n-1,non_desc_idx,-1):
        if nums[swap_idx] > nums[non_desc_idx]:
            nums[non_desc_idx], nums[swap_idx] = nums[swap_idx], nums[non_desc_idx]
            break
    
    reverse_by_swap(non_desc_idx+1, n-1)
    return
```