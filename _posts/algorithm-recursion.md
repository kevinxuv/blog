---
layout: post
title: Recursion in Algorithsm
tags: 
    - python guide
---


```python
def recursion(level, *args, **kwargs):

    # break recursion
    if level > max_level:
        return result

    # process data and get next level params
    args, kwargs = process_data(level, *args, **kwargs)

    # call func itself
    recursion(level + 1, *args, **kwargs)

    # reverse the parent level state if needed
    reverse_state(level)
```
