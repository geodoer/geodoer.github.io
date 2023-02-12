---
title: C++将成员函数转成std::function
tags:
    - C++
    - 成员函数
    - std::function
---

方法一：
```cpp
#define HZ_BIND_EVENT_FN(fn) [this](auto&&... args) -> decltype(auto) { return this->fn(std::forward<decltype(args)>(args)...); }
```

方法二：
```cpp
#define HZ_BIND_EVENT_FN(fn) std::bind(&x, this, std::placeholders::_1)
```