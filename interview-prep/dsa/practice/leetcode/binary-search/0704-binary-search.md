# 0704 - Binary Search

- **LeetCode:** [704. Binary Search](https://leetcode.com/problems/binary-search/)
- **Difficulty:** Easy
- **Topic:** Binary Search
- **Status:** Solved
- **Date:** 2026-11-17

---

## Problem

Given an array of integers `nums` sorted in ascending order and an integer `target`, write a function to search for `target` in `nums`.

If `target` exists, return its index. Otherwise, return `-1`.

You must write an algorithm with $O(\log n)$ runtime complexity.

### Examples

**Example 1**

```js
Input: nums = [-1, 0, 3, 5, 9, 12], target = 9
Output: 4
```

Explanation: `9` exists in `nums` and its index is `4`.

**Example 2**

```js
Input: nums = [-1, 0, 3, 5, 9, 12], target = 2
Output: -1
```

Explanation: `2` does not exist in `nums`.

**Example 3**

```js
Input: nums = [5], target = 5
Output: 0
```

---

## Approach

### Binary Search (Iterative)

Binary Search is an efficient searching algorithm that works on a **sorted array**.

Instead of checking every element sequentially, it repeatedly divides the search space into two halves.

At each iteration:

1. Initialize two pointers:
   - `left`: Start of the search range.
   - `right`: End of the search range.
2. Calculate the middle index using `Math.floor((left + right) / 2)`.
3. Compare the middle element with the target:
   - If equal, return the middle index.
   - If the target is greater, search the right half.
   - If the target is smaller, search the left half.
4. Repeat until the target is found or the search range becomes empty.
5. Return `-1` if the target doesn't exist.

### Algorithm

```text
left = 0
right = nums.length - 1

while left <= right:
    mid = floor((left + right) / 2)

    if nums[mid] == target:
        return mid

    if nums[mid] < target:
        left = mid + 1
    else:
        right = mid - 1

return -1
```

---

## Solution

```js
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number}
 */
var search = function(nums, target) {
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

    return -1;
};
```

---

## Complexity Analysis

- **Time Complexity:** $O(\log n)$

  The search space is reduced by approximately half in every iteration.

- **Space Complexity:** $O(1)$

  The iterative implementation uses a constant amount of additional space.

---

## Edge Cases

| Case | Input | Expected Output |
|---|---|---|
| Single element, target exists | `[5]`, `5` | `0` |
| Single element, target missing | `[5]`, `3` | `-1` |
| Target at beginning | `[1, 3, 5]`, `1` | `0` |
| Target at end | `[1, 3, 5]`, `5` | `2` |
| Target doesn't exist | `[1, 3, 5]`, `4` | `-1` |

---

## Key Takeaways

- Binary Search requires a sorted search space.
- Each iteration eliminates approximately half of the remaining search space.
- `left <= right` ensures the final remaining element is checked.
- `mid + 1` and `mid - 1` exclude the already-checked middle element.
- Iterative Binary Search achieves $O(\log n)$ time and $O(1)$ space complexity.

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
```

**Core idea:** Eliminate half of the search space at every iteration instead of checking elements one by one.