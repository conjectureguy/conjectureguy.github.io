---
title: 'A Small Recursive Program'
date: 2026-09-04
description: 'A compact Python example and a few notes on recursion.'
math: false
---

Recursion is a useful way to describe a problem in terms of smaller versions of itself. Here is the familiar Fibonacci sequence in Python:

```python
def fibonacci(n):
    if n < 2:
        return n

    return fibonacci(n - 1) + fibonacci(n - 2)
```

The base case handles `0` and `1`. Every larger input is reduced until it reaches one of those cases.

## Things to notice

- The function mirrors the mathematical definition.
- The implementation is intentionally simple, not efficient.
- Memoization or iteration avoids repeated work in real programs.

For more about the language, see the [Python documentation](https://docs.python.org/3/).

> Clear examples are often more useful than clever ones.
