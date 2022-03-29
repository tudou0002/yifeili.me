---
title: "Max Rectangle"
date: 2022-03-20T10:50:33-04:00
draft: false
tags: ['algorithm']
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
We start from finding the max rectangle in a histogram, then apply the simialr idea to finding the max rectangle in a 2D array filled with 1. 

## Max rectangle in a histogram
>Given an array of integers heights representing the histogram's bar height where the width of each bar is 1, return the area of the largest rectangle in the histogram.

(from leetcode [84](https://leetcode.com/problems/largest-rectangle-in-histogram/))

#### Explanation
We can start off by doing a case analysis. We want to process the histogram one by one and modify the rectangle size we’ve visited. By observing a histogram array example, there are three trends in the given graph: increasing, consistent, decreasing. 

![case analysis for max rectangle in histogram](/max_histogram.PNG#center)

- increasing: we can extend the width of the rectangles to the right. 
- consistent: we can extend the width of the rectangles to the right. 
- decreasing: we cannot extend the width to the right, thus it’s a good time to compute the rectangles’ size. Notice how we can reversely extend the width of the rectangle.

Since increasing and consistency have the same result, we can merge them into a single condition. 

We will use a stack to store the histogram height and its index. 

- increasing&consistent: add height and index to the stack
- decreasing: pop all elements that are higher than the current height. Compute the rectangle size with the height we just popped and the width is the current_idx-poped_idx. Update the max_size if needed. Push the height to the stack and set the index to be the last index we just poped. (For reversely tracing back the rectangle)

An implementation trick is to append 0 to the end of the histogram so that the iteration will automatically pop the left heights from the stack and compute the max_area.

The overall time complexity and space complexity are both O(n).

#### Example
![max rectanle in histogram example1](/max_histogram_eg1.PNG#center)
![max rectanle in histogram example2](/max_histogram_eg2.PNG#center)

#### Python3 implementation
```python3
def largestRectangleArea(heights: List[int]) -> int:
    s = []
    max_area = 0
    
    for cur_i, cur_h in enumerate(heights+[0]):
        left_i = cur_i  # keep track of the left starting point of a rectangle
        while s and s[-1][0]>cur_h:
            height,prev_i = s.pop()
            width = (cur_i-prev_i)
            max_area = max(max_area, width*height)
            left_i = prev_i
            
        s.append((cur_h, left_i))

    return max_area
```
## Maximal rectangle in a 2d array filled with 1
> Given a rows x cols binary matrix filled with 0's and 1's, find the largest rectangle containing only 1's and return its area.

(from leetcode [85](https://leetcode.com/problems/maximal-rectangle/))
#### Explanation
We can treat the 2d array as multiple histograms. Apply the above algorithm row by row and update the max rectangle area accordingly. If the input array is a $m*n$ 2d array, the time complexity is O(mn) and space complexity is O(n) since we need an extra row to store the hisotgram heights.

#### Example
In this example, the max rectangle size should be 3. It can be found by the `largestRectangleArea()` function with an input of `[2, 1, 1]` as shown in figure(3). 
![max 2d array example](/max_2d_array.PNG)

#### Python3 implementation
```python3
def maximalRectangle(matrix: List[List[str]]) -> int:
    heights = [0]*len(matrix[0])
    result = 0
    for row in matrix:
        for i, c in enumerate(row):
            if c == "1":
                heights[i] += 1
            else:
                heights[i] = 0
        result = max(result, largestRectangleArea(heights))
    return result

```
