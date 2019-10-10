# UEFI

> 全称“统一的可扩展固件接口”(Unified Extensible Firmware Interface)，是一种 PC 固件的体系结构、接口和服务的标准，其主要目的是为了提供一组在 OS 加载之前（启动前）在所有平台上一致的、正确指定的启动服务，被看做是有近20多年历史的 BIOS 的继任者。

## Contents / Mind mapping
- **1 特性**
- **2 结构**
- **3 接口**
- **4 UEFI 启动过程**
- **5 UEFI 开发环境搭建**
  - **5.1 EDK2 Linux 开发环境**

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
- UEFI 提供了异步操作。基于事件的异步操作,提高了 CPU 利用率,减少了总的等待时间。
- UEFI 舍弃了中断这种比较耗时的操作外部设备的方式,仅仅保留了时钟中断。外部设备的操作采用“事件+异步操作”完成。
- 可伸缩的遍历设备的方式,启动时可以仅仅遍历启动所需的设备,从而加速系统启动。
- 当系统的安全启动功能被打开后,UEFI在执行应用程序和驱动前会先检测程序和驱动的证书,仅当证书被信任时才会执行这个应用。



### 2 结构

- UEFI Image
  - UEFI 应用：启动管理、UEFI Shell、诊断、调试、调度和供应。
  - OS Loader：启动操作系统的特殊应用。
  - UEFI 驱动：提供设备间的接口协议。
- 平台初始化框架
  - PEI：EFI 预初始化，主存储器初始化模块、检测和加载驱动执行环境核心。
  - DXE：驱动执行环境，提供了设备驱动和协议接口环境界面。



### 3 接口

- 启动服务 (Boot Services, BS)
  - 事件服务：事件是异步操作的基础。有了事件的支持,才可以在 UEFI 系统内执行并发操作。
  - 内存管理：主要提供内存的分配与释放服务,管理系统内存映射。
  - 协议管理：提供了安装 Protocol 与卸载 Protocol 的服务,以及注册 Protocol 通知函数(该函数在 Protocol 安装时调用)的服务。
  - 协议使用类服务：Protocol 使用类服务:包括 Protocol 的打开与关闭,查找支持 Protocol 的控制器。
  - 驱动管理：包括用于将驱动安装到控制器的 connect 服务,以及将驱动从控制器上卸载的 disconnect 服务。
  - Image 管理：此类服务包括加载、卸载、启动和退出 UEFI 应用程序或驱动。
  - ExitBootServices 服务：用于结束启动服务。
- 运行时服务 (Runtime Service, RT)
  - 时间服务：读取 / 设定系统时间。读取 / 设定系统从睡眠中唤醒的时间。
  - 读写 UEFI 系统变量服务：读取 / 设置系统变量,例如 BootOrder 用于指定启动项顺序。通过这些系统变量可以保存系统配置。
  - 虚拟内存服务：将物理地址转换为虚拟地址。
  - 重启系统服务：包括重启系统的 ResetSystem,获取系统提供的下一个单调单增值等。



### 4 UEFI 启动过程

- SEC (安全验证)
  - 接收并处理系统启动和重启信号:系统加电信号、系统重启信号、系统运行过程中的严重异常信号。
  - 初始化临时存储区域。
  - 递系统参数给下一阶段(即 PEI)。
- PEI (EFI 前期初始化)
  - 为 DXE 准备执行环境,将需要传递到 DXE 的信息组成 HOB(Handoff Block)列表,最终将控制权转交到 DXE 手中。
- DXE (驱动执行环境)
  - 根据 HOB 列表内容初始化系统服务。
  - 调度系统中所有的 Driver。
- BDS (启动设备选择)
  - 初始化控制台设备。
  - 加载必要的设备驱动。
  - 根据系统设置加载和执行启动项。
- TSL (操作系统加载前期)
  - 从操作系统加载器 (OS Loader) 被加载, 到 OS Loader 执行 ExitBootServices() 的这段时间。
- RT (Run Time)
  - 系统进入 RT(Run Time) 阶段后,系统的控制权从 UEFI 内核转交到 OS Loader 手中,随着 OS Loader 的执行,OS 最终取得对系统的控制权。
- AL (系统灾难恢复期)
  - 在 RT 阶段,如果系统(硬件或软件)遇到灾难性错误,系统固件需要提供错误处理和灾难恢复机制,这种机制运行在 AL(After Life)阶段。



### 5 UEFI 开发环境搭建

UEFI 是一种标准,它没有给出具体的实现。软件厂商可以根据 UEFI 标准开发自己的 UEFI 实现,其中常用的开源实现是 EDK2。

#### 5.1 EDK2 Linux 开发环境

1. 安装必要的工具包

```
$ sudo apt-get install build-essential uuid-dev iasl git gcc-5 nasm
```

2. 下载 EDK2 源码

```
$ git clone https://github.com/tianocore/edk2.git
```

3. 编译 BaseTools

```
$ make -C BaseTools
```

4. 建立 build 环境

```
$ source edksetup.sh
```

5. 编译指定模块

```
$ build -a [IA32|X64|ARM|IPC|EBC] -p xxxPkg/xxxPkg.dsc -m xxxPkg/xxxx/xxxx.inf
```

编译后的文件放在 Build 目录下。

6. 运行模拟器

```
$ build run
```



---
Power by Internet.
