# EDK2

> UEFI 具体实现之一。

## Contents / Mind mapping
- **模块**
  - **标准应用程序工程模块**



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



---
Power by Internet.
