# STM32

> STM32 微控制器

## Contents / Mind mapping
- **1 概述**
- **2 系统架构**
- **3 存储器**

---

### 1 概述

STM32系列是专为要求高性能、低成本、低功耗的嵌入式应用设计的 ARM Cortex-M 架构的 32 位微控制器。



### 2 系统架构

系统包括：

- 驱动单元
  - ICode 总线(I-bus)
  - DCode 总线(D-bus)
  - 系统总线(S-bus)
  - GP-DMA(通用 DMA)
- 被动单元
  - 内部 SRAM
  - 内部闪存存储器
  - AHB 到 APB 的桥(AHB2APBx)，它连接所有的 APB 设备



### 3 存储器

程序存储器、数据存储器、寄存器和输入输出端口被组织在同一个 4GB 的线性地址空间内。



---
Power by Internet.
