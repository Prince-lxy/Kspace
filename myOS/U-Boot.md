# U-Boot
> 一种开源的应用广泛的 bootloader 。

## Contents / Mind mapping
- **U-Boot**

---

### U-Boot

```
BL0 =====> BL1 (uboot-spl) =====> BL2 (uboot.bin)
```

- BL0
  - 运行位置：集成在芯片内部的 IROM 中
  - 功能：
    - 初始化系统时钟、特殊设备的控制器、启动设备、看门狗、堆栈、SRAM等
    - 验证BL1镜像
    - 从存储介质或者 USB 加载 BL1 镜像到对应内部 SRAM 上
    - 跳转到BL1镜像所在的地址上
- BL1 (uboot-spl)
  - 运行位置：集成在平台内部的 SRAM 中
  - 功能：
    - 初始化部分时钟（和 SDRAM 相关）
    - 初始化 DDR（外部 SDRAM ）
    - 从存储介质中将BL2镜像加载到 SDRAM 上
    - 验证BL2镜像的合法性
    - 跳转到BL2镜像所在的地址上
  - 代码流程：
    - `_start` : 入口
    - `reset` : 关闭中断
    - `cpu_init_cp15` : 关闭 MMU、TLB
    - `cpu_init_crit -> lowlevel_init` : 板级初始化
    - `_main -> board_init_f` : 加载并跳转到 BL2
- BL2 (uboot.bin)
  - 运行位置：SDRAM
  - 核心功能：
    - 初始化部分硬件，包括时钟、内存等
    - 加载内核到内存上
    - 加载文件系统、atags或者dtb到内存上
    - 根据操作系统启动要求正确配置好一些硬件
    - 启动操作系统
  - 监控功能
    - 进行调试
    - 读写内存
    - 烧写Flash
    - 配置环境变量
    - 命令引导操作系统
  - 代码流程
    - Arch 初始化
      - `_start` : 入口
      - `reset` : 关闭中断，设置 svc 模式
      - `cpu_init_cp15` : 关闭 MMU、TLB
      - `cpu_init_crit -> lowlevel_init` : 关键寄存器设置（时钟、看门狗、内存、串口）
      - `_main` : 进入 board 初始化
    - Board 初始化
      - `board_init_f_alloc_reserve`: 堆栈、GD、early malloc 空间的分配
      - `board_init_f_init_reserve`: 堆栈、GD、early malloc 空间的初始化
      - `board_init_f`: 堆栈、GD、early malloc 空间的初始化（串口、定时、环境变量、I2C、SPI）
      - `relocate_code、relocate_vectors`: uboot 和异常中断向量表的重定向
      - `board_init_r`: uboot relocate后的板级初始化（eMMc、Nand flash、Network、中断）
      - `run_main_loop`: 进入命令行状态，等待终端输入命令以及对命令进行处理



 ---
 Power by Internet.
