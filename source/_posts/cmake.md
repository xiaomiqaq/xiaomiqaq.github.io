---
title: cmake
date: 2023-09-20 20:22:53
tags:
---

# cmake

安装Ninja的一系列命令

```shell
$ mkdir -p ninja
$ ninja_url="https://github.com/Kitware/ninja/releases/download/v1.8.2.g3bbbe.kitware.dyndep-1.jobserver-1/ninja-1.8.2.g3bbbe.kitware.dyndep-1.jobserver-1_x86_64-linux-gnu.tar.gz"
$ curl -Ls ${ninja_url} | tar -xz -C ninja --strip-components=1
$ export PATH=$HOME/Deps/ninja${PATH:+:$PATH}
```



CMake是一个构建系统生成器 ，将描述构建系统(如：Unix Makefile、Ninja、Visual Studio等)应当如何操作才能编译代码。



每个生成器都有自己的文件集



生成器， 编译器

## 配置

配置时，CMake会进行一系列平台测试，以确定哪些编译器可用。。一个合适的编译器不仅取决于我们所使用的平台，还取决于我们想要使用的生成器。



### 查找可用的编译器

``` shell
$ cmake --system-information information.txt
```

## 构建



1. **Debug**：用于在没有优化的情况下，使用带有调试符号构建库或可执行文件。
2. **Release**：用于构建的优化的库或可执行文件，不包含调试符号。
3. **RelWithDebInfo**：用于构建较少的优化库或可执行文件，包含调试符号。
4. **MinSizeRel**：用于不增加目标代码大小的优化方式，来构建库或可执行文件。

## add_library

生成的库的实际名称将由CMake通过在前面添加前缀`lib`和适当的扩展名作为后缀来形成。



## option 

自定义变量并且在配置时指定

```cmake
option(USE_LIBRARY "Compile sources into a library" OFF)
```

自定义一个变量后，可以在配置时通过-D指定。

```shell
cmake -D USE_LIBRARY=ON ..
```

`-D`开关用于为CMake设置任何类型的变量：逻辑变量、路径等等。



## cmake模块

CMake有适当的机制，通过包含模块来扩展其语法和功能，这些模块要么是CMake自带的，要么是定制的。

例如要想使用cmake_dependent_option()，必须包含CMakeDependentOption。

```cmake
include(CMakeDependentOption)
cmake_dependent_option(
    MAKE_STATIC_LIBRARY "Compile sources into a static library" OFF
    "USE_LIBRARY" ON
    )
```





## 变量

### 优先级

1. -D
2. 

### 指定编译器

CMAKE_<LANG>COMPILER 

```shell
通过`-D`指定：
$ cmake -D CMAKE_CXX_COMPILER=clang++ ..
通过环境变量
$ env CXX=clang++ cmake ..
```

> `env` 命令用于显示当前用户环境中定义的环境变量。
>
> *CMake了解运行环境，可以通过其CLI的`-D`开关或环境变量设置许多选项。前一种机制覆盖后一种机制*

### 指定构建类型

配置变量是`CMAKE_BUILD_TYPE`。该变量默认为空，CMake识别的值为:

1. **Debug**：用于在没有优化的情况下，使用带有调试符号构建库或可执行文件。
2. **Release**：用于构建的优化的库或可执行文件，不包含调试符号。
3. **RelWithDebInfo**：用于构建较少的优化库或可执行文件，包含调试符号。
4. **MinSizeRel**：用于不增加目标代码大小的优化方式，来构建库或可执行文件。

使用`CMAKE_CONFIGURATION_TYPES`变量可以对这些生成器的可用配置类型进行调整，该变量将接受一个值列表。例如，在配置时，为Release和Debug配置生成一个构建树。

``` shell
 cmake .. -G"Visual Studio 12 2017 Win64" -D CMAKE_CONFIGURATION_TYPES="Release;Debug"
```

构建时指定

```shell
$ cmake --build . --config Release
```





### 设置编译选项

CMake将编译选项视为目标属性。



为目标设置编译选项：

方法1：

``` cmake

target_compile_options(geometry
  PRIVATE
    ${flags}
  )
```

#### 可见性

- **PRIVATE**，编译选项会应用于给定的目标，不会传递给与目标相关的目标。我们的示例中， 即使`compute-areas`将链接到`geometry`库，`compute-areas`也不会继承`geometry`目标上设置的编译器选项。
- **INTERFACE**，给定的编译选项将只应用于指定目标，并传递给与目标相关的目标。
- **PUBLIC**，编译选项将应用于指定目标和使用它的目标。

方法2：

不用对`CMakeLists.txt`进行修改。使用`-D`CLI标志直接修改`CMAKE_<LANG>_FLAGS_<CONFIG>`变量,这将影响项目中的所有目标，并覆盖或扩展CMake默认值。

```shell
$ cmake -D CMAKE_CXX_FLAGS="-fno-exceptions -fno-rtti" ..
```

#### 检查编译器标志

使用`CheckCXXCompilerFlag.cmake`模块提供的`check_cxx_compiler_flag`函数进行编译器标志的检查:

```
include(CheckCXXCompilerFlag)
check_cxx_compiler_flag("-march=native" _march_native_works)
```

这个函数接受两个参数:

- 第一个是要检查的编译器标志。
- 第二个是用来存储检查结果(true或false)的变量。如果检查为真，将工作标志添加到`_CXX_FLAGS`变量中，该变量将用于为可执行目标设置编译器标志。



#### 根据不同编译器设置不同选项

``` cmake
set(COMPILER_FLAGS)
set(COMPILER_FLAGS_DEBUG)
set(COMPILER_FLAGS_RELEASE)
if(CMAKE_CXX_COMPILER_ID MATCHES GNU)
  list(APPEND CXX_FLAGS "-fno-rtti" "-fno-exceptions")
  list(APPEND CXX_FLAGS_DEBUG "-Wsuggest-final-types" "-Wsuggest-final-methods" "-Wsuggest-override")
  list(APPEND CXX_FLAGS_RELEASE "-O3" "-Wno-unused")
endif()
if(CMAKE_CXX_COMPILER_ID MATCHES Clang)
  list(APPEND CXX_FLAGS "-fno-rtti" "-fno-exceptions" "-Qunused-arguments" "-fcolor-diagnostics")
  list(APPEND CXX_FLAGS_DEBUG "-Wdocumentation")
  list(APPEND CXX_FLAGS_RELEASE "-O3" "-Wno-unused")
endif()

target_compile_option(compute-areas
  PRIVATE
    ${CXX_FLAGS}
    "$<$<CONFIG:Debug>:${CXX_FLAGS_DEBUG}>"
    "$<$<CONFIG:Release>:${CXX_FLAGS_RELEASE}>"
  )
```

### 设置语言标准

```cma	
set_target_properties(animals
  PROPERTIES
    CXX_STANDARD 14
    CXX_EXTENSIONS OFF
    CXX_STANDARD_REQUIRED ON
    POSITION_INDEPENDENT_CODE 1
  )
```

## 检测环境



### 检测操作系统

CMAKE_SYSTEM_NAME 

>  *CMake代码中只使用前斜杠作为路径分隔符，CMake将自动将它们转换为所涉及的操作系统环境。*

#### 为不同系统设置宏

```cmake
if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
  target_compile_definitions(hello-world PUBLIC "IS_LINUX")
endif()
if(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
  target_compile_definitions(hello-world PUBLIC "IS_MACOS")
endif()
if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
  target_compile_definitions(hello-world PUBLIC "IS_WINDOWS")
endif()
```

```cpp
#ifdef IS_WINDOWS
  return std::string("Hello from Windows!");
#elif IS_LINUX
  return std::string("Hello from Linux!");
#elif IS_MACOS
  return std::string("Hello from macOS!");
#else
  return std::string("Hello from an unknown system!");
```

使用`target_compile_definition`来决定预处理阶段的行为。

### 检测CPU架构

CMake的`CMAKE_SIZEOF_VOID_P`变量会告诉我们CPU是32位还是64位

检查空指针类型的大小。CMake的`CMAKE_SIZEOF_VOID_P`变量会告诉我们CPU是32位还是64位。

```cmake
if(CMAKE_SIZEOF_VOID_P EQUAL 8)
  target_compile_definitions(arch-dependent PUBLIC "IS_64_BIT_ARCH")// 会在CPP中添加宏定义IS_64_BIT_ARCH
  message(STATUS "Target is 64 bits") 
else()
  target_compile_definitions(arch-dependent PUBLIC "IS_32_BIT_ARCH")
  message(STATUS "Target is 32 bits")
endif()

```



`CMAKE_HOST_SYSTEM_PROCESSOR `可以检测CPU架构

```cmake
if(CMAKE_HOST_SYSTEM_PROCESSOR MATCHES "i386")
    message(STATUS "i386 architecture detected")
endif()
target_compile_definitions(arch-dependent
  PUBLIC "ARCHITECTURE=${CMAKE_HOST_SYSTEM_PROCESSOR}"
  )
```

还有一个`CMAKE_SYSTEM_PROCESSOR`表示当前正在为其构建的CPU的名称。在交叉编译时很有用

### 检测指令集架构

```cpp
#include "config.h"
#include <cstdlib>
#include <iostream>
int main()
{
  std::cout << "Number of logical cores: "
            << NUMBER_OF_LOGICAL_CORES << std::endl;
  std::cout << "Number of physical cores: "
            << NUMBER_OF_PHYSICAL_CORES << std::endl;
  return 0;
}
```

这个config.h不是直接编辑，而是通过使用`config.h.in`生成。

config.h.in

```cpp
#pragma once
#define NUMBER_OF_LOGICAL_CORES @_NUMBER_OF_LOGICAL_CORES@
#define NUMBER_OF_PHYSICAL_CORES @_NUMBER_OF_PHYSICAL_CORES@
```

然后通过cmake填充`config.h`中的定义

``` cmake
foreach(key
  IN ITEMS
    NUMBER_OF_LOGICAL_CORES
    NUMBER_OF_PHYSICAL_CORES
    )
     cmake_host_system_information(RESULT _${key} QUERY ${key})
endforeach()
configure_file(config.h.in config.h @ONLY)
```

核心函数是`cmake_host_system_information`，它查询运行CMake的主机系统的系统信息。

使用这些变量来配置`config.h.in`中的占位符，输入并生成`config.h`

## 检测外部库

CMake有一组预打包模块，用于检测常用库和程序，`cmake --help-module-list`获得现有模块的列表

### find族命令

- **find_file**：在相应路径下查找命名文件
- **find_library**：查找一个库文件
- **find_package**：从外部项目查找和加载设置
- **find_path**：查找包含指定文件的目录
- **find_program**：找到一个可执行程序

## 属性

全局

项目

目标

源文件：源文件的可用属性列表https://cmake.org/cmake/help/v3.5/manual/cmake-properties.7.html#source-file-properties 

这四个都可以设置属性

## 语法

### 控制流

#### foreach

```cmake
list(
  APPEND sources_with_lower_optimization
    geometry_circle.cpp
    geometry_rhombus.cpp
  )
message(STATUS "Querying sources properties using plain syntax:")
foreach(_source ${sources_with_lower_optimization})
  get_source_file_property(_flags ${_source} COMPILE_FLAGS)
  message(STATUS "Source ${_source} has the following extra COMPILE_FLAGS: ${_flags}")
endforeach()
```

`foreach(loop_var range total)`

`foreach(loop_var IN LISTS [list1[...]])`

### 列表

列表是用分号分隔的字符串组。列表可以由`list`或`set`命令创建。例如，`set(var a b c d e)`和`list(APPEND a b c d e)`都创建了列表`a;b;c;d;e`

