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

## Solution

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

## Problem

Given an integer `n`, find the factorial of that integer.

In mathematics, the factorial of a non-negative integer `n`, denoted by `n!`, is the product of all positive integers less than or equal to `n`.

The factorial of zero is 1.

## Examples

- `factorial(4) = 4 × 3 × 2 × 1 = 24`
    
- `factorial(5) = 5 × 4 × 3 × 2 × 1 = 120`

## Solution

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


# Prime Number
---

## Problem

Given a natural number `n`, determine if the number is prime or not

A prime number is a natural number greater than 1 that is not a product of two smaller natural numbers

## Example

- `isPrime(5) = true (1 * 5 or  5 * 1)`
	
- `isPrime(4) = false (1 * 4 or 2 * 2 or 4 * 1)`

## Solution

> [!question]- Solution
> 
> ```js
> function isPrime(n) {
>   if (n < 2) return false;
> 
>   for (let i = 2; i <= Math.sqrt(n); i++) {
>     if (n % i === 0) return false;
>   }
> 
>   return true;
> }
> ```

> [!question]- What is the time complexity of the solution?
> **Answer:** $O(\sqrt{n})$
> 
> **Explanation:** 
> The loop runs from `2` up to $\sqrt{n}$, checking for factors in constant time on each iteration. Because factors come in pairs, checking past $\sqrt{n}$ is unnecessary.
> 
> $$T(n) = \sqrt{n}$$
> 
> Therefore, the time complexity is $O(\sqrt{n})$.

