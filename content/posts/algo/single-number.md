---
title: "Single Number"
date: 2022-03-24T09:53:57-04:00
draft: false
tags:  ['algorithm', 'array']
author: Yifei Li
showToc: true
TocOpen: false
math: true
hidemeta: false
comments: false
description: "Bit manipulation"
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
In this post, I summarized several questions that share a similar template: *Find an entry that appears x times in an integer array where others appear y times*. All the solutins have a O(n) time complexity and O(1) space complexity.

## Element appears twice except for one appears once
>Given a non-empty array of integers nums, every element appears twice except for one. Find that single one.
You must implement a solution with a linear runtime complexity and use only constant extra space.

(from leetcode [136](https://leetcode.com/problems/single-number/))

#### Explanation
The brute-force solution will be to count the occurrence of each element and return the one that only appears once. The space complexity will be O(n) which violates the O(1) requirement. 

We'll approach this question from the bit-wise persepctive. Notice that any number XOR itself is 0. We can just iterate the whole array and keep XOR-ing the current result. Whatever left in our result is the element that appeared only once. 

#### Example
Assume we are given an array `A = [2, 3, 1, 2, 3]`, we can convert each element into their binary format [$(10)_2$, $(11)_2$, $(01)_2$, $(10)_2$, $(11)_2$]. The step-by-step walkthrough of this solution are:
```
A:          [ 2,  3,  1,  2,  3]
bin_A:      [10, 11, 01, 10, 11]
result:      10  01  00  10  01
```
Another way to process this solution is doing a case analysis bit by bit. A more generalized explanation is provided in the next question. 

#### Python3 Implementation
```python3
def singleNumber(nums: List[int]) -> int:
        result = 0
        for num in nums:
            # bitwise XOR operator: ^
            result ^= num
            
        return result

```


## Element appears three times except for one appears once
> Given an integer array, in which each entry but one appears in triplicate, with the remaining element appearing once, find the element appearing once. 
(from EPI 24.17)

#### Explanation
In the previous example, we applied the XOR operation to get the element that appears only once. The same approach would fail in this problem setting because we will have `a^a^a = 0^a = a` for each triplicate. We'll then analyze this problem from a bit-by-bit perspective. 

Assume the target element that appears once is $x$. We then convert every element into its binary form. At each bit position i, $x_i$ could have two cases:
- $x_i = 0$: Number of 0's at this position across all elements will be $(3n+1)$. Number of 1's at this position across all elements will be $(3m)$. $n, m$ are the number of distinctive elements whose bit is 0, 1 respectively at this position.
- $x_i = 1$: Number of 0's at this position across all elements will be $(3n)$. Number of 1's at this position across all elements will be $(3m+1)$.$n, m$ are the number of distinctive elements whose bit is 0, 1 respectively at this position.

Our goal is to distinguish between the two cases. We can focus on the number of 1's at each bit position, and compare the mod 3 result. 
- $x_i = 0$: $(3m)$%3 = 0
- $x_i = 1$: $(3m+1)$%3 = 1

After settling every bit for target element $x$, we can convert it into its decimal format. We just need a 1*32 array to store each $x_i$ (assume each element is between $[-2^{31}, 2^{31}-1]$), therefore the time complexity is still constant.

#### Example
![single element among triplicates example](/single_element.PNG#center)

#### Python3 Implementation
```python3
def single_number(nums: List[int]) -> int:
    result_bits = [0]*32
    
    for idx in range(len(nums)):
        num = nums[idx]
        for i in range(31, -1, -1):
            result_bits[i] += num & 1  # add ith bit to result
            num >>= 1
    
    result = result_bits[0]%3
    # convert sum of bit array into decimal format
    for b in result_bits[1:]:
        result <<= 1
        result += (b%3)

    # handle negative case
    return result if result<2**31 else result-2**32
```


## Two elements appear twice except for two appear once
> Given an integer array nums, in which exactly two elements appear only once and all the other elements appear exactly twice. Find the two elements that appear only once. You can return the answer in any order.
You must write an algorithm that runs in linear runtime complexity and uses only constant extra space.

(from leetcode [260](https://leetcode.com/problems/single-number-iii/))
#### Explanation
Assume the two elements we want to find is $x$ and $y$.
- Traverse the entire array, XORing every element to get the value of x^y. Let's say `r = x^y`
- Since x is not equal to y, they must differ in some bits and results in 1's in r. We find the position `i` of any 1 in r.
- If we split the entire array into two parts by whether the ith position is 1, we will spilt x and y into two separate parts. We then XOR on either part and get one of results. WLOG, let's assume we've known x by this step. The other result can be computed by y = r^x.

#### Python3 Implementation
```python3
def singleNumber(self, nums: List[int]) -> List[int]:
    xor,idx = 0, 0
    for num in nums:
        xor ^= num
    # find any 1's position in the XOR result
    for bit in range(32):
        if xor & (1<<bit):
            idx = bit
            break
    # find one of the results
    x = 0
    for num in nums:
        if num & (1<<idx):
            x ^= num
    
    return [x, x^xor]
```