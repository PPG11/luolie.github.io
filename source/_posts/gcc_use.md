---
title: gcc使用
top: false
cover: false
toc: true
mathjax: true
date: 2019-03-11 10:09:29
password:
summary:
tags:
  - C/C++
categories:
  - [编程语言, C/C++]
---

## gcc

```
[text] C program (p1.c p2.c)
    |
    |  [Compiler] (gcc -Og -S)
    V
[text] Asm program (p1.s p2.s)
    |
    |  [Assembler] (gcc / as)
    V
[binary] Object program (p1.o p2.o)
    |
    |  [Linker] (gcc / ld)
    V  {add static libraries .a}
[binary] Executable program (p)
```

```
-S : generate asm file .s (or will generate .out)
-Og: basic optimization
```

just like this

```c
// sum.c
long plus(long x, long y);

void sumstore(long x, long y, long *dest)
{
    long t = plus(x, y);
    *dest = t;
}
```

```asm
<!-- sum.s -->
pushq	%rbx
movq	%rdx, %rbx
call	plus@PLT
movq	%rax, (%rbx)
popq	%rbx
ret
```

disassemble

```shell
objdump -d sum.o > sum.d
```

output to `sum.d`

## reg

```asm
parm1: %rdi
parm2: %rsi
parm3: %rdx

ret: %rax
```

## instruction

```
movzbl: move with zero extension byte to long
```

> for `x86-64`:
> any computation where the result is a 32-bit result. it will add zero to the remaining 32 bits of the register (other just like 16-bit or 8-bit will not)

## call func & local variable

`caller saved` and `callee saved`

```
%rax: return value

%rdi
%rsi
%rdx
%rcx
%r8
%r9: 6 reg used to passing arguments

%r10
%r11: caller saved temporaries

%rbx
%r12
%r13
%r14: callee must save

%rbp: callee saved used as frame ptr

%rsp: callee save, restored to original value on exit from procedure
```
