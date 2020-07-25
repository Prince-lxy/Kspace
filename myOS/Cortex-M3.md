# Cortex-M3

> Cortex-M3

## Contents / Mind mapping
- **1 Cortex-M3 MCU**
  - **1.1 Cortex-M3 内核**

---

### 1 Cortex-M3 MCU

Cortex-M3 MCU（微控制器单元）包含 Cortex-M3 处理器内核、调试系统、内部总线、外设、存储器、时钟、I/O。其中的内核和调试系统由 ARM 公司设计，其他部分由生产芯片的厂商设计开发。

#### 1.1 Cortex-M3 内核

- 架构：
  - Cortex‐M3 内核是一个 32 位处理器。内部的数据路径是 32 位的，寄存器是 32 位的，存储器接口也是 32 位的。
  -CM3 采用了哈佛结构，拥有独立的指令总线和数据总线，可以让取指与数据访问并行不悖。另一方面，指令总线和数据总线共享同一个存储器空间（一个统一的存储器系统）。换句话说，不是因为有两条总线，可寻址空间就变成 8GB 了。
- Cortex-M3 组件
  - Processor Core System
    - NVIC
    - Instruction Fetch Unit
    - Decoder
    - Register Bank
    - ALU
    - Memory Interface
    - Trace Interface
  - Memory Protection Unit
  - Bus Interconnect
  - Instruction Bus
  - Data Bus
  - Debug System
  - Debug Interface
- 寄存器
  - 通用寄存器（R0-R12）
    - 注
      - 绝大多数 16 位 Thumb 指令只能访问 R0‐R7，而 32 位 Thumb‐2 指令可以访问所有寄存器。
  - 堆栈指针寄存器（R13）
    - 分类
      - 主堆栈指针（MSP）：复位后缺省使用的堆栈指针，用于操作系统内核以及异常处理例程（包括中断服务例程）
      - 进程堆栈指针（PSP）：由用户的应用程序代码使用。
    - 注
      - Cortex‐M3 拥有两个堆栈指针，然而它们是 banked，因此任一时刻只能使用其中的一个。
      - 堆栈指针的最低两位永远是 0，这意味着堆栈总是 4 字节对齐的。
  - 连接寄存器（R14）
    - 注
      - 当呼叫一个子程序时，由 R14 存储返回地址。
  - 程序计数寄存器（R15）
    - 注
      - 指向当前的程序地址。如果修改它的值，就能改变程序的执行流。
  - 程序状态字寄存器组（PSRs）
    - 注
      - 记录 ALU 标志（0 标志，进位标志，负数标志，溢出标志），执行状态，以及当前正服务的中断号。
  - 中断屏蔽寄存器组
    - 分类
      - PRIMASK: 屏蔽所有中断（除不可屏蔽中断（NMI））
      - FAULTMASK：屏蔽所有 fault（除不可屏蔽中断（NMI））
      - BASEPRI：屏蔽优先级不高于某个具体数值的中断。
  - 控制寄存器（CONTROL）
    - 注
      - 定义特权状态并且决定使用哪一个堆栈指针。
- 操作模式
  - handler 模式：异常服务程序运行时的模式。
  - thread 模式：普通程序运行时的模式，处理器复位后默认进入此模式。
- 特权级
  - 特权级：可以访问所有范围内的存储器(除去MPU禁用的部分)；可以运行 thread 模式的程序，也可以运行 handler 模式的异常服务程序；处理器复位后默认进入特权级。
  - 用户级：只能访问和修改一部分安全的寄存器，可运行的程序不会对处理器产生破坏性的影响。



---
Power by Internet.
- 《Cortex‐M3 权威指南》作者：Joseph Yiu 翻译：宋岩
- 《Cortex-M3 技术参考手册》
