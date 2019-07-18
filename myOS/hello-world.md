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
;; 文件名 hello-world.asm

org 0x7c00

start:
	;; es:bp = 字符串首地址
	mov ax, cs
	mov es, ax
	mov ax, msg
	mov bp, ax

	mov ax, 0x1301		;; ah = 功能代码 al = 写模式
	mov bx, 0x000c		;; bh = 页码 bl = 颜色
	mov cx, msgLen		;; cx = 字符串长度
	mov dx, 0x0000		;; dh = 行 dl = 列
	int 0x10
	jmp $

msg: db "hello world, wooooooool!!!"
msgLen: equ $ - msg	

times 510 - ($ - start) db 0
dw 0xaa55
```



### 2 编译

利用 nasm 编译工具将源代码编译成可执行程序。

```
$ nasm hello-world.asm -o hello-world.bin
```



### 3 制作镜像

制作一个 1.44 软盘镜像，将 hello-world.bin 内容作为软盘镜像前 512 字节的内容。

```
$ dd if=hello-world.bin of=hello-world.img bs=512 count=1
$ dd if=/dev/zero of=hello-world.img seek=1 bs=512 count=2879
```



### 4 测试

通过搭建好的虚拟机进行测试。



---
Power by Internet.
