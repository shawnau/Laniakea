---
title: "Optimize Recursion"
date: 2020-02-11T15:33:49+08:00
draft: false
---

<!--more-->

We can easily optimize a recursion by using a cache, which loads calculated function calls during the recursion process.

```python
from functools import wraps

def memo(func):
    cache = dict()

    @wraps(func)
    def wrap(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    return wrap
```

Some conditional branches could be merged by `and` operator.
```python
@memo
def fib(n: int) -> int:
    if n in {0, 1}:
        return 1
    else:
        return fib(n-1) + fib(n-1)
```


