---
title: cmake
date: 2023-09-20 20:22:53
tags:
---

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
