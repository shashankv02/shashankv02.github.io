---
layout: post
title: Generate next permutation
date: 2020-1-16 12:00:46
tags: [permutations]
categories: [combinatorial]
---

Implement next permutation, which rearranges numbers into the lexicographically next greater permutation of numbers.

If such arrangement is not possible, it must rearrange it as the lowest possible order (ie, sorted in ascending order).

The replacement must be in-place and use only constant extra memory.

[https://leetcode.com/problems/next-permutation/submissions/](https://leetcode.com/problems/next-permutation/submissions/)

```
def nextPermutation(self, nums: List[int]) -> None:
    """
    Do not return anything, modify nums in-place instead.
    """
    # find the inversion point which is right most element smaller than element next to it.
    # swap it with lowest element greater than inversion point to the right of inversion point
    # reverse all elements after the inversion point

    inversion_pt = len(nums)-2
    while inversion_pt >= 0 and nums[inversion_pt] >= nums[inversion_pt+1]:
        inversion_pt -= 1

    if inversion_pt < 0:
        nums.sort()
        return

    for i in reversed(range(inversion_pt+1, len(nums))):
        if nums[i] >  nums[inversion_pt]:
            nums[i], nums[inversion_pt] = nums[inversion_pt], nums[i]
            break

    nums[inversion_pt+1:] = reversed(nums[inversion_pt+1:])
```

This can be used to generate all permutations if there are duplicates by
generating all permutations in lexographical order using nextPermutation.

```

def permuteUnique(self, nums: List[int]) -> List[List[int]]:
    curr = sorted(nums)
    starting = curr[:]
    res = []
    self.nextPermutation(curr)
    res.append(curr[:])
    while curr != starting:
        self.nextPermutation(curr)
        res.append(curr[:])
    return res
```
