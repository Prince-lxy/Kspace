# Arduino

> Arduino 是一款便捷灵活、方便上手的开源电子原型平台。

## Contents / Mind mapping
- **1 概述**
- **2 安装**
- **3 程序结构**
- **4 库函数**
  - **4-1 IO 函数**
- **5 Arduino 进阶**
  - **5-1 脉冲宽度调制(PWM)**
  - **5-2 UART**
  - **5-3 I2C**
  - **5-4 SPI**

---

### 1 概述

Arduino是一个基于易用硬件和软件的原型平台（开源）。它由可编程的电路板（称为微控制器）和称为Arduino IDE（集成开发环境）的现成软件组成，用于将计算机代码写入并上传到物理板。

**特点**
- Arduino板卡能够读取来自不同传感器的模拟或数字输入信号，并将其转换为输出，例如激活电机，打开/关闭LED，连接到云端等多种操作。
- 你可以通过Arduino IDE（简称上传软件）向板上的微控制器发送一组指令来控制板功能。
- 与大多数以前的可编程电路板不同，Arduino不需要额外的硬件（称为编程器）来将新代码加载到板上。你只需使用USB线即可。
- 此外，Arduino IDE使用C++的简化版本，使其更容易学习编程。
- 最后，Arduino提供了一个标准的外形规格，将微控制器的功能打破成更易于使用的软件包。

**Arduion UNO 数据**

|属性|值|
|----|--|
|工作电压|5V|
|输入电压|接上USB时无须外部供电或外部7V~12V DC输入|
|输出电压|5V DC 输出和 3.3V DC 输出和外部电源输入|
|微处理器|ATmega328|
|Bootloader|Arduino Uno|
|时钟频率|16 MHz|
|输入电压( 推荐)|7-12V|
|输入电压(限制)|6-20V|
|支持USB接口协议及供电(不需外接电源)||
|支持ISP下载功能||
|数字I/O端口|14（6个PWM输出口, 3 45 6 9 10 11）|
|模拟输入端口|6 (A0 - A5)|
|UART| 0(RX) 1(TX)|
|直流电流 I/O 端口|40mA|
|直流电流 3.3V 端口|50mA|
|Flash 内存|32 KB (ATmega328) (0.5 KB 用于引导程序）|
|SRAM|2 KB (ATmega328)|
|EEPROM|1 KB (ATmega328)|
|尺寸|75x55x15mm|


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
- analogWrite(pin, value)
  - 功能：向引脚发送模拟值 (PWM 波)。
  - 参数
    - pin: 引脚编号
    - value: 占空比，0 (始终关断) 到 255 (始终导通) 之间。
- analogReference(type)
  - 功能：配置用于模拟输入的参考电压。
  - 参数
    - type
      - DEFAULT: 5伏(5V Arduino板)或3.3伏(3.3V Arduino板)的默认模拟参考值。
      - INTERNAL: 内置参考，在 ATmega168 或 ATmega328 上等于 1.1 伏特，在 ATmega8 上等于 2.56 伏特(不适用于Arduino Mega)。
      - INTERNAL1V1: 内置1.1V参考(仅限Arduino Mega)。
      - INTERNAL2V56: 内置2.56V参考(仅限Arduino Mega)。
      - EXTERNAL: 施加到AREF引脚的电压(仅限0到5V)用作参考。
- randomSeed(seed)
  - 功能：根据 seed 来重置随机数生成器。
  - 参数
    - seed：伪随机数表偏移值，可以通过 analogRead() 函数根据环境噪音来充当其数值。
- random(max)
  - 功能：生成 0 到 max 的伪随机数。
- random(min, max)
  - 功能：生成 min 到 max 的伪随机数。
- attachInterrupt(pin, ISR, mode)
  - 功能：指定中断处理程序和中断捕捉模式到对应的引脚上。
  - 参数
    - pin：引脚编号。
    - ISR：中断处理程序。
    - mode：中断捕捉模式。LOW (在引脚为低电平时触发中断), CHANGE(在引脚更改值时触发中断), FALLING (当引脚从高电平变为低电平时触发中断)。



### 5 Arduino 进阶

#### 5-1 脉冲宽度调制(PWM)

脉冲宽度调制用于调制脉冲的宽度，说白了就是调整一个周期内方波导通 (值为1) 的时间长度，这部分时间称为导通时间 (On-Time)；值为 0 的时间称为 (关断时间 Off-Time)；二者之和为周期；导通时间占总时间的比值为占空比。占空比越高脉冲的能量就越大。

程序中可以通过 `analogRead(pin)` 与 `analogWrite(pin, value)` 来读取和写入 PWM 信号。

- 例如
```
int ledPin = 9;					// LED connected to digital pin 9
int analogPin = 3;				// potentiometer connected to analog pin 3
int val = 0;					// variable to store the read value

void setup() {
   pinMode(ledPin, OUTPUT);			// sets the pin as output
}

void loop() {
   val = analogRead(analogPin);			// read the input pin, analogRead values go from 0 to 1023
   analogWrite(ledPin, (val / 4));		// analogWrite values from 0 to 255
}
```

#### 5-2 UART

Arduino 通过 Serial 对象来控制 UART 串口。

```
void setup() {
   Serial.begin(9600);				// 设定串口的波特率为 9600
}

void loop() {
   if (Serial.available()) {			// 检测是否有数据从 RX 引脚到来
      Serial.print("I received:");		// 将字符串 "I received:" 打印到 IDE 的串口监视窗口
      Serial.write(Serial.read());		// 将 RX 引脚读取到的信息写入 TX 引脚
   }
}
```

#### 5-3 I2C

内部集成电路（I2C）是用于微控制器和新一代专用集成电路之间的串行数据交换系统。当它们之间的距离很短（接收器和发射器通常在同一个印刷电路板上）时使用。通过两根导线建立连接。一个用于数据传输，另一个用于同步（时钟信号）。

在 I2C 中，一个设备始终是主设备，其余设备为从设备。主设备和从设备都可以作为接收器和发射器。

- 主发射器 & 从接收器

```
#include <Wire.h>				//引入 Wire 头文件

void setup() {
   Wire.begin();				// 注册为主设备
}

short age = 0;

void loop() {
   Wire.beginTransmission(2);			// 向 I2C 地址为 2 的从设备建立连接
   Wire.write("age is = ");
   Wire.write(age);				// 发送数据
   Wire.endTransmission();			// 结束传输
   delay(1000);
}
```

```
#include <Wire.h>				// 引入 Wire 头文件

void setup() {
   Wire.begin(2)				// 注册为地址为 2 的从设备
   Wire.onReceive(receiveEvent);		// 注册用于处理接收到信息的回调函数，检测到有数据从主设备传来后调用 receiveEvent 函数
   Serial.begin(9600);
}

void loop() {
   delay(250);
}

void receiveEvent(int howMany) {
   while (Wire.available() > 1) {
      char c = Wire.read();			// 将接收的数据存入字符 c
      Serial.print(c);
   }
}
```
- 主接收器 & 从发射器

```
#include <Wire.h>

void setup() {
   Wire.begin();				 // 注册为主设备
   Serial.begin(9600);
}

void loop() {
   Wire.requestFrom(2, 1);			// 向地址为 2 的从设备请求 1 个字节数据
   while (Wire.available()) {
      char c = Wire.read();			// 将接收到底数据存入字符 c
      Serial.print(c);
   }
   delay(500);
}
```

```
#include <Wire.h>

void setup() {
   Wire.begin(2);				// 注册为地址为 2 的从设备
   Wire.onRequest(requestEvent);		// 注册用于处理请求的回到函数，当接受到其他设备发来的请求信息时调用
}

Byte x = 0;

void loop() {
   delay(100);
}

void requestEvent() {
   Wire.write(x);				// 发送数据
   x++;
}
```

#### SPI

串行外部设备接口总线（SPI）是用于串行通信的系统，最多可使用四个引脚，一个引脚用于数据接收（MISO, 12），一个引脚用于数据发送（MOSI, 11），一个引脚用于同步（SCK, 13），另一个导体用于选择与之通信的设备（SS, 10）。它是一个全双工连接，这意味着数据是同时发送和接收的。

SPI 有四种工作模式：
- 模式0（默认值）：时钟通常为低电平（CPOL = 0），数据在从低电平到高电平（前沿）（CPHA = 0）的转换时采样。
- 模式1：时钟通常为低电平（CPOL = 0），数据在从高电平到低电平（后沿）（CPHA = 1）的转换时采样。
- 模式2：时钟通常为高电平（CPOL = 1），数据在从高电平到低电平（前沿）（CPHA = 0）的转换时采样。
- 模式3：时钟通常为高电平（CPOL = 1），数据在从低电平到高电平（后沿）（CPHA = 1）的转换时采样。

SPI 主机：

```
#include <SPI.h>

void setup (void) {
   Serial.begin(115200);
   digitalWrite(SS, HIGH);			// 关闭从机选择
   SPI.begin ();				// 通过将SCK，MOSI和SS设置为输出来初始化SPI总线，将SCK和MOSI拉低，将SS拉高。
   SPI.setClockDivider(SPI_CLOCK_DIV8);		// 将SPI时钟设置为系统时钟的八分之一
}

void loop (void) {
   char c;
   digitalWrite(SS, LOW);			// 开启从机选择
   for (const char * p = "Hello, world!\r" ; c = *p; p++) {
      SPI.transfer(c);				// 发送数据
      Serial.print(c);
   }
   digitalWrite(SS, HIGH);			// 关闭从机选择
   delay(2000);
}
```

SPI 从机：

```
#include <SPI.h>
char buff [50];
volatile byte indx;
volatile boolean process;

void setup (void) {
   Serial.begin (115200);
   pinMode(MISO, OUTPUT);			// 从机引脚输出设定
   SPCR |= _BV(SPE);				// 启动从机模式
   indx = 0;
   process = false;
   SPI.attachInterrupt();			// 开启中断
}

ISR (SPI_STC_vect) {				// 从机接收数据时的中断处理函数
   byte c = SPDR;				// 从寄存器 SPDR 读取数据
   if (indx < sizeof buff) {
      buff [indx++] = c;
      if (c == '\r')
         process = true;
   }
}

void loop (void) {
   if (process) {
      process = false;
      Serial.println (buff);
      indx = 0;
   }
}
```



---
Power by Internet.
