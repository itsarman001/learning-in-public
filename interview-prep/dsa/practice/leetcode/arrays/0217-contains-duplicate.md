# 217. Contains Duplicate

- **Platform:** [LeetCode](https://leetcode.com/problems/contains-duplicate/)
    
- **Difficulty:** Easy
    
- **Topic:** Arrays, Hash Set
    
- **Status:** Solved
    
- **Language:** JavaScript
    

---

## Problem

Given an integer array `nums`, return `true` if any value appears at least twice in the array, and return `false` if every element is distinct.

## Examples

**Example 1:**

```
Input: nums = [1, 2, 3, 1]
Output: true
```

**Example 2:**

```
Input: nums = [1, 2, 3, 4]
Output: false
```

**Example 3:**

```
Input: nums = [1, 1, 1, 3, 3, 4, 3, 2, 4, 2]
Output: true
```

## Approach: Hash Set

Use a `Set` to keep track of elements that have already been encountered.

1. Initialize an empty `Set`.
    
2. Iterate through the input array.
    
3. Check whether the current element already exists in the set.
    
4. If it exists, return `true` because a duplicate has been found.
    
5. Otherwise, add the current element to the set.
    
6. If the iteration completes without finding a duplicate, return `false`.
    

## Solution

```js
/**
 * @param {number[]} nums
 * @return {boolean}
 */
var containsDuplicate = function(nums) {
    const seen = new Set();

    for (const num of nums) {
        if (seen.has(num)) return true;
        seen.add(num);
    }

    return false;
};
```

## Complexity Analysis

Let `n` be the length of the input array.

- **Time Complexity:** `O(n)` average. Each element is visited once, and `Set.has()` and `Set.add()` take constant time on average.
    
- **Space Complexity:** `O(n)` in the worst case, when all elements are distinct and stored in the set.
    

## Key Takeaways

- A `Set` stores unique values and provides efficient membership checks.
    
- Use a `Set` when you only need to track whether an element has appeared before.
    
- A `Map` is unnecessary when no additional information needs to be associated with each element.
    
- Returning early avoids unnecessary iterations once a duplicate is found.
    

## Initial Approach

Initially, I implemented the solution using a `Map` to track previously encountered numbers.

After reviewing the implementation, I realized that storing each number's index was unnecessary because I only needed to check whether a number had appeared before.

I replaced `Map` with `Set`, resulting in a simpler and more appropriate implementation.

## References

- [LeetCode - 217. Contains Duplicate](https://leetcode.com/problems/contains-duplicate/)
    
- [MDN - Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set)