---
title: macos-install-systemc
top: false
cover: false
toc: true
mathjax: true
date: 2021-04-26 17:15:23
password:
summary:
tags:
  - systemC
categories:
---

尝试在 mac 安装 systemC，可能因为有安装 gem5 的经历，导致对 mac 完全不是 linux 这件事有了深刻的心理阴影。所以在安装 systemC 的时候潜意识也会觉得有一堆问题甚至到最后发现完全不能安装（为此可能也潜意识放弃了很多尝试解决的办法）。今天意外安装成功了，特此记录一下。

### CXX 环境问题

其实最关键的原因就是 mac 默认的 C 编译环境是 clang （无论是输入 `gcc` 还是 `clang` 结果都是使用 `clang`，如果要用 `brew` 安装的 `gcc`，具体需要用 `g++-10` 之类的命令）

## 安装 systemc

### 安装 `gcc` 环境

我此时的版本是 `gcc version 10.2.0 (Homebrew GCC 10.2.0_4)` 但是我觉得应该各个版本都可以

```
brew install gcc
```

但是注意这个时候输入 `gcc` 或者 `g++` 都还是 `clang`，要想使用 `gcc` 应该用 `gcc-10` 或者 `g++-10`

然后设置 `make` 中的 `CXX` 环境

```
export CXX=g++-10
```

### 官网下载 systemc 安装包

https://www.accellera.org/downloads/standards/systemc

我下载的是 SystemC 2.3.3 (Includes TLM)，其中的 zip 和 tar.gz 都可以。

然后解压缩到以后都会保存到的位置，比如我个人习惯在

```
/Users/luolie/Documents/systemc-2.3.3
```

### 新建配置目录

进入相关目录下 (systemc-2.3.3)

```
cd systemc-2.3.3
```

新建配置临时目录

```
mkdir objdir
cd objdir
```

### 进行相关配置

主要是配置 `../configure [option]`

这里的 `[option]` 建议加

1.  `--with-arch-suffix=` : 要不然生成的文件会是 `lib-macosx64` 这样的，如果加上这个参数，则直接就是 `lib`
2.  还可以设置一下以下参数

```
  --disable-shared        do not build shared library (libsystemc.so)
  --enable-debug          include debugging symbols
  --disable-optimize      disable compiler optimization
  --disable-async-updates disable request_async_update support
  --enable-pthreads       use POSIX threads for SystemC processes
  --enable-phase-callbacks
                          enable simulation phase callbacks (experimental)
```

### make

然后 `make` 就好啦，中间会有一些 c 文件的问题，但是不是 error。

```bash
make
make check // 运行example文件检查一下
make install
```

然后可以 `rm -rf objdir` 移除临时目录，但个人建议不要，这个可以留着以后 `make uninstall`

### 自己测试一下

#### 编写代码

随便找个位置（不一定在 systemc 目录）编写如下代码

```c++
// hello.h

#include "systemc.h"

// Hello_world is module name
SC_MODULE(hello_world) {
    SC_CTOR(hello_world){
        // Nothing in constructor
    }
    void say_hello() {
        //Print "Hello World" to the console.
        cout << "Hello World.\n";
    }
};
```

```c++
// hello.cpp
#include "hello.h"

// sc_main in top level function like in C++ main
int sc_main(int argc, char *argv[]){
  hello_world hello("HELLO");
  // Print the hello world
  hello.say_hello();
  return (0);
}
```

#### 测试

1. 首先添加一个 `bash` 的全局环境变量`$SYSTEMC_HOME`（这里用 `export` 但实际使用建议写入 bash）

```bash
# export SYSTEMC_HOME=path/to/systemc-2.3.3
# eg.
export SYSTEMC_HOME=/Users/conflux/Downloads/systemc-2.3.3
echo $SYSTEMC_HOME
```

2. 编译（后面可以为此做一个 make 或者 scons）

```bash
g++-10 hello.cpp -o hello.o -L $SYSTEMC_HOME/lib -I $SYSTEMC_HOME/include -l systemc
```

这里逐个参数讲解一下

- g++-10

3. 运行一下

```bash
./hello.o
```

输出

```
        SystemC 2.3.3-Accellera --- Apr 26 2021 16:46:59
        Copyright (c) 1996-2018 by all Contributors,
        ALL RIGHTS RESERVED
Hello World.
```

耶！

## 参考文献

https://github.com/accellera-official/systemc/blob/master/INSTALL.md

https://stackoverflow.com/questions/25961573/how-to-use-and-install-systemc-in-terminal-mac-os-x
