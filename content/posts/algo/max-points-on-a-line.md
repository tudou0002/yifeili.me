---
title: "Max Points on a Line"
date: 2022-03-24T09:42:06-04:00
draft: false
tags:  ["algorithm"]
author: Yifei Li
showToc: true
TocOpen: false
math: true
hidemeta: false
comments: false
description: "How to design an accurate hash function for a line in 2D space"
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
In image processing, line detection is a task that takes a collection of n edge points and finds all the lines on which these edge points lie (from [wikipedia](https://en.wikipedia.org/wiki/Line_detection)). In the below problem, we are given a collection of points $N$ and want to find the lines that passes the most number of points. 

>Given an array of points where points[i] = [xi, yi] represents a point on the X-Y plane, return the maximum number of points that lie on the same straight line.

(from leetcode [149](https://leetcode.com/problems/max-points-on-a-line/))

#### Explanation
We know that two points can settle a line. We can then inspect the lines between each pair of points in $N$ and check if we've already seen them. Assume there are n points in the collection $N$, we need to compute the line representations $\frac{n(n-1)}{2}$ times in total. In addtion, we need to introduce a hash map with the key as `line representation` and value as `number of points on this line`. 

Given two points $(x_1, y_1)$ and $(x_2, y_2)$ in a 2D space, there are several ways to represent the defined line:

- $ax+by+c = 0, a = y_2-y_1, b=x_1-x_2, c = x_2y_1-x_1y_2$: We can represent each line with a integer tuple `(a, b, c)`.

- $y=kx+b, k=\frac{y_2-y_1}{x_2-x_1}, b = \frac{x_2y_1-x_1y_2}{x_2-x_1}$: This is the most common format I use to define a line. However, we may run into some problem due to the finite precision arithmetic in the division step. To avoid the potential error, we can record the denominator and numerator separately in their integer format. In addition, we can fix a line after knowing two points on it and its $k$ value. Therefore, the line can be represented by $(y_2-y_1, x_2-x_1)$.

- $\rho = x\cos(\theta) + y\sin(\theta)$: This definition is used in the [Hough transformation](https://en.wikipedia.org/wiki/Hough_transform) technique designed for line dectection. It saves computing time by controling the precision of angle $\theta$. This feature significantly contribute to the efficiency in real world application. However, it's not accurate enough in terms of this problem's requirement.

Either the first or the second representation works for this problem, I'll go with the second since it only require two numbers for each line. After choosing the appropriate line representation format, we need to take care of an edge case: vertical lines. We can manually check this case and use `(0, 1)` as a special category. The overall time complexity and space complexity are all O($n^2$)

#### Python3 Implementation

```python3
def maxPoints(points: List[List[int]]) -> int:
    n = len(points)
    max_count = 1   # edge case when there's only one point in points
    
    def compute_line_representation(p1, p2):
        x1, y1 = p1
        x2, y2 = p2
        
        # special case: vertical line
        if x1 == x2:
            return (0, 1)
        else: 
            hy, hx = y2-y1, x2-x1
            gcd = math.gcd(hy, hx)
            hx, hy = hx/gcd, hy/gcd
            if hx < 0:
                return (-hx, -hy)
            else:
                return (hx, hy)
    
    for i in range(n):
        p1 = points[i]
        line_map = dict()
        
        for j in range(i+1, n):
            p2 = points[j]
            
            line_rep = compute_line_representation(p1, p2)
            line_map[line_rep] = line_map.get(line_rep, 1) + 1
            max_count = max(max_count, line_map[line_rep])
    
    return max_count
```