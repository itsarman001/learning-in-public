# Fibonacci Sequence

## Problem

Given a number `n`, find the first `n` elements of the Fibonacci sequence.

In mathematics, the Fibonacci sequence is a sequence in which each number is the sum of the two preceding numbers.

The first two numbers in the sequence are `0` and `1`.

## Examples

```js
fibonacci(0) // []

fibonacci(1) // [0]

fibonacci(2) // [0, 1]

fibonacci(3) // [0, 1, 1]

fibonacci(7) // [0, 1, 1, 2, 3, 5, 8]
```

## Solution

```js
function fibonacci(n) {
  if (n <= 0) return [];
  if (n === 1) return [0];

  const sequence = [0, 1];

  for (let i = 2; i < n; i++) {
    sequence[i] = sequence[i - 1] + sequence[i - 2];
  }

  return sequence;
}
```

## Explanation

What is the time complexity of the solution?

**Answer:** O(n)

**Explanation:** The loop runs `n - 2` times, and each iteration performs a constant-time addition.

T(n) = n - 2

Therefore, the time complexity is linear, O(n). The space complexity is also O(n) because the sequence is stored in an array.

---

# Factorial of a Number

## Problem

Given an integer `n`, find the factorial of that integer.

In mathematics, the factorial of a non-negative integer `n`, denoted by `n!`, is the product of all positive integers less than or equal to `n`.

The factorial of zero is 1.

## Examples

- `factorial(0) = 1`
- `factorial(4) = 4 × 3 × 2 × 1 = 24`
- `factorial(5) = 5 × 4 × 3 × 2 × 1 = 120`

## Solution

```js
function factorial(n) {
  if (n < 0) throw new RangeError("n must be non-negative");

  let result = 1;

  for (let i = n; i >= 1; i--) {
    result *= i;
  }

  return result;
}
```

## Explanation

What is the time complexity of the solution?

**Answer:** O(n)

**Explanation:** The loop runs `n` times, and each iteration performs a multiplication.

T(n) = n

Therefore, the time complexity is O(n). The space complexity is O(1).

> **Note:** Results exceed `Number.MAX_SAFE_INTEGER` from `19!` onward. For exact results with large `n`, use `BigInt`:

```js
function factorialBig(n) {
  if (n < 0) throw new RangeError("n must be non-negative");

  let result = 1n;

  for (let i = 2n; i <= BigInt(n); i++) {
    result *= i;
  }

  return result;
}
```

With `BigInt`, multiplication is no longer truly constant-time, since the numbers grow with each step.

---

# Prime Number

## Problem

Given a natural number `n`, determine if the number is prime or not.

A prime number is a natural number greater than 1 that is not a product of two smaller natural numbers.

## Examples

- `isPrime(5) = true` (1 * 5 or 5 * 1)
- `isPrime(4) = false` (1 * 4 or 2 * 2 or 4 * 1)

## Solution

```js
function isPrime(n) {
  if (n < 2) return false;

  for (let i = 2; i * i <= n; i++) {
    if (n % i === 0) return false;
  }

  return true;
}
```

## Explanation

What is the time complexity of the solution?

**Answer:** O(√n)

**Explanation:** The loop runs from `2` up to √n, checking for factors in constant time on each iteration. Because factors come in pairs, checking past √n is unnecessary.

T(n) = √n

Therefore, the time complexity is O(√n). The space complexity is O(1).

> **Note:** `i * i <= n` is used instead of `i <= Math.sqrt(n)` to avoid calling `Math.sqrt` and doing floating-point work on every iteration.

---

# Power of Two

## Problem

Given a positive integer `n`, determine if the number is a power of 2 or not.

An integer is a power of two if there exists an integer `x` such that `n === 2^x`.

## Examples

- `isPowerOfTwo(1) = true` (2^0)
- `isPowerOfTwo(2) = true` (2^1)
- `isPowerOfTwo(5) = false`

## Solution 

```js
function isPowerOfTwo(n) {
  if (n < 1) return false;

  while (n > 1) {
    if (n % 2 !== 0) return false;
    n = n / 2;
  }

  return true;
}

isPowerOfTwo(1) // true
isPowerOfTwo(2) // true
isPowerOfTwo(5) // false
```

## Explanation

What is the time complexity of the solution?

**Answer:** O(log n)

**Explanation:** Each iteration halves `n`, so the loop runs at most log₂(n) times.

T(n) = log₂(n)

Therefore, the time complexity is O(log n). The space complexity is O(1).

## Solution (Bitwise)

```js
function isPowerOfTwoBitWise(n) {
  if (n < 1) return false;
  return (n & (n - 1)) === 0;
}

isPowerOfTwoBitWise(1) // true
isPowerOfTwoBitWise(2) // true
isPowerOfTwoBitWise(5) // false
```

## Explanation

What is the time complexity of the solution?

**Answer:** O(1)

**Explanation:** A power of two has exactly one bit set, so `n - 1` flips that bit and sets all lower bits. ANDing the two gives `0`. The check is a single bitwise operation, regardless of the size of `n`.

Therefore, the time complexity is O(1). The space complexity is O(1).

> **Note:** JavaScript bitwise operators convert numbers to 32-bit signed integers, so this version is only reliable up to 2³¹ − 1. For larger values, use `BigInt`:

```js
function isPowerOfTwoBig(n) {
  n = BigInt(n);
  return n > 0n && (n & (n - 1n)) === 0n;
}
```

With `BigInt`, the operation takes time proportional to the number of bits.