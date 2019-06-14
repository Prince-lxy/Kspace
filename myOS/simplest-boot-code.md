# 最简单的启动程序
> 在 BIOS 后显示 hello world!

## Contents / Mind mapping
- **1 主程序**
- **2 编译**
- **3 制作镜像**
- **4 测试**

---

### 1 主程序

主要功能：实现在 BIOS 运行后显示 “hello world!”。

```
; 文件名 boot.asm

org 7c00h					; 代码加载到 7c00h 处开始运行

; 调用 10h 中断来显示字符串
mov ax, cs
mov es, ax
mov ax, msg
mov bp, ax					; es:bp 表示字符串首地址
mov cx, msgLen				; cx 存字符长度
mov ax, 1301h				; AH=13h 表示向 TTY 显示字符，AL=01h表示显示属性
mov bx, 000fh				; BH=00h 表示页号，BL=0fh 表示颜色
mov dl, 0					; 列
int 10h						; 调用 10h BIOS 中断

msg: db "hello world!"		; 要显示的字符串
msgLen: equ $ - msg			; 字符串长度
times 510 - ($ - $$) db 0	; 填充剩余部分
dw 0aa55h					; 魔数
```



### 2 编译

利用 nasm 编译工具将源代码编译成可执行程序。

```
$ nasm boot.asm -o boot.bin
```



### 3 制作镜像

制作一个 1.44 软盘镜像，将 boot.bin 内容作为软盘镜像前 512 字节的内容。

```
$ dd if=boot.bin of=boot.img bs=512 count=1
$ dd if=/dev/zero of=boot.img seek=1 bs=512 count=2879
```



### 4 测试

通过搭建好的虚拟机进行测试。



---
Power by Internet.
