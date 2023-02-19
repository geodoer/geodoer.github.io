---
title: C++断言assert
---

```cpp
#include <assert.h>
void assert(int expression);
	//expression == 0时，会向stderr打印一条错误信息，而后调用abort终止程序
```

assert一般用于检查输入参数的合法性，方便程序员调试时定位代码
```cpp
int fun(int size) 
{ 
	assert(size >= 0); 
	assert(size <= 10); 
	... 
}
```

但是频繁的调用会影响程序性能，增加额外的开销。可以定义`NDEBUG`宏来关闭`assert`。
```cpp
#define NDEBUG
#include <assert.h> //assert函数会被取消，程序不会崩溃
```

-  `assert`一般用于程序员调试BUG时，在`DEBUG`环境下的帮手
- 在Release模式下，`NDEBUG`是默认定义的，因此在Relase下，assert不起作用
- 在RelWithDebInfo下也会定义`NDEBUG`，如果想要打开`assert`，可这么做
```cpp
#undef NDEBUG //取消定义NDEBUG
#include<assert.h> //assert函数启用
```

## 注意事项
在使用`assert`应注意以下几点。

一、assert只能包含只读操作，不能影响上下文

```cpp
//错误。会造成，启用assert、未启用assert下，代码的不同
assert(i++); 

//推荐写法
assert(i);
i++;
```

二、一次assert只检验一个条件，为了在调试时，一目了然是哪个条件不成立

```cpp
//不建议
assert(nOffset>=0 && nOffset+nSize<=m_nInfomationSize);

//推荐写法
assert(nOffset >= 0); 
assert(nOffset+nSize <= m_nInfomationSize);
```

三、assert不能代替条件判断，因为在某些条件下，assert并不启用