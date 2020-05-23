# FDT
> Flatted Device Tree

## Contents / Mind mapping
- **1 FDT**

---

### FDT

FDT(Flatted Device Tree):用于存储设备信息的树结构文件。FDT 改变了原来用 hardcode 方式将硬件配置信息嵌入到内核代码的方法，改用 bootloader 传递一个 dtb 的形式。

- DT（Device Tree）
  - Device Tree 的基本单位是 node。这些 node 被组织成树状结构，除了 root node，每个 node 都只有一个 parent。一个 device tree 文件中只能有一个 root node。root node 的 node name 是确定的，必须是“/”。
  - 每个 node 中包含了若干的 property/value 来描述该 node 的一些特性。每个 node 用节点名字（node name）标识，节点名字的格式是 node-name@unit-address。如果该 node 没有 reg 属性，那么该节点名字中必须不能包括 @ 和 unit-address。unit-address 的具体格式是和设备挂在哪个 bus 上相关。例如对于 cpu，其 unit-address 就是从0开始编址，以此加一。而具体的设备，例如以太网控制器，其 unit-address 就是寄存器地址。
  - 要想唯一指定一个 node 必须使用 full path，例如 /node-name-1/node-name-2/node-name-N。
- DTS（Device Tree Source Code）
  - 节点格式：[label:]node-name[@unit-address] { [properties definitions] [child nodes] }
  - .dts 文件用来保存开发板的硬件信息，.dtsi 文件用来保存多个同类型开发板的通用硬件信息，.dts 文件可以通过 include 的方式来引用。
  - 节点中如果存在同名属性，后面的属性值会覆盖前面的属性值。如果存在同名的节点，前后节点会合并。
  - 节点属性的值有三种类型： "string/string list", <32 bit unsigned int>, [binary data]。
  - model 属性：描述该设备属于那个生产商的哪个型号，格式为 "厂商名, 型号"。
  - compatible 属性：描述该设备适配的厂商列表，格式为 "model1", "model2"...
  - #address-cells 属性：描述子节点中需要用多少个 u32 来描述 reg 的 address。
  - #size-cells 属性：描述子节点中需要用多少个 u32 来描述 reg 的 size。
  - chosen 节点：主要用来保存启动 kernel 的 bootargs，chosen 的父节点必须是根节点。
  - aliases 节点：用于定义一些节点的别名，避免每次都使用冗长的节点名称。
  - memory 节点: 所有设备树中必备的节点，memory 节点的 device_type 属性必须是 memory。reg 属性的格式为 <起始地址 长度 起始地址 长度...> 。由于这些数据中间没有明显的分割方式，所以需要父节点中的 #address-cells 和 #size-cells 的属性值来确定。
  - #interrupt-cells 属性：描述需要多少个 u32 来标识一个 interrupts source 。
  - interrupts 属性：用于描述对应具有中断功能节点的设备中断描述地址，格式为 <value value ...>, <value value ...> ...
- DTB (Device Tree Blob)
  - 经过 DTC (Device Tree Compiler) 编译 DTS 所生成的二进制文件。
  - DTB 结构
    - DTB header (struct boot_param_header)
      - magic: 用于识别 DTB 的 magic 数 (0xd00dfeed)
      - totalsize: DTB 总大小
      - off_dt_struct: device-tree structure 偏移量
      - off_dt_string: device-tree strings 偏移量
      - off_mem_resvmap: memory reserve map 偏移量
      - version: DTB 版本
      - last_comp_version: 兼容版本
      - boot_cpuid_phys: 在哪个 CPU 上启动
      - dt_string_size: device-tree strings 大小
      - dt_struct_size: device-tree structure 大小
    - alignment gap
    - memory reserve map: 这个区域包括了若干的 reserve memory 描述符。每个 reserve memory 描述符是由 address 和 size 组成。其中 address 和 size 都是用 u64 来描述。
    - alignment gap
    - device-tree structure: 这部分由`标识 + 标记的固定格式 + 对应内容`的平坦格式来记录 DTS 中的内容，标识有以下五种：
      - FDT_BEGIN_NODE(0x00000001): node 开始标识
      - FDT_END_NODE(0x00000002): node 结束标识
      - FDT_PROP(0x00000003): 属性标识，紧跟这个标识的是两个 u32 数，分别表示属性值的 size 和 属性名称在 device-tree strings 中的偏移量，再紧接着的数据就是这个属性的值。
      - FDT_NOP(0x00000004): 占位标识
      - FDT_END(0x00000009): 结束标识
    - alignment gap
    - device-tree strings: 所有属性名称的字符串所组成的表。
  - 编译工具：dtc
  - 编译命令：`dtc -I dts -O dtb xxx.dts -o xxx.dtb`
  - 反编译命令：`dtc -I dtb -O dts xxx.dtb -o xxx.dts`



 ---
 Power by Internet.
