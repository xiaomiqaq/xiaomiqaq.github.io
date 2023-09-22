---
title: [cmake] find_package详解
date: 2023-09-22 15:42:51
tags: "cmake"
---

# 流程



1. `find_package`在一些目录中查找OpenCV的配置文件。
2. 找到后，`find_package`会将头文件目录设置到`${OpenCV_INCLUDE_DIRS}`中，将链接库设置到`${OpenCV_LIBS}`中。
3. 设置可执行文件的链接库和头文件目录，编译文件。

# 搜索路径

配置模式

1. [`CMAKE_FIND_PACKAGE_REDIRECTS_DIR`](https://cmake.org/cmake/help/latest/variable/CMAKE_FIND_PACKAGE_REDIRECTS_DIR.html#variable:CMAKE_FIND_PACKAGE_REDIRECTS_DIR)





现在回过头来看查找路径的根目录。我认为最重要的一个是`PATH`。由于`/usr/bin/`在`PATH`中，cmake会自动去`/usr/(lib/<arch>|lib|share)/cmake/<name>*/`寻找模块，这使得绝大部分我们直接通过`apt-get`安装的库可以被找到。



不管使用哪一种模式，只要找到`*.cmake`，`*.cmake`里面都会定义下面这些变量：

```cmake
<NAME>_FOUND
<NAME>_INCLUDE_DIRS or <NAME>_INCLUDES
<NAME>_LIBRARIES or <NAME>_LIBRARIES or <NAME>_LIBS
<NAME>_DEFINITIONS
```

