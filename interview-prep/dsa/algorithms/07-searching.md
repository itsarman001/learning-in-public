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
## Binary Search

### Problem
Given an array of `n` elements and a target element `t`, find the index of `t` in the array. Return -1 if the target element is not found

### Example

```js
const arr = [-5, 2, 4, 6, 10]

// t = 10 Should return 4
// t = 6 Should return 3
// t = 20 Should return -1
```

### Solution

```js
function binarySearch(array =[], target) {
    let leftIndex = 0;
    let rightIndex = array.length -1;

    while (leftIndex <= rightIndex) {
        let middleIndex = Math.floor((leftIndex + rightIndex) / 2);
        if(target === array[middleIndex]) return middleIndex;
        if(target > array[middleIndex]) {
            leftIndex = middleIndex + 1;
        } else  {
            rightIndex = middleIndex -1
        }
    }
    return -1
}
```

Time Complexity: $O(\log n)$ 

### Binary search pseudo-code

1. If the array is empty, return -1 as the element cannot be found.
2. if the array has elements, find the middle element in the array. If target is equal to the middle element, return middle element index
3. if target is less than the middle element, binary search left half of the array
4. If target is greater than middle element, binary search right half of the array

---

## Recursive Binary Search

### Problem
Given an array of `n` elements and a target element `t`, find the index of `t` in the array. Return -1 if the target element is not found

### Example

```js
const arr = [-5, 2, 4, 6, 10]

// t = 10 Should return 4
// t = 6 Should return 3
// t = 20 Should return -1
```

### Solution

```js
function recursiveBinarySearch(array, target) {
  return search(array, target, 0, array.length - 1);
}

function search(array, target, leftIndex, rightIndex) {
  if (leftIndex > rightIndex) return -1;
  
  const middleIndex = Math.floor((leftIndex + rightIndex) / 2);
  
  if (array[middleIndex] === target) return middleIndex;
  
  if (target < array[middleIndex]) {
    return search(array, target, leftIndex, middleIndex - 1);
  }
  
  return search(array, target, middleIndex + 1, rightIndex);
}
```

Time Complexity: $O(\log n)$ 