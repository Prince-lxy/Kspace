# EDK2

> UEFI 具体实现之一。

## Contents / Mind mapping
- **模块**
  - **标准应用程序工程模块**
  - **Shell 应用程序工程模块**
  - **main 应用程序工程模块**
  - **库模块**
  - **UEFI 驱动模块**



---

### 模块

结构：源文件 + 工程文件（.inf）

#### 标准应用程序工程模块

- 入口函数：EFI_STATUS UefiMain(ImageHandle, SystemTable)
- 工程文件：相当于 Makefile
  - Defines: 变量集合（必要）
    - INF_VERSION: INF 版本号（必要）
    - BASE_NAME: 模块名称，不能有空格（必要）
    - FILE_GUID: GUID（必要）
    - VERSION_STRING: 版本号（必要）
    - MODULE_TYPE: 模块类型，包括 SEC, PEI_CORE, PEIM, DXE_CORE, DXE_SAL_DRIVER, DXE_SMM_DRIVER, UEF_DRIVER, DXE_DRIVER, DXE_RUNTIME_DRIVER, UEFI_APPLICATION, BASE（必要）
    - ENTRY_POINT: 入口函数（必要）
  - Soureces: 源文件集合（必要）
    - Sources.$(Arch) 块所包含的源文件只用于对应的架构。
    - `源文件 | 工具链` 格式所指定的源文件只用于对应工具链
  - Packages: 引用到的包声明文件 (.dec) 集合（必要）
    - 包声明文件使用相对路径，相对路径的根路径为 EDK2 根目录。
    - 若 Sources 块列出源文件，则 Packages 第一项必须为 MdePkg/MdePkg.dec 。
  - LibraryClasses: 库模块集合（必要）
    - 应用程序工程模块必须链接 UefiApplicationEntryPoint 库。
    - 驱动模块必须链接 UefiDriverEntryPoint 库。
  - Protocols: 协议集合
    - 列出模块所用到的 Protocol 对应的 GUID 变量名。
  - Guids: GUID 集合
  - BuildOptions: 编译链接选项集合
    - `[编译工具链]：[$Target]_[TOOL_CHAIN_TAG]_[$Arch]_[CC|DLINK]_FLAGS[=|==] 选项`
      - 编译工具链：MSFT(VS), INTEL(Intel), GCC, RVCT(ARM)
      - Target: DEBUG, RELEASE
      - TOOL_CHAN_TAG: VS2008, VS2010, GCC4X, GCC5
      - Arch: IA32, X64, IPF, EBC, ARM, AARCH64
      - CC: 编译选项
      - DLINK: 链接选项
　　　- '=' 表示选项追加到默认选项后，'==' 表示弃用默认选项。
      - Pcd: 平台配置数据库，库中的变量可以被整个 UEFI 系统访问
- 编译过程：
  - .c 源文件被编译为 .obj 目标文件。
  - .obj 目标文件被链接成 .dll（此过程会将入口函数封装为 UefiApplicationEntryPoint 库中的 '_ModuleEntryPoint', 次入口函数在模块入口函数前后增加了构造函数和析构函数）。
  - GenFw 工具将 .dll 文件转换成 .efi 文件。
- 加载和执行过程：
  - Shell 调用 gBS->LoadImage() 将 efi 文件加载入内存并生存 Image 对象。
  - Shell 调用 gBS->StartImage() 启动这个 Image 对象。
  - 进入 eif 文件入口 _ModuleEntryPoint。
  - 进入模块入口函数 UefiMain。

#### Shell 应用程序工程模块

- 入口函数：INTN ShellAppMain(UINTN Argc, CHAR16 ** Argv)
- 工程文件
  - Defines
    - MODULE_TYPE: UEFI_APPLICATION
    - ENTRY_POINT: ShellCEntryLib
  - Packages
    - MdePkg/MdePkg.dec 与 ShellPkg/ShellPkg.dec 必须列出。
  - LibraryClasses
    - ShellCEntryLib 必须列出。
    - UefiBootServicesTableLib 和 UefiLib 通常也要列出。
- 执行过程
  - 模块入口函数 ShellCEntryLib 的参数与标准应用程序相同。
  - 模块入口函数调用 EFI_SHELL_PARAMETERS_PROTOCOL 获取命令行参数并传给入口函数 ShellAppMain。

#### main 应用程序工程模块

- 入口函数：int main(int argc, char ** argv)
- 工程文件
  - Defines
    - MODULE_TYPE: UEFI_APPLICATION
    - ENTRY_POINT: ShellCEntryLib
  - Packages
    - MdePkg/MdePkg.dec, ShellPkg/ShellPkg.dec, StdLib/StdLib.dec 必须列出。
  - LibraryClasses
    - ShellCEntryLib 提供 ShellCEntryLib 函数。
    - LibC 提供 ShellAppMain 函数。
    - LibStdio 提供标准 C 输入输出函数，如 printf()。
- 编译
  - main 应用程序工程模块要在 AppPkg 环境下才能编译成功，首先要将本工程的工程文件添加到 AppPkg/AppPkg.dsc 文件的 Components 块后通过 `build -p AppPkg/AppPkg.dsc -m xxx.inf` 进行编译。
- 执行过程
  - 模块入口函数 ShellCEntryLib 的参数与标准应用程序相同。
  - 模块入口函数调用 EFI_SHELL_PARAMETERS_PROTOCOL 获取命令行参数并传给入口函数 ShellAppMain。
  - ShellAppMain 函数调用 main 函数。

#### 库模块

- 工程文件
  - MODULE_TYPE: BASE
  - LIBRARY_CLASS: 库名字
    - 指明本库模块可以被引用的模块类型：`库名字 | 模块类型１ 模块类型２`
  - 不要设置：ENTRY_POINT
  - CONSTRUCTOR: 如果在使用之前需要初始化该库，就在本项后面填写本库的构造函数。
  - DESTRUCTOR: 如果在使用之后需要清理库中占用的资源，就在本项后面填写本库的析构函数。
- 库使用规则
  - 模块所属的包声明文件(.dsc)中 LibraryClasses 下声明该库，语法为：`库名称 | 库工程文件相对于 EDK2 根目录的相对路径`　。
  - 模块的工程文件中 LibraryClasses 下声明该库的名称。

#### UEFI 驱动模块

- 分类
  - 符合 UEFI 驱动模型的驱动：UEFI_DRIVER
  - DXE 驱动：DXE_DRIVER, DXE_SAL_DRIVER, DXE_SMM_DRIVER, DXE_RUNTIME_DRIVER
- UEFI 驱动工程文件
  - Defines:
    - MODULE_TYPE: UEFI_DRIVER
  - Sources:
    - 通常都约定 ComponentName.c 中定义驱动名字。
  - LibraryClasses:
    - 必须包含 UefiDriverEntryPoint, 该模块中包含 UEFI_ENTRYPOINT



---
Power by Internet.
