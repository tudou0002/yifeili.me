---
title: "Morris Traversal"
date: 2022-07-19T15:28:31-04:00
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
A common operation on a binary tree is to **traverse** all the nodes in the tree. Normally, there are three ways to do the traversing: pre-order, in-order and post-order. Recursive algorithms are well suited for this traversing task. However, we need a stack to keep track of the previous nodes we want to go back, leading to a O(n) space complexity.   

In this post, I will introduce an algorithm called **Morris Traversal** that is designed to do the in-order traversal with O(1) space complexity and O(n) time complexity. The intuition is we manipulate the right pointers of the rightmost nodes in each left subtree so we can directly access the next node when finish visiting the left subtree.

## Binary Search Tree Iterator
> Implement the BSTIterator class that represents an iterator over the in-order traversal of a binary search tree (BST)

(from leetcode [173](https://leetcode.com/problems/binary-search-tree-iterator/))

#### Explanation
For the in-order traversal, the general rule is to follow the `left child -> node -> right child` visiting order for each node. The steps of the Morris traversal for in-order traversal are:
- we start from `current_node = root`
- while `current_node` is not NULL:
    - if `current_node` has a left child (go to left half and change right child pointer if needed):
        - find the rightmost node in its left subtree as `prev_node`
            - if the `prev_node`'s right child is `current_node`, reset the right child pointer and visit `current_node`. Assign `current_node` to be `current_node`'s right child
            - else, set `prev_node`'s right child to be `current_node`. Assign `current_node` to be `current_node`'s left child
    - else (no left subtree to check, time to visit the current node and then move on to the right half):
        - visit `current_node`, assign `current_node` to be `current_node`'s right child

To implement an iterator with this algorithm, we just need to keep track of an attribute `self.cur`. This attribute stores the node pointer of the node we want to output next. The naming can be a little misleading though, I named it as `self.cur` just to be consistent with the variable name I use in the above explanation.

#### Example
Here's a step-by-step illustration of how morris traversal works on a binary search tree. A node with orange-filled color means this node has been visited. 
![Morris traversal step-by-step](/morris.PNG)

#### Python3 Implementation
```python3
    class BSTIterator:

    def __init__(self, root: Optional[TreeNode]):
        self.cur = root

    def next(self) -> int:
        while self.cur:
            if self.cur.left:
                prev = self.cur.left
                while prev.right and prev.right!=self.cur:
                    prev = prev.right
                if prev.right == self.cur:
                    prev.right = None
                    res = self.cur.val
                    self.cur = self.cur.right
                    return res
                else:
                    prev.right = self.cur
                    self.cur = self.cur.left
            else:
                res = self.cur.val
                self.cur = self.cur.right
                return res

    def hasNext(self) -> bool:
        return self.cur!=None
```

## Recover Binary Search Tree
> You are given the root of a binary search tree (BST), where the values of exactly two nodes of the tree were swapped by mistake. Recover the tree without changing its structure.

(from leetcode [99](https://leetcode.com/problems/recover-binary-search-tree/))

#### Explanation
We need to find a way to detect the two nodes being swapped in the tree. The in-order traversal of a normal binary search tree is an array of non-decreasing sequence. We can in turn observe what the in-order traversal will be like when exactly two nodes are swapped.   
Assume the in-order traversal of a correct BST is `1,2,3,4,5,6`
- `4` and `5` are swapped, traversal output will be `1,2,3,5,4,6`. There's a decreasing trend between `5` and `4`.
- `3` and `5` are swapped, traversal output will be `1,2,5,4,3,6`. There are two decreasing trend: `5` -> `4` and `4` -> `3`.
- `2` and `5` are swapped, traversal output will be `1,5,3,4,2,6`. There are two decreasing trend: `5` -> `3` and `4` -> `2`.

We can observe from the above examples that if we keep track of all the numbers involved in decreasing trends, the first and the last are the original swapping numbers. 

Now we can just use morris traversal and swap the two nodes we find before function returns.


#### Python3 Implementation
```python3
def recoverTree(self, root: Optional[TreeNode]) -> None:
    cur = root
    last = TreeNode(float('-inf'))
    swaps = []
    while cur:
        if cur.left:
            prev = cur.left
            while prev.right and prev.right!=cur:
                prev = prev.right
            if prev.right == cur:
                prev.right = None
                if cur.val<last.val:
                    # last = cur.val
                    swaps.append(last)
                    swaps.append(cur)
                last = cur
                cur = cur.right
                
            else:
                prev.right = cur
                cur = cur.left
        else:
            if cur.val<last.val:
                swaps.append(last)
                swaps.append(cur)
            last = cur
            cur = cur.right

    swaps[0].val, swaps[-1].val = swaps[-1].val, swaps[0].val
```