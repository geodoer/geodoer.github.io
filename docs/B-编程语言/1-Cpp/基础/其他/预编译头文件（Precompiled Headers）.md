
## 背景

即使没有使用模板，C++头文件也可以变大非常庞大，然后导致编译时间过长。而模板加重了这种问题，而各个编译器厂商提供了一种名为“预编译头文件”（precompiled header）的机制，来缓解这个问题。

- “预编译头文件”并不是标准库的范畴，是编译器厂家提供的方案

## 示例：MSVC
来源：[TheCherno/Hazel: Hazel Engine (github.com)](https://github.com/TheCherno/Hazel)

### 代码

1. `hzpch.h`
```cpp
#pragma once

#include "Hazel/Core/PlatformDetection.h"

#ifdef HZ_PLATFORM_WINDOWS
	#ifndef NOMINMAX
		// See github.com/skypjack/entt/wiki/Frequently-Asked-Questions#warning-c4003-the-min-the-max-and-the-macro
		#define NOMINMAX
	#endif
#endif

#include <iostream>
#include <memory>
#include <utility>
#include <algorithm>
#include <functional>

#include <string>
#include <sstream>
#include <array>
#include <vector>
#include <unordered_map>
#include <unordered_set>

#include "Hazel/Core/Base.h"

#include "Hazel/Core/Log.h"

#include "Hazel/Debug/Instrumentor.h"

#ifdef HZ_PLATFORM_WINDOWS
	#include <Windows.h>
#endif
```

2. `hzpch.cpp`
```cpp
#include "hzpch.h"
```

3. 例如，使用vector，无需`#include<vector>`，直接`#include "hzpch.h"`

### 工程配置
#### Visual Studio

1. 项目 > 右键 > 配置 > 配置属性 > C/C++ > 预编译头
2. 预编译头 > 设置成“使用”
3. 预编译头文件 > 输入"hzpch.h"

#### Premake
```lua
project "Hazel"
	pchheader "hzpch.h"
    pchsource "src/hzpch.cpp"

	-- ...
```