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

|Case|Time Complexity|
|---|---|
|Best Case (Already Sorted)|O(n)|
|Average Case|O(n²)|
|Worst Case (Reverse Sorted)|O(n²)|

**Space Complexity:** O(1)

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