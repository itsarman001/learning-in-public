# 242. Valid Anagram

- **Platform:** [LeetCode](https://leetcode.com/problems/valid-anagram/)
    
- **Difficulty:** Easy
    
- **Topic:** Strings, Hash Map
    
- **Pattern:** Frequency Counting
    
- **Status:** Solved
    
- **Language:** JavaScript
    

---

## Problem

Given two strings `s` and `t`, return `true` if `t` is an anagram of `s`, and `false` otherwise.

An anagram is a word or phrase formed by rearranging the letters of another word or phrase, using all the original letters exactly once.

## Examples

**Example 1:**

```
Input: s = "anagram", t = "nagaram"
Output: true
```

**Example 2:**

```
Input: s = "rat", t = "car"
Output: false
```

## Approach: Hash Map (Frequency Counting)

Use a `Map` to count the frequency of each character in the first string, then decrement those frequencies while iterating through the second string.

1. Check whether both strings have the same length. If not, return `false`.
    
2. Initialize an empty `Map` to store character frequencies.
    
3. Iterate through string `s` and count the occurrences of each character.
    
4. Iterate through string `t`:
    
    - Check whether the character exists in the map and its frequency is greater than zero.
        
    - If not, return `false`.
        
    - Otherwise, decrement its frequency.
        
5. If the iteration completes successfully, return `true`.
    

## Solution

```js
/**
 * @param {string} s
 * @param {string} t
 * @return {boolean}
 */
var isAnagram = function(s, t) {
    if (s.length !== t.length) return false;

    let map = new Map();

    for (let char of s) {
        map.set(char, (map.get(char) || 0) + 1);
    }

    for (let char of t) {
        if (!map.has(char) || map.get(char) === 0) return false;

        map.set(char, map.get(char) - 1);
    }

    return true;
};
```

## Complexity Analysis

Let `n` be the length of the strings.

- **Time Complexity:** `O(n)` average. Both strings are traversed once, and map operations take constant time on average.
    
- **Space Complexity:** `O(k)`, where `k` is the number of distinct characters in the strings. In the worst case, this is `O(n)`.
    

## Key Takeaways

- Anagrams contain exactly the same characters with identical frequencies.
    
- A Hash Map can efficiently track character frequencies.
    
- Counting frequencies avoids sorting the strings.
    
- Decrementing frequencies while processing the second string helps identify missing or excess character occurrences.
    
- Checking string lengths first eliminates unnecessary processing.
    

## References

- [LeetCode - 242. Valid Anagram](https://leetcode.com/problems/valid-anagram/)
    
- [MDN - Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)