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

```
[-6 20  8 -2  4]
[-6  8 20 -2  4]  Swap since 20 > 8
[-6  8 -2 20  4]  Swap since 20 > -2
[-6  8 -2  4 20]  Swap since 20 > 4
```

End of array. Elements swapped? Yes. Repeat the comparison.

#### Second Pass

```
[-6  8 -2  4 20]
[-6 -2  8  4 20]  Swap since 8 > -2
[-6 -2  4  8 20]  Swap since 8 > 4
```

End of array. Elements swapped? Yes. Repeat the comparison.

#### Third Pass

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
| Best Case (Already Sorted)  | $O(n)$          |
| Average Case                | $O(n²)$         |
| Worst Case (Reverse Sorted) | $O(n²)$         |

**Space Complexity:** $O(1)$

Bubble Sort sorts the array in place without requiring additional space proportional to the input size.

### Note

Bubble Sort is one of the least efficient sorting algorithms for large datasets.

- It is useful for understanding sorting algorithms and practicing for coding interviews.
- It is rarely used in professional applications because more efficient algorithms are available.
- Algorithms such as Quick Sort and Merge Sort are generally preferred for larger datasets.
- In JavaScript, built-in `Array.prototype.sort()` is commonly used for sorting.

**Remember:** Bubble Sort repeatedly compares adjacent elements and swaps them until a complete pass is made without any swaps.

---

## Insertion Sort

Insertion Sort is a simple sorting algorithm that builds a sorted array one element at a time.

It works by dividing the array into a sorted and an unsorted part. It takes one element from the unsorted part and inserts it into its correct position in the sorted part.

It repeatedly shifts larger elements to the right to make room for the selected element.

### Problem

Given an array of integers, sort the array in ascending order.

### Example

```
const arr = [-6, 20, 8, -2, 4];

insertionSort(arr);
// Returns [-6, -2, 4, 8, 20]
```

### How It Works

1. Virtually divide the array into a sorted and an unsorted part.
2. Assume the first element is already sorted, and the remaining elements are unsorted.
3. Select an element from the unsorted part.
4. Compare the selected element with the elements in the sorted part, starting from right to left.
5. Shift all elements that are greater than the selected element one position to the right.
6. Insert the selected element into its correct position.
7. Repeat until all elements have been inserted into the sorted part.

#### Initial Array

`[-6, 20, 8, -2, 4]`

The first element `-6` is considered sorted.

#### First Pass

Select `20` and compare it with `-6`.

```css
[-6  20   8  -2   4]
```

`-6 > 20?` No. Keep `20` in its current position.

#### Second Pass

Select `8` and compare it with elements in the sorted part.

```css
[-6  20   8  -2   4]
[-6  20  20  -2   4]  Shift since 20 > 8
[-6   8  20  -2   4]  Insert 8
```

#### Third Pass

Select `-2` and compare it with elements in the sorted part.

```css
[-6   8  20  -2   4]
[-6   8  20  20   4]  Shift since 20 > -2
[-6   8   8  20   4]  Shift since 8 > -2
[-6  -2   8  20   4]  Insert -2
```

#### Fourth Pass

Select `4` and compare it with elements in the sorted part.

```css
[-6  -2   8  20   4]
[-6  -2   8  20  20]  Shift since 20 > 4
[-6  -2   8   8  20]  Shift since 8 > 4
[-6  -2   4   8  20]  Insert 4
```

End of array. All elements are inserted into their correct positions. The array is sorted.

![[insertion_sort_example.png]]

### Solution

```js
function insertionSort(arr) {
  for (let i = 1; i < arr.length; i++) {
    let elementToInsert = arr[i];
    let previousElement = i - 1;

    while (previousElement >= 0 && arr[previousElement] > elementToInsert) {
      arr[previousElement + 1] = arr[previousElement];
      previousElement = previousElement - 1;
    }

    arr[previousElement + 1] = elementToInsert;
  }

  return arr;
}

const arr = [-6, 20, 8, -2, 4];

insertionSort(arr); // Output: [-6, -2, 4, 8, 20]
```

### Time and Space Complexity

| Case                        | Time Complexity |
| --------------------------- | --------------- |
| Best Case (Already Sorted)  | $O(n)$          |
| Average Case                | $O(n²)$         |
| Worst Case (Reverse Sorted) | $O(n²)$         |

**Space Complexity:** $O(1)$

Insertion Sort sorts the array in place without requiring additional space proportional to the input size.

### Note

Insertion Sort is more efficient for small or nearly sorted datasets but becomes inefficient for large datasets.

- It performs well when the array is already sorted or nearly sorted.
- It is useful for understanding sorting algorithms and practicing for coding interviews.
- It is an in-place sorting algorithm, meaning it does not require additional space proportional to the input size.
- It is a stable sorting algorithm because equal elements maintain their relative order.
- Algorithms such as Merge Sort and Quick Sort are generally preferred for larger datasets.
- In JavaScript, built-in `Array.prototype.sort()` is commonly used for sorting.

**Remember:** Insertion Sort builds a sorted array one element at a time by shifting larger elements to the right and inserting the selected element into its correct position.

---

## Quick Sort

Quick Sort is a divide-and-conquer sorting algorithm that works by selecting a pivot element and partitioning the array around it.

It divides the array into two parts:

- **Left:** Elements smaller than or equal to the pivot.
- **Right:** Elements greater than the pivot.

It recursively applies the same process to the left and right arrays until each array contains fewer than two elements.

Finally, it combines the sorted left array, pivot and sorted right array to produce the sorted array.

### Problem

Given an array of integers, sort the array in ascending order.

### Example

```js
const arr = [-6, 20, 8, -2, 4];

quickSort(arr);
// Returns [-6, -2, 4, 8, 20]
```

### How It Works

1. Select a pivot element from the array.

   There are several ways to choose a pivot:
   - Pick the first element as the pivot.
   - Pick the last element as the pivot (our approach).
   - Pick a random element as the pivot.
   - Pick the median element as the pivot.

2. Partition the array into two parts:
   - Place elements smaller than or equal to the pivot into the `left` array.
   - Place elements greater than the pivot into the `right` array.

3. Recursively apply Quick Sort to the individual `left` and `right` arrays.

4. Continue until each array contains fewer than two elements. An array with zero or one element is already sorted by definition.

5. Combine the sorted `left` array, pivot and sorted `right` array.

6. Repeat the process until one completely sorted array remains.

#### Initial Array

`[-6, 20, 8, -2, 4]`

#### First Partition

Select the last element `4` as the pivot.

```js
[-6, 20, 8, -2, 4];

Pivot = 4;

Left = [-6, -2];
Right = [20, 8];
```

Combine:

```js
[-6, -2] + [4] + [20, 8];
```

#### Second Partition

Apply Quick Sort to the left and right arrays.

For the left array:

```js
[-6, -2];

Pivot = -2;

Left = [-6];
Right = [];
```

For the right array:

```js
[20, 8];

Pivot = 8;

Left = [8];
Right = [20];
```

#### Final Result

Combine the sorted left array, pivot and sorted right array.

```js
[-6] + [-2] + [4] + [8] + [20];
```

The resulting array is:

```js
[-6, -2, 4, 8, 20];
```

![[quick_sort_example.png]]

### Solution

```js
function quickSort(arr) {
  if (arr.length < 2) return arr;

  let pivot = arr[arr.length - 1];
  let left = [];
  let right = [];

  for (let i = 0; i < arr.length - 1; i++) {
    if (arr[i] > pivot) {
      right.push(arr[i]);
    } else {
      left.push(arr[i]);
    }
  }

  return [...quickSort(left), pivot, ...quickSort(right)];
}

const arr = [-6, 20, 8, -2, 4];

quickSort(arr); // Output: [-6, -2, 4, 8, 20]
```

### Time and Space Complexity

| Case         | Time Complexity |
| ------------ | --------------- |
| Best Case    | $O(n log n)$    |
| Average Case | $O(n log n)$    |
| Worst Case   | $O(n²)$         |

**Space Complexity:** $O(n)$ for this implementation.

Quick Sort divides the array into smaller subarrays and recursively sorts them.

- **Best Case:** $O(n log n)$. The pivot divides the array into roughly equal-sized partitions.
- **Average Case:** $O(n log n)$. The partitions are reasonably balanced on average.
- **Worst Case:** $O(n²)$. The pivot repeatedly produces highly unbalanced partitions, such as when the array is already sorted and the last element is consistently selected as the pivot.

The implementation uses additional arrays for partitioning and creates new arrays using the spread operator. Therefore, its auxiliary space usage is $O(n)$ on average, with $O(n)$ space for the recursion stack in the worst case.

### Note

Quick Sort is generally an efficient sorting algorithm for large datasets, especially when partitions are reasonably balanced.

- It follows the divide-and-conquer approach.
- Choosing a good pivot helps avoid highly unbalanced partitions.
- Randomized pivot selection can help reduce the likelihood of consistently unbalanced partitions.
- Quick Sort is generally not a stable sorting algorithm.
- In-place implementations can reduce auxiliary space usage compared to this implementation.
- Algorithms such as Merge Sort and Quick Sort are commonly used when efficient sorting is required.
- In JavaScript, built-in `Array.prototype.sort()` is commonly used for sorting.

**Remember:** Quick Sort selects a pivot, partitions the array around it, recursively sorts the partitions and combines them to produce a sorted array.

---

## Merge Sort

Merge Sort is a divide-and-conquer sorting algorithm that divides an array into smaller subarrays until each subarray contains a single element.

It then repeatedly merges the subarrays in sorted order until only one sorted array remains.

An array containing only one element is considered sorted by definition.

### Problem

Given an array of integers, sort the array in ascending order.

### Example

```js
const arr = [-6, 20, 8, -2, 4];

mergeSort(arr);
// Returns [-6, -2, 4, 8, 20]
```

### How It Works

1. Divide the array into two halves.
2. Recursively divide each half into smaller subarrays.
3. Continue dividing until each subarray contains only one element.
4. Merge two subarrays by comparing their first unprocessed elements.
5. Place the smaller element into the new sorted array.
6. Continue until one subarray is empty, then append the remaining elements from the other subarray.
7. Repeatedly merge the sorted subarrays until one sorted array remains.

#### Initial Array

`[-6, 20, 8, -2, 4]`

#### First Division

Divide the array into two halves.

```
[-6, 20]    [8, -2, 4]
```

#### Second Division

Continue dividing the subarrays.

```
[-6] [20]    [8] [-2, 4]
```

#### Third Division

Divide the remaining subarray.

```
[-6] [20]    [8] [-2] [4]
```

Every subarray contains one element and is sorted.

#### First Merge

Merge `[-6]` and `[20]`.

```
[-6, 20]
```

Merge `[-2]` and `[4]`.

```
[-2, 4]
```

#### Second Merge

Merge `[8]` and `[-2, 4]`.

Compare the elements and place them in ascending order.

```
[-2, 4, 8]
```

#### Final Merge

Merge `[-6, 20]` and `[-2, 4, 8]`.

```
[-6, -2, 4, 8, 20]
```

The array is sorted.

### Solution

```js
function mergeSort(arr) {
  if (arr.length < 2) return arr;

  const mid = Math.floor(arr.length / 2);
  const leftArr = arr.slice(0, mid);
  const rightArr = arr.slice(mid);

  return merge(mergeSort(leftArr), mergeSort(rightArr));
}

function merge(leftArr, rightArr) {
  const sortedArr = [];
  let i = 0;
  let j = 0;

  while (i < leftArr.length && j < rightArr.length) {
    if (leftArr[i] <= rightArr[j]) {
      sortedArr.push(leftArr[i]);
      i++;
    } else {
      sortedArr.push(rightArr[j]);
      j++;
    }
  }

  return [...sortedArr, ...leftArr.slice(i), ...rightArr.slice(j)];
}

const arr = [-6, 20, 8, -2, 4];

mergeSort(arr); // Output: [-6, -2, 4, 8, 20]
```

### Time and Space Complexity

| Case         | Time Complexity |
| ------------ | --------------- |
| Best Case    | $O(n log n)$    |
| Average Case | $O(n log n)$    |
| Worst Case   | $O(n log n)$    |

**Space Complexity:** $O(n)$

Merge Sort divides the array into halves recursively, resulting in O(log n) recursion depth.

During merging, it creates temporary arrays to store sorted elements, requiring $O(n)$ additional space.

### Note

Merge Sort is an efficient sorting algorithm for large datasets.

- It follows the divide-and-conquer approach.
- Its time complexity is $O(n log n)$ in all cases.
- It is a stable sorting algorithm because equal elements maintain their relative order.
- It requires additional memory for merging, unlike in-place sorting algorithms.
- It is particularly useful when predictable performance is important.
- Algorithms such as Quick Sort and Merge Sort are commonly used for efficient sorting.
- In JavaScript, built-in `Array.prototype.sort()` is commonly used for sorting.

**Remember:** Merge Sort divides the array into smaller subarrays until each contains one element, then repeatedly merges them in sorted order until one sorted array remains.
