# 0001 - Two Sum

- **LeetCode:** [1. Two Sum](https://leetcode.com/problems/two-sum/)
- **Difficulty:** Easy
- **Topic:** Array, Hash Map
- **Status:** Solved
- **Date:** 2026-11-17

---

## Problem Understanding

Given an array of integers `nums` and an integer `target`, return the indices of the two numbers whose sum equals the target.

**Constraints:**

- Each input has exactly one valid solution.
- You cannot use the same element twice.
- You can return the answer in any order.

### Examples

**Example 1**

```js
Input: nums = [2, 7, 11, 15], target = 9
Output: [0, 1]
```

Explanation: `nums[0] + nums[1] === 9`.

**Example 2**

```js
Input: nums = [3, 2, 4], target = 6
Output: [1, 2]
```

Explanation: `nums[1] + nums[2] === 6`.

**Example 3**

```js
Input: nums = [3, 3], target = 6
Output: [0, 1]
```

Explanation: Both elements have the value `3`, but they are at different indices.

---

## Approach

### 1. Brute Force (Nested Loops)

The straightforward approach is to check every possible pair of elements.

**Algorithm:**

1. Iterate through the array using an outer loop.
2. Use an inner loop starting from the next element (`i + 1`).
3. Check whether the sum of both elements equals the target.
4. If a matching pair is found, return their indices.
5. If no pair is found, return an empty array.

This approach avoids checking the same element twice or comparing the same pair in reverse order.

### 2. Optimized Approach (Hash Map)

The brute-force approach repeatedly checks pairs of elements, resulting in `O(n²)` time complexity.

We can optimize this using a Hash Map (`Map` in JavaScript) to perform efficient lookups.

**Key Observation:**

Instead of searching for a pair, calculate the number needed to reach the target.

\[
complement = target - currentNumber
\]

For each element, check whether its complement already exists in the Map.

Store previously encountered numbers as keys and their indices as values.

**Algorithm:**

1. Create an empty Map.
2. Iterate through the array.
3. Calculate the complement using `target - nums[i]`.
4. Check whether the complement exists in the Map.
5. If it exists, return the stored index and the current index.
6. Otherwise, store the current number and its index in the Map.
7. Return an empty array if no matching pair is found.

**Important:** Check for the complement before inserting the current number. This ensures that the same element isn't used twice.

---

## Implementation

### 1. Brute Force

```js
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
var twoSum = function(nums, target) {
    for (let i = 0; i < nums.length; i++) {
        for (let j = i + 1; j < nums.length; j++) {
            if (nums[i] + nums[j] === target) {
                return [i, j];
            }
        }
    }

    return [];
};
```

### 2. Optimized (Hash Map)

```js
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
var twoSum = function(nums, target) {
    const map = new Map();

    for (let i = 0; i < nums.length; i++) {
        const complement = target - nums[i];

        if (map.has(complement)) {
            return [map.get(complement), i];
        }

        map.set(nums[i], i);
    }

    return [];
};
```

---

## Complexity Analysis

### Brute Force

- **Time Complexity:** `O(n²)`

  Two nested loops potentially examine every unique pair of elements.

- **Space Complexity:** `O(1)`

  Only a constant amount of additional space is used.

### Optimized (Hash Map)

- **Time Complexity:** `O(n)`

  The array is traversed once, with average `O(1)` lookup and insertion operations in the Map.

- **Space Complexity:** `O(n)`

  The Map can store up to `n` elements.

### Comparison

| Approach    | Time Complexity | Space Complexity |
| ----------- | --------------- | ---------------- |
| Brute Force | `O(n²)`         | `O(1)`           |
| Hash Map    | `O(n)`          | `O(n)`           |

---

## Key Takeaways

- Nested loops can solve pair-searching problems but may result in quadratic time complexity.
- A Hash Map allows efficient lookups using key-value pairs.
- Mathematical observations can help eliminate unnecessary searching.
- The complement of a number is calculated using `target - currentNumber`.
- Checking for the complement before inserting the current element prevents using the same element twice.
- Optimizing an algorithm often involves trading additional memory for reduced execution time.

### Pattern to Remember

**Hash Map + Complement**

When looking for two values that satisfy a target sum:

1. Calculate the required complement.
2. Check whether it has already been encountered.
3. If found, return the stored index and current index.
4. Otherwise, remember the current value and its index.

**Core idea:** Remember previously encountered values so you don't have to search for them repeatedly.