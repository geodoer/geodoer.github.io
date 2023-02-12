官方文档：[doctest/doc/markdown at master · doctest/doctest (github.com)](https://github.com/doctest/doctest/tree/master/doc/markdown)


## 集成
### CMake Head-Only

### CMake FetchContent
Head-only引入的话，编译速度会慢。因此建议编译后链接。

```cmake
include(FetchContent)  
FetchContent_Declare(
	doctest
	GIT_REPOSITORY https://github.com/doctest/doctest.git
	GIT_TAG v2.4.9         
	GIT_SHALLOW TRUE
)
FetchContent_MakeAvailable(doctest)  ​  

enable_testing()
add_executable(test main.cpp)
target_link_libraries(test PRIVATE doctest_with_main)
```

main.cpp
```cpp
#include "doctest/doctest.h"

int factorial(int number) { return number <= 1 ? number : factorial(number - 1) * number; }

TEST_CASE("testing the factorial function") {
    CHECK(factorial(1) == 1);
    CHECK(factorial(2) == 2);
    CHECK(factorial(3) == 6);
    CHECK(factorial(10) == 3628800);
}
```

## 参考文章
[C++工程实践必备技能 - 掘金 (juejin.cn)](https://juejin.cn/post/7194435171633299513)