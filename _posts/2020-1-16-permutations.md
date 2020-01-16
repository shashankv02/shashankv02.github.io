---
layout: post
title: Generate all permutations
date: 2020-1-16 12:00:45
tags: [permutations]
categories: [combinatorial]
---

Given a collection of distinct integers, return all possible permutations.
[https://leetcode.com/problems/permutations/](https://leetcode.com/problems/permutations/)

** Thought Process **

We know for a n element array, there are n! permutations. To generate a
permutation, we need fill in each place by selecting one element from the
original array. Once we have filled the last position, we have generated a
single permutation. At each position, we need to try every element. A straight
forward code for this is:

```
def permute(self, nums: List[int]) -> List[List[int]]:
    res = []
    def helper(curr):
        if len(curr) == len(nums):
            res.append(curr)
            return
        for e in nums:
            if e not in curr:
                helper(curr + [e])
    helper([])
    return res
```

Here we are creating new list with one extra element with every recursive call.
Instead we can generate the permutation using the same list by swapping the
elements.

```
def permute(self, nums):
    """
    :type nums: List[int]
    :rtype: List[List[int]]
    """
    res = []
    def perm(i):
        if i == len(nums):
            res.append(nums[:])
        for j in range(i, len(nums)):
            # fill element in position i
            nums[i], nums[j] = nums[j], nums[i]
            perm(i+1)
            nums[i], nums[j] = nums[j], nums[i]
    perm(0)
    return res
```
