---
title: "Count Complete Binary Tree"
date: 2022-04-12T08:03:09-04:00
draft: false
tags: ['algorithm', 'binary tree']  
author: Yifei Li
showToc: true
TocOpen: false
math: true
hidemeta: false
comments: false
description: "Just want to document this beautiful solution XD"
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

> Given the root of a complete binary tree, return the number of the nodes in the tree.
According to Wikipedia, every level, except possibly the last, is completely filled in a complete binary tree, and all nodes in the last level are as far left as possible. It can have between 1 and $2^h$ nodes inclusive at the last level h.
Design an algorithm that runs in less than O(n) time complexity.

(from leetcode [222](https://leetcode.com/problems/count-complete-tree-nodes/))

## Explanation
The solution is introduced by this discussion [post](https://leetcode.com/problems/count-complete-tree-nodes/discuss/61958/Concise-Java-solutions-O(log(n)2)). According to this solution, we define the height of a node to be the number of nodes along the path when we keep going to the left. We'll then count the total nodes recursively:
- when `root's height - 1 == root.right's height` (situation in Fig(1)): left subtree of root is a full binary tree with $2^{(root.right's height)}-1$ nodes
- when `root's height - 1 != root.right's height` (situation in Fig(2)): right subtree of root is a full binary tree with $2^{(root.right's height)}-1$ nodes
![Example of two conditions](/count_binary_tree.PNG#center)
Either of the above situation, we can increment the count by the full subtree plus the root itself (red nodes in the figures above) and move on to the other subtree. 

## Python3 Implementation
```python3
def countNodes(root: Optional[TreeNode]) -> int:
    if not root:
        return 0
    def height(n):
        if not n:
            return -1
        else:
            return 1 + height(n.left)
    
    h = height(root)
    if height(root.right) == h-1:
        # left half subtree guaranteed to be complete with height of (h)
        return (1<<h) + countNodes(root.right)
    else:
        # right hald subtree guaranteed to be complete with height of (h-1)
        return (1<<h-1) + countNodes(root.left)
```