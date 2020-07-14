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



---
Power by Internet.
