# stlink

> STM32 设备的程序下载器。

## Contents / Mind mapping
- **1 概述**
- **2 安装**
  - **2-1 Linux**

---

### 1 概述

STLink 是一个开源工具集，用于对 STMicroelectronics 制造的 STM32 设备和电路板进行编程和调试。它支持几个所谓的STLINK程序员板（及其克隆），它们使用微控制器芯片将命令从USB转换为JTAG/SWD。



### 2 安装

#### 2-1 Linux

- 二进制安装

```
$ sudo apt install stlink-tools
```

- 源码安装
```
$ git clone https://github.com/stlink-org/stlink.git
$ mkdir build && cd build
$ cmake ..
$ make
$ sudo make install
```



---
Power by Internet.
