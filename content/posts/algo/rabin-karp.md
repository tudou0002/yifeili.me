---
title: "Rabin Karp"
date: 2022-04-20T22:28:06-04:00
draft: false
tags: ['algorithm', 'string'] 
author: Yifei Li
showToc: true
TocOpen: false
math: true
hidemeta: false
comments: false
description: "An elegant hash function for encoding strings."
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

Rabin-Karp algorithm is commonly used in some string problem when it comes to finding the occurrences of a substring. The motivation of this algorithm is to devise a hash function that can efficiently represent distinctive strings. Therefore, this kind of hash function are sometimes refered to as a "rolling hash". This hash function is usually paired with sliding window because it's very efficient to compute the hashed result with small shifts.

## Repeated DNA Sequences
> The DNA sequence is composed of a series of nucleotides abbreviated as 'A', 'C', 'G', and 'T'. Given a string s that represents a DNA sequence, return all the 10-letter-long sequences (substrings) that occur more than once in a DNA molecule. You may return the answer in any order.

(from leetcode [187](https://leetcode.com/problems/repeated-dna-sequences/)

#### Explanation
Given a fixed size 10, how can we find duplicated substrings that have a size of 10? An intuition is to inspect all possible 10-letters substrings with all the other 10-letters substrings. When comparing two substrings, the brute-force method has a time complexity of O(n) by comparing the characters one by one. We can use Rabin-Karp to improve the average time complexity. By applying a rolling hash function and comparing hash values, the time complexity of comparing two substrings will be O(1) but we cannot fully avoid the collision so the worst case is still O(n). 

Step by step algorithm of Rabin-Karp is:
1. Define the `base` value. Usually `base` a prime number and >= maximum the maximum of all possible chars to avoid collision
2. Compute hash values of substring(s)
3. Iterate through the whole string, if a substring's hash value is seen, we compare the two substrings letter-by-letter. Update the substring's hash value. The detailed computation step is shown in the example below

#### Example
For the simplicity of the example, I'll find 3-letter-long duplicate instead of 10-letter-long one. And I'll use a letter to number map as `{'A': 1, 'C':2, 'G': 3, 'T':4}`. You can use your own choice for this L2N map or use the `ord()` function as long as each character is represented by distinctive numbers. I'll choose `base=5` to avoid collision. 

```
s: "TACGACGA"
result: ["ACG", "CGA"]
```
![Example of finding the duplicate substring using rabin karp](/rabin_karp.PNG#center)

#### Python3 implementation
```python3
def findRepeatedDnaSequences(s: str) -> List[str]:
    n = len(s)
    if n<10:
        return []
    results = set()
    base = 5
    seen_hash = set()
    power = base**9

    hash_val = functools.reduce(lambda h,c:h*base+ord(c), s[:10], 0)
    
    for i in range(10, n):
        if hash_val in seen_hash:
            results.add(s[i-10:i])
        else:
            seen_hash.add(hash_val)
        # move window to right and update rolling hash
        hash_val -= ord(s[i-10])*power
        hash_val = hash_val*base + ord(s[i])
    
    #inspect the last 10 letters' hash
    if hash_val in seen_hash:
        results.add(s[n-10:n])
    return list(results)
```

## Longest Duplicate Substring

>Given a string s, consider all duplicated substrings: (contiguous) substrings of s that occur 2 or more times. The occurrences may overlap.
>Return any duplicated substring that has the longest possible length. If s does not have a duplicated substring, the answer is "".

(from leetcode [1044](https://leetcode.com/problems/longest-duplicate-substring/))

#### Explanation
This problem can be decomposed into two steps: inspect substrings with different length + test if a substring with fixed length is duplicated. The first part can be quickly implemented by a binary search approach (for more info about binary search: Binary Search [Basic](https://www.yifeili.me/posts/binary-search-basic/) and [Advanced](https://www.yifeili.me/posts/binary-search-advanced/)). The second part can be a helper function validating if such a fixed length substring exists. The helper function is implemented with Rabin-karp and base is selected to be 27 since all characters are lower case English letters.

#### Python3 implementation
```python3
def longestDupSubstring(s: str) -> str:
        n = len(s)
        left, right = 1, n
        res = ''
        
        def rabin_karp(size):
            mapping = {} # fingerprint: actual value
            base = 27
            s_hash = functools.reduce(lambda h, c: h*base+ord(c), s[:size],0)
            power_s = base**max(size-1,0)
            
            # i is the index of the next char for the current substr
            for i in range(size, n):
                if s_hash in mapping:
                    return (True, s[i-size:i])
                mapping[s_hash] = i-1
                
                s_hash -= ord(s[i-size])*power_s
                s_hash = ord(s[i]) + s_hash * base
                
            if s_hash in mapping:
                return (True, s[~size+1:])
            
            return (False, '')
                
        while left<=right:
            size = left + (right-left)//2
            flag, record = rabin_karp(size)
            if flag:
                if size > len(res):
                    res = record
                    # print(res)
                left = size+1
            else:
                # move to smaller size
                right = size - 1

        return res
```