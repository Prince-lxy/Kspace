# EDK2

> UEFI 具体实现之一。

## Contents / Mind mapping
- **模块**
  - **标准应用程序工程模块**
  - **Shell 应用程序工程模块**
  - **main 应用程序工程模块**
  - **库模块**
  - **UEFI 驱动模块**
- **包**
  - **.dsc 文件**
  - **.dec 文件**
- **协议**
  - **协议在 UEFI 中的存在形式**
  - **协议的使用**



---

### 模块

结构：源文件 + 工程文件（.inf）

#### 标准应用程序工程模块

- 主函数、模块入口函数：EFI_STATUS UefiMain(ImageHandle, SystemTable)
- 工程文件：相当于 Makefile
  - Defines: 变量集合（必要）
    - INF_VERSION: INF 版本号（必要）
    - BASE_NAME: 模块名称，不能有空格（必要）
    - FILE_GUID: GUID（必要）
    - VERSION_STRING: 版本号（必要）
    - MODULE_TYPE: 模块类型，包括 SEC, PEI_CORE, PEIM, DXE_CORE, DXE_SAL_DRIVER, DXE_SMM_DRIVER, UEF_DRIVER, DXE_DRIVER, DXE_RUNTIME_DRIVER, UEFI_APPLICATION, BASE（必要）
    - ENTRY_POINT: 模块入口函数（必要）
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
  - .obj 目标文件被链接成 .dll（此过程会将 Image 入口函数封装为 UefiApplicationEntryPoint 库中的 '_ModuleEntryPoint', Image 入口函数在模块入口函数前后增加了构造函数和析构函数）。
  - GenFw 工具将 .dll 文件转换成 .efi 文件。
- 加载和执行过程：
  - Shell 调用 gBS->LoadImage() 将 efi 文件加载入内存并生存 Image 对象。
  - Shell 调用 gBS->StartImage() 启动这个 Image 对象。
  - 进入 eif 文件入口 _ModuleEntryPoint。
  - 进入模块入口函数 UefiMain。

#### Shell 应用程序工程模块

- 主函数：INTN ShellAppMain(UINTN Argc, CHAR16 ** Argv)
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
  - 模块入口函数调用 EFI_SHELL_PARAMETERS_PROTOCOL 获取命令行参数并传给主函数 ShellAppMain。

#### main 应用程序工程模块

- 主函数：int main(int argc, char ** argv)
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
  - 模块入口函数调用 EFI_SHELL_PARAMETERS_PROTOCOL 获取命令行参数并传给函数 ShellAppMain。
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



### 包

- 结构
  - 平台描述文件（.dsc）+ 平台声明文件（.dec）+ 模块集合
  - 如果需要生成固件，还需要有 Flash 描述文件（.fdf）
- 编译工具即功能
  - build: 用于编译包，需要一个 .dec 文件，一个 .dsc 文件，多个 .inf 文件。
  - GenFW: 用于制作固件，需要一个 .dec 文件，一个 .fdf 文件，多个 .efi 文件。

#### .dsc 文件

- Defines
  - 功能：设置 build 全局宏变量。
  - 语法：`变量名 = 值`，用 DEFINE/EDK_GLOBAL 修饰的变量可以在 .dsc .fdf 文件中通过 $(变量名) 的方法使用。
  - 变量：
    - DSC_SPECIFICATION: 通常为 0x00010005。
    - PLATFORM_GUID: 平台 GUID。
    - PLATFORM_VERSION: 平台版本号。
    - PLATFORM_NAME: 平台名称。
    - SKUID_IDENTIFIER: 通常为 Default。
    - SUPPORTED_ARCHITECTURES: 已支持的平台，用'|'分割不同平台。
    - BUILD_TARGETS: DEBUG|RELEASE
    - OUTPUT_DIRECTORY: 目标文件路径。（可选）
    - FLASH_DEFINITION: FDF 文件。（可选）
    - BUILD_NUMBER: 用于 Makefile。（可选）
    - FIX_LOAD_TOP_MEMORY_ADDRESS: 驱动、应用在内核中的地址。（可选）
    - TIME_STAMP_FILE: 该文件包含了一个时间戳，所有编译过程中生成的文件都使用该时间戳。（可选）
    - DEFINE: `DEFINE XXX = XXX` 此处定义的宏可以在 .dsc 文件的后续部分使用。（可选）
- LibraryClasses
  - 功能：定义了库的名字和库 .inf 文件相对路径。这些库可以被 Components 中的模块引用。
  - 语法
    - 标签：`[LibraryClasses.$(Arch).$(MODULE_TYPE), LibraryClasses.$(Arch1).$(MODULE_TYPE1)...]`
    - 模块：`LibraryName | Path/LibraryName.inf`
- Components
  - 功能：包含了会被 build 编译的模块集合。
  - 语法
    - 标签：`[Components.$(Arch)]`
    - 模块：`Path/xxx.inf`
    - 嵌套：`{<LibraryClasses> LibraryName | Path/LibraryName.inf}` 可以通过在模块名后面用大括号嵌套独立使用的库标签。
- BuildOptions
  - 功能：与 .inf 文件中的对应标签功能大致相同，此处的编译选项对本 .dsc 的所有模块有效。
  - 语法
    - 标签：`[BuildOptions.$(Arch)]`
    - 内容：`[编译器]:[$(Target)]_[Tool]_[$(Arch)]_[CC|DLINK]_FLAGS=`
- PCD
  - 功能：定义平台配置变量。
  - 语法
    - 定义：`命名空间.变量名 | 值 | 类型 | 值最大长度`
    - 使用：LibPcdGetPtr(_PCD_TOKEN_变量名)

#### .dec 文件

- Defines
  - 功能：用于提供包名称、GUID、版本号等信息。
  - 变量
    - DEC_SPECIFICATION: 通常为 0x00010005。
    - PACKAGE_NAME: 包名。
    - PACKAGE_GUID: 包 GUID。
    - PACEAGE_VERSION: 包版本号。
- Includes
  - 功能：列出了本包所提供的头文件的位置。
  - 语法
    - 标签：`[Includes.$(Arch)]`
    - 内容：相对于包中 .dsc 文件所在目录的相对目录。
- LibraryClasses
  - 功能：包可以通过 .dec 文件对外提供库，库的头文件位于 Include/Library 目录下。
  - 语法
    - 标签：`[LibraryClasses.$(Arch)]`\
    - 内容：`LibraryName | Path/LibraryHeader.h`
- Guids
  - 功能：包的 Include/Guid 中的头文件内定义了很多仅仅是声明的 GUID，这里保存了那些 GUID 的值，编译时 build 会将这些值复制到 AutoGen.c 中。
  - 语法
    - 标签：`[Guids.$(Arch)]`
    - 内容：`GUIDName = GUIDValue`
- Protocols
  - 功能：包的 Include/Protocols 中的头文件内定义了很多仅仅是声明的 Protocol GUID，这里保存了那些 GUID 的值。
  - 语法
    - 标签：`[Protocols.$(Arch)]`
    - 内容：`ProtocolName = GUIDValue`



### 协议

协议本身是服务器和客户端之间的约定，而 EDK2 中的协议是广义上的协议，提供服务（函数实现）的一方是服务器，使用服务（函数调用）的一方是客户端。

#### 协议在 UEFI 中的存在形式

EDK2 用 c 的方式 实现了 c++ 的面向对象。

当一个 .efi 文件被加载如内存以后，UEFI 将其描述为 IHANDLE 对象：

```
typedef struct {
	UINTN Signature;			// 此 Handle 类别
	LIST_ENTRY ALLHandles;			// 所有 IHANDLE 组成的链表
	LIST_ENTRY Protocols;			// 此 Handle 的 Protocols 链表
	UINTN LocateRequest;
	UINTN Key;
} IHANDLE;
```

IHANDLE 对象中的 Protocols 成员是双向链表，链表中的每一个元素是 PRPTOCOL_INTERFAC 对象：

```
typedef struct {
	UINTN Signature;
	LIST_ENTRY Link;
	IHANDLE *Handle;
	LIST_ENTRY ByProtocol;
	PROTOCOL_ENTRY *Protocol;		// Protocol 的 GUID
	VOID *Interface;			// Protocol 的实例
	LIST_ENTRY OpenList;
	UINTN OpenListCount;
} PROTOCOL_INTERFACE;
```

PROTOCOL_INTERFACE 对象的 Interface 成员表示某个具体的 PROTOCOL_ENTRY 对象：

```
typedef struct {
	UINTN Signature;
	LIST_ENTRY AllEntries;
	EFI_GUID ProtocolID;
	LIST_ENTRY Protocols;
	LIST_ENTRY Notify;
} PROTOCOL_ENTRY;
```

#### 协议的使用

- 寻找 Protocol 对象
  - gBS->OpenProtocol
    - 功能：用于查询指定的 Handle 中是否有支持制定的 Protocol ，如果支持，则打开该 Protocol ，如果不支持则返回错误信息。
    - 参数
      - IN EFI_HANDLE Handle: 指定的 Handle
      - IN EFI_GUID *Protocol: 要打开的 Protocol, 指向该 Protocol 的 GUID 指针
      - OUT VOID **Interface: 返回打开的 Protocol 对象
      - IN EFI_HANDLE AgentHandle: 打开此 Protocol 的 Image
      - IN EFI_HANDLE ControllerHandle: 使用此 Protocol 的控制器
      - IN UINT32 Attributes: 打开 Protocol 的方式
    - 特性
      - 如果 Handle 的 Protocol 链表中有对应的 Protocol ，则将 Protocol 对象的指针写入 *Interface 中，否则返回错误信息。
      - 如果是驱动调用这个函数，ControllerHandle 是该驱动的控制器，AgentHandle 是 EFI_DRIVER_BINDING_PROTOCOL 对象的 Handle 。
      - 如果调用驱动的是应用程序，AgentHandle 是该应用程序的 Image, ControllerHandle 此时可以忽略。
    - 打开方式
      - EFI_OPEN_PROTOCOL_BY_HANDLE_PROTOCOL 0x00000001
      - EFI_OPEN_PROTOCOL_GET_PROTOCOL 0x00000002
      - EFI_OPEN_PROTOCOL_TEST_PROTOCOL 0x00000004
      - EFI_OPEN_PROTOCOL_BY_CHILD_CONTROLLER 0x00000008
      - EFI_OPEN_PROTOCOL_BY_DRIVER 0x00000010
      - EFI_OPEN_PROTOCOL_EXCLUSIVE 0x00000020
  - gBS->HandleProtocol
    - 功能：实际上调用的还是 OpenProtocol ，只是做了参数上的简化。
    - 参数
      - IN EFI_HANDLE Handle: 指定的 Handle
      - IN EFI_GUID *Protocol: 要打开的 Protocol, 指向该 Protocol 的 GUID 指针
      - OUT VOID **Interface: 返回打开的 Protocol 对象
    - 特性
      - AgentHandle 固定为 gDxeCoreImageHandle 。
      - ControllerHandle 固定为 NULL 。
      - Attributes: 固定为 EFI_OPEN_PROTOCOL_BY_HANDLE_PROTOCOL 。
  - gBS->LocateProtocol
    - 功能：OpenProtocol 与 HandleProtocol 的使用需要知道对应设备和对应的 Protocol ，但是在我们不清楚 Protocol 在哪个设备上的时候，可以通过这个函数直接向 UEFI 内核寻找 Protocol 的位置。
    - 参数
      - IN EFI_GUID *Protocol: 要打开的 Protocol, 指向该 Protocol 的 GUID 指针
      - IN VOID *Registration, OPTIONAL: 可选参数，从 RegisterProtocolNotify() 中获得的 key
      - OUT VOID **Interface: 返回打开的 Protocol 对象
    - 特性
      - UEFI 内核中所包含的 Protocol 对象通常都不止一个，本函数采用的是顺序查找的方式，返回找到的第一个该 Protocol 对象 。
  - gBS->LocateHandleBuffer
    - 功能：查找支持某个 Protocol 的所有设备。
    - 参数
      - IN EFI_LOCATE_SEARCH_TYPE SearchType: 查找方式
      - IN EFI_GUID *Protocol OPTIONAL: 指定的 Protocol
      - IN VOID *SearchKey OPTIONAL: PROTOCOL_NOTIFY 类型
      - IN OUT UINTN *NoHandles: 返回找到的 Handle 数量
      - OUT EFI_HANDLE **Buffer: 分配 Handle 数组并返回
    - 特性
      - SearchType 有三种：AllHandles 表示找出系统中所有的 Handle; ByRegisterNotify 用于从 RegisterProtocolNotify 中找出匹配 SearchKey 的 Handle; ByProtocol 用于从系统数据库中找设备。
      - LocateHandle 函数与本函数类似，只是 Buffer 需要调用者自行管理，参数中多一个调用者自己开辟的 Buffer 。
  - gBS->ProtocolPerHandle
    - 功能：获取制定设备所支持的所有 Protocol 。
    - 参数
      - IN EFI_HANDLE Handle: 指定设备
      - OUT EFI_GUID ***ProtocolBuffer: 返回 Protocol GUID 数组
      - OUT UINTN *ProtocolBufferCount: 返回 Protocol 的数目
  - gBS->OpenProtocolInformation
    - 功能：获取指定设备上指定 Protocol 的打开信息。
    - 参数
      - IN EFI_HANDLE Handle: 指定设备
      - IN EFI_GUID *Protocol: 指定 Protocol
      - OUT EFI_OPEN_PROTOCOL_INFORMATION_ENTRY **EntryBuffer: 返回信息数组
      - OUT UINTN *EntryCount: EntryBuffer 数组元素个数
    - 特性
      ```
      typedef struct {
      	EFI_HANDLE AgentHandle;
      	EFI_HANDLE ControllerHandle;
      	UINT32 Attributes;
      	UINT32 OpenCount;
      } EFI_OPEN_PROTOCOL_INFORMATION_ENTRY;
      ```
- 使用 Protocol 提供的服务
- 关闭 Protocol
  - gBS->CloseProtocol
    - 功能：关闭打开的 Protocol 。
    - 参数
      - IN EFI_HANDLE Handle: 指定的设备
      - IN EFI_GUID *Protocol: 指定的 Protocol
      - IN EFI_HANDLE AgentHandle: 使用 Protocol 的对象
      - IN EFI_HANDLE ControllerHandle: 控制器对象
    - 特性
      - 通过 HandleProtocol 和 LocateProtocol 打开的 Protocol 因为没有指定对应的 AgentHandle ，所以无法直接关闭，如果要关闭，必须使用 OpenProtocolInformation() 获取 AgentHandle 和 ControllerHandle 去关闭。



---
Power by Internet.
