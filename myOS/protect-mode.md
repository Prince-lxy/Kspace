# 实模式-保护模式
> CPU 的两种工作模式

## Contents / Mind mapping
- **1 地址**

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
  - 段值保存在两个描述符表中（GDT LDT），表格的每一项（Descriptor）记录了段的起始地址、段的长度、段的属性。这两个表格的首地址分别在寄存器 gdtr 和 ldtr 中。在保护模式下段寄存器记录了某个段在表格中相对于表格起始地址的偏移量（Selector）。这样通过段寄存器中的值结合描述符表寄存器中的地址就可以从描述符表中找到对应段的首地址了。



---
Power by Internet.