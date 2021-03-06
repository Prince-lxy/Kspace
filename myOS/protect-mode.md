# 实模式-保护模式
> CPU 的两种工作模式

## Contents / Mind mapping
- **1 地址**
- **2 中断和异常**

---

### 1 地址

- 实模式
  - 开机状态下默认进入实模式。
  - 物理地址 = 段值 × 16 + 偏移量，段值直接保存在段寄存器中。
- 保护模式
  - 保护模式需要特殊方法开启。
    - 准备全局描述符表（GDT）。
    - 用 lgdt 命令将 GDT 的首地址加载到 gdtr 寄存器。
    - 打开 A20 地址线。
    - 置位 cr0 寄存器 PE 位。
    - 跳转到保护模式。
  - 段描述符（段属性表）
    - 段值保存在两个描述符表中（GDT LDT），表格的每一项（Descriptor）记录了段的起始地址、段的长度、段的属性。这两个表格的首地址分别在寄存器 gdtr 和 ldtr 中。在保护模式下段寄存器记录了某个段在表格中相对于表格起始地址的偏移量（Selector）。这样通过段寄存器中的值结合描述符表寄存器中的地址就可以从描述符表中找到对应段的首地址了。
    - LDT 表所在的段要在 GDT 表中注册，加载 LDT 表是在进入保护模式之后，所以与 gdtr 不同，ldtr 保存的不是实际的物理地址，而是 LDT 段在 GDT 表中的段描述符偏移量。
  - 段特权级（段等级）
    - 段描述符中通过特权级来保护代码，特权级别从０到３分成四个等级，数字越小等级越高（下面讨论的特权等级与数字无关）。
    - 数据段：高特权等级可以查看低特权等级的数据段，低特权等级无法访问高特权等级的数据段。
    - 非一致代码段（默认代码段）：同级别之间可以访问，不同级别不可以访问。
    - 调用门：调用门自身拥有特权级，特权级相同或者高于调用门特权级的代码都可以使用调用门。但能否访问成功还需要看调用门的另一侧代码段的特权等级，调用者的特权等级要低于或者等于这部分代码的特权等级。
    - 一致代码段：专门用来给低特权等级访问的代码段（特权级高者无法访问），访问一致代码段时特权级不会发生变化。
    - **只有调用门+非一致代码段+call指令可以在调用后改变特权等级（jmp命令不会改变堆栈，所以不能用在特权级发生变化的跳转过程中），调用门+一致代码段不会改变特权等级，只是暂时调用一致代码段的代码，也不会有堆栈的改变。**
  - 从特权级0进入特权级3的方法：
    - 准备 TSS 并且通过 ltr 命令将其写入 tr 寄存器。
    - 准备特权级3的堆栈。
    - 依次入栈：特权级3的 ss 寄存器，特权级3的 esp 寄存器，特权级3的 cs 寄存器，特权级3的 eip 寄存器。
    - 利用 retf 函数进行特权级转换。
  - 从特权级3进入特权级1的方法：
    - 通过调用门调用非一致代码段,会实现特权级的变化（此时只能用 call 命令，因为 jmp 命令不会改变堆栈，而特权级转换需要堆栈也进行相应的改变）。
  - 分页机制
    - 只开启分段模式时，通过（段选择子+偏移量）的方式指定执行位置，cpu 将其转化为（段描述符+偏移量）的线性地址，此时线性地址就是物理地址。在开启分页模式后，cpu 将线性地址重新转化成物理地址：页目录表偏移量（前 10 bits）+ 页表偏移量（中 10 bits）+ 页内偏移量（后 12 bits）。
    - 真实的物理地址与页表的构成有关，从此地址的直观大小失去了意义。
    - 开启分页的步骤：初始化页表、初始化页目录表、将页目录表基地址（实地址）放入 cr3、将 cr0 的分页位置位。



### 2 中断和异常

> ***中断和异常都是程序执行过程中的强制性转移，转移到相应的处理程序。***
> 中断通常在程序执行时因为硬件而随机发生。除此之外软件通过 “int n” 指令也可以产生中断。
> 异常通常在处理器执行命令并检测到错误时发生，可以分成以下三种：
>   - Fault: 可以被更正的异常，异常处理程序结束后重新执行产生异常的指令。
>   - Traps: 可以被更正的异常，异常处理程序技术后执行产生异常命令的吓一条指令。
>   - Aborts: 不可以被更正的异常，产生异常后程序不允许继续执行。

- 实模式
  - 实模式下处理中断和异常的函数在 BIOS 的中断向量表中，中断向量表按顺序保存了对应中断向量的处理函数的首地址。

- 保护模式
  - 保护模式中的中断向量表的每一项由实模式中的函数首地址变成了具有保护能力的门描述符，通过增加属性的方式来提供保护能力。

---
Power by Internet.
