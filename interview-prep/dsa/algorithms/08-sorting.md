# Sorting

## Bubble Sort

Bubble Sort is a simple sorting algorithm that compares adjacent elements in an array and swaps their positions if they are not in the intended order.

It repeatedly steps through the array, comparing two adjacent elements at a time.

Once you step through the whole array without making any swaps, the array is sorted.

### Problem

Given an array of integers, sort the array in ascending order.

### Example

```js
const arr = [-6, 20, 8, -2, 4];

bubbleSort(arr);
// Returns [-6, -2, 4, 8, 20]
```

### How It Works

Initial array:

`[-6, 20, 8, -2, 4]`

#### First Pass

Compare adjacent elements and swap if the left element is greater than the right element.

text

```
[-6 20  8 -2  4]
[-6  8 20 -2  4]  Swap since 20 > 8
[-6  8 -2 20  4]  Swap since 20 > -2
[-6  8 -2  4 20]  Swap since 20 > 4
```

End of array. Elements swapped? Yes. Repeat the comparison.

#### Second Pass

text

```
[-6  8 -2  4 20]
[-6 -2  8  4 20]  Swap since 8 > -2
[-6 -2  4  8 20]  Swap since 8 > 4
```

End of array. Elements swapped? Yes. Repeat the comparison.

#### Third Pass

text

```
[-6 -2  4  8 20]
[-6 -2  4  8 20]
```

End of array. Elements swapped? No. The array is sorted.

### Solution

```js
function bubbleSort(arr) {
  let swapped;

  do {
    swapped = false;

    for (let i = 0; i < arr.length - 1; i++) {
      if (arr[i] > arr[i + 1]) {
        [arr[i], arr[i + 1]] = [arr[i + 1], arr[i]];

        swapped = true;
      }
    }
  } while (swapped);

  return arr;
}

const arr = [-6, 20, 8, -2, 4];

bubbleSort(arr); // Output: [-6, -2, 4, 8, 20]
```

### Time and Space Complexity

| Case                        | Time Complexity |
| --------------------------- | --------------- |
| Best Case (Already Sorted)  | O(n)            |
| Average Case                | O(n²)           |
| Worst Case (Reverse Sorted) | O(n²)           |

**Space Complexity:** O(1)

Bubble Sort sorts the array in place without requiring additional space proportional to the input size.

### Note

Bubble Sort is one of the least efficient sorting algorithms for large datasets.

- It is useful for understanding sorting algorithms and practicing for coding interviews.
- It is rarely used in professional applications because more efficient algorithms are available.
- Algorithms such as Quick Sort and Merge Sort are generally preferred for larger datasets.
- In JavaScript, built-in `Array.prototype.sort()` is commonly used for sorting.

**Remember:** Bubble Sort repeatedly compares adjacent elements and swaps them until a complete pass is made without any swaps.
