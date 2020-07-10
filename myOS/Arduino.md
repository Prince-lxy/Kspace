# Arduino

> Arduino 是一款便捷灵活、方便上手的开源电子原型平台。

## Contents / Mind mapping
- **1 概述**
- **2 安装**
- **3 程序结构**
- **4 库函数**
  - **4-1 IO 函数**

---

### 1 概述

Arduino是一个基于易用硬件和软件的原型平台（开源）。它由可编程的电路板（称为微控制器）和称为Arduino IDE（集成开发环境）的现成软件组成，用于将计算机代码写入并上传到物理板。

**特点**
- Arduino板卡能够读取来自不同传感器的模拟或数字输入信号，并将其转换为输出，例如激活电机，打开/关闭LED，连接到云端等多种操作。
- 你可以通过Arduino IDE（简称上传软件）向板上的微控制器发送一组指令来控制板功能。
- 与大多数以前的可编程电路板不同，Arduino不需要额外的硬件（称为编程器）来将新代码加载到板上。你只需使用USB线即可。
- 此外，Arduino IDE使用C++的简化版本，使其更容易学习编程。
- 最后，Arduino提供了一个标准的外形规格，将微控制器的功能打破成更易于使用的软件包。



### 2 安装

- 硬件安装：利用 USB 线将开发板与电脑进行连接。
- 软件安装：通过[下载 Arduino IDE](https://www.arduino.cc/en/Main/Software) 来处理开发板的连接和镜像文件的传输与调试。



### 3 程序结构

Arduino软件是开源的。Java环境的源代码在GPL下发布，C/C++微控制器库在LGPL下。

Arduino 程序以 `Void setup()` 和 `Void loop()` 函数为主体。程序启动时通过 setup 函数来初始化变量，引脚模式，启用库等。setup()函数只能在Arduino板的每次上电或复位后运行一次。而 loop 函数作为程序的主体，用于主动控制 Arduino 板。




### 4 库函数

#### 4-1 IO 函数
- pinMode(pin, mode)
  - 功能：将特定的引脚配置为输入或输出。
  - 参数
    - pin: 引脚编号
    - mode: INPUT(悬空状态, 随机输入状态), OUTPUT(输出状态), INPUT_PULLUP(带有上拉电阻的输入状态)
- digitalRead(pin)
  - 功能：检测引脚是否有电压输入，返回 LOW 表示没有电压输入，返回 HIGH 表示有。
  - 参数
    - pin: 引脚编号
- digitalWrite(pin, mode)
  - 功能：向数字引脚写入 HIGH 或 LOW 值。
  - 参数
    - pin：引脚编号
    - mode: HIGH(高电平 5V/3.3V), LOW(低电平 0V)
  - 特性
    - 当引脚处于 INPUT/INPUT_PULLUP 模式时，本函数用于启用(HIGH)或禁止(LOW)引脚内部上拉电阻。
    - 当引脚处于 OUTPUT 模式时，本函数用于设定相应的电平值。
- analogRead(pin)
  - 功能：获取 "analog In" 引脚的输入电压值。此函数返回 0~1023 之间的数字，来对应 0~5 伏特之间的电压。例如，如果施加到 0 号引脚的电压为 2.5V，则 analogRead(0) 返回 512。
  - 参数
    - pin: 引脚编号
- analogReference(type)
  - 功能：配置用于模拟输入的参考电压。
  - 参数
    - type
      - DEFAULT: 5伏(5V Arduino板)或3.3伏(3.3V Arduino板)的默认模拟参考值。
      - INTERNAL: 内置参考，在 ATmega168 或 ATmega328 上等于 1.1 伏特，在 ATmega8 上等于 2.56 伏特(不适用于Arduino Mega)。
      - INTERNAL1V1: 内置1.1V参考(仅限Arduino Mega)。
      - INTERNAL2V56: 内置2.56V参考(仅限Arduino Mega)。
      - EXTERNAL: 施加到AREF引脚的电压(仅限0到5V)用作参考。



---
Power by Internet.
