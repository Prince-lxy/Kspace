# 构建虚拟机
> 利用虚拟机代替真机进行开发。

## Contents / Mind mapping
- **1 虚拟机**
- **2 安装**
  - **2-1 bochs**
- **3 运行**
  - **3-1 bochs**

---

### 1 虚拟机

虚拟机指的是通过软件模拟出来的计算机，这种计算机和实体计算机功能完全相同。

由于我们的目的是开发操作系统，因此使用完全软件化的虚拟机比较利于调试。



### 2 安装

#### 2-1 bochs

由于要进行调试和源码查看甚至修改 bochs 源代码，所以我们用源码来安装。

通过[官网](https://sourceforge.net/projects/bochs/files/bochs/)下载 bochs 源码安装包。

解压源码安装包，通过 configure、make、make install 安装。编译安装过程中出现问题都可以网络解决:)



### 3 运行

#### 3-1 bochs

制作镜像文件 floppya.img:

```
$ dd if=/dev/zero of=floppya.img bs=512 count=2880
```


添加并根据具体安装情况修改配置文件 bochsrc:

```
###############################################
# Configuration file for Bochs
###############################################

# ram大小，单位MB
megs: 32

# BIOS和VGA BIOS.
romimage: file=$BXSHARE/BIOS-bochs-latest
vgaromimage: file=$BXSHARE/VGABIOS-lgpl-latest

# 启动盘
floppya: 1_44=floppya.img, status=inserted

# 启动盘符
boot: floppy

# 日志文件
log: bochsout.log

# 开启或关闭某些功能。
# 关闭鼠标，并打开键盘。
mouse: enabled=0
keyboard: keymap=$BXSHARE/keymaps/x11-pc-us.map
```

运行 bochs:

```
bochs -f bochsrc -q
```



---
Power by Internet.
