# L1 DP1

## 1. Dynamic Programming Overview

- Fibonacci numbers
- Longest increasing subsequence(LIS)
- Longest common subsequence(LCS)
- Knapsack
- Chain matrix multiply
- Shortest path algorithms

## 2. Fibonacci Numbers

Toy example:
Fibonacci numbers:
Input: Interger n >= 0
Output: nth Fibonacci number

## 3. FIB1: Recursive Algorithm

for n > 1:
$F_n = F_{n-1} + F_{n-2}$

```python
Fib1(n):
    input: integer n>=0
    output: Fn
    if n = 0, return (0)
    if n = 1, return (1)
    return (Fib1(n-1) + Fib1(n-2))
```

Let $T(n) =  \text{steps for Fib1(n)}$
