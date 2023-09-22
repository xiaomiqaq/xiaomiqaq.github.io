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

## 命令

大多数 CMake 命令在配置的时候执行。



### 生成器表达式

最简单的生成器表达式是信息表达式，其形式为 `$<KEYWORD>`；它会评估和当前配置相关的一系列信息。信息表达式的另一个形式是 `$<KEYWORD:value>`，其中 `KEYWORD` 是一个控制评估的关键字，而 `value` 则是被评估的对象（ 这里的 `value` 中也允许使用信息表达式，如下面的 `${CMAKE_CURRENT_SOURCE_DIR}/include` ）。如果 `KEYWORD` 是一个可以被评估为0或1的生成器表达式或者变量，如果（`KEYWORD`被评估）为1则 `value` 会在这里被保留下来，而反之则不会。

### add_library

生成的库的实际名称将由CMake通过在前面添加前缀`lib`和适当的扩展名作为后缀来形成。



### option 

自定义变量并且在配置时指定

```cmake
option(USE_LIBRARY "Compile sources into a library" OFF)
```

自定义一个变量后，可以在配置时通过-D指定。

```shell
cmake -D USE_LIBRARY=ON ..
```

`-D`开关用于为CMake设置任何类型的变量：逻辑变量、路径等等。





## 选项

使用 `-D` 设置选项。你能使用 `-L` 列出所有选项，或者用 `-LH` 列出人类更易读的选项列表。

## 变量,缓存,属性

CMake 支持缓存选项。CMake 中的变量可以被标记为 "cached"，这意味着它会被写入缓存（构建目录中名为 `CMakeCache.txt` 的文件）。

### 变量	

```CMake
set(MY_VARIABLE "value")
```

在 CMake 中如果一个值没有空格，那么加和不加引号的效果是一样的。这使你可以在处理知道不可能含有空格的值时不加引号。

当一个变量用 `${}` 括起来的时候，空格的解析规则和上述相同。对于路径来说要特别小心，路径很有可能会包含空格，因此你应该总是将解析变量得到的值用引号括起来，也就是，应该这样 `"${MY_PATH}"` 。

#### cmake设置环境变量

通过 `set(ENV{variable_name} value)` 和 `$ENV{variable_name}` 来设置和获取环境变量

### 缓存

缓存实际上就是个文本文件，`CMakeCache.txt` ，当你运行 CMake 构建目录时会创建它。 CMake 可以通过它来记住你设置的所有东西，因此你可以不必在重新运行 CMake 的时候再次列出所有的选项。

### 属性

属性本质还是一个变量，

- 属性是与CMake的目标（如可执行文件、库、自定义目标等）相关联的元数据。这些属性可以控制目标的构建过程和行为。
- 属性是目标特定的，可以设置不同的属性值来控制不同目标的行为。

像通常我们设置的CMAKE_开头的变量，往往是一个全局属性

```cmake
set_property(TARGET TargetName
             PROPERTY CXX_STANDARD 11)

set_target_properties(TargetName PROPERTIES
                      CXX_STANDARD 11)
```

第一种方式更加通用 ( general ) ，它可以一次性设置多个目标、文件、或测试，并且有一些非常有用的选项。第二种方式是为一个目标设置多个属性的快捷方式。此外，你可以通过类似于下面的方式来获得属性：

```cmake
get_property(ResultVariable TARGET TargetName PROPERTY CXX_STANDARD)
```

目标可以分为下列：全局、项目、目标、源文件。

源文件的可用属性列表https://cmake.org/cmake/help/v3.5/manual/cmake-properties.7.html#source-file-properties 

这四个都可以设置属性



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



# cmake 行为准则

- **不要使用具有全局作用域的函数**：这包含 `link_directories`、 `include_libraries` 等相似的函数。
- **不要添加非必要的 PUBLIC 要求**：你应该避免把一些不必要的东西强加给用户（-Wall）。相比于 **PUBLIC**，更应该把他们声明为 **PRIVATE**。
- **不要在file函数中添加 GLOB 文件**：如果不重新运行 CMake，Make 或者其他的工具将不会知道你是否添加了某个文件。值得注意的是，CMake 3.12 添加了一个 `CONFIGURE_DEPENDS` 标志能够使你更好的完成这件事。
- **将库直接链接到需要构建的目标上**：如果可以的话，总是显式的将库链接到目标上。
- **当链接库文件时，不要省略 PUBLIC 或 PRIVATE 关键字**：这将会导致后续所有的链接都是缺省的。
- **把 CMake 程序视作代码**：它是代码。它应该和其他的代码一样，是整洁并且可读的。
- **建立目标的观念**：你的目标应该代表一系列的概念。为任何需要保持一致的东西指定一个 （导入型）INTERFACE 目标，然后每次都链接到该目标。
- **导出你的接口**：你的 CMake 项目应该可以直接构建或者安装。
- **为库书写一个 Config.cmake 文件**：这是库作者为支持客户的体验而应该做的。
- **声明一个 ALIAS 目标以保持使用的一致性**：使用 `add_subdirectory` 和 `find_package` 应该提供相同的目标和命名空间。
- **将常见的功能合并到有详细文档的函数或宏中**：函数往往是更好的选择。
- **使用小写的函数名**： CMake 的函数和宏的名字可以定义为大写或小写，但是一般都使用小写，变量名用大写。
- **使用 `cmake_policy` 和/或 限定版本号范围**： 每次改变版本特性 (policy) 都要有据可依。应该只有不得不使用旧特性时才降低特性 (policy) 版本。

# 特定场景

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

# 检测环境



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

# 检测外部库

CMake有一组预打包模块，用于检测常用库和程序，`cmake --help-module-list`获得现有模块的列表

## find族命令

- **find_file**：在相应路径下查找命名文件
- **find_library**：查找一个库文件
- **find_package**：从外部项目查找和加载设置
- **find_path**：查找包含指定文件的目录
- **find_program**：找到一个可执行程序

### find_package

CMake模块文件称为`Find<name>.cmake`，当调用`find_package(<name>)`时，模块中的命令将会运行。`find_package`是用于发现和设置包的CMake模块的命令。

查找模块还会设置了一些有用的变量，反映实际找到了什么，也可以在自己的`CMakeLists.txt`中使用这些变量。例如对于Python解释器，PYTHONINTERP

- **PYTHONINTERP_FOUND**：是否找到解释器
- **PYTHON_EXECUTABLE**：Python解释器到可执行文件的路径
- **PYTHON_VERSION_STRING**：Python解释器的完整版本信息
- **PYTHON_VERSION_MAJOR**：Python解释器的主要版本号
- **PYTHON_VERSION_MINOR** ：Python解释器的次要版本号
- **PYTHON_VERSION_PATCH**：Python解释器的补丁版本号

## 检测Python库

### 将Python解释器嵌入到C或C++程序中

需要下列条件：

- Python解释器的工作版本
- Python头文件Python.h的可用性
- Python运行时库libpython

```cmake
cmake_minimum_required(VERSION 3.5 FATAL_ERROR)
project(recipe-02 LANGUAGES C)
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_EXTENSIONS OFF)
set(CMAKE_C_STANDARD_REQUIRED ON)
//找到Python解释器
find_package(PythonInterp REQUIRED) 
//找到Python头文件和库的模块，EXACT关键字，限制CMake检测特定的版本
find_package(PythonLibs ${PYTHON_VERSION_MAJOR}.${PYTHON_VERSION_MINOR} EXACT REQUIRED)
//目标包含python的include目录
target_include_directories(hello-embedded-python
  PRIVATE
      ${PYTHON_INCLUDE_DIRS}
    )
```

### 检测Python模块和包

检测Numpy模块

``` cmake

```

#### 每次更改时复制文件

``` cmake
add_custom_command(
  OUTPUT
      ${CMAKE_CURRENT_BINARY_DIR}/use_numpy.py
  COMMAND
      ${CMAKE_COMMAND} -E copy_if_different ${CMAKE_CURRENT_SOURCE_DIR}/use_numpy.py
      ${CMAKE_CURRENT_BINARY_DIR}/use_numpy.py
  DEPENDS
      ${CMAKE_CURRENT_SOURCE_DIR}/use_numpy.py
  )
```







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



