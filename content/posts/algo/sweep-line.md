---
title: "Sweep Line"
date: 2022-05-23T10:31:54-04:00
draft: false
tags:  ['algorithm']
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
Sweep line algorithm (normally) uses a vertical line to sweep given intervals and increment the result when events occurs. The common paradigm for a sweep line algorithm is

1. **Define** the events/items from the problem setting. What are the attributes of an event we want to keep track of?
2. Converts the given input into an array of events/items and **sort** them. The sorting key may vary based on the problem setting.
3. **Identify** the situation when we want to update our result and update it

## Meeting Room II
> Given an array of meeting time intervals consisting of start and end times [[s1,e1],[s2,e2],...] (si < ei), find the minimum number of conference rooms required.

(from lintcode [919](https://www.lintcode.com/problem/919))

#### Explanation
To paraphrase the problem, it’s the same as counting the maximum concurrent meetings across the overall time frame. We can visualize it by using a vertical sweep line on a horizontal time axis. We can keep track of a counter. When the sweep line hits the left end point of a meeting, we know there's one meeting start therfore increment the counter by 1. When the sweep line hits the right end point of a meeting, it's an indicator of meeting end, so we decrement the counter by 1 (We don't care which meeting ends). 

From the analysis above, events can be defined as "an end point of a meeting". Useful attributes include time of the end point and whether it's a left end point. I use `collections.namedtuple()` to define an event for the readability. After converting meeting info into events, we sort them based on the `time` attribute. With all these preparation, we can then update the counter as we discussed. In the end, we return the maximum counter we've ever seen during the process. 

The time complexity is O(n) and space complexity is O(1).

#### Example
```python3
Input: [(3,5),(1,8), (6,7)]
Output: 2
```
![Illustration of how sweep line moves in meeting room problem](/meeting_room.PNG#center)
#### Python3 Implementation
```python3
"""
Definition of Interval:
class Interval(object):
    def __init__(self, start, end):
        self.start = start
        self.end = end
"""

class Solution:
    """
    @param intervals: an array of meeting time intervals
    @return: the minimum number of conference rooms required
    """
    def min_meeting_rooms(self, intervals: List[Interval]) -> int:
        Event = collections.namedtuple('Event',('time','is_start'))
        events = []
        for i in intervals:
            events.append(Event(i.start, True))
            events.append(Event(i.end, False))
        
        events.sort(key=lambda x:x.time)
        res = 0
        cur = 0
        for e in events:
            if e.is_start:
                cur += 1
                res = max(cur, res)
            else:
                cur -= 1
        return res
```

## Rectangle Area II
> You are given a 2D array of axis-aligned rectangles. Each rectangle[i] = [xi1, yi1, xi2, yi2] denotes the ith rectangle where (xi1, yi1) are the coordinates of the bottom-left corner, and (xi2, yi2) are the coordinates of the top-right corner.
Calculate the total area covered by all rectangles in the plane. Any area covered by two or more rectangles should only be counted once.
Return the total area. Since the answer may be too large, return it modulo $10^9 + 7$.

(from leetcode [850](https://leetcode.com/problems/rectangle-area-ii/))

#### Explanation
Following the steps in the introduction section, we have:
1. We define each event with attributes `(x_ccord, y_lower_coord, y_upper_coord, is_left)`. We care about the x coordinate of our sweep line. Area = length*width, two y coordinates represents the width info for each rectangles. When we sweep the line along x axis, we need to keep track of when is the start of a rectangle and the end of a rectangle, i.e. we need keep track of if the event is a left edge or a right edge.
2. Convert input format in to an array of events and sort events based on their `x_coord`.
3. Keep track of a list of y intervals that we are visiting. Increment the total area by (x difference of two sweep lines)*(length being covered on the y axis). The `merge()` function is the same as solution in leetcode [56](https://leetcode.com/problems/merge-intervals/).

#### Example
```python3
Input : [[1,1,4,3], [2,2,3,4], [2,5,3,6]]
Output: 8
```
![Step-wise illustration of rectangle area problem](/rect_area.PNG#center)

#### Python3 Implementation
```python3
def rectangleArea(self, rectangles: List[List[int]]) -> int:
    Item = collections.namedtuple('Item', ('x','y1','y2','is_left'))
    def merge(heights):
        # input has a format [[y1,y2],[y1,y2]...]
        out = []
        for i in sorted(heights):
            if out and i[0] <= out[-1][1]:
                out[-1][1] = max(out[-1][1], i[1])
            else:
                out.append([i[0],i[1]])
        return sum(b-a for a,b in out)
    
    # convert rectangles into Items
    items = []
    for r in rectangles:
        items.append(Item(r[0],r[1],r[3],True))
        items.append(Item(r[2],r[1],r[3],False))
    
    # sort by x coord
    items.sort(key=lambda a:a.x)
    area = 0
    prev_x = items[0].x
    visiting = []
    # operate the vertical sweep line 
    for item in items:
        area += (item.x-prev_x)*merge(visiting)
        if item.is_left:
            visiting.append([item.y1,item.y2])
        else:
            visiting.remove([item.y1,item.y2])
        prev_x = item.x
    
    return area % (10**9+7)
```