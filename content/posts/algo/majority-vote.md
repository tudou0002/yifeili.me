---
title: "Majority Vote"
date: 2022-03-18T22:12:35-04:00
draft: true
tags: ['algorithm']
author: Yifei Li
showToc: true
math: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Boyer–Moore majority vote algorithm "
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

Boyer–Moore majority vote algorithm is an algorithm that is used to detect the majority elements in an array. In this post, I'll explain its application in the majority element problem and a more general case. 

## Majority element

>Given an array nums of size n, return the majority element.
>The majority element is the element that appears more than ⌊n / 2⌋ times. You may assume that the majority element always exists in the array.

(from Leetcode [169](https://leetcode.com/problems/majority-element/))

We can think of the elements as multiple parties against each other. Each element can cancel out one element that is different from itself. Since major element will always out-number the non-major ones (by problem's definition). We know that if we keep track of the number of elements that have’t been canceled out, the final answer after we traverse all array is the major element. 

We can use two variables to help with the process: `majority` and `count`. For each element $a$ we visit, 
- if `a==majority`, we increment the `count` by 1.
- if `a!=majority and count==0`, we change `majority` to be $a$ and increment `count` by 1. This means that all the elements we've visited have cancelled each other.
- if `a!=majority and count!=0`, we decrement `count` by 1.
The final value stored in `majority` is the majority element we want to return. The time complexity is O(n) and space complexity is O(1).

#### Example
Given an array [7, 5, 5, 7, 3, 7, 7], the result should be 7. 
```
array:       [7, 5, 5, 7, 3, 7, 7]
majority:     7, 7, 5, 5, 3, 3, 7
count:        1, 0, 1, 0, 1, 0, 1
```

#### Python3 implementation
```python3
def majority_element(A:List[int]) -> int :
    majority = A[0]
    count = 0

    for a in A:
        if a == majority:
            count += 1
        else:
            if count == 0:
                majority = a
                count += 1
            else:
                count -= 1

    return majority
```

## Heavy hitter problem
> You are reading a sequence of numbers. You are allowed to read the sequence twice. Devise an algorithm that uses O(k) memory to identify the words that occur more than $n/k$ times, where n is the length of the sequence.

(From EPI 24.31)  

#### Explanation
The gerneralized algorithm for this problem is a bit unituitive. Here's an excellent :star:[explanation](https://cs.stackexchange.com/a/91805 ) from Stackexchange.

#### Example
An example of this algorithm is: given an array [1, 5, 3, 1, 1, 5, 3, 3, 3, 1, 4]. The steps are as follows:
![heavy-hitter-steps](/majority-vote.PNG#center)

After visiting all 11 elements, all three blocks are filled and 4 elements are in the trash section. The second pass is crucial because we can only guarantee that the elements stored in the blocks appear more than n/(k+1) times instead of n/k times. Therefore, in the second pass, we need to count the actual occurrence of the elements in the blocks and filter out the elements that fails to meet the n/k requirements.

Naturally, we can use a hashmap to store the visited elements and their counts in the blocks. And we don't need to keep track of the trash since these elements won't be used in the final result.

The time complexity is still O(n) and space complexity is related to the size of k so it's O(k).

#### Python3 implementation
```python3
def heavy_hitter(A:List[int], k:int) -> List[int]: 
    buckets = {}
    n = len(A)

    for a in A:
        if len(buckets)==k:
            if a in buckets:
                buckets[a] += 1
            else:
            # if buckets are full and a is not in the buckets
            # we want to remove one element from every bucket
                for b in buckets:
                    buckets[b] -= 1
        else:
            buckets[a] = 1 + buckets.get(a, 0)
        % remove all entries where value ==0
        buckets = dict((k,v) for k,v in buckets.items() if v>0)

    % reset count in buckets
    for b in buckets:
        buckets[b] = 0

    % update actual counts for elements in buckets
    for a in A:
        if a in buckets:
            buckets[a] += 1
    
    return [e for e, c in buckets.items() if c>(n/k)]
    
```









