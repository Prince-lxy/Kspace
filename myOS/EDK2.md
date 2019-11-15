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
- **Protocol**
  - **Protocol 在 UEFI 中的存在形式**
  - **Protocol 的使用**
- **系统表**
  - **构成**
  - **使用**
- **启动服务**
  - **事件服务**
  - **内存管理服务**
  - **Protocol 管理服务**
  - **Protocol 使用服务**
  - **驱动管理服务**
  - **Image 管理服务**
  - **ExitBootServices**
  - **其他服务**
- **运行时服务**
  - **时间服务**
  - **读写系统变量**
  - **虚拟内存服务**
  - **其他服务**
- **时钟中断**
- **设备路径**
- **硬盘**
  - **MBR**
  - **GPT**
  - **硬盘 Protocol**
  - **文件系统**



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
  - Protocols: 用到的 Protocol 集合
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



### Protocol

Protocol 本身是服务器和客户端之间的约定，而 EDK2 中的协议是广义上的协议，提供服务（函数实现）的一方是服务器，使用服务（函数调用）的一方是客户端。

#### Protocol 在 UEFI 中的存在形式

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

#### Protocol 的使用

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



### 系统表

#### 构成

- EFI_TABLE_HANDER Hdr: 表头
  - UINT64 Signature: 64 位无符号整数，可以通过 SIGNATURE_64(A,B,C,D,E,F,G,H) 宏将 ASCII 字符串转化
  - UINT32 Revision
  - UINT32 HeaderSize: 整个表的长度
  - UINT32 CRC32: 整个表的校验码
  - UINT32 Reserved
- UINT16 * FirmwareVendor: 固件提供商
- UINT32 FirmwareRevision: 固件版本号
- EFI_HANDLE ConsoleInHandle: 输入控制台设备
- EFI_SIMPLE_TEXT_INPUT_PROTOCOL * ConIn: 输入 Protocol
- EFI_HANDLE ConsoleOutHandle: 输出控制台设备
- EFI_SIMPLE_TEXT_OUTPUT_PROTOCOL * ConOut: 输出 Protocol
- EFI_HANDLE StandardErrorHandle: 标准错误控制台设备
- EFI_SIMPLE_TEXT_OUTPUT_PROTOCOL * StdErr: 标准错误控制台 Protocol
- EFI_RUNTIME_SERVICES * RuntimeServices: 启动服务表
- EFI_BOOT_SERVICES * BootServices: 运行时服务表
- UINTN NumberOfTableEntries: 配置表数组大小
- EFI_CONFIGURATION_TABLE * Configurationtable: 系统配置表
  - EFI_GUID VendorGuid: 配置表标识符
  - VOID * VendorTable: 配置表数据

#### 使用

- 通过标准应用程序模块参数获取: UefiMain 入口函数的两个参数 ImageHandle 和 SystemTable 可以直接使用。
- 通过全局变量 gST, gBS: 在 Shell 应用程序和驱动程序中，我们可以调用 UefiBootServicesTableLib 库中的全局变量 gST, gBS, gImageHandle 。



### 启动服务

系统进入 DXE 阶段时启动服务表被初始化，最终通过 SystemTable 指针将启动服务表传递给 UEFI 应用程序或驱动程序。

#### 事件服务

事件是异步操作的基础，有了事件的支持，才可以在 UEFI 系统内执行并发操作。

- CreateEvent
  - 功能: 生成事件对象。
  - 参数
    - IN UINT32 Type: 事件类型
    - IN EFI_TPL NotifyTPL: 事件等级
    - IN EFI_EVENT_NOTIFY NotifyFunction, OPTIONAL: 事件通知函数
    - IN VOID * NotifyContext, OPTIONAL: 事件通知函数第二个参数
    - OUT EFI_EVENT * Event: 生成的事件
  - 特性
    - 事件类型
      - 0
      - EVT_TIMER
      - EVT_RUNTIME
      - EVT_NOTIFY_WAIT
      - EVT_NOTIFY_SIGNAL
      - EVT_SIGNAL_EXIT_BOOT_SERVICES
      - EVT_SIGNAL_VIRTUAL_ADDRESS_CHANGE
      - EVI_RUNTIME_CONTEXT
    - 事件类型可以是其中的某一个，也可以是其中某几个的组合。
    - 优先级: 0 到 31 的一个整数。
      - TPL_APPLICATION (4)
      - TPL_CALLBACK (8)
      - TPL_NOTIFY (16)
      - TPL_HIGH_LEVEL (31)
    - EVT_NOTIFY_WAIT 或 EVT_NOTIFY_SIGNAL 属性的事件才会执行事件通知函数，前者在等待过程中每次事件状态被检查时调用通知函数，后者在事件触发时调用通知函数。
- CreateEventEx
  - 功能: 生成事件对象并将该事件加入到一个组内。
  - 参数
    - IN UINT32 Type: 事件类型
    - IN EFI_TPL NotifyTpl: 事件等级
    - IN EFI_EVENT_NOTIFY NotifyFunction, OPTIONAL: 事件通知函数
    - IN VOID * NotifyContext, OPTIONAL: 事件通知函数第二个参数
    - IN CONST EFI_GUID * EventGroup OPTIONAL: 事件组
    - OUT EFI_EVENT * Event: 事件
  - 特性
    - 本函数生成的事件会被加入到 EventGroup 组中，当其中任何一个事件被触发后，组内其他成员都会被触发。同组内所有通知函数都会被加入到执行队列，NotifyTpl 等级高者优先执行。
    - 事件类型不能为 EVT_SIGNAL_EXIT_BOOT_SERVICES 或 EVT_SIGNAL_VIRTUAL_ADDRESS_CHANGE ，因为这两种事件有各自对应的组 gEfiEventExitBootServicesGuid 和 gEfiEventVirtualAddressChangeGuid ，ExitBootService() 与 SetVirtualAddressMap() 调用时分别触发对应组内所有的事件。
    - Memory Map 改变时会触发 gEfiEventMemoryMapChangeGuid 组内所有的事件。
    - Boot Manager 加载并执行一个启动项时触发 gEfiEventReadyToBootGuid 组内所有的事件。
- CloseEvent
  - 功能: 关闭事件对象。
  - 参数
    - IN EFI_EVENT Event
  - 特性
    - 通常的原则是事件所有者调用本函数来删除内核中对应的事件对象。
- SignalEvent
  - 功能: 将事件设置为触发状态。
  - 参数
    - EFI_EVENT Event
  - 特性
    - 如果事件类型为 EVT_NOTIFY_SIGNAL ，则将该事件的通知函数加入就绪队列；如果该事件属于一个组，则将组内所有 EVT_NOTIFY_SIGNAL 事件都设置为触发状态并将通知函数都加入到就绪队列执行。
- WaitForEvent
  - 功能：等待事件数组中的任一事件被触发。
  - 参数
    - IN UINTN NumberOfEvents: Event 数组内 Event 个数
    - IN EFI_EVENT * Event: Event 数组
    - OUT UINTN * Index: 触发的事件在数组中的下标
  - 特性
    - 本函数为阻塞操作，函数从前到后循环轮训 Event 数组，直到事件被触发或出错后返回。如果想要等待一段事件后退出等待，可以在 Event 数组中加入定时器事件。
    - 当检测到某个事件处于触发态后，将该事件在 Event 数组中的下标赋值给 Index 参数，并在返回前重置该事件为非触发状态。
    - 本函数必须运行在 TPL_APPLICATION 级别，否则返回 EFI_UNSUPPORTED 。
    - 当检查到某个事件为 EVT_NOTIFY_SIGNAL 类型时，Index 赋值为该事件在 Event 数组中的下标，并返回 EFI_INVALID_PARAMETER 。
- CheckEvent
  - 功能: 检查事件状态。
  - 参数
    - IN EFI_EVENT Event: 事件对象
  - 特性
    - 如果事件类型为 EVT_NOTIFY_SIGNAL ，则返回错误 EFI_INVALID_PARAMETER 。
    - 如果事件为触发状态，返回 EFI_SUCCESS 并在返回前将事件置为非触发状态。
    - 如果事件处于非触发状态并且没有通知函数，则返回 EFI_NOT_READY 。
    - 如果事件处于非触发状态但是有通知函数，则执行通知函数。
- SetTimer
  - 功能: 设定 EVT_TIMER 事件的定时器属性。
  - 参数
    - IN EFI_EVENT Event: 事件对象
    - IN EFI_TIMER_DELAY Type: 定时器类别
    - IN UINT64 TriggerTime: 定时器过期时间，单位是 100ns
  - 特性
    - 定时器类别
      - TimerCancel: 用于取消定时器。
      - TimerPeriodic: 重复型定时器，每隔 TriggerTime * 100ns 触发一次。
      - TimerRelative: 一次性定时器，TriggerTime * 100ns 时触发一次。
    - 如果 TriggerTime 是 0 ，则时钟定时器单位从 100ns 变为一个时钟滴答。
- RaiseTPL
  - 功能: 提升任务优先级。
  - 参数
    - EFI_TPL NewTpl
- RestoreTPL
  - 功能: 恢复任务优先级。
  - 参数
    - EFI_TPL OldTpl
  - 特性
    - RaiseTPL 与 RestoreTPL 必须成对出现，提升优先级后要尽快恢复。
    - 当优先级被提升到最高时会关闭中断，回复后中断会被重新打开。
    - 在优先级恢复到原来之前，所有高于原来优先级的触发态事件的通知函数都要执行完毕。

#### 内存管理服务

提供内存的分配和释放，管理系统内存映射。

- AllocatePool: 分配内存区域。
- FreePool: 释放内存区域。
- AllocatePages: 分配内存页。
- FreePages: 释放内存页。
- GetMemoryMap: 获取内存映射。

#### Protocol 管理服务

提供安装与卸载 Protocol 的服务，以及注册 Protocol 通知服务。

- InsatllProtocolInterface: 安装 Protocol 到设备上。
- UninstallProtocolInterface: 卸载设备上的 Protocol 。
- ReinstallProtocolInterface: 重新安装 Protocol 到设备上。
- RegisterProtocolNotify: 为指定的 Protocol 注册通知事件。
- InstallMultipleProtocolInterfaces: 安装多个 Protocol 到设备上。
- UninstallMultipleProtocolInterfaces: 卸载多个设备上的 Protocol 。

#### Protocol 使用服务

提供 Protocol 查找、打开、关闭等服务。

- OpenProtocol: 打开 Protocol 。
- HandleProtocol: 简化版本 OpenProtocol 。
- LocateProtocol: 找出系统中指定 Protocol 的第一个对象。
- LocateProtocolBuffer: 找出指定 Protocol 所有的 Handle ，系统分配 Buffer ，调用者释放 Buffer 。
- LocateHandle: 找出指定的 Protocol ，调用者分配、释放 Buffer 。
- OpenProtocolInformation: 获取 Protocol 打开信息。
- ProtocolsPerHandle: 获取指定设备上安装的所有 Protocol 。
- CloseProtocol: 关闭 Protocol 。
- LocateDevicePath: 在指定设备路径下找出支持指定 Protocol 的设备。

#### 驱动管理服务

提供驱动的安装与卸载服务。

- ConnectController: 将驱动安装到指定设备控制器。
- DisconnnectController: 卸载指定设备控制器的指定驱动。

#### Image 管理服务

提供 Image 的加载、卸载、启动、退出等服务。

- LoadImage: 加载 .efi 文件至内存并生成 Image 。
- UnloadImage: 卸载 Image 。
- StartImage: 启动 Image 。
- Exit: 退出 Image 。

#### ExitBootServices

结束启动服务，进入 RT 期。

#### 其他服务

- InstallConfigurationTable: 管理系统配置表。
- GetNextMonotonicCount: 获取系统单调计数器数值。
- Stall: 暂停 CPU 指定微秒数。
- SetWatchdogTimer: 设置看门狗定时器。
- CalculateCrc32: 计算 CRC 校验码。
- CopyMem: 复制内存。
- SetMem: 设定指定内存区域数值。



### 运行时服务

从进入 DXE 阶段运行时服务被初始化开始，知道操作系统结束，运行时服务都存在。

#### 时间服务

提供读取、设置系统时间和读取、设置系统从睡眠中唤醒的时间。

- GetTime: 获取硬件时间。
- SetTIme: 设置硬件时间。
- GetWakeupTime: 读取唤醒定时器时间。
- SetWakeupTime: 设置唤醒定时器时间。

#### 读写系统变量

读取、设置系统变量。

- GetVariable: 读取变量值。
- GetNextVariableName: 遍历系统变量。
- SetVariable: 更新、创建变量。

#### 虚拟内存服务

转换物理地址为虚拟地址。

- SetVirtualAddressMap: 设置系统内存映射。
- ConvertPointer: 查询指定物理地址的虚拟地址。

#### 其他服务

重启系统、获取系统的下一个单调增值。

- GetNextHighMonotonicCount: 获取系统的下一个单调增值。
- ResetSystem: 重启系统。
- UpadteCapusule
- QueryCapsuleCapabilities
- QueryVariableInfo: 查询关于 EFI 变量存储的信息。



### 时钟中断

UEFI 用事件机制取代了 BIOS 的中断机制，不在向开发者提供中断接口，但其核心还是使用了时钟中断。

UEFI 时钟中断时调用时钟中断函数 CoreTimerTick() , 该函数在更新完系统时间后，会检查事件列表中第一个事件是否到期，如果到期，则触发 mEfiCheckTimerEvent 事件，该事件会检查并触发所有到期的事件。



### 设备路径

系统中每一个设备都通过一个唯一的路径来表示，这个路径就是设备路径。

设备路径中的每个节点称为设备节点，设备路径的随后一个节点为设备结束节点。

每个设备节点都有一个共同的属性 EFI_DEVICE_PATH_PROTOCOL , 用面向对象的角度来看，EFI_DEVICE_PATH_PROTOCOL 是所有设备节点的基类。

EFI_DEVICE_PATH_PROTOCOL：
- UINT8 Type: 主类型
- UINT8 SubType: 次类型
- UINT8 Length[2]: 节点字节数

UEFI 设备类型：
- 0x1: 硬件设备
  - 0x1: PCI 设备
  - 0x2: PCCARD 设备
  - 0x3: 内存映射设备
  - 0x4: vendor 自定义设备
  - 0x5: 控制器设备
- 0x2: ACPI 设备 Type=0x02
  - 0x1: ACPI 设备
  - 0x2: 扩展 ACPI 设备
  - 0x3: _ADR 设备
- 0x3: Messaging 设备 Type=0x03
  - 0x1: ATAPI 设备
  - 0x2: SCSI 设备
  - 0x3: 光纤
  - 0x4: 1394
  - 0x5: USB
  - 0xc: IPv4
  - 0xd: IPv6
  - 0xf: USB
  - 0x12: SATA
  - 0x14: 802.1q
- 0x4: 介质设备 Type=0x04
  - 0x1: 硬盘
  - 0x2: 光驱
  - 0x3: vender 自定义设备
  - 0x4: 文件路径
  - 0x5: 介质 Protocol
  - 0x6: 固件文件路径
  - 0x7: 固件卷
- 0x5: BIOS 启动设备
- 0x7F: 结束节点设备
  - 整个设备路径结束
  - 前一个设备路径结束；新的设备路径开始



### 硬盘

硬盘的分区方法有两种: MBR (Master Boot Record) 和 GPT (GUID Partition Table)。

#### MBR

MBR 位于硬盘的 0 号扇区，记录了该硬盘的分区方式。由于只有 4 个分区表项目，所以最多分层 4 个主分区。由于分区表的起始扇区号只占 4 个字节，所以这种分区方式做多可以用到硬盘前 4G 的扇区，也就是 2T 空间。

结构（单位：字节）：

- 0~439   ：启动代码
- 440~443 ：磁盘标识符
- 444~445 ：0
- 446~509 ：四个分区表项目
  - 0     ：引导标志 0x00 表示非活动分区；0x80 表示活动分区
  - 1~3   ：起始 CHS （目前不适用）
  - 4     ：分区类型、文件系统类型
  - 5~7   ：结束 CHS （目前不适用）
  - 8~11  ：起始扇区号
  - 12~15 ：扇区个数
- 510~511 ：0x55 和 0xAA 。

### GPT

GPT 支持 64 位寻址，支持更大的容量、更多的分区和安全性。

扇区结构（单位：扇区、字节）：

- 0     ：保护性 MBR，主要用于兼容 MBR，但是 启动代码区域对 UEFI 无效；分区表第一项必须包含整个硬盘，引导标志必须为 0 ，分区类型必须为 0xEE ；其他分区必须为 0 。
- 1     ：主 GPT 头
  - 0~7   : "EFI PART" 字符串
  - 8~11  : 版本号
  - 12~15 : 本表字节数
  - 16~19 : 表头 CRC32 校验码
  - 20~23 : 0
  - 24~31 : 本表所在扇区
  - 32~39 : 本表所在扇区备份
  - 40~47 : 可用于分区的第一个扇区地址
  - 48~55 : 可用于分区的最后一个扇区地址
  - 56~71 : 硬盘 GUID
  - 72~79 : 分区表所在扇区
  - 80~83 : 分区表项数目
  - 84~87 : 每个分区表项占用的字节数，必须是 128 的 2^n 倍
  - 88~91 : 分区表 CRC32 校验码
  - 92~511: 0
- 2     ：分区表项 1~4
  - 0~15  : 分区类型 GUID
  - 16~31 : 分区标志 GUID
  - 32~39 : 分区首扇区
  - 40~47 : 分区尾扇区
  - 48~55 : 属性
    - 0    : 必须分区，置位表示该分区对系统至关重要
    - 1    : 无 BlockIo 分区，置位表示 UEFI 不为该分区生成块设备
    - 2    : 传统 BIOS 引导，只用于 BIOS 使用
    - 3~47 : 保留区 1
    - 48~63: 保留区 2
  - 55~128: 0
- 3~33  ：分区表项 5~128
- 34~-34：内容扇区
- -33~-3：备份分区表 1~4
- -2    ：备份分区表 5~128
- -1    ：备份 GPT 头

#### 硬盘 Protocol

- BlockIo
  - 功能：按块访问硬盘的阻塞函数。
  - EFI_BLOCK_IO_PROTOCOL:
    - UINT64 Revision: 版本号
    - EFI_BLOCK_IO_PROTOCOL Media: 设备信息
      - UINT32 MediaId
      - BOOLEAN RemovableMedia
      - BOOLEAN MediaPresent: 设备中是否有介质
      - BOOLEAN LogicalPartition: 该介质是否是分区（TRUE），还是整个设备（FALSE）
      - BOOLEAN ReadOnly
      - BOOLEAN WriteCaching: 是否用 cache 方式写硬盘
      - UINT32 BlockSize
      - UINT32 IoAlign: 读写缓冲区地址对其字节数，0 或 2^n
      - EFI_LBA LastBlock: 设备最后一个扇区地址
      - EFI_LBA LowestAlignedLba
      - UINT32 LogicalBlocksPerPhysicalBlock
      - UINT32 OptimalTransferLengthGranularity
    - EFI_BLOCK_RESET Reset: 重置设备
    - EFI_BLOCK_READ ReadBlocks: 读扇区
      - IN EFI_BLOCK_IO_PROTOCOL *This
      - IN UINT32 MediaId: 设备中的介质号
      - IN EFI_LBA LBA: 起始扇区号
      - IN UINTN BufferSize: 读取字节数（块的整数倍）
      - OUT VOID *Buffer: 目标缓存区缓存区由开发者开辟和释放
    - EFI_BLOCK_WRITE WriteBlocks: 写扇区
      - IN EFI_BLOCK_IO_PROTOCOL *This
      - IN UINT32 MediaId: 设备中的介质号
      - IN EFI_LBA LBA: 起始扇区号
      - IN UINTN BufferSize: 写入字节数（块的整数倍）
      - OUT VOID *Buffer: 目标缓存区缓存区由开发者开辟和释放
    - EFI_BLOCK_FLUSH FlushBlocks: 将设备缓存中的数据全部更新到介质中
      - IN EFI_BLOCK_IO_PROTOCOL *This
- BlockIo2
  - 功能：按块访问硬盘的异步函数。
  - EFI_BLOCK_IO2_PROTOCOL
    - EFI_BLOCK_IO_MEDIA * Media
    - EFI_BLOCK_IO_RESET_EX Reset
    - EFI_BLOCK_IO_READ_EX ReadBlocksEx
      - IN EFI_BLOCK_IO2_PROTOCOL * This
      - IN UINT32 MediaId
      - IN EFI_LBA LBA
      - IN OUT EFI_BLOCK_IO2_TOKEN * Token
      - IN UINTN BufferSize
      - OUT VOID * Buffer
    - EFI_BLOCK_IO_Write_EX WriteBlocksEx
      - IN EFI_BLOCk_IO2_PROTOCOL * This
      - IN UINT32 MediaId
      - IN EFI_LBA LBA
      - IN OUT EFI_BLOCK_IO2_TOKEN * Token
      - IN UINTN BufferSize
      - OUT VOID * Buffer
    - EFI_BLOCK_IO_FLUSH_EX FlushBlocksEx
      - IN EFI_BLOCK_IO2_PROTOCOL * This
      - IN OUT EFI_BLOCK_IO2_TOKEN * Token
  - EFI_BLOCK_IO2_TOKEN: 事务
    - EFI_EVENT Event: 事务对应的事件
    - EFI_STATUS TransactionStatus: 事务完成后的状态
- DiskIo
  - 功能：按字节访问硬盘的阻塞函数。
  - EFI_DISK_IO_PROTOCOL
    - UINT64 Revision
    - EFI_DISK_READ ReadDisk
      - IN EFI_DISK_IO_RPOTOCOL * This
      - IN UINT32 MediaId
      - IN UINT64 Offset
      - IN UINTN BufferSize
      - OUT VOID * Buffer
    - EFI_DISK_WRITE WriteDisk
      - IN EFI_DISK_IO_PROTOCOL * This
      - IN UINT32 MediaId
      - IN UINT64 Offset
      - IN UINTN BufferSize
      - IN VOID * Buffer
- DiskIo2
  - 功能：按字节访问硬盘的异步函数。
  - EFI_DISK_IO2_PROTOCOL
    - UINT64 Revision
    - EFI_DISK_CANCEL_EX Cancel: 取消磁盘上用于等待的事务
      - EFI_DISK_IO2_PROTOCOL * This
    - EFI_DISK_READ_EX ReadDiskEx
      - IN EFI_DISK_IO_PROTOCOL * This
      - IN UINT32 MediaId
      - IN UINT64 Offset
      - IN OUT EFI_DISK_IO2_TOKEN * Token
      - IN UINTN BufferSize
      - OUT VOID *Buffer
    - EFI_DISK_WRITE_EX WriteDiskEx
      - IN EFI_DISK_IO2_PROTOCOL * This
      - IN UINT32 MediaId
      - IN UINT64 Offset
      - IN OUT EFI_DISK_IO2_TOKEN * Token
      - IN UINTN BufferSize
      - IN VOID *Buffer
    - EFI_DISK_FLUSH_EX FlushDiskEx
      - IN EFI_DISK_IO2_PROTOCOL * This
      - IN OUT EFI_DISK_IO2_TOKEN * Token
  - EFI_DISK_IO2_TOKEN
    - EFI_EVENT Event: 无论事务成功或失败，事务结束后，Event 会触发
    - EFI_STATUS TransactionStatus: 表示 Event 触发后事务的状态
- PassThrough
  - 功能：构建命令包实现复杂操作。
  - EFI_ATA_PASS_THRU_PROTOCOL
    - EFI_ATA_PASS_THRU_MODE * Mode: 设备模式
    - EFI_ATA_PASS_THRU_PASSTHRU PassThru: 向 ATA 设备发送 ATA 命令
    - EFI_ATA_PASS_THRU_GET_NEXT_PORT GetNextPort
    - EFI_ATA_PASS_THRU_GET_NEXT_DEVICE GetNextDevice
    - EFI_ATA_PASS_THRU_BUILD_DEVIC_PATH BuildDevicePath
    - EFI_ATA_PASS_THRU_GET_DEVICE GetDevice
    - EFI_ATA_PASS_THRU_RESET_PORT ResetPort
    - EFI_ATA_PASS_THRU_RESET_DEVICE ResetDevice
  - EFI_SCSI_PASS_THRU_PROTOCOL
    - EFI_SCSI_PASS_THRU_MODE * Mode: 设备模式
    - EFI_SCSI_PASS_THRU_PASSTHRU PassThru: 向 SCSI 设备发送 SCSI 命令
    - EFI_SCSI_PASS_THRU_GET_NEXT_DEVICE GetNextDevice
    - EFI_SCSI_PASS_THRU_BUILD_DEVICE_PATH BuildDevicePath
    - EFI_SCSI_PASS_THRU_GET_TARGET_LUN GetTargetLun
    - EFI_SCSI_PASS_THRU_RESET_CHANNEL ResetChannel
    - EFI_SCSI_PASS_THRU_RESET_TARGET ResetTarget

#### 文件系统

UEFI 内置了 FileSystemIo (EFI_SIMPLE_FILE_SYSTEM_PROTOCOL) 用于操作 FAT 文件系统。 FileSystemIo 建立在 DiskIo 的基础上。

- FileSystemIo
  - 功能: 操作 FAT 文件系统。
  - EFI_SIMPLE_FILE_SYSTEM_PROTOCOL
    - UINT64 Revision
    - EFI_SIMPLE_FILE_SYSTEM_PROTOCOL_OPEN_VOLUME OpenVolume: 获取根目录指针
      - IN EFI_SIMPLE_FILE_SYSTEM_PROTOCOL * This
      - OUT EFI_FILE_PROTOCOL ** Root

- 文件对象
  - 功能: 对文件进行操作。
  - EFI_FILE_PROTOCOL
    - UINT64 Revision
    - EFI_FILE_OPEN Open
    - EFI_FILE_CLOSE Close
    - EFI_FILE_DELETE Delete
    - EFI_FILE_READ Read
    - EFI_FILE_WRITE Write
    - EFI_FILE_GET_POSITION GetPosition
    - EFI_FILE_SET_POSITION SetPosition
    - EFI_FILE_GET_INFO GetInfo
    - EFI_FILE_SET_INFO SetInfo
    - EFI_FILE_FLUSH Flush
    - EFI_FILE_OPEN_EX OpenEx
    - EFI_FILE_READ_EX ReadEx
    - EFI_FILE_WRITE_EX WriteEx
    - EFI_FILE_FLUSH_EX FlushEx



---
Power by Internet.
