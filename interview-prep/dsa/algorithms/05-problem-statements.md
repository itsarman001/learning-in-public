# Fibonacci Sequence
---

## Problem

Given a number `n`, find the first `n` elements of the Fibonacci sequence.

In mathematics, the Fibonacci sequence is a sequence in which each number is the sum of the two preceding numbers.

The first two numbers in the sequence are `0` and `1`.

## Examples

```js
fibonacci(2) // [0, 1]

fibonacci(3) // [0, 1, 1]

fibonacci(7) // [0, 1, 1, 2, 3, 5, 8]
```

> [!question]- Solution
> 
> 
> ```js
> function fibonacci(n) {
>   const sequence = [0, 1];
> 
>   for (let i = 2; i < n; i++) {
>     sequence[i] = sequence[i - 1] + sequence[i - 2];
>   }
> 
>   return sequence;
> }
> ```

> [!question]- What is the time complexity of the solution?
> **Answer:** $O(n)$
> 
> **Explanation:** The loop runs `n - 2` times, and each iteration performs a constant-time addition. Therefore, the time complexity is linear, $O(n)$.


# Factorial of a Number
---
Problem: Given an integer `n`, find the factorial of that integer.

In mathematics, the factorial of a non-negative integer `n`, denoted by `n!`, is the product of all positive integers less than or equal to `n`.

The factorial of zero is 1.

### Examples

- `factorial(4) = 4 × 3 × 2 × 1 = 24`
    
- `factorial(5) = 5 × 4 × 3 × 2 × 1 = 120`

> [!question]- Solution
>
> ```js
> function factorial(n = 1) {
>   let result = 1;
>
>   for (let i = n; i >= 1; i--) {
>     result *= i;
>   }
>
>   return result;
> }
> ```

> [!question]- What is the time complexity of the solution?
> **Answer:** $O(n)$
>
> **Explanation:**
> The loop runs `n` times, and each iteration performs a multiplication.
>
> $$T(n) = n$$
>
> Therefore, the time complexity is $O(n)$.

