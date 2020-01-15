---
layout: post
title: Test if a binary tree is height balanced
date: 2020-1-15 12:00:45
tags: [binary-tree]
categories: []
---

Given a binary tree, determine if it is height-balanced.

[https://leetcode.com/problems/balanced-binary-tree/](https://leetcode.com/problems/balanced-binary-tree/)

**Notes and clarifications**

The tricky part of the question is in the definition of **height balanced** - a
binary tree in which the left and right subtrees of **every node** differ in
height by no more than 1. The property of height balance should be held at every
node in the tree.

**Thought process**

Let's think about how to check if tree is height balanced at a single node -

To know if tree height balanced at node, we need to know the height
of left subtree and right subtree. If their difference is less than or equal to
1, the tree is height balanced at that node.

But we need to make sure the tree is height balanced at every node. So we need
to visit every node and so we need to use a tree traversal algorithm like
pre-order, post-order or inorder. We cannot use inorder because, we need heights
of left and right subtrees to check if current node is balanced. During
traversal if we find any node is not height balanced, then the tree is not
height balanced and we can stop terminate early. So along with height, we also
need to return an extra value to indicate if subtree is balanced or not to the
parent.

```
def is_height_balanced(root) -> bool:
    def post_order(node):
        # Returns a tuple of height and a boolean
        # indicating if tree at node is balanced
        if not node:
            return 0, True
        left = post_order(node.left)
        if not left[1]:
            return 0, False
        right = post_order(node.right)
        if not right[1]:
            return 0, False
        height = max(left[0], right[0]) + 1
        is_balanced = abs(left[0] - right[0]) <= 1
        return height, is_balanced
    return post_order(root)[1]
```

**Complexity:**

Time complexity is same as post order - `O(n)`

If the tree is not height balanced, we terminate early.

Space complexity is - `O(h)` implicit stack space where `h` is height of tree.
