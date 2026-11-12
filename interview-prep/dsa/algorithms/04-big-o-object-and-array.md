# Object – Big-O

An **object** is a collection of key-value pairs. Each key is unique and is used to access its corresponding value.

```js
const person = {
    name: "John Doe",
    email: "johndoe@email.com"
};
```

## Time Complexity

| Operation | Time Complexity |
|---|---|
| Insert, Remove, Access | $O(1)$ |
| Search | $O(n)$ |
| `Object.keys()`, `Object.values()`, `Object.entries()` | $O(n)$ |

**Notes:**

- Insert, remove, and access operations are generally $O(1)$ when the key is known.
- Searching for a value requires iterating through the object properties, making it $O(n)$.
- `Object.keys()`, `Object.values()`, and `Object.entries()` iterate over the object's properties, making them $O(n)$.

# Array – Big-O

An **array** is an ordered collection of values. Each element is stored at an index, starting from `0`.

```js
const odd = [1, 3, 5, 7, 9];
```

## JavaScript Array Operations – Time Complexity

| Category | Operations / Methods | Time Complexity |
|---|---|---|
| Access | Access by index | $O(1)$ |
| Add / Remove at End | `push()`, `pop()` | $O(1)^*$ |
| Add / Remove at Beginning | `unshift()`, `shift()` | $O(n)$ |
| Insertion / Deletion | `splice()` | $O(n)$ |
| Searching | `find()`, `findIndex()`, `includes()`, `indexOf()` | $O(n)$ |
| Iteration / Transformation | `forEach()`, `map()`, `filter()`, `reduce()` | $O(n)$ |
| Copying | `slice()` | $O(n)$ |
| Merging | `concat()` | $O(n + m)$ |

**Notes:**

- **Access — $O(1)$:** Elements can be accessed directly using their index.
- **Search — $O(n)$:** In the worst case, every element must be checked.
- **Beginning operations — $O(n)$:** Existing elements need to be shifted to accommodate insertion or removal.
- **Iteration methods — $O(n)$:** Each element is processed once.

### Important: Method Chaining and Nested Iterations

- **Sequential method chaining:** Calling `map()` and `filter()` one after another results in $O(n) + O(n) = O(n)$ time complexity.
- **Nested iterations:** Using `map()` or `filter()` inside another iteration can result in $O(n^2)$ time complexity.

Example of sequential method chaining:

```js
const result = arr
    .map(item => item * 2)
    .filter(item => item > 5);
```

Each method processes the array once, so the overall time complexity is $O(n)$.

Example of nested iterations:

```js
const result = arr.map(item =>
    arr.filter(value => value > item)
);
```

For every iteration of `map()`, `filter()` traverses the entire array.

If the array contains `n` elements:

$$O(n \times n) = O(n^2)$$

Therefore, the overall time complexity is **Quadratic — $O(n^2)$**.

### Additional Notes

- **Amortized complexity:** `push()` and insertion at the end are amortized $O(1)$. Occasionally, an array may need to resize, making an individual insertion $O(n)$.
- **Array merging:** For `concat()`, `n` and `m` represent the lengths of the two arrays. Both arrays must be copied into the resulting array, giving $O(n + m)$ complexity.