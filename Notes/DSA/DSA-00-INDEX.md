# DSA for DevOps Interviews — Focused Index

> **last_reviewed:** 2026-06-30  
> DevOps/SRE interviews need **light coding** — not SWE-level DP and graphs. This subset covers ~90% of what you'll see.

Advanced parts (trees, graphs, DP) are archived in [`_archive/`](./_archive/) for optional deep study.

---

## Core curriculum (study these)

| Part | Document | Topics | Time |
|------|----------|--------|------|
| 1 | [Fundamentals](./DSA-01-Fundamentals.md) | Big O, complexity | 1 week |
| 2 | [Arrays & Strings](./DSA-02-Arrays-Strings.md) | Two pointers, sliding window | 1 week |
| 3 | [Hash Maps & Sets](./DSA-03-HashMaps-Sets.md) | Frequency, Two Sum | 1 week |
| 4 | [Problem-Solving Patterns](./DSA-09-Problem-Solving-Patterns.md) | Strategy, must-do list | 1 week |

**Total: ~4 weeks** alongside PetClinic and infra study.

---

## What DevOps interviews actually ask

| Pattern | Example | Doc |
|---------|---------|-----|
| Parse logs / strings | Count errors, find IP | Arrays & Strings |
| Hash map counting | Top N URLs, duplicates | Hash Maps |
| Basic automation logic | Merge config dicts | Fundamentals |
| Simple BFS (rare) | Dependency order | _archive/ graphs |

They rarely ask: dynamic programming, advanced trees, graph shortest path.

---

## 25 must-do problems (DevOps subset)

From [Problem-Solving Patterns](./DSA-09-Problem-Solving-Patterns.md):

**Arrays & strings:** Two Sum, Valid Anagram, Contains Duplicate, Best Time Buy/Sell Stock, Maximum Subarray, Valid Parentheses

**Hash maps:** Group Anagrams, Top K Frequent Elements

**Sliding window:** Longest Substring Without Repeating

**Practice platform:** [Exercism Python](https://exercism.org/tracks/python) or [LeetCode](https://leetcode.com) Easy/Medium filter

1. Two Sum
2. Valid Anagram
3. Contains Duplicate
4. Best Time to Buy and Sell Stock
5. Maximum Subarray
6. Valid Parentheses
7. Group Anagrams
8. Top K Frequent Elements
9. Longest Substring Without Repeating Characters
10. Merge Intervals
11. Binary Search (basic)
12. Reverse String
13. FizzBuzz (warmup)
14. Ransom Note
15. Majority Element
16. Product of Array Except Self
17. Encode and Decode Strings (concept)
18. Logger Rate Limiter (design)
19. Implement Queue using Stacks
20. First Unique Character
21. Intersection of Two Arrays
22. Move Zeroes
23. Is Subsequence
24. Plus One
25. Climbing Stairs (only DP worth knowing)

---

## Archived (optional / SWE interviews)

| Part | Location |
|------|----------|
| Linked Lists, Stacks, Queues | [_archive/DSA-04](./_archive/DSA-04-LinkedLists-Stacks-Queues.md) |
| Trees | [_archive/DSA-05](./_archive/DSA-05-Trees.md) |
| Graphs | [_archive/DSA-06](./_archive/DSA-06-Graphs.md) |
| Dynamic Programming | [_archive/DSA-07](./_archive/DSA-07-Dynamic-Programming.md) |
| Advanced structures | [_archive/DSA-08](./_archive/DSA-08-Advanced-DataStructures.md) |
| Full interview guide | [_archive/DevOps-18](./_archive/DevOps-18-DSA-Interview.md) |

---

## Study schedule (with full-time DevOps job)

```
Week 1: Part 1 + Exercism python/lasagna, hello-world
Week 2: Part 2 + 8 LeetCode easy
Week 3: Part 3 + 8 LeetCode easy/medium
Week 4: Part 9 patterns + 9 more problems + mock interview
```

**Also study:** [Python for DevOps](../Programming/DevOps-15-Programming.md) — automation beats LeetCode for daily work.

---

## Related

- [Programming (Python/Go)](../Programming/DevOps-15-Programming.md)
- [PetClinic project](../../Projects/PetClinic.md)
- [Certifications](../../CERTIFICATIONS.md)
