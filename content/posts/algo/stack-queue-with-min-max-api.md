---
title: "Stack/Queue With Min/Max API"
date: 2022-08-03T19:10:32-04:00
draft: false
tags:  ['algorithm', 'queue', 'stack']
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

## Min stack
> Design a stack that supports push, pop, top, and retrieving the minimum element in constant time.
Implement the MinStack class:  
> `MinStack()` initializes the stack object.  
`void push(int val)` pushes the element val onto the stack.  
`void pop()` removes the element on the top of the stack.  
`int top()` gets the top element of the stack.  
`int getMin()` retrieves the minimum element in the stack.  
You must implement a solution with O(1) time complexity for each function.  

(from leetcode [155](https://leetcode.com/problems/min-stack/))

#### Explanation
Due to the special characteristic of stack, the later added elements will be poped first. Assume we have two elements `a` and `b`. We first push `a` to a stack, then we push `b` to the same stack. Let's say the minimum element unto the element `a` is `min_a` and the minimum element after we push `b` is `min_b`. We then have `min_b = min(min_a, b)`. Also `min_a` is not affected regarding the later elements we push. Because no matter what elements after `a` will be poped before `a`. we can implement a caching variable to store the minimum value upto each element. Here, I use `namedtuple` for its readability. We use O(n) extra space to trade for O(1) time complexity for each function.

#### Python3 Implementation
```python3
class MinStack:
    ElementWithCachedMin = collections.namedtuple('ElementWithCachedMin',
                                               ('element', 'min'))

    def __init__(self):
        self._stack_with_cached_min: List[ElementWithCachedMin] = []

    def push(self, val: int) -> None:
        self._stack_with_cached_min.append(self.ElementWithCachedMin(val,
            val if not self._stack_with_cached_min else min(val, self.getMin())))

    def pop(self) -> None:
        return self._stack_with_cached_min.pop().element
            
    def top(self) -> int:
        return self._stack_with_cached_min[-1].element

    def getMin(self) -> int:
        return self._stack_with_cached_min[-1].min

```
(Modified from code in EPI P101)

## Max Queue
> Implement a queue with enqueue, dequeue, and max operations. The `max` operation returns the maximum element currectly stored in the queue.

(from EPI 8.9)

#### Explanation
A simple cache storing the maximum element so far along with each element won't work for a queue. Because queue follows the "first in, first out" order. Every time we enqueue an element `x`, the previously added elements will have to be aware of `x`. Since `x` will stay in the queue longer than any of its previously added elements. On the other hand, we don't care the previous elements once there's a larger element newly added to the queue. This special feature indicates we can use an additional monotonic queue as a cache for the maximum values. Here's a post I wrote about [monotonic queue](http://yifeili.me/posts/algo/monotonic-queue/).


#### Python3 Implementation
```python3
Class MaxQueue:
    def __int__(self):
        self._queue = collections.deque()
        self._max_deque = collections.deque()

    def enqueue(self, x):
        self._queue.append(x)
        while self._max_deque and self._max_deque[-1]<x:
            self._max_deque.pop()
        self._max_deque.append(x)

    def dequeue(self):
        result = self._queue.popleft()
        if result == self._max_deque[0]:
            self._max_deque.popleft()
        return result

    def max(self):
        return self._max_deque[0]

```