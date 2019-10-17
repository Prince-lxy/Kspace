# UEFI

> 全称“统一的可扩展固件接口”(Unified Extensible Firmware Interface)，是一种 PC 固件的体系结构、接口和服务的标准，其主要目的是为了提供一组在 OS 加载之前（启动前）在所有平台上一致的、正确指定的启动服务，被看做是有近20多年历史的 BIOS 的继任者。

## Contents / Mind mapping
- **1 特性**
- **2 结构**
- **3 内容**
- **4 UEFI 启动过程**
- **5 UEFI 开发环境搭建**
  - **5.1 EDK2 Linux 开发环境**
  - **5.2 OVMF 开发环境**

---

### 1 特性

| VS | BIOS | UEFI |
| -- | ---- | ---- |
| 可识别硬盘分区格式 | MBR( BIOS 硬盘采用 32 位地址,因而引导扇区的最大逻辑块地址是 2^32 , 换算成字节地址, 即 2^32×512=2TB ) | GPT |
| 编码格式 | 主要是汇编语言。 | 编码99%都是由C语言完成。 |
| 性能 | BIOS 基本输入 / 输出服务需要通过中断来完成,开销大,并且BIOS 没有提供异步工作模式,大量的时间消耗在等待上。 | UEFI 提供了异步操作。基于事件的异步操作,提高了 CPU 利用率,减少了总的等待时间。UEFI 舍弃了中断这种比较耗时的操作外部设备的方式,仅仅保留了时钟中断。外部设备的操作采用“事件+异步操作”完成。 |
| 扩展性 | BIOS 代码采用静态链接,增加硬件功能时,必须将 16 位代码放置在 0x0C0000 ~ 0x0DFFFF 区间,初始化时将其设置为约定的中断处理程序。而且 BIOS 没有提供动态加载设备驱动的方案。 | 具有良好的拓展性，驱动开发灵活。 |
| 安全性 | BIOS 运行过程中对可执行代码没有安全方面的考虑。 | 当系统的安全启动功能被打开后,UEFI在执行应用程序和驱动前会先检测程序和驱动的证书,仅当证书被信任时才会执行这个应用。 |
| 兼容性 | 体系驱动并由直接运行在 CPU 上的机器码组成。 | 用 EFI Byte Code 编写而成的，类似于 Java，具有良好的向下兼容性。 |



### 2 结构

- UEFI Image
  - UEFI 应用：启动管理、UEFI Shell、诊断、调试、调度和供应。
  - OS Loader：启动操作系统的特殊应用, 如果执行失败会通过 Exit() 函数结束运行并将控制器交还给固件， 如果运行成功可以通过 ExitBootServices() 函数将控制权交给加载到内存的 OS。。
  - UEFI 驱动：提供设备间的接口协议, 包括 Boot Serveces 和 Runtime Services。
    - Boot Service Drivers 在 OS Loader 调用 ExitBootServices() 函数后会被清除。
    - Runtime Drivers 在 OS Loader 将控制权递交到 OS 后仍然可以使用。

- 平台初始化框架
  - PEI：EFI 预初始化，主存储器初始化模块、检测和加载驱动执行环境核心。
  - DXE：驱动执行环境，提供了设备驱动和协议接口环境界面。



### 3 内容

- Image
  - UEFI Image 包含 PE32+ 头部, 相比常规 PE32 头支持 64-bit 重定向功能。
  - PE32+ 头中的 Subsystem 控制 Image 类型， 该类型影响 Image 被固件加载后的内存类型和退出方式。
    | Type | Value | Code Memory Type | Data Memory Type |
    | ---- | ----- | ---------------- | ---------------- |
    | EFI_IMAGE_SUBSYSTEM_EFI_APPLICATION | 10 | EfiLoaderCode | EfiLoaderData |
    | EFI_IMAGE_SUBSYSTEM_EFI_BOOT_SERVICE_DRIVER | 11 | EfiBootServiceCode | EfiBootServicesData |
    | EFI_IMAGE_SUBSYSTEM_EFI_RUNTIME_DRIVER | 12 | EfiRuntimeServicesCode | EfiRuntimeServicesData |
  - PE32+ 头中的 Machine 控制 Image 所适用的平台架构类型。
    | Type | Value |
    | ---- | ----- |
    | EFI_IMAGE_MACHINE_IA32 | 0x014c |
    | EFI_IMAGE_MACHINE_IA64 | 0x0200 |
    | EFI_IMAGE_MACHINE_EBC | 0x0EBC |
    | EFI_IMAGE_MACHINE_x64 | 0x8664 |
  - UEFI Image 通过 LoadImage() 函数来加载运行，通过 Exit() 函数来结束运行并交换控制权。

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

> build-essential: 构建必备软件包的信息列表 
> uuid-dev: 通用唯一 ID 库头文件和静态库 
> iasl: 英特尔 ASL 编译器 / 反编译器 
> git: 版本控制工具 
> gcc-5: GNU C 编译器 
> nasm: 通用 X86 汇编器

2. 下载 EDK2 源码并更新子模块

```
$ git clone https://github.com/tianocore/edk2.git
$ cd edk2
$ git submodule update --init --recursive
```

> EDK2 工程包含了多个子模块： 
> 'SoftFloat' (https://github.com/ucb-bar/berkeley-softfloat-3.git) 
> 'CryptoPkg/Library/OpensslLib/openssl' (https://github.com/openssl/openssl) 
> 'boringssl' (https://boringssl.googlesource.com/boringssl) 
> 'krb5' (https://github.com/krb5/krb5) 
> 'pyca.cryptography' (https://github.com/pyca/cryptography.git) 

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

编译后的 efi 格式文件放在 Build 目录下。

6. 运行模拟器

```
$ build run
```

#### 5.2 OVMF 开发环境

OVMF ( Open Virtual Machine Firmware,开放虚拟机固件)是用于虚拟机上的 UEFI 固件。在模拟器中测试非常方便,但模拟器功能有限,并且模拟器只能测试 32 位程序。在虚拟机中测试无疑是一种方便、快捷的方式,它既能较好地模拟真实环境,又可以做到快速、方便。EDK2 提供了制作虚拟机固件的方法,称为 OVMF。

1. 制作 OVMF 固件 OVMF.fd 文件

64位：

```
$ build -a X64 -p OvmfPkg\OvmfPkgX64.dsc
```

32位：

```
$ build -a IA32 -p OvmfPkg\OvmfPkgIa32.dsc
```

apt-get:

```
$ apt-get install ovmf
```

文件位于 /usr/share/ovmf/OVMF.fd

2. 安装 qemu 虚拟机

```
$ apt-get install qemu
```

3. 制作启动盘并运行虚拟机

```
$ dd if=/dev/zero of=hda.img bs=xxx count=xxx
$ mkfs.vfat hda.img
$ sudo qemu-system-x86_64 -bios /usr/share/ovmf/OVMF.fd -hdd hda.img
```



---
Power by Internet.
