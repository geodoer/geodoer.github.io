### 背景：旧标准的auto
在旧标准中，**auto**代表“具有自动存储期的 **局部变量**”
```cpp
auto int i = 0;		//具有自动存储期的局部变量
//C++98/03，可以默认写成int i=0;
static int j = 0;	//静态类型的定义方法
```

- 实际上，我们很少使用`auto`，因为非static即“具有自动存储期的”，所以不用人为去写`auto`

因此，在C++11中，auto关键字不再表示存储类型指示符（如static、register、mutable等都属于storage-class-specifiers），而是改成了一个类型指示符（type-specifier）。

### auto初探
不同于Python等动态类型语言的运行时变量类型推导，隐式类型定义（如`auto x = 5;`）的类型推导 **发生在编译期**。它的作用是让编译器自动 **推断出这个变量的类型**，而不需要显式指定类型
```cpp
auto x = 5;					//OK: x是int类型
//字面量5是const int类型，变量x将被推导为int类型（const被丢弃）
auto pi = new auto(1);		//OK: pi被推导为int*
const auto *v = &x, u=6;	//OK: v是const int*类型，u是const int类型
//因为v，u已经被推导为const int类型
//1. u必须初始化，不然编译不予通过
//2. u的初始化不能使编译器产生二义性。如u=6.6，编译器会报错
static auto y - 0.0;		//OK: y是double类型
auto int r;					//error: auto不再表示存储类型指示符，编译器报错
auto s;						//error: auto无法推导出s的类型，编译器报错
```

1. 隐式的类型定义也是 **强类型定义** （auto所在那行，已经准确推断出其类型了）
2. auto只能用于 **类型推导** ，不能用于存储类型指示符的推导，比如`auto int r =5;`
3. auto并不能代表一个实际类型的声明（如s的编译器错误），它只是一个类型声明的“占位符”
4. auto声明的变量必须马上初始化，以让编译器推断出它的实际类型，并在编译时将其替换为真正的类型

### 推导规则
auto可以通指针、引用结合起来使用，还可以带上cv限定符（cv-qualifier, const和volatile限定符的统称）

推导规则

1. 当**不声明**为指针或引用时，auto的推导结果和初始化表达式**抛弃引用和cv限定符**后类型一致
2. 当声明为指针或引用时，auto的推导结果将保持初始化表达式的cv属性
```cpp
int x = 0;

auto *a = &x;		//a -> int*			auto被推导为int
auto b = &x;		//b -> int* 		auto被推导为int*
//auto不声明为指针，也可以推导出指针类型
auto &c = x;		//c -> int&			auto被推导为int
auto d = c;			//d -> int			auto被推导为int
//当表达式是一个引用类型时，auto会把引用类型抛弃

const auto e = x;	//e -> const int	auto被推导为int
auto f = e;			//f -> int			auto被推导为int(non-const int)
//当表达式带有const（volatile也一样）属性时，auto会把const属性抛弃掉

const auto& g = x;	//e -> const int&	auto被推导为int
auto& h = g;		//f -> const int&	auto被推导为const int
    //auto和引用（或指针）结合时，auto的推导将保留表达式的const属性
```

### auto的推导和函数模板参数的自动推导
auto的推导和函数模板参数的自动推导有相似之处

上面的例子中的auto，和下面的模板参数自动推导出来的类型是一致的
```cpp
template<typename T> void func(T x) {}		//T		-> auto
template<typename T> void func(T *x) {}		//T* 	-> auto *
template<typename T> void func(T& x) {}		//T&	-> auto &

template<typename T> void func(const T x) {}	//const T -> const auto
template<typename T> void func(const T* x) {}	//const T* -> const auto*
template<typename T> void func(const T& x) {}	//const T& -> const auto&
```

因此，在熟悉auto推导规则时，可以借助函数模板的参数自动推导规则来帮助和加强理解。

### auto的限制
```cpp
void func(auto a = 1) {}	//error: auto不能用于函数参数

struct Foo{
    auto var1_ = 0;					//error: auto不能用于非静态成员变量
    static const auto var2_ = 0;	//OK: var2_ -> static const int
        //auto仅能用于推导static const的整形或枚举成员
        //1. 因为其他静态类型在C++标准中无法就地初始化
        //2. 虽然C++11可以接受非静态成员变量的就地初始化，但却不支持auto类型非静态成员的初始化
};

template<typename T>
struct Bar{};

void main() {
    int arr[10] = {0};
    auto aa = arr;		//OK: aa -> int*
        //aa不会推导为int[10]，而是int*
    auto rr[10] = arr;	//error: auto无法定义数组
    
    Bar<int> bar;
    Bar<auto> bb = bar; //error: auto无法推导出模板参数
}
```

### 什么时候用auto

1. 迭代器类型等场景
2. auto简化函数定义

不加选择地随意使用auto，会带来代码可读性和维护性的严重下降。因此，在使用auto的时候，一定要权衡好。

#### auto简化函数定义
```cpp
class Foo{
public:
    static int get(void) {return 0;}
};
class Bar{
public:
    static const char* get(void) {return "0";}
};

template<class A>
void func(void){
    //对类型A都调用静态函数get来获取val
    auto val = A::get();
    //对val做统一处理
    //...
}

func<Foo>();
func<Bar>();
```

如果不用auto，就不得不对func再添加一个模板参数，并在外部调用时手动指定get()的返回值类型
```cpp
class Foo{
public:
    static int get(void) {return 0;}
};
class Bar{
public:
    static const char* get(void) {return "0";}
};

template<class A, class B>
void func(void){
    B val = A::get();
    //...
}

func<Foo, int>();
func<Bar, const char*>();
```

### 参考文章

1. 参考书籍《深入应用C++11》




