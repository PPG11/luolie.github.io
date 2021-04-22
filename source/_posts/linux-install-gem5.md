---
title: linux 安装 gem5
top: false
cover: false
toc: true
mathjax: true
date: 2021-04-22 13:53:15
password:
summary:
tags:
categories:
---

官方文档 https://www.gem5.org/documentation/learning_gem5/part1/building/

ubuntu18.05 版本

```
sudo apt install build-essential git m4 scons zlib1g zlib1g-dev libprotobuf-dev protobuf-compiler libprotoc-dev libgoogle-perftools-dev python-dev python
```

最后 编译

```
scons build/X86/gem5.opt -j <NUMBER OF CPUs ON YOUR PLATFORM>
```

### 出现的错误：

1. 权限问题

```
/tmp/tmpi2synp81: 1: util/cpt_upgrader.py: Permission denied
scons: *** [build/X86/sim/tags.cc] Error 126
scons: building terminated because of errors.
```

解决: 给该文件权限 777

```
sudo chmod 777 ./util/cpt_upgrader.py
```

### 参考文献

https://yuchen112358.github.io/2016/03/21/gem5-install/
