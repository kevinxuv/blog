---
title: [Python Basics] Build in Functions
tags: 
    - python
    - python basics
---
python build in functions：

- [2.7.*](https://docs.python.org/2.7/library/functions.html#func-list)
- [3.7.*](https://docs.python.org/3/library/functions.html#func-list)

## list

```python
import dis
def func_a():
    return []
def func_b():
    return list()
dis.dis(func_a)
dis.dis(func_b)
```
