---
title: systemC 值保持器
top: false
cover: false
toc: true
mathjax: true
date: 2021-05-24 18:23:31
password:
summary:
tags:
  - systemC
categories:
---

## 在 `systemC` 中变量分三种

- 变量(variable)：普通 C 变量
- 信号(signal)：组件内连接（`sc_signal`）
- 端口(port)：I/O 口（`sc_in / sc_out`）

注意这里也有“变量”，所以在 `systemC` 中一般把常规意义上的“变量”称呼为“值保持器”

```C++
// 1. 变量
// type v1, v2;
int v1;

// 2. 信号
// sc_signal<type> s1, s2;
sc_signal<bool> s1;

// 3. 端口
// sc_in<type>, sc_out<type>, sc_inout<type>
sc_in<bool>
```

注意这里信号和端口都是用来描述硬件结构的，所有想要对于“值保持器”的操作都应该传递给“变量”后再操作，具体见下节。

所有的值保持器都可以作为 C 语法中的“变量”，使用数组、指针等。

> 在 SC 中，特色的可作为左值和右值的有五种：
>
> - 变量
> - 信号
> - 端口
> - 位选取结果 `[]`
> - 位区间选取结果 `.range()`

## 位选取`[]`和位区间选取`.range()`

```c++
sc_signal<sc_bv<4>> dval;
sc_in<sc_bv<8>> addr;

sc_bv<4> var_dval;
sc_bv<8> var_addr;
bool ready;

// ...

// for sc_in port, need to use read()
var_addr = addr.read();
// do something, eg:
ready = var_addr[2];

var_dval = dval;
var_dval.range(2, 0) = "011";
dval = var_dval;
```

还有一种比较有特色的分配方式，只有当是位向量 vector 类型时（如 sc_bv 或者 sc_lv 等）

```c++
sc_bv<8> ctrl_bus;
sc_bv<4> mult;

mult = (ctrl_bus[0], ctrl_bus[2], ctrl_bus[0], ctrl_but[2])
```

## 位 / 逻辑类型

> 此类型只能做逻辑运算与或非，不能做算数运算

位类型 `('1' / '0')`

```c++
// 1 bit
bool v1;
// vector
sc_bv<4> v2;
```

逻辑类型 `('1' / '0' / 'X' / 'Z')`

```C++
sc_logic_0 // '0'
sc_logic_1 // '1'
sc_logic_X // 'X'
sc_logic_Z // 'Z'
```

```c++
// 1 bit
sc_logic v1;
// vector
sc_lv<8> v2;
```

## 有/无符号整形
