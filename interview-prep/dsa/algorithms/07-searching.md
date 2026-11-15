# Search

## Linear Search

### Problem
Given an array of `n` elements and a target element `t`, find the index of `t` in the array. Return -1 if the target element is not found

### Example

```js
const arr = [-5, 2, 10, 4, 6]

// t = 10 Should return 2
// t = 6 Should return 6
// t = 20 Should return -1
```

### Solution

#### Using array method

```js
function findPreviousElement(arr = [], target) {
  const index = arr.findIndex((num) => num === target);
  if (index <= 0) return -1;
  return arr[index - 1];
}

findPreviousElement(arr, 2);
findPreviousElement(arr, 6);
findPreviousElement(arr, 20);
```

| Time complexity | Space complexity |
| --------------- | ---------------- |
| $O(n)$          | $O(1)$           |

#### Using For Loop

```js
function linearSearch(arr = [], t) {
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] === t) return i > 0 ? arr[i - 1] : -1;
  }
  return -1;
}

linearSearch(arr, 2);
linearSearch(arr, 6);
linearSearch(arr, 20);
```

| Time complexity | Space complexity |
| --------------- | ---------------- |
| $O(n)$          | $O(1)$           |

----
