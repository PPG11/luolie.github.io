---
title: cmake-learn
top: false
cover: false
toc: true
mathjax: true
date: 2021-05-20 15:58:59
password:
summary:
tags:
  - C/C++
categories:
---

## CMake 语法

CMake 语句主要有 3 类用法：

- 设置变量:

  - `set`
  - `file`
  - `list`
  - `find_library`
  - `aux_source_directory`
  - `$<...>`: generator expressions

- 设置`target`: 构建的目标（一般来说就是库或者可执行文件）

  - `add_library`
  - `add_executable`

- 设置`target`的属性: 定义如何生成 target（源文件的路径、编译选项、要链接的库...）
  - `add_definitions`
  - `target_link_libraries`
  - `link_directories`
  - `include_directories`
  - `target_include_directories`

### 预处理

1. `project`

设置项目的名字

```cmake
project(SYSZUXrtp)

# or more parm
project(
    libdeepvac
    LANGUAGES CXX
    VERSION "1.0.0")
```

### 设置变量

1. `set(var content)`

其中 `content` 可以有空格换行等，第一个空格前是变量名

```cmake
set(SYSZUX_HEADERS
    include/detail/class.h
    include/detail/common.h
    include/detail/descr.h
    include/detail/init.h
    include/internals.h
    include/detail/typeid.h)
```

2. `file`

使用正则匹配文件，并将文件路径赋值给第一个参数（为变量）

3. `list`

针对`list`进行各种操作，如增删改查

4. `find_library`

寻找一个库，将找到的库的绝对路径赋值给变量。如

```cmake
find_library(LIBGEMFIELD_PATH libgemfield.so PATHS ${CUDA_TOOLKIT_ROOT_DIR}/lib64/)
```

5. `aux_source_directory(<dir> <var>)`

找到`dir`下所有的源文件赋值给`var`，如

```cmake
aux_source_directory(${gemfield_root}/include gemfield_src)
```

6. `$<...>`: generator expressions

生成表达式，暂略

### 设置`target`

1. `add_library`

```cmake
add_library(<name> [STATIC | SHARED | MODULE]
            [EXCLUDE_FROM_ALL]
            source1 [source2 ...])

# eg
add_library(gemfield_static STATIC ${gemfield_src_list})
```

将一系列库整合命名为`<name>`

2. `add_executable`

```cmake
add_executable(<name> [WIN32] [MACOSX_BUNDLE]
               [EXCLUDE_FROM_ALL]
               source1 [source2 ...])

# eg
add_executable(gemfield_proxy ${gemfield_src_list})
```

将源文件生成可执行文件`<name>`

### 设置`target`属性

1. `add_definitions`

定义一些属性

```cmake
add_definitions(-DENABLE_GEMFIELD)
```

2. `target_link_libraries`

链接

```cmake
target_link_libraries(<target> [item1 [item2 [...]]]
                      [[debug|optimized|general] <item>] ...)

# eg
target_link_libraries(
        gemfield_proxy
        shared_static
        json_static
        mpeg_static
        ${LINK_LIB_LIST})
```

链接`gemfield_proxy`的时候需要有后面的库

## CMake 控制

### `if-else`

```cmake
if (xx AND aa)
# something
elseif(NOT yy)
# something
else(zz)
# something
endif()
```

### `for`

```cmake
foreach(i list_i)
# something
endforeach()
```

## 内置变量

### 系统变量

- `MAKE_MAJOR_VERSION` : major version number for CMake, e.g. the "2" in CMake 2.4.3
- `CMAKE_MINOR_VERSION` : minor version number for CMake, e.g. the "4" in CMake 2.4.3
- `CMAKE_PATCH_VERSION` : patch version number for CMake, e.g. the "3" in CMake 2.4.3
- `CMAKE_TWEAK_VERSION` : tweak version number for CMake, e.g. the "1" in CMake X.X.X.1. Releases use tweak < 20000000 and development versions use the date format CCYYMMDD for the tweak level.
- `CMAKE_VERSION` : The version number combined, eg. 2.8.4.20110222-ged5ba for a Nightly build. or 2.8.4 for a Release build.
- `CMAKE_GENERATOR` : the generator specified on the commandline.
- `BORLAND` : is TRUE on Windows when using a Borland compiler
- `WATCOM` : is TRUE on Windows when using the Open Watcom compiler
- `MSVC, MSVC_IDE, MSVC60, MSVC70, MSVC71, MSVC80, CMAKE_COMPILER_2005, MSVC90, MSVC10 (Visual Studio 2010)` : Microsoft compiler
- `CMAKE_C_COMPILER_ID` : one of "Clang", "GNU", "Intel", or "MSVC". This works even if a compiler wrapper like ccache is used.
- `CMAKE_CXX_COMPILER_ID` : one of "Clang", "GNU", "Intel", or "MSVC". This works even if a compiler wrapper like ccache is used；
- `cmake_minimum_required`：设置所需 CMake 的最小版本；

### 编译相关变量!!

- `CMAKE_CXX_STANDARD`：设置 C++ 标准；
- `CMAKE_CXX_FLAGS`：设置 C++ 编译参数；
- `CMAKE_C_FLAGS`：设置 C 编译参数

```cmake
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -w")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -w")
```

- `BUILD_SHARED_LIBS` : if this is set to ON, then all libraries are built as shared libraries by default. SET(BUILD_SHARED_LIBS ON) ；
- `CMAKE_BUILD_TYPE` : A variable which controls the type of build when using a single-configuration generator like the Makefile generator. It is case-insensitive；If you are using the Makefile generator, you can create your own build type like this:

```cmake
set(CMAKE_BUILD_TYPE distribution)
set(CMAKE_CXX_FLAGS_DISTRIBUTION "-O3")
set(CMAKE_C_FLAGS_DISTRIBUTION "-O3")
```

## 参考文献

### 关键词

https://www.cnblogs.com/binbinjx/p/5626916.html

### 多源文件

https://blog.csdn.net/dyyzlzc/article/details/105189374

### 一些内容

https://elloop.github.io/tools/2016-04-10/learning-cmake-2-commands

https://wangpengcheng.github.io/2019/08/13/learn_cmake/
