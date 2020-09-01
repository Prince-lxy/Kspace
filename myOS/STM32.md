# STM32

> STM32F103ZET6

## Contents / Mind mapping
- **1 平台简介**
- **2 硬件资源**
- **3 最小系统**
- **4 开发工具**
  - **4-1 MDK5**
  - **4-2 FlyMcu**

---

### 1 平台简介

STM32系列是专为要求高性能、低成本、低功耗的嵌入式应用设计的 ARM Cortex-M 架构的 32 位微控制器。



### 2 硬件资源

根据芯片名称参考芯片选型手册，大致了解芯片的硬件资源。

- 144 引脚，112 IO口（除模拟通道外，大部 IO 口都耐 5V）
- 支持调试：SWD（只需要两根数据线），JTAG
- 512K FLASH
- 64K SRAM
- 2.0 ~ 3.6V 电源
- 时钟
  - 4 ~ 16M 的外部高速晶振，通常使用 8M
  - 内部 8M 高速 RC 振荡器
  - 内部锁相环（PLL，将外部晶振倍频后提供给内部芯片使用）
  - 外部低速 32.768K 晶振，主要做 RTC 时钟源
- 低功耗
  - 支持睡眠、停止、待机
  - 可用电池为 RTC 和备份寄存器供电
- AD 模数转换
  - 3 个 12 位的 AD，最多 21 个外部测量通道
  - 转换范围 0 ~ 3.6V
  - 内部通道可以测量温度
  - 内置参考电压
- DA 数模转换
  - 2 个 12 位 DA
- DMA
  - 7 通道 DMA1 和 5 通道 DMA2
  - 支持 ADC，DAC，SDIO，I2C，SPI，I2S，USART
- 定时器
  - 4 个通用定时器
  - 2 个基本定时器
  - 2 个高级定时器
  - 1 个系统定时器
  - 2 个看门狗定时器
- 通信接口
  - 2 个 I2C
  - 5 个 USART
  - 3 个 SPI
  - 1 个 CAN2.0
  - 1 个 USB FS
  - 1 个 SDIO



### 3 最小系统

- 供电
- 复位
- 时钟：2 个外部晶振
- Boot 启动选择模式
- 下载电路（串口，JTAG，SWD）
- 后备电池



### 4 开发工具

#### 4-1 MDK5

MDK5 源自德国的 KEIL 公司，是目前针对 ARM 处理器，尤其是 Cortex M 内核处理器的最佳开发工具。

**MDK5 组成**：
- MDK5 核心
  - 编辑器
  - C/C++ 编译器
  - 包安装器
  - 调试器
- MDK5 插件包
  - Cortex 软件接口标准
  - 硬件设备支持
  - 驱动

**MDK5 安装**：
- MDK5 下载：http://www.keil.com/demo/eval/arm.htm
- 软件包下载： http://www.keil.com/dd2/pack
- STM32F1 支持包：Keil.STM32F1xx_DFP.1.0.5.pack

**MDK5 建立工程**：
- 建立工程文件夹目录和以下子目录
```
- <Project name>
  - CORE		# 存放芯片初始化汇编文件
  - SYS			# 存放常用的 delay sys uart 函数
  - LIB			# 存放外设库函数
  - OBJ			# 存放编译时产生的中间文件
  - USER		# 存放 main 函数
```
- 打开 MDK5 新建工程，在工程文件夹中建立同名工程文件，选择芯片 `STM32F103ZE`
- 根据真实的目录结构调整 MDK5 中 Target1 的目录树结构（此处跳过 OBJ目录和自动生成的 Listings 和 Objects 目录）
- 设置工程
  - Output
    - 勾选 `Create HEX File`
    - 选择 Objects 文件存放目录： `OBJ`
  - Listing
    - 选择 Listing 文件存放目录： `OBJ`
  - C/C++
    - 填写处理器宏定义：`STM32F10X_HD`
    - 添加头文件目录
  - 点击 OK 保存设定
- 删除 MDK5 自动生成的 Listings 和 Objects 目录（旧版本 MDK 不自动生成，出于兼容性考虑），功能与我们自己创建的 OBJ 相同。
- 添加 main 函数

**MDK5 调试**：
- MDK5 软件仿真调试
  - 步骤
    - Options for Target 的 Debug 选项卡
    - 勾选 `Use Simulator` 和 `Run to main()`
    - 设置 Dialog DLL 为 `DARMATM.DLL`
    - 设置 Dialog Parameter 为 `-p STM32F103ZE`
  - 注意
    - 模拟信号监控只能用于此模式
    - UART 监控只能用于此模式
- MDK5 ST-Link 硬件仿真调试
  - 步骤
    - Options for Target 的 Debug 选项卡
      - 勾选 `Use ST-Link Debugger` 和 `Run to main()`
      - 设置 Dialog DLL 为 `TARMATM.DLL`
      - 设置 Dialog Parameter 为 `-p STM32F103ZE`
      - 通过 `Use ST-Link Debugger` 右边的 Settings 设置调试模式 SW/JTAG
    - Options for Target 的 Utilities 选项卡
      - 勾选 `Use Debug Driver`
      - `Use Debug Driver` 左下方的 Settings 中勾选 `Reset and Run`
  - 注意
    - 串口监控需要使用第三方串口软件来实现，例如 XCOM

#### 4-2 FlyMcu

功能：通过 USB 串口下载程序到开发板

- 手动下载步骤
  - RXD 接 PA9，TXD 接 PA10
  - BOOT0 接 3.3V，BOOT1 接 GND，按下 reset 键开始下载代码
  - BOOT0 接 GND，BOOT1 接 GND，按下 reset 启动程序
- 一键下载
  - 搜索串口，设置波特率为 460800
  - 选择需要下载的 Hex 文件
  - 勾选`编程前重装文件`，`编程后执行`
  - 选择`DTR 低电平复位，RTS 高电平进 BootLoader`
  - 点击`开始编程`



---
Power by Internet.
