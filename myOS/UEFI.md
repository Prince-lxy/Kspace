# UEFI

> 全称“统一的可扩展固件接口”(Unified Extensible Firmware Interface)，是一种 PC 固件的体系结构、接口和服务的标准，其主要目的是为了提供一组在 OS 加载之前（启动前）在所有平台上一致的、正确指定的启动服务，被看做是有近20多年历史的 BIOS 的继任者。

## Contents / Mind mapping
- **1 特性**
- **2 结构**
- **3 UEFI 启动过程**
- **4 接口**

---

### 1 特性

- 分区各式：BIOS 采用 MBR，UEFI 采用 GPT。
- 编码99%都是由C语言完成。
- 一改之前的中断、硬件端口操作的方法，而采用了Driver/protocol的新方式。
- 将不支持X86实模式，而直接采用Flat mode。
- 输出也不再是单纯的二进制code，改为Removable Binary Drivers。
- OS启动不再是调用Int19，而是直接利用protocol/device Path。
- 对于第三方的开发，前者基本上做不到，除非参与BIOS的设计，但是还要受到ROM的大小限制，而后者就便利多了。
- 弥补BIOS对新硬件的支持不足的问题。
- UEFI 体系驱动并不是由直接运行在 CPU 上的代码组成的，而是用 EFI Byte Code 编写而成的，类似于 Java，具有良好的向下兼容性。

### 2 结构

- UEFI Image
  - UEFI 应用：启动管理、UEFI Shell、诊断、调试、调度和供应。
  - OS Loader：启动操作系统的特殊应用。
  - UEFI 驱动：提供设备间的接口协议。
- 平台初始化框架
  - PEI：EFI 预初始化，主存储器初始化模块、检测和加载驱动执行环境核心。
  - DXE：驱动执行环境，提供了设备驱动和协议接口环境界面。

### 3 UEFI 启动过程

- 平台初始化
- EFI Image 加载
  - EFI 驱动
  - EFI APP
- EFI OS 加载
- 操作系统加载

### 4 接口

- 启动服务
  - 事件服务
  - 内存管理
  - 协议管理
  - 协议使用类服务
  - 驱动管理
  - Image 管理
  - ExitBootServices 服务
- 运行时服务
  - 时间服务
  - 读写 UEFI 系统变量服务
  - 虚拟内存服务
  - 重启系统服务



---
Power by Internet.
