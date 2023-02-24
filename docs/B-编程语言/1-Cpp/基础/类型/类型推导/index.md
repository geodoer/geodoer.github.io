---
title: 类型推导
---

> 诚如C#语言中，可以通过var关键字来实现类型的推导，以此简化一些不需要关注变量类型的场景。而C++11也引入了类型推导。

## auto(C++11)

鉴于`auto`关键字在C++98/03中极少用到（因为非`static`即`auto`），故C++11也为`auto`赋予了新的含义，即类型推导。

auto定义的变量必须要有初始值，因为编译器通过初始值来进行类型的推导，从而获得变量的类型。

```cpp
//普通类型
int a=1, b=3;
auto c = a+b; //c为int

//const类型
const int i = 5;
auto j = i;  //变量i是顶层const，会被忽略，因此j类型是int
auto k = &i; //变量i是一个常量，对常量取地址是一种底层const，所以k是const int*
const auto l = i;  //如果希望推断出顶层const，则需要在auto前加上const

//引用和指针类型
int x = 2;
int& y = x;
auto z = y;   //z是int型，不是int&
auto& p1 = y; //p1是int&型
auto p2 = &x; //p2是指针类型int* 
```

## decltype(C++11)
`auto`只能通过初始值来进行类型推断，这很不够用。  如果我们希望，通过表达式来推断出变量的类型，那么可以使用`decltype`。

`decltype`会让编译器分析表达式，并获得它的类型，却不计算表达式的值。

```cpp
// 函数返回值
int func() {return 0;}
decltype(func()) sum = 5; // sum的类型是函数func()的返回值的类型int, 但是这时不会实际调用函数func()

//普通类型
int a = 0;
decltype(a) b =4; // a的类型是int, 所以b的类型也是int

//不论是顶层const还是底层const, decltype都会保留   
const int c = 3;
decltype(c) d = c; // d的类型和c是一样的, 都是顶层const
int e = 4;
const int* f = &e; // f是底层const
decltype(f) g = f; // g也是底层const

//不论是顶层const还是底层const, decltype都会保留   
const int c = 3;
decltype(c) d = c; // d的类型和c是一样的, 都是顶层const
int e = 4;
const int* f = &e; // f是底层const
decltype(f) g = f; // g也是底层const

//在所有编译器中都适用
typedef decltype(nullptr) nullptr_t;	//通过编译器关键字nullptr定义类型nullptr_t
typedef decltype(sizeof(0)) size_t;

//抽取变量类型
vector<int> v;
decltype(v)::value_type i = 0;

//模板中的妙用
template<class ContainerT>
class Foo{
	decltype(ContainerT().begin()) it_;
public:
	void func(ContainerT& container) {
    	it_ = container.begin();
    }
    //...
};
```


## 返回值后置(C++11)
在写模板时，可能会遇到以下情况。在coding阶段，我们无法确定返回值的类型，只有在使用模板时，才能确定返回值具体的类型。

此时就可以用C++11中的 **“返回值后置”** 语法。

1. 用`auto`代替返回值
2. 在函数括号后，使用`decltype`自动推断出返回值类型

```cpp
template<typename T, typename U>
auto add(T t, U u) -> decltype(t + u) {
	return t + u;
}
```

更复杂的例子
```cpp
int& foo(int& i);
float foo(float& f);

template<typename T>
auto func(T& val) -> decltype(foo(val)) {
	return foo(val);
}
```

## decltype(auto) (C++14)
在C++14中，新增了`decltype(auto)`，它可以用来声明变量。  

1. 编译器会将“`=`“右边的表达式类型，替换掉`auto`
2. 再根据`decltype`的语法规则来确定类型

```cpp
int e = 4;
const int* f = &e;    //f是底层const
decltype(auto) j = f; //j的类型是const int* 并且指向的是e
```

## 参考文章

1. [C++11新标准-01-20 | 阿秀的学习笔记](https://interviewguide.cn/notes/03-hunting_job/02-interview/01-03-01-C++11.html)
