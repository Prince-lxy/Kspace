# myOS
> 这是一件很好玩的事情:)

## Contents / Mind mapping
- **1 计算机**
  - **1-1 hardware**
  - **1-2 BIOS**
  - **1-3 bootloader**
  - **1-4 kernel**

---

### 1 计算机

计算机的启动过程：hardware -> BIOS -> bootloader -> kernel

以上的每一个部分都很有趣:)

#### 1-1 hardware

硬件和软件相辅相成，是逻辑的不同展现形式，软件逻辑在不断硬件化，硬件也可以通过虚拟变得软件化。

组成：
- cpu
  - 运算器
  - 控制器
- 存储器
  - 内存：ROM RAM
  - 外存: 光盘 软盘 硬盘
- 输入输出设备
  - 输入设备：键盘 鼠标
  - 输出设备：显示器

这次为了便于调试，通过[搭建虚拟机](build-virtual-pc.md)的方式来虚拟硬件部分。

#### 1-2 BIOS

计算机运行靠一条一条的指令来实现，当主板上的电源键被按下时，一切从 BIOS(Basic Input Output System) 开始了。

#### 1-3 bootloader

由于 BIOS 只能从外部设备读取 512 字节的内容，而操作系统的大小远远超过了这个值，因此我们只好通过写一个不超过 512 字节大小的程序，来把 kernel 加载到内存。

这个程序一开始像一个快递员，后来搬运的东西复杂了，也变成了一个系统。

通过[hello world 程序](hello-world.md)来体会以下裸机程序的开发。

通过[探索保护模式](protect-mode.md)来体会从实模式进入保护模式的流程。

#### 1-4 kernel

一个程序复杂到了一定程度，我们就把它称之为系统，kernel 与前面的部分相比，只是逻辑更复杂了而已，包含的东西更多了而已。



---
Power by Internet.
