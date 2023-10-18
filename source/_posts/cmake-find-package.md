---
title: cmake find_package详解
date: 2023-09-22 15:42:51
tags: "cmake"
---

# 



1. `find_package`在一些目录中查找OpenCV的配置文件。
2. 找到后，`find_package`会将头文件目录设置到`${OpenCV_INCLUDE_DIRS}`中，将链接库设置到`${OpenCV_LIBS}`中。
3. 设置可执行文件的链接库和头文件目录，编译文件。



# find_package找什么

**Moudle**模式下找的是`Find<PackageName>.cmake`

**Config**模式下是要查找名为`<PackageName>Config.cmake`或`<package-name>-config.cmake`

管使用哪一种模式，只要找到`*.cmake`，`*.cmake`里面都会定义下面这些变量：

```cmake
<NAME>_FOUND	#是否找到
<NAME>_INCLUDE_DIRS or <NAME>_INCLUDES	#include目录
<NAME>_LIBRARIES or <NAME>_LIBRARIES or <NAME>_LIBS	#lib目录
<NAME>_DEFINITIONS
```



# 搜索路径

Moudle模式：

1. [`CMAKE_FIND_PACKAGE_REDIRECTS_DIR`](https://cmake.org/cmake/help/latest/variable/CMAKE_FIND_PACKAGE_REDIRECTS_DIR.html#variable:CMAKE_FIND_PACKAGE_REDIRECTS_DIR)
1. `CMAKE_ROOT`：cmake安装目录

如果Moudle模式下没有找到，则切换到Configure模式寻找

Configure模式：

这些目录要么是根目录，要么是非跟目录

1. 名为`<PackageName>_DIR`的CMake变量或环境变量路径，默认为空。非根目录
2. 名为`CMAKE_PREFIX_PATH`、`CMAKE_FRAMEWORK_PATH`、`CMAKE_APPBUNDLE_PATH`的CMake变量或**环境变量**路径。默认都为空。 根目录
3. `PATH`环境变量路径，默认为系统环境`PATH`环境变量值。根目录

> 遍历`PATH`环境变量中的各路径，如果该路径如果以bin或sbin结尾，则**自动回退到上一级目录**得到根目录。这使得绝大部分我们直接通过`apt-get`安装的库可以被找到。

上述指明的是根目录路径时，CMake会首先检查这些根目录路径下是否有名为<PackageName>Config.cmake或<package-name>-config.cmake的模块文件，如果没有，CMake会继续检查或匹配这些根目录下的以下路径（<PackageName>_DIR路径不是根目录路径）：

```
<prefix>/(lib/<arch>|lib|share)/cmake/<name>*/
<prefix>/(lib/<arch>|lib|share)/<name>*/ 
<prefix>/(lib/<arch>|lib|share)/<name>*/(cmake|CMake)/
```

整个`(lib/<arch>|lib|share)`为可选路径。其中<arch>为系统架构名，如Ubuntu下一般为：`/usr/lib/x86_64-linux-gnu`

name为包名，不区分大小写`<name>*`意思是包名后接一些版本后等字符也是合法的。





现在回过头来看查找路径的根目录。我认为最重要的一个是`PATH`。由于`/usr/bin/`在`PATH`中，cmake会自动去`/usr/(lib/<arch>|lib|share)/cmake/<name>*/`寻找模块，



