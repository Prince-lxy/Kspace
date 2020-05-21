# FDT
> Flatted Device Tree

## Contents / Mind mapping
- **1 FDT**

---

### FDT

FDT(Flatted Device Tree):用于存储设备信息的树结构文件。FDT改变了原来用 hardcode 方式将硬件配置信息嵌入到内核代码的方法，改用 bootloader 传递一个 dtb 的形式。

- DT（Device Tree）
  - Device Tree 的基本单位是 node。这些 node 被组织成树状结构，除了root node，每个 node都只有一个 parent。一个 device tree 文件中只能有一个 root node。root node 的 node name 是确定的，必须是“/”。
  - 每个node中包含了若干的 property/value 来描述该 node 的一些特性。每个 node 用节点名字（node name）标识，节点名字的格式是node-name@unit-address。如果该 node 没有 reg 属性，那么该节点名字中必须不能包括 @ 和 unit-address。unit-address 的具体格式是和设备挂在哪个 bus 上相关。例如对于 cpu，其 unit-address 就是从0开始编址，以此加一。而具体的设备，例如以太网控制器，其 unit-address 就是寄存器地址。
  - 要想唯一指定一个node必须使用 full path，例如 /node-name-1/node-name-2/node-name-N。
- DTS（Device Tree Source Code）
  - 节点格式：[label:]node-name[@unit-address] { [properties definitions] [child nodes] }
  - .dts 文件用来保存开发板的硬件信息，.dtsi 文件用来保存多个同类型开发板的通用硬件信息，.dts 文件可以通过 include 的方式来引用。
  - 节点属性的值有三种类型： "string/string list", <32 bit unsigned int>, [binary data]。
  - #address-cells 属性：描述子节点中需要用多少个 u32 的 cell 来描述 reg 的 address。
  - #size-cells 属性：描述子节点中需要用多少个 u32 的 cell 来描述 reg 的 size。
  - chosen 节点：主要用来保存启动 kernel 的 bootargs，chosen 的父节点必须是根节点。
  - aliases 节点：用于定义一些节点的别名，避免每次都使用冗长的节点名称。



 ---
 Power by Internet.
