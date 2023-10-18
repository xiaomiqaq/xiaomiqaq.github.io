---
title: cmake-ctest
date: 2023-09-26 10:54:39
tags: "cmake" 
---

# ctest

可以自定义测试命令，可以以任何编程语言运行测试集。CTest关心的是，通过命令的返回码测试用例是否通过。CTest遵循的标准约定是，**返回零意味着成功，非零返回意味着失败**。可以返回零或非零的脚本，都可以做测试用例。

测试的具体信息会输出在`build/test/Temporary/lasttestsfailure.log`
