---
title: CMake学习笔记
tags:
  - CMake
categories:
  - 笔记
katex: false
abbrlink: 43493
date: 2025-02-04 04:15:29
---

由于大部分情况下只有在项目创建时才会写一下CMake，甚至在用了项目模板之后连创建的时候也不需要写，因此老是忘记一些东西要怎么写，故写个笔记方便以后查看。目前只记录一些最常用的函数，后续再逐渐补充，并添加一些教程。

## 特性

- 在CMake中不论Linux还是Windows都使用`/`作为路径分割符

## 常用函数

### 通用函数

- `cmake_minimum_required`
这个函数用来指定脚本所需的CMake最低版本，比如你用到了一些高版本CMake的语法，就可以用这个函数来指定最低版本，一般放在CMakeLists.txt的第一行，比如`cmake_minimum_required(VERSION 3.13)`。

- `project`
指定项目名称，LANGUAGES字段可以指定项目使用的语言，目前支持以下语言：
  - C，C语言
  - CXX，C++
  - ASM，汇编
  - Fortran，Fortran语言
  - CUDA，英伟达的CUDA
  - OBJC，苹果的Objective-C
  - OBJCXX，苹果的Objective-C++
  - ISPC，一种英特尔的自动SIMD编程语言
VERSION字段可以指定项目版本，CMake使用x.y.z这样的版本号，
简单的示例`project(test LANGUAGES CXX C)`。

- `message`
输出信息，可以指定信息类型，支持以下类型
  - STATUS 正常的信息
  - WARNING 输出警告，不会中断脚本
  - AUTHOR_WARNING 只对作者的警告，可以通过命令行`-Wno-dev`关闭
  - SEND_ERROR 错误，会中断构建，但会继续执行脚本
  - FATAL_ERROR 错误，会中断脚本运行
简单示例`message(STATUS "sources: ${sources}")`

- `set`
这个函数用来定义变量，比如`set(sources main.cpp)`，即定义一个sources变量存储main.cpp文件。通过`${sources}`来使用变量。除了自定义变量外，也可以用来改变CMake内置变量的值，比如`set(CMAKE_CXX_STANDARD 20)`，即设置C++标准为C++20，使用CMAKE开头的一般都是CMake的内置变量，用来定义相关的配置。

### 文件相关

- `file`
这个函数用来批量查找文件，比如`file(GLOB sources *.cpp *.h)`，就会搜索所有后缀为cpp和h的文件，并存在sources变量中。把GLOB改为GLOB_RECURSE就会递归的搜索子目录。在大型项目中一般不使用，因为不会自动添加文件，如果新增了文件需要重新配置cmake，并且文件数量过多时可能导致性能问题，当然个人项目还是很方便的，可以使用CONFIGURE_DEPENDS选项来自动更新文件列表，比如`file(GLOB sources CONFIGURE_DEPENDS *.cpp)`。

### 库相关

- `add_library`
添加一个库，可以选择静态库还是动态库。
简单示例

```cmake
add_library(mylib STATIC mylib.cpp) # 静态库
add_library(mylib SHARED mylib.cpp) # 动态库
```

除了常规的静态库动态库外，CMake还可以使用一种特殊的对象库，对象库和静态库类似，但是不生成静态库文件，只是CMake内部组织代码的一种方式，可以认为是一种逻辑上的库。使用方式`add_library(mylib OBJECT mylib.cpp)`。

- `find_package`
查找指定库文件并生成伪对象，可以用`COMPONENTS`指定组件，`REQUIRED`指定找不到时报错。在linux下，这个函数会在`/usr/lib/cmake`目录下寻找对应库的配置文件（{包名}Config.cmake，例如ClangConfig.cmake），生成伪对象，包含了include路径等信息，在链接时这些配置会扩散到构建目标上，无需单独配置。在Windows上则需要额外的配置，推荐的做法是定义`{包名}_DIR`的变量指向配置文件所在路径，比如定义`Qt5_DIR`指向`D:/Qt/5.15.2/msvc2019_64/lib/cmake`。对于Qt这种有多个组件的库，必须指定组件。
简单示例

```cmake
find_package(Qt5 REQUIRED COMPONENTS Core Gui Widgets)
add_executable(test main.cpp mainwindow.h mainwindow.cpp)
target_link_libraries(test PUBLIC Qt5::Core Qt5::Gui Qt5::Widgets)
```

### 构建相关

- `add_executable`
这个函数用来添加一个可执行文件作为构建目标，比如`add_executable(main main.cpp)`，就是使用main.cpp文件构建main这个可执行文件，可以用空格分割多个文件。

### target_ 相关函数

这个系列的函数都是作用与一个对象

- `target_sources`
给目标添加源文件，简单示例

```cmake
add_executable(main)
target_sources(main main.cpp)
```

- `target_link_libraries`
给目标链接库。

## 命令行

- `cmake -B`
指定build目录，一般用法为`cmake -B build`，会自动创建build目录并生成项目文件

- `cmake --build`
开始构建，一般用法为`cmake --build build`

- `cmake -D`
配置CMake变量，比如`cmake -DCMAKE_BUILD_TYPE=Release`，即设置构建类型为release。-D指令设置的是缓存变量，也就是说会写入CMakeCache.txt，下一次执行cmake命令时就不需要再次设置。

- `cmake -G`
指定生成器，比如`cmake -GNinja`，若不指定，则在Linux上默认使用Makefile，Windows上默认使用MSBuild。由于Ninja跨平台，性能好，一般都使用Ninja。

## 内置变量

- `CMAKE_BUILD_TYPE`
指定构建类型，有以下四种：

  - Debug 调试模式，包含调试信息
  - Release 发布模式，优化程度最高
  - MinSizeRel 最小体积发布，生成文件较Release小，节省空间
  - RelWithDebInfo 带调试信息的发布，较Release比多了调试信息

默认空字符串，此时为Debug模式。

- `CMAKE_CURRENT_SOURCE_DIR`
当前CMakeLists.txt所在的路径

- `CMAKE_CURRENT_BINARY_DIR`
当前输出目录的位置

- `PROJECT_SOURCE_DIR`
当前项目的源文件路径，也就是使用了`project`的CMakeLists.txt所在的路径。

- `PROJECT_NAME`
当前项目名

- `PROJECT_VERSION`
当前项目版本号

- `CMAKE_CXX_STANDARD`
C++标准版本

- `CMAKE_CXX_STANDARD_REQUIRED`
BOOL类型的变量，表示是否一定要支持设定的C++标准，若设置为ON则会在编译器不支持时报错。

- `CMAKE_CXX_EXTENSIONS`
是否支持编译器扩展，比如GCC支持C99的一些特殊写法，但这些写法在标准C++里是没有的，如果要兼容性就会设置为OFF。

## 基本使用

一个简单的CMake脚本主要由以下几个部分组成

1. 配置基本设置，如cmake版本要求，C++标准，项目名等
2. 寻找三方库
3. 收集源文件
4. 添加构建目标
5. 链接库
