---
layout: post
title:  CMake Bugs
date:   2024-06-27 11:10:16
description: debugging notes.
tags: linux, programming
categories: mix-posts
---

# Table of Content
- [Table of Content](#table-of-content)
- [undefined symbol](#undefined-symbol)

# undefined symbol
Error Message: 
```
Traceback (most recent call last):
  File "test_vis.py", line 3 in <module>
    from cpp.build import pcl
ImportError: /xxx/pcl.cpython-38-x86_64-linux-gnu.so: undefined symbol: _ZN12PointMatcherIfE12ICPChainBase10setDefaultEv
```

Cause: the built module is not linked to the necessary libraries.

Solution: add
```
pybind11_add_module(pcl, pcl.cpp)
target_link_libraries(pcl PRIVATE ${MY_LIBRARIES})
```

[ref](https://github.com/pybind/pybind11/issues/1403)


[Back to Top](#table-of-content)

