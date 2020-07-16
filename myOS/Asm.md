# Asm

> 汇编。

## Contents / Mind mapping
- **1 概述**
- **2 语法**

---

### 1 概述

最接近机器语言的程序语言。



### 2 语法

> .equ

功能：定义常量，类似于 #define

```
.equ	NUM	0x1234
```

> .text

功能：声明代码段起始位置。

> .align

功能：声明字节对齐宽度。

```
.align 4	# 表示对齐宽度为4个字节，不足 4 字节部分会自动补齐。
```

> .thumb

功能：声明

> .thumb

功能：声明接下来的指令都为 thumb 指令。

> .syntax unified

功能：可以让我们使用 thumb 指令时不用区分其版本。

> .type xxx %function

功能：声明 xxx 为一个函数。

> cpsid i(IRQ)/f(FIQ)

功能：屏蔽可配置优先级的中断。

> cpsie i(IRQ)/f(FIQ)

功能：开启可配置优先级的中断。

> push {r4, r5}

功能：把寄存器 r4 r5 压栈。

> pop {r4, r5}

功能：把栈顶的内容出栈并存储到寄存器 r4 r5 中。

> ldr r0, =50		;; r0 <- 50
> ldr r0, [r1, #16]	;; r0 <- * (r1 + 16)
> ldr r0, [r1], #16	;; r0 <- * (r1); r1 <- r1 + 16;

功能：字数据加载指令。

> ldm r0, {r4, r5}	;; r0[0] -> r4; r0[1] -> r5;

功能：连续字数据加载指令。

> str r1, [r2]		;; r1 -> * r2
> str r5, [r4, #4]	;; r5 -> * (r4 + 4)
> str r5, [r4], #4	;; r5 -> * r4; r4 <- r4 + 4

功能：字数据存储到指令（register to memory)

> stm r0, {r4, r5}	;; r0[0] <- * r4; r0[1] <- * r5;

功能：连续字数据存储指令（registers to memory)

> add r1, r1, #1	;; r1 <- r1 + 1

功能：加法指令

> sub r1, r1, #1	;; r1 <- r1 - 1

功能：减法指令

> mov r1, #0		;; r1 <- 0

功能：赋值指令

> msr primask, r1	;; primask <- r1

功能：mov r1 to 程序状态寄存器 primask 域。

> mrs r1, psp		;; r1 <- psp

功能：mov process stack pointer to r1 寄存器。

> bx 目标地址

功能：跳转指令

> cbz

功能：为零跳转。

> orr lr, lr, #0x0	;; lr <- lr | 0x0;

功能：逻辑或指令



---
Power by Internet.
