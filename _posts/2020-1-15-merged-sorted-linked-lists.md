---
layout: post
title: Merged two sorted linked lists
date: 2020-1-15 12:00:45
tags: [linked-lists]
categories: []
---

Merge two sorted linked lists and return it as a new list. The new list should
be made by splicing together the nodes of the first two lists.

[https://leetcode.com/problems/merge-two-sorted-lists/](https://leetcode.com/problems/merge-two-sorted-lists/)

**Thought process**

This is straightforward problem of maintaing two pointers for each list and
moving the smaller one forward.

_Tip:_ Usually with linked list problems, using a
dummy head variable whose next pointer refers to the actual head makes the code
simpler in handling the edge cases of empty lists etc.

```
class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None

def merge_lists(l1: ListNode, l2: ListNode) -> ListNode:
    dummy_head = itr = ListNode(0)
    while l1 and l2:
        if l1.val <= l2.val:
            itr.next = l1
            itr = itr.next
            l1 = l1.next
        else:
            itr.next = l2
            itr = itr.next
            l2 = l2.next
    remaining_list = l1 or l2
    while remaining_list:
        itr.next = remaining_list
        remaining_list = remaining_list.next
        itr = itr.next
    return dummy_head.next
```

**Complexity**
Time - O(n) where is n is sum of number of nodes in l1 and l2
Space - O(1)
