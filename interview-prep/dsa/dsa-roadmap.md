# DSA Roadmap: Blind 75 in JavaScript

75 problems, ordered by NeetCode's roadmap graph (each pattern builds on the ones before it). About 13 weeks at 1-2 hours a day and roughly 6 problems a week. Solve on LeetCode in JavaScript, tick boxes here.

## The rules

1. **Learn first, briefly.** For each new pattern, spend at most 20-30 minutes on one explainer (a NeetCode video, another YouTube video, or one blog post). Then go solve.
2. **Attempt before help.** Give each problem 25-30 minutes. If stuck, read or watch a solution, close it, and re-solve from scratch.
3. **Dry-run by hand.** Trace your code on a small input before submitting.
4. **Write a note** for every problem in `learning-in-public`: approach, time and space complexity, one mistake you made.
5. **Re-solve** each problem after 3 days and again after about a week. The last item of each week is a revision slot for this.
6. **JS utility of the week.** One interview-style JS function per week (listed below), written from scratch.

Notes path: `interview-prep/dsa/practice/leetcode/<pattern>/NNNN-problem-name.md`, with pattern folders named after the sections below (`arrays-hashing`, `two-pointers`, ...).

## Where to learn (use whichever works for you)

|Source|Use|
|---|---|
|NeetCode video for each problem|After your own attempt|
|Codevolution playlist|Data structure videos when you reach that topic (see `playlist-duration.xlsx`)|
|Any other YouTube video or blog|For the pattern explainer. Search "<pattern> pattern leetcode" and pick one. Stop at one.|
|GreatFrontEnd, LeetCode "30 Days of JavaScript"|JS utility functions and frontend questions|

Premium note: problems marked **(P)** are paywalled on LeetCode. Check whether NeetCode offers a free version; if not, skip them. The other patterns are still fully covered.

## Weekly plan

### Week 1: Arrays & Hashing

- [x] 217 Contains Duplicate
- [x] 1 Two Sum
- [ ] 242 Valid Anagram
- [ ] 49 Group Anagrams
- [ ] 347 Top K Frequent Elements
- [ ] 238 Product of Array Except Self
- [ ] 128 Longest Consecutive Sequence
- [ ] 271 Encode and Decode Strings (P)
- [ ] JS: closures + `debounce`
- [ ] Re-solve the ones you struggled with

### Week 2: Two Pointers, Stack, Binary Search

- [ ] 125 Valid Palindrome
- [ ] 15 3Sum
- [ ] 11 Container With Most Water
- [ ] 20 Valid Parentheses
- [ ] 153 Find Minimum in Rotated Sorted Array
- [ ] 33 Search in Rotated Sorted Array
- [ ] JS: `throttle`
- [ ] Re-solve Week 1 problems

(You have already solved 704 Binary Search and 35 Search Insert Position, which are not on the Blind 75 list but are good groundwork for this week.)

### Week 3: Sliding Window

- [ ] 121 Best Time to Buy and Sell Stock
- [ ] 3 Longest Substring Without Repeating Characters
- [ ] 424 Longest Repeating Character Replacement
- [ ] 76 Minimum Window Substring (hard, give it extra time)
- [ ] JS: `flatten`
- [ ] Re-solve Week 2 problems

### Week 4: Linked List

- [ ] 206 Reverse Linked List
- [ ] 21 Merge Two Sorted Lists
- [ ] 141 Linked List Cycle
- [ ] 19 Remove Nth Node From End of List
- [ ] 143 Reorder List
- [ ] 23 Merge k Sorted Lists (hard)
- [ ] JS: `deepClone`
- [ ] Re-solve Week 3 problems

### Week 5: Trees, part 1

- [ ] 226 Invert Binary Tree
- [ ] 104 Maximum Depth of Binary Tree
- [ ] 100 Same Tree
- [ ] 572 Subtree of Another Tree
- [ ] 235 Lowest Common Ancestor of a BST
- [ ] 102 Binary Tree Level Order Traversal
- [ ] JS: `Promise.all`
- [ ] Re-solve Week 4 problems

### Week 6: Trees, part 2

- [ ] 98 Validate Binary Search Tree
- [ ] 230 Kth Smallest Element in a BST
- [ ] 105 Construct Binary Tree from Preorder and Inorder Traversal
- [ ] 124 Binary Tree Maximum Path Sum (hard)
- [ ] 297 Serialize and Deserialize Binary Tree (hard)
- [ ] JS: `curry`
- [ ] Re-solve Week 5 problems

### Week 7: Tries, Backtracking, Heap

- [ ] 208 Implement Trie
- [ ] 211 Design Add and Search Words Data Structure
- [ ] 212 Word Search II (hard)
- [ ] 39 Combination Sum
- [ ] 79 Word Search
- [ ] 295 Find Median from Data Stream (hard)
- [ ] JS: `memoize`
- [ ] Re-solve Week 6 problems

### Week 8: Graphs

- [ ] 200 Number of Islands
- [ ] 133 Clone Graph
- [ ] 417 Pacific Atlantic Water Flow
- [ ] 207 Course Schedule
- [ ] 323 Number of Connected Components in an Undirected Graph (P)
- [ ] 261 Graph Valid Tree (P)
- [ ] 269 Alien Dictionary (P, hard)
- [ ] JS: `EventEmitter`
- [ ] Re-solve Week 7 problems

### Week 9: 1-D Dynamic Programming, part 1

- [ ] 70 Climbing Stairs
- [ ] 198 House Robber
- [ ] 213 House Robber II
- [ ] 322 Coin Change
- [ ] 139 Word Break
- [ ] JS: `bind`, `call`, `apply` polyfills
- [ ] Re-solve Week 8 problems

### Week 10: 1-D Dynamic Programming, part 2

- [ ] 5 Longest Palindromic Substring
- [ ] 647 Palindromic Substrings
- [ ] 91 Decode Ways
- [ ] 152 Maximum Product Subarray
- [ ] 300 Longest Increasing Subsequence
- [ ] JS: `map`, `filter`, `reduce` polyfills
- [ ] Re-solve Week 9 problems

### Week 11: 2-D DP, Greedy, Intervals

- [ ] 62 Unique Paths
- [ ] 1143 Longest Common Subsequence
- [ ] 53 Maximum Subarray
- [ ] 55 Jump Game
- [ ] 56 Merge Intervals
- [ ] 57 Insert Interval
- [ ] 435 Non-overlapping Intervals
- [ ] 252 Meeting Rooms (P)
- [ ] 253 Meeting Rooms II (P)
- [ ] JS: `Promise.race` and `Promise.allSettled`
- [ ] Re-solve Week 10 problems

### Week 12: Bit Manipulation, Math & Geometry

- [ ] 191 Number of 1 Bits
- [ ] 338 Counting Bits
- [ ] 190 Reverse Bits
- [ ] 268 Missing Number
- [ ] 371 Sum of Two Integers
- [ ] 48 Rotate Image
- [ ] 54 Spiral Matrix
- [ ] 73 Set Matrix Zeroes
- [ ] Re-solve Week 11 problems

### Week 13: Revision and mocks

- [ ] Re-solve every problem marked hard or that took you more than one attempt
- [ ] 3-4 timed mock sessions (2 problems in 45 minutes)
- [ ] 2 machine-coding practice rounds (build a UI component in 45-60 minutes)
- [ ] Clean up your `learning-in-public` notes and README

## Checkpoints

|After|You have covered|
|---|---|
|Week 4|The patterns most junior frontend interviews use (hashing, two pointers, sliding window, binary search, linked list)|
|Week 6|Trees as well, which covers most easy and medium DSA rounds|
|Week 13|The full Blind 75|

If an interview is coming up soon, do Weeks 1-4 first and keep the JS utilities going.

## Progress log

| Week | Done | Hardest problem | Notes |
| ---- | ---- | --------------- | ----- |
| 1    |      |                 |       |
| 2    |      |                 |       |
| 3    |      |                 |       |
| 4    |      |                 |       |
| 5    |      |                 |       |
| 6    |      |                 |       |
| 7    |      |                 |       |
| 8    |      |                 |       |
| 9    |      |                 |       |
| 10   |      |                 |       |
| 11   |      |                 |       |
| 12   |      |                 |       |
| 13   |      |                 |       |