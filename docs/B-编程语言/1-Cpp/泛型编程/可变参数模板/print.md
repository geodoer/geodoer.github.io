---
tags:
    - C++
    - 可变参数模板
---

## C++11

```cpp
template<typename Last>
void print(const Last& last)
{
	std::cout << last << std::endl;
}
template<typename First, typename Rest>
void print(const First& first, const Rest&... rest)
{
	std::cout << first << " ";
	print(rest...); //使用...将参数包展开
}
```

## C++17
使用C++17的折叠表达式

```cpp
template<typename... Args>
void print(const Args&... args) //折叠表达式折叠的是一个集合，这里的集合就是一堆参数
{
	( //折叠表达式，折叠的是操作
		(std::cout << args << " ") //初始操作
		,...   //逗号表达式：展开args，并重复初始操作
	);
}
```