# 自定义后端

## 概述

当框架内置后端无法满足需求时，可以利用MindSpore的自定义后端功能实现自己的后端。本教程提供一个简单的自定义后端用例展示。

## 实现自定义后端

自定义后端的实现需要完成以下步骤：

1. 引用 `mindspore/include/custom_backend_api.h` 头文件
2. 继承 `BackendBase` 类并实现 `Build` 和 `Run` 接口
3. 使用 `MS_REGISTER_BACKEND` 宏注册自定义后端

以下是一个简单的自定义后端示例，仅添加打印信息：

```cpp
#include <string>
#include <memory>
#include <vector>
#include "mindspore/include/custom_backend_api.h"

namespace mindspore {
namespace backend {
constexpr auto kCustomBackendName = "my_custom_backend";

class MSCustomBackendBase : public BackendBase {
 public:
  BackendGraphId Build(const FuncGraphPtr &func_graph, const BackendJitConfig &backend_jit_config) {
    MS_LOG(INFO) << "MSCustomBackendBase use the origin ms_backend to build the graph.";
  }

  RunningStatus Run(BackendGraphId graph_id, const VectorRef &inputs, VectorRef *outputs) {
    MS_LOG(INFO) << "MSCustomBackendBase use the origin ms_backend to run the graph.";
  }
};
MS_REGISTER_BACKEND(kCustomBackendName, MSCustomBackendBase);
}  // namespace backend
}  // namespace mindspore
```

## 编译自定义后端

将示例代码保存为 `custom_backend.cc`，使用以下CMake脚本编译成 `libcustom_backend.so` 动态库：

```cmake
cmake_minimum_required(VERSION 3.16)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(MINDSPORE_INCLUDE_DIR ${MINDSPORE_ROOT}/include)
set(MINDSPORE_LIB_DIRS ${MINDSPORE_ROOT}/lib)
message(STATUS "Using MindSpore from: ${MINDSPORE_ROOT}")

set(CMAKE_BUILD_TYPE "Release")

include_directories(${CMAKE_CURRENT_SOURCE_DIR})

if(MINDSPORE_INCLUDE_DIR)
    include_directories(${MINDSPORE_INCLUDE_DIR})
    include_directories(${MINDSPORE_INCLUDE_DIR}/mindspore)
    include_directories(${MINDSPORE_INCLUDE_DIR}/mindspore/core/include)
    include_directories(${MINDSPORE_INCLUDE_DIR}/mindspore/ccsrc/include)
    include_directories(${MINDSPORE_INCLUDE_DIR}/mindspore/ccsrc)
    include_directories(${MINDSPORE_INCLUDE_DIR}/third_party)
    include_directories(${MINDSPORE_INCLUDE_DIR}/third_party/include)
    include_directories(${MINDSPORE_INCLUDE_DIR}/third_party/pybind11)
endif()

find_package(Python3 COMPONENTS Interpreter Development)
if(Python3_FOUND)
    set(PYTHON_INCLUDE_DIRS "${Python3_INCLUDE_DIRS}")
    set(PYTHON_LIBRARIES "${Python3_LIBRARIES}")
    if(WIN32)
        if(Python3_DIR)
            message("Python3_DIR set already: " ${Python3_DIR})
        else()
            string(LENGTH ${PYTHON_LIBRARIES} PYTHON_LIBRARIES_LEN)
            string(LENGTH "libpythonxx.a" Python3_NAME_LEN)
            math(EXPR Python3_DIR_LEN ${PYTHON_LIBRARIES_LEN}-${Python3_NAME_LEN})
            string(SUBSTRING ${PYTHON_LIBRARIES} 0 ${Python3_DIR_LEN} Python3_DIR)
            message("Python3_DIR: " ${Python3_DIR})
        endif()
        link_directories(${Python3_DIR})
    endif()
else()
    find_python_package(py_inc py_lib)
    set(PYTHON_INCLUDE_DIRS "${py_inc}")
    set(PYTHON_LIBRARIES "${py_lib}")
endif()
include_directories(${PYTHON_INCLUDE_DIRS})

file(GLOB_RECURSE SRC_SOURCES "*.cc")

add_library(custom_backend SHARED ${SRC_SOURCES})

target_link_libraries(custom_backend
    ${MINDSPORE_LIB_DIRS}/libmindspore_backend_manager.so
    ${MINDSPORE_LIB_DIRS}/libmindspore_core.so
    ${MINDSPORE_LIB_DIRS}/libmindspore_common.so
)

if(CMAKE_SYSTEM_NAME MATCHES "Linux")
    if(NOT ENABLE_GLIBCXX)
        add_compile_definitions(_GLIBCXX_USE_CXX11_ABI=0)
    endif()
endif()

target_compile_options(custom_backend PRIVATE
    -fPIC
    -std=c++17
    -Wall
    -Wextra
)

install(TARGETS custom_backend
    LIBRARY DESTINATION lib
    RUNTIME DESTINATION bin
)
```

编译命令：

```bash
cmake . -DMINDSPORE_ROOT=/path/to/mindspore
make
```

其中 `/path/to/mindspore` 为MindSpore的安装路径。

## 使用自定义后端

使用 `mindspore.graph.register_custom_backend` 接入后端，并通过 `mindspore.jit` 接口选择使用：

```python
import mindspore
import numpy as np
from mindspore import jit, mint

custom_path = "/data1/libcustom_backend.so"
success = mindspore.graph.register_custom_backend(backend_name="my_custom_backend", backend_path=custom_path)
assert success, "Plugin registration failed"

x = mindspore.Tensor(np.ones([2, 2], np.float32))
y = mindspore.Tensor(np.zeros([2, 2], np.float32))

@jit(backend="my_custom_backend")
def net1(x):
    return mint.sin(x)
x = net1(x)

@jit(backend="ms_backend")
def net2(x):
    return mint.cos(x)
y = net2(y)
```

`net1` 将使用自定义后端 `my_custom_backend`，`net2` 将使用MindSpore内置后端 `ms_backend`。
