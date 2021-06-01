---
title: C++-multiproject-compile
top: false
cover: false
toc: true
mathjax: true
date: 2021-06-01 11:30:53
password:
summary:
tags:
  - C/C++
categories:
---

主要是在使用 SystemC 的时候遇到的，一般 C++ 的文件组织形式都是 `h` 文件放在 `include/` 下，但是在使用 SystemC 时，每个 `.h` 和 `.cpp` 都是成套出现的，这里就想要按照每个 module 一起的组织的方式。

即想组织成如下结构

```
.
├── main.cpp
├── CMakeLists.txt
├── Module1
│   ├── module1.h
│   ├── module1.cpp
│   └── CMakeLists.txt
└── Module2
    ├── module2.h
    ├── module2.cpp
    └── CMakeLists.txt
```

下面开始吧

## 文件组织

```
.
├── main.cpp
├── CMakeLists.txt
├── hello
│   ├── CMakeLists.txt
│   ├── hello.cpp
│   └── hello.h
└── world
    ├── CMakeLists.txt
    ├── world.cpp
    └── world.h
```

## 顶层目录

```C++
// main.cpp
#include "hello.h"
#include "world.h"
#include <stdio.h>

int main() {
    hello();
    world();
    return 0;
}
```

```cmake
# CMakeLists.txt

cmake_minimum_required(VERSION 2.8)
project(helloworld)

# Add the source in project root directory
aux_source_directory(. DIRSRCS)

# Add header file include directories
include_directories(
    ./
    ./hello
    ./world
)

# Add block directories
add_subdirectory(hello)
add_subdirectory(world)

# Target
add_executable(helloworld ${DIRSRCS})
target_link_libraries(helloworld hello world)

```

## `hello/`

```C++
// hello.h
#ifndef HELLO_H
#define HELLO_H

#include <stdio.h>
void hello();

#endif
```

```C++
// hello.cpp
#include "hello.h"
void hello() { printf("hello\n"); }
```

```cmake
# hello/CMakeLists.txt

aux_source_directory(. DIR_HELLO_SRCS)
add_library(hello ${DIR_HELLO_SRCS})
```

## `world/`

```C++
// world.h
#ifndef WORLD_H
#define WORLD_H

#include <stdio.h>
void world();

#endif
```

```C++
// world.cpp
#include "world.h"
void world() { printf("world\n"); }
```

```cmake
# world/CMakeLists.txt

aux_source_directory(. DIR_WORLD_SRCS)
add_library(world ${DIR_WORLD_SRCS})
```

## 原理

其实主要看其中的三个 `CMakeLists.txt` 的文件就可以了

在子模块中的 `CMake` 就是把当前目录下所有文件写入对应名字的`library`中，如`hello`、`world`

然后顶层目录文件需要做三件事

1. `include_directories`：将 `#include` 的目录放进来:
2. `add_subdirectory`：将刚才写好的`library`放进来（找到对应目录下的 sub-library）
3. `target_link_libraries`：将刚才的`library`和主文件链接上

即可
