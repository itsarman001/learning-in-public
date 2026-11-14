# Recursion

## What is Recursion?

Recursion is a problem-solving technique where the solution depends on solutions to smaller instances of the same problem.

In programming, recursion occurs when a function calls itself. It is a powerful technique that helps simplify complex solutions.

If you find yourself breaking down a problem into smaller versions of the same problem, recursion can be very useful.

### Important Notes

- Every recursive solution must have a base case — a condition that terminates the recursion. Without it, the function may get stuck in an infinite loop.
    
- Recursion might simplify problem-solving, but it doesn't always result in a faster solution. A recursive solution can be significantly less efficient than an iterative solution.
    
- Recursion is not the most straightforward topic to understand. It takes time and practice to become comfortable with it.
    

## 1. Recursive Fibonacci Sequence

### Problem

Given a number `n`, find the `n`th element of the Fibonacci sequence.

The Fibonacci sequence is a sequence in which each number is the sum of the two preceding numbers.

The first two numbers in the sequence are `0` and `1`.

Sequence:

`0, 1, 1, 2, 3, 5, 8, 13, 21, ...`

### Examples

```js
recursiveFibonacci(0) // 0
recursiveFibonacci(1) // 1
recursiveFibonacci(6) // 8
```

### Solution

```js
function recursiveFibonacci(n) {
  if (n < 2) return n;

  return recursiveFibonacci(n - 1) + recursiveFibonacci(n - 2);
}

recursiveFibonacci(0) // 0
recursiveFibonacci(1) // 1
recursiveFibonacci(6) // 8
```

### Explanation

The Fibonacci function works by breaking the problem into two smaller recursive calls:

F(n)=F(n−1)+F(n−2)F(n) = F(n-1) + F(n-2)

Base Case:

```js
if (n < 2) return n;
```

- If `n` is `0`, return `0`.
    
- If `n` is `1`, return `1`.
    

Recursive Case:

```js
return recursiveFibonacci(n - 1) + recursiveFibonacci(n - 2);
```

The function calls itself with the two previous numbers and adds their results.

### Complexity Analysis

- Time Complexity: $O(2^n)$ — The function makes overlapping recursive calls, resulting in exponential time complexity.
    
- Space Complexity: $O(n)$ — The maximum recursion call stack depth is proportional to `n`.
    

## 2. Recursive Factorial of a Number

### Problem

Given an integer `n`, find the factorial of that integer.

The factorial of a non-negative integer `n`, denoted by `n!`, is the product of all positive integers less than or equal to `n`.

The factorial of zero is `1`.

### Examples

```js
factorial(4) = 4 * 3 * 2 * 1 = 24

factorial(5) = 5 * 4 * 3 * 2 * 1 = 120

factorial(0) = 1
```

### Solution

```js
function recursiveFactorial(n) {
  if (n === 0) return 1;

  return n * recursiveFactorial(n - 1);
}

recursiveFactorial(0) // 1
recursiveFactorial(1) // 1
recursiveFactorial(5) // 120
```

### Explanation

The factorial function works by multiplying the current number by the factorial of the previous number.

n!=n×(n−1)!n! = n \times (n-1)!

Base Case:

```js
if (n === 0) return 1;
```

When `n` reaches `0`, return `1` to terminate the recursion.

Recursive Case:

```js
return n * recursiveFactorial(n - 1);
```

The function multiplies `n` by the result of calling itself with `n - 1`.

For example, `recursiveFactorial(5)` works as follows:

$$
\begin{array}{rcl}
5! &=& 5 \times 4! \\
   &=& 5 \times 4 \times 3! \\
   &=& 5 \times 4 \times 3 \times 2! \\
   &=& 5 \times 4 \times 3 \times 2 \times 1! \\
   &=& 5 \times 4 \times 3 \times 2 \times 1 \\
   &=& 120
\end{array}
$$

### Complexity Analysis

- Time Complexity: $O(n)$ — The function makes one recursive call for each number until it reaches `0`.
    
- Space Complexity: $O(n)$ — The recursion call stack grows proportionally to `n`.
    

## Key Takeaways

| Concept          | Fibonacci | Factorial |
| ---------------- | --------- | --------- |
| Base Case        | `n < 2`   | `n === 0` |
| Recursive Calls  | 2         | 1         |
| Time Complexity  | $O(2^n)$  | $O(n)$    |
| Space Complexity | $O(n)$    | $O(n)$    |

Remember: Every recursive solution has two important parts:

1. Base Case: When should the recursion stop?
    
2. Recursive Case: How can the problem be broken down into smaller instances of itself?