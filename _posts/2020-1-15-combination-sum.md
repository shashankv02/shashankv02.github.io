---
layout: post
title: Combination Sum/Coin Change/Number of score combinations
date: 2020-1-15 12:00:45
tags: [linked-lists]
categories: []
---

Given a set of candidate numbers (candidates) (without duplicates) and a target
number, find all unique combinations in candidates where the candidate
numbers sums to target.

The same repeated number may be chosen from candidates unlimited number of times.

Other variants of the same problem include given a set of denominations, find
the number of ways to make change for an amount or given a different ways to
gaining points in a game, calcuate number of ways to reach a target.

**Thought process**

The key to solve these type of problems is check if solving a smaller problem
helps in solving the actual problem i,e., identify the recurrance relation.
There are two parameters to the function we need to solve, the target and list
of candidates. An immediate approach that might look like it would work is:

```
candidates = [2,3,7]
target = 12
f(target, candidates) = f(target-2, candidates) + f(target-3, candidates) + f(target-7, candidates)
f(12) = f(9) + f(8) + f(7)  where f(0) = 1 as base condition
```

The mistake in the above recurrance relation is that it actually calculates the
number of permutations - not number of combinations. If it is not apparent, draw
the call graph and you would see. Since we need to combinations, we need to
enforce an order on the `canidates` parameter. We need it to be involved in the
recurrance relation.

```
If there are n candidates:
f(target, n) = f(target, n-1) + f(target - candidates[n-1], n)
```

There are repeating problems, so we can cache them if we are solving the problem
top down.

We can actually solve this problem space efficiently in bottom-up approach. See
[https://stackoverflow.com/a/59551834/5459201](https://stackoverflow.com/a/59551834/5459201)
to understand the thought process on arriving at the space efficient bottom up
approach.

```
def number_of_combinations(target, candidates)
    cache = [0] * (target + 1)
    cache[0] = 1
    for cand in candidates:
        for c in range(target + 1):
            if c < cand:
                continue
            cache[c] += cache[c-s]
    return cache[-1]
```

**Complexity**

Time: O(target x number of candidates)

Space: O(target)
