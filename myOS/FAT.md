# FAT
> File Allocation Table

## Contents / Mind mapping
- **FAT12/FAT16**
  - **启动扇区**
- **FAT32**
  - **启动扇区**

---

### FAT12/FAT16

FAT12/FAT 16 包含启动扇区、FAT 区、根目录区和数据区。

#### 启动扇区

|偏移（字节）|长度（字节）|说明|
|------------|------------|----|
|0x00|3|跳转指令|
|0x03|8|OEM 字符串|
|0x0b|2|逻辑扇区大小（字节）|
|0x0d|1|每个簇的扇区数|
|0x0e|2|保留扇区数|
|0x10|1|FAT 数|
|0x11|2|根目录文件条目数|
|0x13|2|扇区总数，如果是 0，则使用 0x20 处的 4 字节值|
|0x15|1|媒体描述符|
|0x16|2|每个 FAT 所占用的扇区数|
|0x18|2|每个磁道的扇区数|
|0x1a|2|磁头数|
|0x1c|4|隐藏扇区数|
|0x20|4|总扇区数|
|0x24|1|物理驱动器号|
|0x25|1|保留|
|0x26|1|签名|
|0x27|4|卷 ID|
|0x2b|11|卷标|
|0x36|8|文件系统类型|
|0x3e|448|空|
|0x1fe|2|结束符 0x55AA|


### FAT32

FAT32 的根目录区保存在数据区的任意位置处，与数据区中的数据共享数据区。

#### 启动扇区

|偏移（字节）|长度（字节）|说明|
|------------|------------|----|
|0x00|3|跳转指令|
|0x03|8|OEM 字符串|
|0x0b|2|逻辑扇区大小（字节）|
|0x0d|1|每个簇的扇区数|
|0x0e|2|保留扇区数(x)|
|0x10|1|FAT 数|
|0x11|2|0x0000|
|0x13|2|扇区总数，如果是 0，则使用 0x20 处的 4 字节值|
|0x15|1|媒体描述符|
|0x16|2|0x0000|
|0x18|2|每个磁道的扇区数|
|0x1a|2|磁头数|
|0x1c|4|隐藏扇区数|
|0x20|4|总扇区数|
|0x24|4|每个 FAT 所占用的扇区数|
|0x28|2|FLAG|
|0x2a|2|版本号|
|0x2c|4|根目录起始簇|
|0x30|2|文件系统信息扇区号|
|0x32|2|引导扇区备份|
|0x34|12|保留|
|0x40|1|逻辑驱动号|
|0x41|1|保留|
|0x42|1|签名|
|0x43|4|序列号|
|0x47|11|卷标|
|0x52|8|文件系统类型|
|0x5a|420|空|
|0x200(x) - 2|2|结束符 0x55AA|


 ---
 Power by Internet.
