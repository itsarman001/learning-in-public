# 0035 - Search Insert Position

- **LeetCode:** [35. Search Insert Position](https://leetcode.com/problems/search-insert-position/)
- **Difficulty:** Easy
- **Topic:** Binary Search
- **Status:** Solved
- **Date:** 2026-11-17

---

## Problem Understanding

Given a sorted array of distinct integers `nums` and an integer `target`, return the index if the target is found.

If the target is not found, return the index where it would be inserted to maintain the array's ascending order.

The solution must have an $O(\log n)$ runtime complexity.

### Examples

**Example 1**

```js
Input: nums = [1, 3, 5, 6], target = 5
Output: 2
```

Explanation: The target `5` exists at index `2`.

**Example 2**

```js
Input: nums = [1, 3, 5, 6], target = 2
Output: 1
```

Explanation: The target `2` should be inserted at index `1`.

**Example 3**

```js
Input: nums = [1, 3, 5, 6], target = 7
Output: 4
```

Explanation: The target `7` should be inserted at the end of the array.

---

## Approach

### Binary Search (Iterative)

Since the array is sorted in ascending order, Binary Search allows us to find the target or determine its correct insertion position efficiently.

Instead of checking every element, we repeatedly divide the search space into two halves.

### Algorithm

1. Initialize two pointers:
   - `leftIndex` at the beginning of the array (`0`).
   - `rightIndex` at the end of the array (`nums.length - 1`).
2. Calculate the middle index using `Math.floor((leftIndex + rightIndex) / 2)`.
3. Compare the middle element with the target:
   - If equal, return the middle index.
   - If the target is greater, move `leftIndex` to `middleIndex + 1`.
   - If the target is smaller, move `rightIndex` to `middleIndex - 1`.
4. Repeat until the target is found or the search range becomes empty.
5. If the target is not found, return `leftIndex`.

### Why return `leftIndex`?

When the loop terminates, the search boundaries cross:

```js
leftIndex > rightIndex
```

At this point, `leftIndex` represents the first position where the target can be inserted without breaking the ascending order.

This works because all elements before `leftIndex` are smaller than the target, while all elements from `leftIndex` onward are greater than the target.

---

## Implementation

```js
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number}
 */
var searchInsert = function(nums, target) {
    let leftIndex = 0;
    let rightIndex = nums.length - 1;

    while (leftIndex <= rightIndex) {
        let middleIndex = Math.floor((leftIndex + rightIndex) / 2);

        if (target === nums[middleIndex]) return middleIndex;

        if (target > nums[middleIndex]) {
            leftIndex = middleIndex + 1;
        } else {
            rightIndex = middleIndex - 1;
        }
    }

    return leftIndex;
};
```

---

## Complexity Analysis

- **Time Complexity:** $O(\log n)$

  The search space is reduced by approximately half in every iteration.

- **Space Complexity:** $O(1)$

  The iterative implementation uses only a constant amount of additional space.

---

## Key Takeaways

- Binary Search can be used to find both an existing element and its correct insertion position.
- A sorted array allows us to eliminate half of the search space at each iteration.
- When the search range becomes empty, `leftIndex` points to the correct insertion position.
- Unlike LeetCode 704, which returns `-1` when the target is absent, this problem returns `leftIndex`.
- The same Binary Search implementation can solve multiple problems by changing what happens after the search terminates.

### Pattern to Remember

```js
while (left <= right) {
    const mid = Math.floor((left + right) / 2);

    if (nums[mid] === target) return mid;

    if (nums[mid] < target) {
        left = mid + 1;
    } else {
        right = mid - 1;
    }
}

return left;
```

**Core idea:** When Binary Search terminates, the left pointer identifies the insertion position of the target.