---
title: "Divide and Conquer"
date: 2022-04-28T21:19:46-04:00
draft: false
tags:  ['algorithm', 'divide&conquer']
author: Yifei Li
showToc: true
TocOpen: false
math: true
hidemeta: false
comments: false
description: "Examples of divide and conquer paradigm"
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
A divide-and-conquer algorithm recursively breaks down a problem into two or more subproblems of the same or related type, until these become simple enough to be solved directly(from [Wikipedia](https://en.wikipedia.org/wiki/Divide-and-conquer_algorithm)). Some common algorithms like merge sort, quick sort, and binary search all fall into this category. In this post, I'll explain several problems that apply the divide and conquer paradigm.

## Sort List
Merge Sort - classic application of divide and conquer
>Given the head of a linked list, return the list after sorting it in ascending order.

(from leetcode [148](https://leetcode.com/problems/sort-list/))

#### Explanation
There are two steps for merge sort algorithm in general:
1. split the current list into two sublists unless the current list only has one element or no elements
2. use two pointers to merge the two sublists by ascending/descending order

In this problem, we are given a singly linked list with no additional information about length. We can apply a fast&slow pointer technique to find the mid point in the linked list and split the list by manipulating pointers. The `merge()` function uses a dummy head pointer for simplicity.

#### Python3 Implementation
```python3
def sortList(self, head: ListNode) -> ListNode:
    if not head or not head.next:
        return head
    mid  = self.split_and_get_mid(head)
    left = self.sortList(head)
    right = self.sortList(mid)
    return self.merge(left, right)
    
def split_and_get_mid(self,node):
    # split the list into two lists
    # use slow-fast pointers to get the middle node
    # return the head node of the second sublist
    slow, fast = node, node
    while fast.next and fast.next.next:
        slow = slow.next
        fast = fast.next.next

    mid = slow.next
    slow.next = None
    return mid

def merge(self,L1, L2):
    # actual sort happens in the merge() function
    dummy = tail = ListNode()

    while L1 and L2:
        if L1.val<=L2.val:
            tail.next, L1 = L1, L1.next
        else:
            tail.next, L2 = L2, L2.next
        tail = tail.next
    tail.next = L1 or L2
    return dummy.next
```

## Kth Largest Element in an Array
Quick Select - divide and conquer combined with randomization
> Given an integer array nums and an integer k, return the kth largest element in the array.
Note that it is the kth largest element in the sorted order, not the kth distinct element.

(from leetcode [215](https://leetcode.com/problems/kth-largest-element-in-an-array/))

#### Explanation
A typical solution for finding the largest k elements in an array is to use a heap. The time complexity is O(nlogk) in this case. The same problem can be solved in a divide and conquer approach that takes O(n) in average. 

The algorithm is called quick select. Each time we pick an element `e` and **partition** the array into subarrays: `larger than e` and `smaller than e`. `e` should stay in the middle of the two subarrays. We count the number of elements that are before `e`(larger than `e`).

- if there are k-1 elements before it. we just found the target.
- if there are more than k-1 elements: e is too small, we can consider a larger e by inspect the subarray before e
- if there are less than k-1 elements: e is too large, we can consider a smaller e by inspect the subarray after e

The partition() function modifies array in place by swapping elements larger than `e` before `e`.
(, we need to be careful when dealing with the swapping and pivot initialization. Also, duplicates is another pain. )

(below should be a step-by-step walk through of the partition() function, explain the what and why)

- first record the value of pivot and swap pivot to be the last element in the array.
- we start from the leftmost element and keep track of `split` index. This index will later be the position where we insert `e`. While traversing, if we find a larger element, we swap current element with `arr[split]` and increment `split` by 1. Otherwise we ignore current element.
- We also want to keep track of the duplicated pivot value if there's any: If current element is the same as the pivot value, increment the counter. This duplicate counter can tell us if the largest k element is one of the duplicated pivot. 

The average time complexity is O(n) and the worst-case time is O($n^2$). The space complexity is O(1) since we are just swapping elements inplace.

:sparkles: A much more elegant solution with O(n) time and space complexity can be found [here](https://leetcode.com/problems/kth-largest-element-in-an-array/discuss/1019513/Python-QuickSelect-average-O(n)-explained).
:sparkles: When the input array size $n$ is large and k is rather small, we can break input into fixed size arrays and run quick select on one batch, accumulate the largest k elements into the next batch. By using $2k-1$ as the batch size, the time complexity for each batch is almost certain O(k). It's run every $k-1$ element, suggesting an O(n) time complexity in total. (from EPI 24.16)
#### Example
Given an integer array `[4,5,3,3,2,3]`, we want to find the 4th largest element in the array which should be 3. Assume we first randomly pick `4` as the pivot and in the next round we pick `3` as the pivot. Here's what will happen step-by-step:
![Detailed walkthrough of the quick select example](/quick_select.PNG#center)

#### Python3 Implementation
```python3
def findKthLargest(nums: List[int], k: int) -> int:
    l, r = 0, len(nums)-1
    def partition(l, r, pivot_idx):
        pivot_val = nums[pivot_idx]
        split_idx = l
        nums[pivot_idx], nums[r] = nums[r], nums[pivot_idx]
        # count of elements same as pivot except for this pivot itself
        dup_cnt = 0
        for i in range(l,r):
            if nums[i]> pivot_val:
                nums[i], nums[split_idx] = nums[split_idx], nums[i]
                split_idx += 1
            if nums[i]== pivot_val:
                dup_cnt += 1
        nums[r], nums[split_idx] = nums[split_idx], nums[r]
        return split_idx, dup_cnt

    while l<=r:
        pivot = random.randint(l,r)
        split_idx, dup_cnt = partition(l, r, pivot)
        if split_idx==k-1 or (split_idx<=k-1 and split_idx+dup_cnt-1>=k-1):
            return nums[split_idx]
        elif k-1<split_idx:
            #pivot is too small
            r = split_idx-1
        elif k-1>split_idx:
            # pivot is too large
            l = split_idx+1
            
    return -1
```

## Find a Peak Element 
Binary Search - when the search interval is not sorted
> A peak element in a 2D grid is an element that is strictly greater than all of its adjacent neighbors to the left, right, top, and bottom.
Given a 0-indexed m x n matrix mat where no two adjacent cells are equal, find any peak element mat[i][j] and return the length 2 array [i,j].
You may assume that the entire matrix is surrounded by an outer perimeter with the value -1 in each cell.
You must write an algorithm that runs in O(m log(n)) or O(n log(m)) time.

(from leetcode [1901](https://leetcode.com/problems/find-a-peak-element-ii/))

#### Explanation
In this problem, we will see that binary search is not restricted to the sorted interval. Whether we can use binary search really depends on the problem definition.

The algorithm starts from the middle columns in the current inspecting interval. We then iterate the entire column to find the max element. 
- If it’s larger than it’s horizontal neighbors, return it's index because this is the peak. 
- If right element is larger, move to right . 
- Otherwise, move to left.

#### Python3 Implementation
```python3
def findPeakGrid(mat: List[List[int]]) -> List[int]:
    ROWS, COLS = len(mat), len(mat[0])
    
    def validate(mid):
        # given a column index, return the local max we can find
        local_max = (0, mid)
        for r in range(ROWS):
            if mat[local_max[0]][local_max[1]]<mat[r][mid]:
                local_max = (r, mid)
        
        # check its horizontal neighbors 
        left = mat[local_max[0]][local_max[1]-1] if local_max[1]-1>=0 else -1
        right = mat[local_max[0]][local_max[1]+1] if local_max[1]+1<COLS else -1
        if mat[local_max[0]][local_max[1]]>right and mat[local_max[0]][local_max[1]] > left:
            return local_max
        elif right>mat[local_max[0]][local_max[1]]:
            return (local_max[0], local_max[1]+1)
        else:
            return (local_max[0], local_max[1]-1)
    
    l, r = 0, COLS-1
    while l<=r:
        m = l+(r-l)//2
        (i,j) = validate(m)
        if j==m:
            return (i,j)
        elif j>m:
            l = m+1
        else:
            r = m-1
    
    return (0,0)
```

