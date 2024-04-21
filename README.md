## 1 基础知识

- 汇编指令是机器指令的助记符，同机器指令一一对应
- 每一种CPU都有自己的汇编指令集
- CPU可以直接使用的信息在存储器中存放
- 在存储器中指令和数据没有任何区别，都是二进制信息
- 存储单元从零开始顺序编号
- 一个存储单元可以存储8个bit，及8位二进制数
- 每个CPU芯片都有很多管脚，这些管脚和总线相连。也可以说，这些管脚引出总线。一个CPU可以引出3中总线的宽度标志了这个CPU的不同方面的性能：
  - 地址总线：决定了CPU的寻址能力
  - 数据总线：决定了CPU与其他器件进行数据传送时的一次数据传送量
  - 控制总线：决定了CPU对系统中其他器件的控制能力



8080、8088、8086、80286、80386的地址总线宽度分别为

16、20、20、24、32。数据总线宽度分别为

8、8、16、16、32。



内存地址空间

最终运行程序的是CPU，我们用汇编语言编程的时候，必须要从CPU的角度考虑问题。对CPU来讲，系统中的所有存储器单元都处于一个统一的逻辑存储器中，它的容量受CPU寻址能力的限制。这个逻辑存储器就是我们所说的内存地址空间。



8086CPU的内存地址空间分配：

00000～9FFFF是主储存器地址空间（RAM）

A0000～BFFFF是显存地址空间

C0000～FFFFF是各类ROM地址空间

## 2 寄存器

8086CPU有14个寄存器：AX、BX、CX、DX、SI、DI、SP、BP、IP、CS、SS、DS、ES、PSW。它们都是16位的。指令的两个操作对象的位数应当是一致的。

### 物理地址

CPU在访问内存时，用一个基础地址和一个相对基础地址的偏移地址相加，给出内存单元的物理地址。

特别地，对于8086，**段地址x16+偏移地址=物理地址**。

### 段的概念

内存其实并没有分段，段的划分来自于CPU，由于8086CPU使用“基础地址（段地址x16）+偏移地址=物理地址”的方式给出内存单元的物理地址，使得我们可以用分段的方式来管理内存。

段的起始地址一定是16的整数倍，偏移地址为16位，变化范围为0～FFFFH，所以一个段的长度最大为64K。

一个物理地址可以表示成不同的端地址和偏移地址的组合。

对于21F60H的内存单元，一般描述成2000:1F60或2000段中的1F60单元。

一数据存放在20000H单元，现给定段地址为SA，若想用偏移地址寻到此单元。则SA应满足的条件是：最小1001H，最大2000H。

### 段寄存器

段地址在段寄存器中存放，8086有4个段寄存器：CS、DS、SS、ES。

### CP和IP

CS和IP是8086中两个最关键的寄存器，指示了CPU当前要读取的指令的地址。CS为代码段寄存器、IP为指令指针寄存器。任何时刻，CPU将CS:IP指向的内容当作指令执行。

读取和执行指令的步骤：

1. 从CS:IP指向的内存单元中读取指令，读取的指令进入指令缓冲器
2. IP=IP+所读取的指令长度，从而指向下一条指令
3. 执行指令。跳到步骤1，重复这个过程



8086CPU加电启动或复位后，CS和IP被设置为CS=FFFFH，IP=0000H，所以启动的时候从FFFF0H单元开始执行指令，这个地址通常是ROM-BIOS中的地址。

### 修改CS和IP的指令

mov指令被称为**传送指令**，它不能用于设置CS和IP。能够改变CS和IP内容的指令被统称为**转移指令**。其中最简单的就是jmp指令。

```assembly
jmp 2AE3:3	; CS=2AE3H，IP=0003H
jmp ax		; 类似 move IP, ax，仅修改IP的内容
```



### 实验1  查看CPU和内存，用机器指令和汇编指令编程

- Debug

Debug是DOS下的提供的实模式（8086方式）程序的调试工具。使用它，可以查看CPU各种寄存器中的内容、内存的情况和在机器码级跟踪程序的运行。[debug程序](https://www.jianshu.com/p/147be16882e9)

- Debug的功能
  - R：查看和修改寄存器
  - D：查看内存的内容
  - E：修改内存
  - U：将内存中的机器指令翻译成汇编指令
  - T：执行一条机器指令
  - A：以汇编指令的格式在内存中写入一条机器指令



## 3 寄存器（内存访问）

### DS和[offset]

DS寄存器通常用来存放要访问数据的段地址，8086CPU不支持将字面数值直接送入段寄存器的操作，所以需要先送入一般寄存器。下面的例子，将从1000:0读取数据到al寄存器：

```assembly
mov bx, 1000H
mov ds, bx
mov al, [0]
```

其中的`[0]`表示内存单元的偏移地址是0，它的段地址默认放在ds中。



mov, add, sub指令的形式：

```assembly
mov ax, 8
mov ax, bx
mov ax, [0]
mov [0], ax
mov ds, ax
mov ax, ds
mov [0], cs
mov ds, [0]
```



### CPU栈机制SS:SP

**任意时刻，SS:SP指向栈顶元素**。push和pop指令执行时，CPU从SS和SP中得到栈顶地址，出栈和入栈操作都是以字为单位的。栈从高地址向低地址方向增长。

push ax时，先将SP=SP-2，然后将ax的值送入SS:SP指向的字单元。

pop ax时，先将SS:SP指向的字单元处的数据送入ax，然后SP=SP+2。

8086CPU只记录栈顶，不知道栈的范围，栈空间的大小需要我们自己管理。

push和pop可以作用于寄存器、段寄存器和内存单元。其本质就是一种内存传送指令。但是mov指令只需要一步操作，push和pop指令却需要两步操作。



一段内存，可以既是代码的存储空间，又是数据的存储空间，还可以是栈空间，也可以什么也不是。关键在于CPU中寄存器的设置，即CS、IP、SS、SP、DS的指向。



### 实验2 用机器指令和汇编指令编程

d、e、a、u这些命令也可以直接使用段寄存器表示内存单元的段地址。

T命令在执行修改寄存器SS的指令时，下一条指令也紧接着被执行（涉及中断机制）。



## 4 第一个程序

### 源程序

汇编语言源程序中，包含两种指令，一种是汇编指令，一种是伪指令。汇编指令有对应的机器码，可以被编译为机器指令，最终为CPU所执行。伪指令没有对应的机器指令，是由编译器来执行的指令。

```assembly
assume cs:codesg

codesg segment
		mov ax, 0123H
		mov bx, 0456H
		add ax, bx
		add ax, ax
		
		mov ax, 4c00H
		int 21H
codesg ends

end
```

上面的例子中出现了3种伪指令：

- segment和ends定义一个，前面的codesg是段的名称
- end标记整个程序结束
- assume将某一个段寄存器和程序中的某一个段相关联

其中最后两条汇编指令，实现**程序返回**，暂时不用去理解。

### 编译和链接

[下载编译器masm和链接器link](https://github.com/xDarkLemon/DOSBox_MASM/tree/master/masm)



写如下测试程序1.asm

```assembly
assume cs:code
code segment

    mov ax, 2
    add ax, ax
    add ax, ax

    mov ax, 4c00H
    int 21H

code ends
end
```

我们在dosbox中运行masm，进行编译：

```
C:\>masm
Microsoft (R) Macro Assembler Version 5.10
Copyright (C) Microsoft Corp 1981, 1988. All rights reserved.

Source filename [.ASM]: 1
Object filename [1.OBJ]:
Source listing [NUL.LST]:
Cross-reference [NUL.CRF]:

 50180 + 463225 Bytes symbol space free
 
     0 Warning Errors
     0 Severe  Errors
```

其中Source listing和Cross-reference是编译过程的中间结果，可以直接按Enter不生成文件。

结束之后，可以看到目录下已经生成了1.OBJ。

接下来进行链接：

```
C:\>link
Microsoft (R) Overlay Linker Version 3.64
Copyright (C) Microsoft Corp 1983-1988. All rights reserved.

Object Modules [.OBJ]: 1
Run File [1.EXE]:
List File [NUL.MAP]:
Libraries [.LIB]:
LINK : warning L4021: no stack segment

C:\>
```

List File映射文件同样可以不生成，我们的程序也没有用到库。最后出现了一个警告，我们先不管。

结束之后，可以看到目录下已经生成了1.EXE。



简化的方式进行编译和链接

```
C:\>masm 1;

C:\>link 1;
```



### 程序执行

DOS中执行程序时，会找到一个容量足够的空闲内存区起始地址为SA:0，在这段内存的前256个字节，创建一个程序前缀段（PSP）的数据去，用于和被加载程序进行通信。然后把程序装入SA+10H:0中，然后将内存区的段地址存入ds，即DS=SA，初始化其他相关寄存器后设置CS:IP指向程序的入口SA+10H:0。



```
C:\>debug 1.exe
...
t
t
t
...
p
Program terminated normally
```

最后执行到int 21的时候要用p命令。



## 5 [BX]和loop命令

`mov ax, [bx]`，中的`[bx]`同样表示偏移地址，偏移地址在bx中，段地址在ds中。

后续用`(X)`表示寄存器或内存单元中的内容。使用`idata`表示数字常量。

loop指令：

- (cx)=(cx)-1
- cs中的值不为零，则转至标号处执行程序，为零则向下

```assembly
assume cs:code
code segment
	mov ax, 2
	mov cx, 11
s:	add ax, ax
	loop s
	
	mov ax, 4c00H
	int 21H
code ends

end
```

上面的代码计算2的12次方。



汇编源程序中，数据不能以字母开头。

[idata]解释为idata，要表示内存单元的偏移地址，需要显式地给出段寄存器，例如`ds:[0]`，或者类似`[bx]`默认段地址在ds中。



debug的g命令，执行到指定IP处，类似gdb中的until。

debug的p命令，自动重复执行loop，直到cx的值为0.



在DOS（实模式）下，可以直接用汇编去操作真实的硬件，但在运行于CPU保护模式下的操作系统中，不理会操作系统，用汇编语言去操作真实的硬件，是根本不可能的。

为了防止随意写内存导致系统问题，后续会使用0:200~0:2ff这256个字节的空间，这段空间一般DOS和其他合法的程序都不会使用。

### 实验4

1. 向内存0:200~0:23F依次传送数据0~63(3FH)，只能使用9条指令，包括实现程序返回的两条

   ```assembly
   assume cs:code
   code segment
       mov ax, 20H
       mov ds, ax
       mov cx, 40H         ; 初始循环次数
       mov bx, 0H
   s:  mov [bx], bx
       inc bx
       loop s
   
       mov ax, 4c00H
       int 21H
   code ends
   
   end
   ```

   

2. 将"mov ax, 4c00h"之前的程序指令复制到内存0:200处

   ```assembly
   assume cs:code
   code segment
       mov ax, cs
       mov ds, ax
       mov ax, 0020H
       mov es, ax
       mov bx, 0
       mov cx, 17H		; 需要查看机器码长度
   s:  mov al, [bx]
       mov es:[bx], al
       inc bx
       loop s
   
       mov ax, 4c00H
       int 21H
   code ends
   
   end
   ```

   

## 6 包含多个段的程序

前面的程序只有一个代码段，如果我们的程序需要其他空间来存放数据，该怎么办？0:200～0:2FF是相对安全的，但是只有256个字节。

在操作系统中，可以通过操作系统取得空间。

程序取得所需空间的方法有两种，一是在加载程序时为程序分配，再就是程序执行的过程中向系统申请。我们不讨论第二种情况。

### 在代码段中使用数据

```assembly
assume cs:code
code segment
    dw 0123h, 0456h, 0789h, 0abc, 0defh, 0fedh, 0cbah, 0987h

    mov bx, 0
    mov ax, 0
    mov cx, 8
s:  add ax, cs:[bx]
    add bx, 2
    loop s

    mov ax, 4c00H
    int 21H
code ends

end
```

dw定义字型数据，它们占了代码段的前16个字节，所以要执行程序中的指令，需要在debug中将IP设置为10h，从而使CS:IP指向程序中的第一条指令。

那么如何让这个程序在编译之后可以在系统中直接运行呢？我们可以在源程序中指明程序的入口所在：

```assembly
assume cs:code
code segment
    dw 0123h, 0456h, 0789h, 0abc, 0defh, 0fedh, 0cbah, 0987h

    start:  mov bx, 0
            mov ax, 0
            mov cx, 8
        s:  add ax, cs:[bx]
            add bx, 2
            loop s

            mov ax, 4c00H
            int 21H
code ends

end start
```

通过伪指令end，通知编译器程序的入口在什么地方。编译连接之后，被转化为一个入口地址，存储在可执行程序的描述信息中。

### 在代码段中使用栈

```assembly
assume cs:code
code segment
    dw 0123h, 0456h, 0789h, 0abc, 0defh, 0fedh, 0cbah, 0987h
    dw 0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0      ; 定义16个字节的字型数据
                                            ; 后面程序中用来当栈使用

    start:  mov ax, cs
            mov ss, ax
            mov sp, 30h                     ; 设置栈顶cs:30

            mov bx, 0
            mov cx, 8
        s:  push cs:[bx]                    ; 将前16字节的数据依次入栈
            add bx, 2
            loop s

            mov bx, 0
            mov cx, 8
        s0: pop cs:[bx]                     ; 将8个字型数据依次出栈
            add bx, 2
            loop s0

            mov ax, 4c00H
            int 21H
code ends

end start                                   ; 指明程序的入口在start处
```

这个程序通过使用栈，将数据逆序存放。



## 7 将数据、代码、栈放到不同的段

前面将它们放到一个段中，使程序显得混乱，且总共不能超过64KB。

```assembly
assume cs:code, ds:data, ss:stack
data segment

    dw 0123h, 0456h, 0789h, 0abc, 0defh, 0fedh, 0cbah, 0987h

data ends

stack segment

    dw 0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0      ; 定义16个字节的字型数据
                                            ; 后面程序中用来当栈使用
stack ends

code segment

start:  mov ax, stack
        mov ss, ax
        mov sp, 20h                     ; 设置栈顶stack:20

        mov ax, data
        mov ds, ax                      ; ds:bx指向data段的第一个单元

        mov bx, 0
        mov cx, 8
    s:  push [bx]                       ; 将前16字节的数据依次入栈
        add bx, 2
        loop s

        mov bx, 0
        mov cx, 8
    s0: pop [bx]                        ; 将8个字型数据依次出栈
        add bx, 2
        loop s0

        mov ax, 4c00H
        int 21H
code ends

end start                               ; 指明程序的入口在start处
```



注意，段用来做什么，完全是我们的安排，并不是定义了stack段，CPU就把它当作栈来处理了。assume关联也不会将cs指向code，我猜是让编译器、调试程序不要用对应空间，防止冲突。



## 7 更灵活的定位内存地址的方法

### and和or指令

按位与和按位或

```assembly
mov al, 01100011B
and al, 00111011B	; 结果00100011B

mov al, 01100011B
or  al, 00111011B	; 结果01111011B
```



字符形式给出数据

```assembly
data segment
 db 'unIX'
 db 'foRK'
data ends

code segment
 start: mov al, 'a'
 		mov bl, 'b'
code ends
```



大小写字母除了第5位外，其他位都一样，小写字母大20H。



### [bx+idata]

偏移地址是(bx)+200，下面几种写法都是可以的

```assembly
mov ax, [bx+200]
mov ax, [200+bx]
mov ax, 200[bx]
mov ax, [bx].200
```

### SI和DI

是8086CPU中和bx功能相近的寄存器，si和di不能分成两个8位寄存器来使用。

```assembly
mov ax, [si]
mov ax, [si+123]
mov ax, 16[si]
```

### [bx+si]和[bx+di]

偏移地址是(bx)+(si)

```assembly
mov ax, [bx+si]
mov ax, [bx][si]
```

### [bx+si+idata]和[bx+di+idata]

```assembly
mov ax, [bx+200+si]
mov ax, [200+bx+si]
mov ax, 200[bx][si]
mov ax, [bx].200[si]
mov ax, [bx][si].200
```

### 不同寻址方式的灵活运用

不同的寻址方式，使我们可以从更加结构化的角度来看待所要处理的数据。

多层循环的时候，在执行内存loop之前要先将cx的值保存起来。但是CPU中的寄存器数量毕竟是有限的，如果其他寄存器也都被使用了，那么就只能暂存到内存中了。

**一般来说，我们需要暂存数据的时候，我们应该使用栈**。这样就不需要记住将数据放到哪个单元了，程序不容易混乱。

### 实验6

下面的程序将每个单词的前4个字母改成大写。

```assembly
assume cs:code, ds:data, ss:stack

stack segment
    dw 0,0,0,0,0,0,0,0
stack ends

data segment
    db '1. display      '
    db '2. brows        '
    db '3. replace      '
    db '4. modify       '
data ends

code segment

start:  mov ax, stack
        mov ss, ax
        mov sp, 10h             ; 设置栈顶stack:10

        mov ax, data
        mov ds, ax              ; 设置数据段ds

        mov bx, 0               ; 每行的起始地址
        mov cx, 4               ; 外层循环计数

s0:     push cx
        mov si, 0
        mov cx, 4               ; 内存循环计数

s:      mov al, [bx+si+3]
        and al, 11011111b       ; 改成大写
        mov [bx+si+3], al

        inc si
        loop s

        add bx, 16              ; 指向下一行
        pop cx
        loop s0
        
        mov ax, 4c00H
        int 21H
code ends

end start                       ; 指明程序的入口在start处
```



## 8 数据处理的两个基本问题

本章主要是对前面所有内容的总结。计算机进行数据处理、运算的两个基本问题：

1. 处理的数据在什么地方
2. 要处理的数据有多长



后面将使用reg和sreg表示一个寄存器和一个段寄存器。

reg: ax,bx,cx,dx,ah,al,…,dh,dl,sp,bp,si,di

sreg: ds, ss, cs, ds

### bx、si、di和bp

1. 在8086CPU中，只有这4个寄存器可以用在`[]`里进行内存单元寻址。
2. 在`[]`中这4个寄存器可以单个出现，或只能以4种组合出现：bx和si、bx和di，bp和si、bp和di
3.  只要在`[]`中使用寄存器bp，而指令中没有显式地给出段地址，段地址就默认在ss中。

### 数据位置

汇编语言中用3个概念来表达数据的位置：

立即数、寄存器、段地址+偏移地址

### 寻址方式

| 寻址方式      | 含义               | 名称             | 常用格式举例                                                 |
| ------------- | ------------------ | ---------------- | ------------------------------------------------------------ |
| [idata]       | EA=idata           | 直接寻址         | [idata]                                                      |
| [bx]          | EA=(bx)            | 寄存器间接寻址   | [bx]                                                         |
| [bx+idata]    | EA=(bx)+idata      | 寄存器相对寻址   | 用于结构体：<br />[bx].idata<br />用于数组：<br />idata[si], idata[di]<br />用于二维数组<br />[bx]\[idata] |
| [bx+si]       | EA=(bx)+(si)       | 基址变址寻址     | 用于二维数组：<br />[bx]\[si]                                |
| [bx+si+idata] | EA=(bx)+(si)+idata | 相对基址变址寻址 | 用于表格结构中的数组项：<br />[bx].idata[si]<br />用于二维数组：<br />idata[bx]\[si] |

### 指令要处理的数据有多长

8086CPU指令可以处理两种尺寸的数据：byte和word。

1. 通过寄存器名指明长度，如`mov ax, 1`

2. 用操作符`xxx ptr`指明长度，如

   ```assembly
   mov word ptr ds:[0], 1
   mov byte ptr ds:[0], 1
   ```

3. 其他，有些指令默认了访问的是字单元还是字节单元，如push

### div指令

div是除法指令。

1. 除数：有8位和16位两种，在一个reg或内存单元中
2. 被除数：默认放在AX或DX+AX，如果除数是8位，被除数则为16位，默认在AX中；如果除数是16位，则被除数位32位，在DX+AX 中存放，DX存放高16位，AX存放低16位。
3. 结果：如果除数位8位，则AL存储除法操作的商，AH存储除法操作的余数；如果除数位16位，则AX存储除法操作的商，DX存储除法操作的余数。

```assembly
div byte ptr ds:[0]
div word ptr es:[0]
div byte ptr [bx+si+8]
div word ptr [bx+si+8]
```

### 伪指令dd

前面分别用db和dw定义字节型数据和字型数据，dd用来定义双字型数据。

### dup操作符

dup是一个操作符，也是由编译器识别处理的符号，配合db、dw、dd等数据定义伪指令配置使用，用来进行数据的重复。

```assembly
db 3 dup (0)		; 定义了3个字节，值都是0
db 3 dup (0, 1, 2)	; 9个字节，0,1,2,0,1,2,0,1,2
db 3 dup ('abc', 'ABC');
```

### 实验7

将data段中的数据格式化到table段中，并计算人均收入（取整），结果也按照下面的格式保存在table段中

`年份(4字节)|空格|收入(4)｜空格｜雇员数(2)|空|人均收入(2)|空格`



注意没有直接内存传送到内存的指令。

```assembly
assume cs:code

data segment
    ; 21个年份，共84字节
    db '1975','1976','1977','1978','1979','1980','1981'
    db '1982','1983','1984','1985','1986','1987','1988'
    db '1989','1990','1991','1992','1993','1994','1995'

    ; 21个收入，共84字节
    dd 16,22,384,1356,2390,8000,16000,24486,50065,97479
    dd 140417,197514,345980,590827,803530,1183000
    dd 1843000,2759000,3753000,4649000,5937000

    ; 21个雇员数
    dw 3,7,9,13,28,38,130,220,476,778,1001,1442,2258
    dw 2793,4037,5635,8226,11542,14430,15257,17800
data ends

table segment
    db 21 dup ('year summ ne ?? ')
table ends

stack segment
    dw 0, 0
stack ends

code segment

start:  mov ax, data
        mov ds, ax              ; 设置数据段ds
        mov ax, table
        mov es, ax

        mov ax, stack
        mov ss, ax
        mov sp, 16

        mov bx, 0               ; 年份/收入数组计数+4
        mov bp, 0               ; table数组计数+16
        mov di, 0               ; 雇员计数+2
        mov cx, 21              ; 外层循环计数

s:      push cx
        mov si, 0               ; 年份字符计数
        mov cx, 2               ; 内层循环计数

s1:     mov ax, [bx+si]         ; 拷贝年份一个字
        mov es:[bp][si], ax     ; 拷贝年份一个字
        mov ax, 84[bx][si]      ; 拷贝收入一个字
        mov es:[bp].5[si], ax   ; 拷贝收入一个字
        add si, 2
        loop s1

        mov ax, 168[di]         ; 拷贝雇员数
        mov es:[bp].0ah, ax     ; 拷贝雇员数

        mov dx, es:[bp].7       ; 高位在高地址
        mov ax, es:[bp].5       ; 低位在低地址
        div word ptr es:[bp].0ah         ; 计算人均收入

        mov es:[bp].0dh, ax     ; 拷贝商

        add bx, 4
        add di, 2
        add bp, 16
        pop cx
        loop s

        mov ax, 4c00H
        int 21H
code ends

end start                       ; 指明程序的入口在start处
```



## 9 转移指令的原理

**能够修改IP，或者同时修改CS和IP的指令统称为转移指令**。

8086CPU的转移行为有以下几类：

- 只修改IP时，称为段内转移，比如：`jmp ax`
- 同时修改CS和IP时，称为段间转移，如：`jmp 1000:0`

根据转移指令对IP的修改范围不同，段内转移又分为短转移和近转移：

- 短转移IP的修改范围为-128~127
- 近转移IP的修改范围为-32768~32767

8086CPU的转移指令分为以下几类：

- 无条件转移指令（如：jmp）
- 条件转移指令
- 循环指令（如：loop）
- 过程
- 中断

这些转移指令的前提条件可能不同，但是转移的基本原理是相同的。本章主要通过深入无条件转移指令jmp来理解CPU执行转移指令的基本原理

### offset操作符

是由编译器处理的符号，功能是取得标号的偏移地址，如

```assembly
start:	mov ax, offset start	; mov ax, 0
s:		mov ax, offset s		; mov ax, 3
```

如下程序将s处的一条指令复制到s0处

```assembly
assume cs:codesg
codesg segment
s:	mov ax, bx		; 机器码占两个字节
	mov si, offset s
	mov di, offset s0
	mov ax, cs:[si]
	mov cs:[di], ax
s0:	nop				; nop的机器码占1个字节
	nop
codesg ends
end s
```

### jmp指令

为无条件转移指令，可以只修改IP，也可以同时修改CS和IP。jmp指令需要给出两种信息：

- 转移的目的地址
- 转移的距离（段间转移、段内短转移、段内近转移）

### 依据位移进行转移的jmp指令

- `jmp short 标号`

这种格式的jmp指令实现的是段内短转移，对IP的修改范围是-128～127

```assembly
assume cs:codesg
codesg segment

start:	mov ax, 0
		jmp short s
		add ax, 1
s:		inc ax
codesg ends
```

该程序翻译成机器码之后

```assembly
-u
0BBD:0000 B80000	MOV		AX,0000
0BBD:0003 EB03		JMP		0008
0BBD:0005 050100	ADD		AX,0001
0BBD:0008 40		INC		AX
```

可以看到jmp指令机器码并没有包含转移的目的地址，而是要转移的位移（这里是3），用补码表示。

根据位移进行转移，方便了程序段在内存中的浮动装配。

- `jmp near 标号`

段内近转移，16位位移，用补码表示。

### 转移的目的地址在机器码中的jmp

`jmp far ptr 标号`

机器码中包含了转移的目的地址，如

```
-u
0BBD:0006 EA0B01BD0B	JMP		0BBD:010B
```

低地址是偏移地址，高地址是段地址。

### 转移地址在寄存器中的jmp

`jmp 16位reg`

用寄存器中的值，修改ip

### 转移地址在内存中的jmp

1. `jmp word ptr 内存单元地址`

   段内转移，内存单元地址处存放着一个字，是转移的目的偏移地址

2. `jmp dword ptr 内存单元地址`

   段间转移，内存单元地址处存放着两个字，低地址处的字是转移的目的偏移地址，高地址处的字是转移的目的段地址。

### jcxz指令

`jcxz 标号`

是有条件转移指令，所有的有条件转移指令都是短转移。

当(cx)为0时转移，否则啥都不做，程序向下执行。

### loop指令

`loop 标号`

循环指令，所有的循环指令都是短转移。

当(cx)=(cx-1)；如果(cx)!=0，转移。

### 实验9

80*25彩色字符模式显示缓冲区的结构：

B8000H~BFFFFH共32KB空间，为80*25彩色字符模式的显示缓冲区，向这个地址空间写入数据，写入的内容将立即出现在显示器上。

在80*25彩色字符模式下，显示器可以显示25行，每行80个字符，每个字符可以有256种属性（背景色、前景色、闪烁、高亮等）。

这样一个字符在显示缓冲区中就要占用两个字节，分别存放ASCII码和属性。80*25模式下，一屏的内容在显示缓冲区中共占4000个字节。

显示缓冲区分为8页，每页4KB，显示器可以显示任意一页的内容，一般情况下，显示第0页的内容（B8000H～B8F9FH）。

在一页显示缓冲区中：

偏移000～09F对应显示器上的第1行；

偏移A00～13F对应显示器上的第2行；

偏移140～1DF对应显示器上的第3行；

属性字节的格式：

```
         7  6 5 4 4 2 1 0
含义      BL R G B I R G B
         闪烁背景 高亮 前景
```



编程：在屏幕中间分别显示绿色、绿底红色、白底蓝色的字符串'welcome to masm!'



注意`[bp]`默认是ss段。

```assembly
assume cs:codesg, ds:datasg, ss:stack

datasg segment
    db 'welcome to masm!'
    ;  绿色       绿底红色   白底蓝色
    db 00000010b, 00100100b, 01110001b
datasg ends

stack segment
    dw 0,0
stack ends

codesg segment
start:  mov ax, datasg
        mov ds, ax      ; 数据段
        mov ax, stack
        mov ss, ax      ; 栈段
        mov sp, 16
        mov ax, 0b800h  ; 显示段
        mov es, ax

        mov bx, 0       ; 字符串显示开始地址
        mov cx, 11
s0:     add bx, 160
        loop s0
        add bx, 64      ; 初始化为11行64列内存开始地址

        mov si, 0       ; 行数计数
        mov cx, 3       ; 外层循环，3行

s1:     push cx
        mov dl, 16[si]  ; 获取属性
        mov di, 0       ; 显示一行内偏移计数
        mov bp, 0       ; 字符串一行内偏移计数
        mov cx, 16      ; 内存循环，16个字符
s2:     mov al, ds:[bp]         ; 字符
        mov es:[bx][di], al
        mov es:[bx][di].1, dl   ; 属性

        add di, 2       ; 显存向后移动2个字节
        inc bp          ; 字符串向后移动1个字节
        loop s2

        inc si          ; 行数+1
        add bx, 160     ; 显存指向下一行
        pop cx
        loop s1

        mov ax, 4c00h
        int 21h
codesg ends

end start
```



## 10 CALL和RET指令

这两个都是转移指令，经常被用来实现子程序的设计。

### ret和retf

ret指令用栈中的数据，修改IP，从而实现近转移

retf指令用栈中的数据，修改IP和CS，从而实现远转移。先弹出IP，再弹出CS。

### call指令

1. 将当前的IP或CS和IP压入栈中
2. 转移

call指令不能实现短转移，除此之外，call指令实现转移的方法和jmp指令的原理相同。

### 依据位移进行转移的call指令

`call 标号`，将当前的IP压栈后，转到标号处执行指令

16位位移，用补码表示

### 转移的目的地址在机器码中的call

`call far ptr 标号`，实现的是段间转移

将当前CS和IP依次入栈，然后转移。

### 转移地址在寄存器中的call

`call 16位reg`。先将当前IP压栈，然后转移到16位寄存器中的偏移地址处。

### 转移地址在内存中的call

1. `call word ptr 内存单元地址`

   先将当前IP压栈，然后转移到内存单元地址中指定的IP

2. `call dword ptr 内存单元地址`

   先依次将CS和IP入栈，然后转移到内存单元地址中指定的IP和CS。

### call和ret配合使用

```assembly
assume cs:code
code segment
start:	mov ax, 1
		mov cx, 3
		call s
		mov bx, ax		; (bx)=8
		mov ax, 4c00h
		int 21h
s:		add ax, ax
		loop s
		ret
code ends
end start
```

上面的程序调用s子程序，计算2的3次方。

### mul指令

- 两个相乘的数要么都是8位，要么都是16位。如果是8位，一个默认放在AL中，另一个放在8位reg或内存字节单元中；如果是16位，一个默认在AX，另一个放在16位reg或内存字单元中。

- 结果：如果是8位乘法，结果默认放在AX中；如果是16位乘法，结果高位默认在DX，低位在AX中。

```
mul reg
mul 内存单元
```

### 参数和结果传递

计算N的3次方

```assembly
assume cs:codesg, ds:datasg

datasg segment
    dw 1,2,3,4,5,6,7,8
    dd 8 dup (0)
datasg ends

codesg segment
start:  mov ax, datasg
        mov ds, ax      ; 数据段
        mov si, 0       ; 第一组word单元
        mov di, 16      ; 第二组dword单元

        mov cx, 8
s:      mov bx, [si]
        call cube
        mov [di], ax
        mov [di].2, dx

        add si, 2
        add di, 4
        loop s

        mov ax, 4c00h
        int 21h

        ; 说明: 计算N的3次方
        ; 参数: (bx)=N
        ; 结果: (dx:ax)=N^3
cube:   mov ax, bx
        mul bx
        mul bx
        ret

codesg ends

end start
```



### 批量数据的传递

寄存器数量终究有限，如果有很多参数需要传递，可以将批量数据放到内存中，然后将内存空间的首地址放在寄存器中，传递给需要的子程序。批量返回结果也可以用同样的方法。



注意：and不要写成add！



```assembly
assume cs:codesg, ds:datasg

datasg segment
    db 'conversation'
datasg ends

codesg segment
start:  mov ax, datasg
        mov ds, ax      ; 数据段
        mov si, 0       ; 指向字符串所在空间的首地址
        mov cx, 12      ; cx存放字符串长度

        call upcase

        mov ax, 4c00h
        int 21h

        ; 说明: 将ds:si为首地址的字符串转化为大写
        ; 参数: (cx)=N 表示字符串长度
        ; 返回: 无
upcase: and byte ptr [si], 11011111b
        inc si
        loop upcase
        ret

codesg ends

end start
```



### 使用栈来传递参数

与高级语言编译器的工作原理密切相关。

```assembly
; 说明: 计算(a-b)^3，a和b为字型数据
; 参数: 进入子程序时，栈顶存放IP，后面依次存放a和b
; 结果: (dx:ax)=(a-b)^3
difcube:	push bp
			mov bp, sp
			mov ax, [bp+4]	; a的值送入ax
			sub ax, [bp+6]	; a - b
			mov bp, ax
			mul bp
			mul bp
			
			pop bp
			ret 4
```

ret n的含义可以描述为：

```assembly
pop ip
add sp, 4
```

栈的情况

```
                                   bp    ip    a     b
1000:0000 00 00 00 00 00 00 00 00 xx xx xx xx 03 00 01 00 
```

下面来看据称eC语言的例子。注意C语言中，局部变量也在栈中存储。

```c
void add(int, int, int);

main()
{
    int a=1;
    int b=2;
    int c=0;
    add(a,b,c);
    c++;
}

void add(int a, int b, int c)
{
    c = a + b;
}
```

编译成汇编程序

```assembly
mov bp, sp
sub sp, 6
mov word ptr [bp-6], 0001;	; main局部变量a
mov word ptr [bp-4], 0002;	; main局部变量b
mov word ptr [bp-2], 0000;	; main局部变量c

push [bp-2]		; 参数c压栈
push [bp-4]		; 参数b压栈
push [bp-6]		; 参数a压栈

call addfn
add sp, 6		; 还原栈，清除参数
inc word ptr [bp-2]

addfn:	push bp
		mov bp, sp
		mov ax, [bp+4]
		add ax, [bp+6]
		mov [bp+8], ax
		mov sp, bp
		pop bp
		ret
```



### 寄存器冲突的问题

在子程序的开头，将子程序所有用到的寄存器中的内容都保存起来，在子程序防止前再恢复。注意寄存器入栈和出栈的顺序相反。

### 实验10：编写子程序

#### 显示字符串

```assembly
; 名称 show
; 功能 在指定的位置，用指定的颜色显示一个用0结束的字符串
; 参数 (dh)=行号(0~24)，(dl)=列号(0~79)
;      (cl)=颜色，ds:di指向字符串的首地址
; 返回 无
assume cs:codesg, ds:datasg, ss:stack

datasg segment
    db 'welcome to masm!', 0
datasg ends

stack segment
    dw 8 dup (0)
stack ends

codesg segment
start:  mov ax, datasg
        mov ds, ax      ; 数据段
        mov ax, stack
        mov ss, ax      ; 栈段
        mov sp, 16

        mov si, 0
        mov dh, 8
        mov dl, 3
        mov cl, 00000010b

        call show

        mov ax, 4c00h
        int 21h
        
show:   push es
        push ax
        push bx
        push cx
        push si

        mov ax, 0b800h  ; 显示段起始地址
        mov es, ax

        mov al, 160     ; 计算显示起始位置
        mul dh
        mov bx, ax
        mov al, 2
        mul dl
        add bx, ax      ; 显示起始位置

        mov al, cl      ; 保存颜色

        mov cx, 0
s:      mov cl, [si]
        jcxz ok
        mov es:[bx], cl     ; 字符
        mov es:[bx].1, al   ; 颜色
        add bx, 2
        inc si
        jmp short s

ok:     pop si
        pop cx
        pop bx
        pop ax
        pop es
        ret

codesg ends

end start
```



#### 解决除法溢出问题

如果div做除法时，结果的商过大，超出了寄存器所能存储的范围，将引发CPU的一个内部错误：**除法溢出**。

这时候不能直接用div来计算，可以子程序分解除法运算：

`X/N=int(H/N)*65536 + [rem(H/N)*65536+L]/N`



```assembly
; 名称 divdw
; 功能 进行不会产生溢出的除法运算，被除数为dword，除数为word，结果为dword
; 参数 (ax)=dword型数据的低16位
;      (dx)=dword型数据的高16位
;      (cx)=除数
; 返回 (dx)=结果的高16位，(ax)=结果的低16位
;      (cx)=余数
; 公式 X/N = int(H/N)*65536 + [rem(H/N)*65536+L]/N
assume cs:codesg, ss:stack
stack segment
    dw 8 dup (0)
stack ends

codesg segment
start:  mov ax, datasg
        mov ds, ax      ; 数据段
        mov ax, stack
        mov ss, ax      ; 栈段
        mov sp, 16

        mov ax, 4240h
        mov dx, 000fh
        mov cx, 0ah

        call divdw

        mov ax, 4c00h
        int 21h

divdw:  push bx
        push di

        mov bx, ax      ; 低位L先保存
        mov ax, dx      ; 高位H移到ax
        mov dx, 0
        div cx          ; H/N

        mov di, ax      ; int(H/N)

        mov ax, bx      ; 低位L移到ax，rem(H/N)已经在dx中
        div cx          ; [rem(H/N)*65536+L]/N

        mov cx, dx      ; 余数
        mov dx, di      ; 商高位，低位已经在ax中

        pop di
        pop bx
        ret

codesg ends

end start
```



#### 数值显示

将word型数据以十进制的形式显示。字符0~9对应的ASCII为30H~39H。

主程序如下

```assembly
codesg segment
start:  mov ax, datasg
        mov ds, ax      ; 数据段
        mov si, 0
        mov ax, stack
        mov ss, ax      ; 栈段
        mov sp, 16

        mov ax, 12666
        call dtoc

        mov dh, 8
        mov dl, 3
        mov cl, 00000010b
        call show

        mov ax, 4c00h
        int 21h
```

dtoc函数

```assembly
; 名称 dtoc
; 功能 将word型数据转变为表示十进制数的字符串，字符串以0为结尾符
; 参数 (ax)=word型数据
;      ds:si 指向字符串的首地址
; 返回 无
dtoc:   push ax
        push bx
        push cx
        push dx
        push si

dtocs:  mov dx, 0
        mov bx, 10
        div bx

        add dl, 30h  ; 余数转成字符
        mov [si], dl ; 余数保存
        inc si
        mov cx, ax
        jcxz dtocok  ; 商为0结束
        jmp short dtocs

dtocok:
        mov byte ptr [si], 0 ; 补0
        pop si
        pop dx
        pop cx
        pop bx
        pop ax
        call rev
        ret
; dtoc end
```

其中调用了rev函数，对字符串进行反转

```assembly
; rev
; 名称 翻转字符串
; 功能 将以0结束的字符串翻转
; 参数 ds:di指向字符串的首地址
; 返回 无
rev:    push cx
        push ax
        push si

        mov ax, 0
        mov ch, 0
revs:   mov cl, [si]
        jcxz revok      ; 遇到0结束
        push cx         ; 字符入栈
        inc si
        inc ax
        jmp short revs
revok:  mov cx, ax      ; cx是字符串长度
        sub si, cx      ; si归位
revs1:  jcxz revret
        pop ax          ; 字符出栈
        mov [si], al    ; 写到数据段
        inc si
        dec cx
        jmp short revs1
revret:
        pop si
        pop ax
        pop cx
        ret
```

show函数不再展示

### 课程设计1

将之前实验的power idea公司的数据按照下面的格式在屏幕显示，dtoc程序需要扩展成支持dword。

```
  1975    16        3         5   
  ...
  1995    5937000   17800     333
```

主程序如下：

```assembly
assume cs:codem, ds:data, ss:stack

data segment
    ; 21个年份，共84字节
    db '1975','1976','1977','1978','1979','1980','1981'
    db '1982','1983','1984','1985','1986','1987','1988'
    db '1989','1990','1991','1992','1993','1994','1995'

    ; 21个收入，共84字节
    dd 16,22,384,1356,2390,8000,16000,24486,50065,97479
    dd 140417,197514,345980,590827,803530,1183000
    dd 1843000,2759000,3753000,4649000,5937000

    ; 21个雇员数，共42字节
    dw 3,7,9,13,28,38,130,220,476,778,1001,1442,2258
    dw 2793,4037,5635,8226,11542,14430,15257,17800

    db 81 dup(0)
data ends

stack segment
    dw 16 dup (0)
stack ends

start:  mov ax, data
        mov ds, ax              ; 设置数据段ds

        mov ax, stack
        mov ss, ax
        mov sp, 32

        mov bx, 0               ; 年份/收入数组计数+4
        mov di, 0               ; 雇员计数+2
        mov bp, 2               ; 显示行号+1
        mov cx, 21              ; 外层循环计数

s:      push cx

s1:     mov si, 210             ; 临时缓冲区偏移
        mov cx, 4
        call fill               ; 前面填两个空格

        mov ax, [bx]            ; 转移年份一个字
        mov [si], ax            ; 转移年份一个字
        mov ax, [bx].2
        mov [si].2, ax

        add si, 4
        mov cx, 8               ; 内层循环计数
s2:     mov byte ptr [si], 20H  ; 空格
        inc si
        loop s2

        mov ax, 84[bx]          ; 转移收入低位字
        mov dx, 84[bx].2        ; 转移收入高位字
        call dtoc               ; ax返回字符数, si已更新
        mov cx, 12
        sub cx, ax
s3:     mov byte ptr [si], 20H  ; 空格
        inc si
        loop s3
        
        mov ax, 168[di]         ; 雇员数
        mov dx, 0
        call dtoc               ; ax返回字符数, si已更新
        mov cx, 12
        sub cx, ax
        call fill

        mov ax, 84[bx]          ; 计算人均收入
        mov dx, 84[bx].2
        mov cx, 168[di]
        call divdw

        call dtoc               ; ax返回字符数, si已更新
        mov cx, 8
        sub cx, ax
        call fill

        mov cx, 32              ; 80 - 4 - 12 * 3 - 8 = 32
        call fill               ; 后面补空格

        mov ax, bp
        mov dh, al
        mov dl, 0
        mov cl, 00000010b
        mov si, 210             ; si还原
        call show

        add bx, 4
        add di, 2
        inc bp

        pop cx
        loop s

        mov ax, 4c00H
        int 21H
```

其他子函数不再展示，其中fill用于填充空格，dtoc将dword型数字转成字符串，show将字串显示在屏幕。



## 11 标志寄存器

CPU内部有一种特殊的寄存器：

1. 用来存储相关指令的某些执行结果
2. 用来为CPU执行相关指令提供行为依据
3. 用来控制CPU的相关工作方式

这种特殊的寄存器在8086CPU中，被称为标志寄存器。8086CPU的标志寄存器有16位，其中存储的信息通常被称为**程序状态字（PSW）**。

flag寄存器是按位起作用的，8086CPU的flag寄存器的结果如下：

```
 15 14 13 12 11 10 9  8  7  6  5  4  3  2  1  0
|  |  |  |  |OF|DF|IF|TF|SF|ZF|  |AF|  |PF|  |CF|
```

### ZF标志（Zero）

零标志位，它记录相关指令执行后，其结果是否为0。如果为0，则zf=1；否则，zf=0.

```assembly
mov ax, 1
sub ax, 1	; 结果为0，zf=1

mov ax, 1
add ax, 0	; 结果为0，zf=0
```

注意，在8086指令集中有的指令执行是影响标志寄存器的，比如，add、sub、mul、div、inc、or、and等，它们大都是运算指令；有的则对标志寄存器没有影响，如mov、push、pop等，它们大都是传送指令。

### PF标志(Parity)

奇偶标志位，它记录相关指令执行后，其结果的所有bit位中1的个数是否为偶数。如果1的个数为偶数，pf=1；为奇数，pf=0。

### SF标志（Signed）

符号标志位，它记录相关指令执行后，其结果是否为负。如果为负，sf=1；否则，sf=0。

计算机通常用补码来表示有符号数据。对于同一个二进制数据，计算机可以将它当作无符号数据，也可以当作有符号数，关键在与我们的程序需要哪一种结果。

SF标志，就是CPU对**有符号数**运算结果的一种记录，它记录数据的正负。如果我们将数据当作无符号数来运算，SF的值则没有意义。

### CF标志（Carry）

进位标志位，一般情况下，在进行**无符号数**运算的时候，它记录了运算结果的最高有效位向更高位的进位值，或从更高位的借位值。

```assembly
mov al, 98H
add al, al		; (al)=30H, cf=1
add al, al		; (al)=60H, cf=0

mov al, 97H
sub al, 98H		; (al)=FFH, cf=1
sub al, al		; (al)=0, cf=0
```

### OF标志(overflow)

溢出标志位，在进行**有符号数**运算的时候，如果结果发生了溢出，则of=1；否则，of=0。

### adc指令

adc是带进位的加法，它利用了CF位上记录的进位值。

```assembly
adc ax, bx	; (ax)=(ax)+(bx)+cf
```



下面的指令和add ax, bx具有相同的结果：

```assembly
add al, bl
adc ah, bh
```

可以看出adc和add配合可以对更大的数据进行加法运算。adc执行之后，也可能产生进位，所以也会对CF位进行设置。由于有这样的功能，我们就可以对任意大的数据进行加法运算。

```assembly
; 计算 1EF000H + 201000H，结果放在ax(高16位)和bx（低16位）
mov ax, 001EH
mov bx, 0F000H
add bx, 1000H
adc ax, 0020H
```

下面的程序对两个128位数据进行相加

```assembly
; 名称 add128
; 功能 两个128位数相加
; 参数 ds:si指向存储第一个数的内存空间，因为数据位128位，所以需要8个字单元
;     低地址保存低位，结果存储在第一个数的存储空间里
;      ds:di指向存储第二个数的内存空间
; 返回 无
; 注意 inc和loop不影响CF位
add128:
		push ax
		push cx
		push si
		push di
		
		sub ax, ax		; 将cf清零
		mov cx, 8
s:
		mov ax, [si]
		adc ax, [di]
		mov [si], ax
		inc si
		inc si
		inc di
		inc di
		loop s
		
		pop di
		pop si
		pop cx
		pop ax
		ret
```

### sbb指令

带借位的减法指令，

```asse
sbb ax, bx	; (ax)=(ax)-(bx)-cf
```



### cmp指令

比较指令，功能相当于减法，只是不保存结果。cmp指令执行后，将对标志寄存器产生影响，其他相关指令通过识别这些被影响的标志寄存器位来得知比较结果。

```assembly
mov ax, 8
mov bx, 3
cmp ax, bx	;(ax)=8, zf=0, pf=1, sf=0, cf=0, of=0
```

通过判断标志寄存器，可以得知无符号数的比较结果

zf=1，说明(ax)=(bx)

zf=0，说明(ax)!=(bx)

cf=1，说明(ax)<(bx)

cf=0，说明(ax)>=(bx)

cf=0且zf=0，说明(ax)>(bx)

cf=1或zf=1，说明(ax)<=(bx)



对于有符号数，需要考察of和sf，其中sf记录的是实际结果的正负，不是逻辑上真正结果的正负。以cmp ah, bh为例：

sf=1, of=0，没有溢出，实际结果正负=逻辑结果正负，所以(ah) < (bh)

sf=1, of=1，有溢出，实际结果正负!=逻辑结果正负，所以(ah) > (bh)

sf=0, of=1，有溢出，实际结果正负!=逻辑结果正负，所以(ah) < (bh)

sf=0,of=0，没有溢出，实际结果正负=逻辑结果正负，所以(ah) >= (bh)

在配合zf，可以判断是否为0。

### 检测比较结果的条件转移指令

条件转移指令只能短转移，除了jcxz之外，其他条件转移指令大多数检测标志寄存器，它们通常和cmp相配合使用。

下面是常用的根据无符号数的比较结果进行转移的条件转移指令：

| 指令 | 含义         | 检测的标志位 |
| ---- | ------------ | ------------ |
| je   | 等于则转移   | zf=1         |
| jne  | 不等于转移   | zf=0         |
| jb   | 小于则转移   | cf=1         |
| jnb  | 不小于则等于 | cf=0         |
| ja   | 高于则转移   | cf=0且zf=0   |
| jna  | 不高于则转移 | cf=1或zf=1   |

在选择条件转移指令时，例如等于时有额外操作，那么可以选择使用jne使程序精简一点。

### DF标志和串传送指令, rep, cld, std

方向标志位。在串处理指令中，控制每次操作后si、di的增减方向。df=0，递增；df=1，递减。

- `movsb`

  相当于执行下面几步操作

  1. ((es)\*16+(di))=((ds)\*16+(si))
  2. 如果df=0，(si)=(si)+1，(di)=(di)+1
  3. 如果df=1，(si)=(si)-1，(di)=(di)-1

- `movsw`

  类似movsb，不过是操作字单元，每次增减2

一般都配合rep使用

`rep movsb`相当于

```assembly
s:movsb
  loop s
```

可见rep的作用是根据cx的值，重复后面的串传送指令



设置df的指令

- cld指令：将df位置0
- std指令：将df位置1



### pushf和popf

分别将标志寄存器的值压栈和出栈，为直接访问标志寄存器提供了一种方法。

### 标志寄存器在Debug中的表示



```
NV UP EI PL NZ NA PO NC
^
OF DF    SF ZF    PF CF
```

| 标志 | 值为1 | 值为0 |
| ---- | ----- | ----- |
| of   | OV    | NV    |
| df   | DN    | UP    |
| sf   | NG    | PL    |
| zf   | ZR    | NZ    |
| pf   | PE    | PO    |
| cf   | CY    | NC    |

### 实验11

将包含任意字符，以0结尾的字符串中的小写字母转成大写。

```assembly
; 名称: letterc
; 功能: 将以0结尾的字符串中的小写字母转变成大写
; 参数: ds:si指向字符串首地址
; 返回: 无
assume cs:codesg
datasg segment
    db "Beginner's All-purpose Symbolic Instruction Code.",0
datasg ends

codesg segment
start:
        mov ax, datasg
        mov ds, ax
        mov si, 0
        call letterc

        mov ax, 4c00h
        int 21h

letterc:
        push cx
        push si
        mov ch, 0
s:
        mov cl, [si]
        jcxz ok
        cmp cl, 61h             ; a
        jb next
        cmp cl, 7ah             ; z
        ja next
        and cl, 11011111b       ; 改成大写
        mov [si], cl

next:
        inc si
        jmp s
ok:
        pop si
        pop cx
        ret
codesg ends
end start
```



## 12 内中断

任何一个通用CPU，都具备一种能力，可以在执行完当前指令之后，检测到从CPU外部发送过来的或内部产生的一种特殊信息，并且可以立即对所接收到的信息进行处理。

这种特殊的信息，我们可以称其为：中断信息。中断的意思是，CPU不再接着向下执行，而是转去处理这个特殊信息。

### 内中断的产生

对于8086CPU，当CPU内部有下面的情况发生时，将产生相应的中断信息。

1. 除法错误，比如div指令除法溢出
2. 单步执行
3. 执行into指令
4. 执行int指令

区分不同的中断信息来源，中断信息中必须包含识别来源的编码，8086中称为**中断类型码**。它是一个字节型数据，可以表示256种中断信息来源。

上述4种中断源，在8086中的中断类型码如下：

1. 除法错误：0
2. 单步执行：1
3. 执行into指令：4
4. 执行int指令，int n，n为字节型立即数

### 中断处理程序

如何对中断信息进行处理，可以由我们编程决定。我们编写的用来处理中断信息的程序被称为中断处理程序。

那么CPU在收到中断信息之后，如何确定其中断处理程序的入口？CPU的设计者必须在中断信息和其处理程序的入口地址之间建立某种联系。

### 中断向量表

CPU用8位的中断类型码通过中断向量表找到相应的中断处理程序的入口地址。

中断向量表在内存中保存，其中存放着256个中断源所对应的中断处理程序的入口。

那么如何找到中断向量表呢？对于8086CPU，中断向量表指定放在内存地址0处。从0000:0000到0000:03FF的1024个单元中存放着中断向量表。每个中断向量两个字，高地址字存放段地址，低地址字存放偏移地址。

### 中断过程

根据中断类型码找到中断向量之后，用它设置CS和IP，这个工作是由CPU的硬件自动完成的。CPU硬件完成这个工作的过程被称为中断过程。

1. （从中断信息中）取得中断类型码
2. 标志寄存器的值入栈
3. 设置标志寄存器的第8位TF和第9位IF的值为0
4. CS的内容入栈
5. IP的内容入栈
6. 从内存地址为中断类型码\*4和中断类型码\*4+2的两个字单元中读取中断处理程序入口地址

用简洁的语言描述如下：

1. 取得中断类型码N
2. pushf
3. TF=0, IF=0
4. push CS
5. push IP
6. (IP)=(N\*4), (CS)=(N\*4+2)



### 中断处理程序和iret指令

由于CPU随时都可能检测到中断信息，所以中断处理程序必须一直存储在内存中。而中断处理程序的入口地址，即中断向量，必须存储在对应的中断向量表表项中。

中断处理程序的编写方法和子程序的比较相似，下面是常规的步骤：

1. 保存用到的寄存器
2. 处理中断
3. 恢复用到的寄存器
4. 用iret指令返回

iret指令的功能可以用下面的汇编语言描述：

```assembly
pop IP
pop CS
popf
```

iret指令执行之后，CPU回到执行中断程序之间的执行点继续执行程序。

### 除法错误中断的处理

前面提到，内存空间的前1KB空间是来存放中断向量表的，一般情况下，实际中断用不了这么多，我们使用0000:0200~0000:02FF这256个字节来存放中断处理程序。

重新编写0号中断处理程序，需要做一下几件事情：

1. 编写中断处理程序do0
2. 将do0送入内存0000:0200处
3. 将do0入口地址0000:0200存储在中断向量表0号表项中

程序的框架如下：

```assembly
assume cs:code
code segment
start:
        do0安装程序
        设置中断向量表
        mov ax, 4c00h
        int 21h

do0:
        显示字符串"overflow!"
        mov ax, 4c00h
        int 21h

code ends

end start
```

### 安装

代码块的长度可以由编译器来计算。

```assembly
assume cs:code
code segment
start:
        mov ax, cs
        mov ds, ax
        mov si, offset do0      ; ds:si指向源地址

        mov ax, 0
        mov es, ax
        mov di, 200h            ; es:si指向目的地址

        mov cx, offset do0end - offset do0

        cld                     ; df=0
        rep movsb

        mov word ptr es:[0*4], 200h ; 设置中断向量
        mov word ptr es:[0*4+2], 0

        mov ax, 4c00h
        int 21h

do0:
        显示字符串"overflow!"
        mov ax, 4c00h
        int 21h
do0end:
        nop

code ends

end start
```



### do0

完整的do0程序如下，注意字符串数据放在了处理程序代码中，所以执行时先跳到实际代码处。

```assembly
do0:
        jmp short do0start
        db "overflow!"
do0start:
        mov ax, cs
        mov ds, ax
        mov si, 202h            ; 指向字符串，第一个jmp指令占两个字节

        mov ax, 0b800h
        mov es, ax
        mov di, 12*160+36*2     ; 指向显存中间位置

        mov cx, 9               ; 字符串长度
s:
        mov al, [si]
        mov es:[di], al
        inc si
        add di, 2
        loop s

        mov ax, 4c00h
        int 21h
do0end:
        nop
```

### 单步中断

CPU在执行完一条指令之后，如果检测到标志寄存器的TF位位1，则产生单步中断。

Debug程序就是利用了单步中断的功能，那么它是如何实现的呢？

首先，Debug提供了中断处理程序，功能为显示所有寄存器的内容后等待输入命令。然后在执行t指令时，Debug将TF设置为1，使得CPU工作于单步中断方式下，则在执行完这条指令之后引发单步中断，执行单步中断处理程序。

### 响应中断的特殊情况

在有些情况下，CPU在执行完当前指令后，即便是发生中断，也不会响应。这里举一个例子。

在执行完向ss段寄存器传送数据的指令后，即便发生中断，CPU也不会响应。因为ss:sp联合指向栈顶，如何执行完设置ss的指令之后，CPU响应中断，中断过程会压栈标志寄存器、CS和IP。而当前ss:sp指向的不是正确的栈顶，将引起错误。



## 13 int指令

程序中可以直接用`int n`指令调用任何一个中断处理程序。以后我们将中断处理程序称为中断例程。

中断例程编写和安装方式跟前面int0的例子类似，注意如果中断例程中用到了寄存器要先入栈，返回前出栈，最后iret返回。

### 对int、iret和栈的深入理解

问题：用7ch中断例程完成loop指令的功能

应用举例：在屏幕中间显示80个'!'

```assembly
assume cs:code

code segment
start:
        mov ax, 0b800h
        mov es, ax
        mov id, 160*12

        mov bx, offset s - offset se
        mov cx, 80
s:
        mov byte ptr es:[di], '!'
        add di, 2
        int 7ch
se:
        nop
        mov ax, 4c00h
        int 21h
code ends
end start
```

其中cx控制循环次数，bx用来传递转移位移。

对应的中断例程如下：

```assembly
lp:     push bp
        mov bp, sp
        dec cx
        jcxz lpret
        add [bp+2], bx
lpret:  pop bp
        iret
```



问题：用7ch中断例程完成jmp near ptr s指令的功能

应用举例：在屏幕第12行，显示data段中的以0结尾的字符串。

```assembly
assume cs:code
data segment
    db 'conversation',0
data ends

code segment
start:
        mov ax, data
        mov ds, ax
        mov si, 0
        mov ax, 0b800h
        mov es, ax
        mov di, 160*12
s:
        cmp byte ptr [si], 0
        je ok
        mov al, [si]
        mov es:[di], al
        inc si
        add di, 2
        mov bx, offset s - offset ok
        int 7ch
ok:
        mov ax, 4c00h
        int 21h
code ends
end start
```

对应的中断例程

```assembly
jnear:  
		push bp
        mov bp, sp
        add [bp+2], bx
jnret:  pop bp
        iret
```



### BIOS和DOS所提供的中断例程

BIOS中主要包含一下几部分内容：

1. 硬件系统的检测和初始化程序
2. 外部中断和内部中断的中断例程
3. 用于对硬件设备进行I/O操作的中断例程
4. 其他和硬件系统相关的中断例程

操作系统DOS也提供了中断例程，从操作系统角度看，DOS的中断例程就是操作系统向程序员提供的编程资源。程序员在编程的时候可以用int指令直接调用BIOS和DOS提供的中断例程，来完成某些工作。

和硬件相关的DOS中断例程中，一般都调用到了BIOS的中断例程。

### BIOS和DOS中断例程的安装过程

1. 开机加电后，初始化(CS)=0FFFFH，(IP)=0，自动从FFFF:0单元开始执行程序。FFFF:0处由一条跳转指令，CPU执行该指令后，转去执行BIOS中的硬件系统检测和初始化程序
2. 初始化程序将建立BIOS所支持的中断向量，即将BIOS提供的中断例程的入口地址登记在中断向量表中。注意，BIOS的中断例程是固化到ROM中的，一直在内存中存在，所以只需将入口地址登记到中断向量表即可。
3. 硬件系统检测和初始化完成后，调用int 19h进行操作系统的引导。从此将计算机交给操作系统控制
4. DOS启动后，除完成其他工作外，还将它所提供的中断例程装入内存，并建立相应的中断向量。

### BIOS中断例程应用

一般来说，一个供程序员调用的中断例程中往往包括多个子程序，中断例程内部用传递进来的参数来决定执行哪一个子程序。BIOS和DOS提供的中断例程，都用ah来传递内部子程序的编号。

int 10h中断例程包含了多个和屏幕输出相关的子程序

int 10h中断例程的设置光标位置功能

```assembly
mov ah, 2	; 置光标
mov bh, 0	; 第0页
mov dh, 5	; 行号
mov dl, 12	; 列号
int 10h
```

int 10h中断例程在光标位置显示字符功能

```assembly
mov ah, 9	; 在光标位置显示字符
mov al, 'a'	; 字符
mov bl, 7	; 颜色属性
mov bh, 0	; 第0页
mov cx, 3	; 字符重复个数
int 10h
```

将前面两个组合起来，编程：在屏幕的5行12列显示3个红底高亮闪烁绿色的'a'

```assembly
assume cs:code
code segment
    mov ah, 2   ; 置光标
    mov bh, 0   ; 第0页
    mov dh, 5   ; 行号
    mov dl, 12  ; 列号
    int 10h

    mov ah, 9   ; 在光标位置显示字符
    mov al, 'a' ; 字符
    mov bl, 11001010b   ; 颜色属性
    mov bh, 0   ; 第0页
    mov cx, 3   ; 字符重复个数
    int 10h

    mov ax, 4c00h
    int 21h
code ends
end
```

### DOS中断例程应用

我们前面一直在使用int 21h中断例程的4ch号功能，即程序返回功能。

```assembly
mov ah, 4ch	; 程序返回
mov al, 0	; 返回值
int 21h
```

int 21h中断例程在光标位置显示字符串的功能

```assembly
ds:dx 指向字符串	; 要显示的字符串需以$结尾
mov ah, 9		; 功能号9
int 21h
```

编程：在屏幕的5行12列显示字符串“Welcome to masm!”

```assembly
assume cs:code
data segment
    db 'Welcome to masm!','$'
data ends

code segment
start:
    mov ah, 2   ; 置光标
    mov bh, 0   ; 第0页
    mov dh, 5   ; 行号
    mov dl, 12  ; 列号
    int 10h

    mov ax, data
    mov ds, ax
    mov dx, 0
    mov ah, 9   ; 在光标位置显示字符串
    int 21h

    mov ax, 4c00h
    int 21h
code ends
end start
```

### 实验13：编写、应用中断例程

（1）编写并安装int 7ch中断例程，功能为显示一个用0结尾的字符串，安装在0:200处。

参数：(dh)=行号，(dl)=列号，(cl)=颜色，ds:si指向字符串首地址

```assembly
assume cs:code
data segment
    db 'Welcome to masm!',0
data ends

code segment
start:
    mov dh, 10  ; 行号
    mov dl, 10  ; 列号
    mov cl, 00000010b   ;颜色

    mov ax, data
    mov ds, ax
    mov si, 0
    int 7ch

    mov ax, 4c00h
    int 21h
code ends
end start
```

中断例程及安装程序

```assembly
assume cs:code
code segment
start:
        mov ax, cs
        mov ds, ax
        mov si, offset do0      ; ds:si指向源地址

        mov ax, 0
        mov es, ax
        mov di, 200h            ; es:si指向目的地址

        mov cx, offset do0end - offset do0

        cld                     ; df=0
        rep movsb

        mov word ptr es:[7ch*4], 200h ; 设置中断向量
        mov word ptr es:[7ch*4+2], 0

        mov ax, 4c00h
        int 21h

do0:
        push ax
        push cx
        push es
        push di
        push si
        mov ax, 0b800h
        mov es, ax
        mov ax, 160             ; 计算位置
        mul dh
        mov di, ax
        mov ax, 2
        mul dl
        add di, ax
        mov al, cl      ; 保存颜色

        mov cx, 0
show:
        mov cl, [si]
        jcxz ok
        mov es:[di], cl     ; 字符
        mov es:[di].1, al   ; 颜色
        add di, 2
        inc si
        jmp short show

ok:
        pop si
        pop di
        pop es
        pop cx
        pop ax
        iret
do0end:
        nop

code ends

end start
```



## 14 端口

在PC机系统中，和CPU通过总线相连的芯片除了各种存储器之外，还有以下3种芯片。

1. 各种接口卡（比如网卡、显卡）上的接口芯片，它们控制接口卡进行工作
2. 主板上的接口芯片，CPU通过它们对部分外设进行访问
3. 其他芯片，用来存储相关的系统信息，或进行相关的输入输出处理

在这些芯片中，都有一组可以由CPU读写的寄存器。

1. 它们都和CPU的总线相连，当然这种连接是通过它们所在的芯片进行的；
2. CPU对它们进行读或写的时候都通过控制线向它们所在的芯片发出端口读写命令

从CPU的角度，将这些寄存器都当作端口，对它们进行统一编址。

CPU可以直接读写以下3个地方的数据：

1. CPU内部的寄存器
2. 内存单元
3. 端口

### 端口的读写

端口地址和内存地址一样，通过地址总线来传送。在PC机系统中，最多可以定位64KB个不同的端口。端口地址的范围0～65535。

端口的读写指令只有两条：in和out。

`in al, 60h`从60h号端口读如一个字节。

1. CPU通过地址线将地址信息60h发出
2. CPU通过控制线发出端口读命令，选中端口所在的芯片，并通知它，将要从中读数据
3. 端口所在的芯片将60h端口中的数据通过数据线送入CPU



注意：in和out指令中，只能使用ax或al来存放从端口中读如的数据或要发送到端口的数据。访问8位端口用al，访问16位端口用ax。

```assembly
; 对0～255范围内的端口进行读写
in al, 20h	; 从20h端口读如一个字节
out 20h, al	; 往20h端口写入一个字节

; 对256～65535的端口进行读写，端口号放在dx中
mov dx 3f8h	; 将端口号3f8h送入dx
in al, dx	; 从3f8h端口读如一个字节
out dx, al	; 向3f8h端口写入一个字节
```



### CMOS RAM芯片

CMOS RAM芯片：

1. 包含一个实时钟和一个有128个存储单元的RAM储存器
2. 该芯片靠电池供电。所以，关机后其内部的实时钟仍可以正常工作
3. 128个字节的RAM中，内部实时钟占用0～0dh单元来保存时间信息，其余大部分用来保存系统配置信息，供系统启动时BIOS程序读取。BIOS也提供了相关的程序，使我们可以在开机的时候配置CMOS RAM中的系统信息
4. 该芯片内部有两个端口，端口地址为70h和71h
5. 70h为地址端口，存放要访问的CMOS RAM单元的地址；71h为数据端口，存放要读写的数据。

所以CPU对CMOS RAM的读写分两步进行，比如读2号单元：

1. 将2送入端口70h
2. 从端口71h读出2号单元的内容

### shl和shr指令

逻辑移位指令。

shl的功能如下：

1. 将一个寄存器或内存单元中的数据向左移位
2. 将最后移出的一位写入CF中
3. 最低位用0补充



```assembly
mov al, 01001000b
shl al, 1			; 执行后(al)=10010000b, CF=0
```

如果移动位数大于1，必须将移动位数放在cl中

```assembly
mov al, 01010001b
mov cl, 3
shl al, cl
```



shr的功能如下：

1. 将一个寄存器或内存单元中的数据向右移位
2. 将最后移出的一位写入CF中
3. 最高位用0补充

同样地，如果移动位数大于1，必须将移动位数放在cl中



### CMOS RAM中存储的时间信息

CMOS中年月日时分秒信息的长度都是一个字节，存放单元为：

秒：0，分：2，时：4，日：7，月：8，年：9

这些数据以BCD码的形式存放

```
十进制数码  0    1    2    3    4    5    6    7    8    9
BCD码     0000 0001 0010 0011 0100 0101 0110 0111 1000 1001
```

可见一个字节可表示两个BCD码，高4位表示十位，低4位表示个位。

### 实验14 CMOS时间

访问CMOS RAM，以“年/月/日 时:分:秒”的格式显示当前的日期时间。

```assembly
assume cs:code, ds:data
data segment
    db 'yy/MM/dd hh:mm:ss',0 ; 显示的字符串
data ends
code segment
        db 9, 8, 7, 4, 2, 0 ; 存放内存单元号
start:
        mov ax, data
        mov ds, ax
        mov si, 0           ; +3
        mov di, 0           ; +1

        mov cx, 6           ; 循环次数
lp:     push cx
        mov al, cs:[di]     ; 获取内存单元号
        out 70h, al         ; 写到控制端口
        in al, 71h          ; 从数据端口读取

        mov ah, al
        mov cl, 4           ; 右移位数
        shr ah, cl          ; ah中为十位
        and al, 00001111b   ; al中为个位
        add ah, 30h         ; 对应ascii码
        add al, 30h         ; 对应ascii码
        mov [si], ah
        mov [si].1, al

        add si, 3
        inc di
        pop cx
        loop lp

        mov si, 0
        mov dh, 12
        mov dl, 36
        mov cl, 00000100b
        call show

        mov ax, 4c00h
        int 21h
```



## 15 外中断

CPU如何得知外设的输入？CPU从何处得到外设的输入？

端口和中断机制，是CPU进行I/O的基础。

### 接口芯片和端口

CPU通过端口和外部设备进行联系。外设的输入，送入相关的接口芯片的端口中，CPU向外设的输出也是先送到端口中，再有相关的芯片送到外设。CPU还可以向外设输出控制命令，而这些控制命令也是先送到相关芯片的端口中，再由相关芯片根据命令对外设实施控制。

### 外中断信息

在PC系统中，外中断源主要由两类：

#### 可屏蔽中断 sti, cli

CPU是否响应可屏蔽中断，要看标志寄存器位IF的设置。当CPU检测到可屏蔽中断信息时，如果IF=1，则CPU在执行完当前指令后响应中断，引发中断过程；如IF=0，则不响应可屏蔽中断。

中断过程基本上跟内中断相同。因为中断信息来自CPU外部，中断类型码是通过数据总线送入CPU的；而内中断的中断类型码是在CPU内部产生的。

之所以在中断过程中，要将IF置0，就是在进入中断处理程序后，禁止其他的可屏蔽中断。

当然，如果在中断处理程序中需要处理可屏蔽中断，可以用指令将IF置1。8086CPU提供的设置IF的指令如下：

sti，设置IF=1

cli，设置IF=0

#### 不可屏蔽中断

不可屏蔽中断时CPU必须响应的外中断。对于8086CPU，不可屏蔽中断的中断类型码固定为2。所以在中断过程中，不需要取中断类型码。其中断过程为：

1. 标志寄存器入栈，IF=0，TF=0
2. CS、IP入栈
3. (IP)=(8), (CS)=(0AH)

几乎所有由外设引发的外中断，都是可屏蔽中断。不可屏蔽中断是在系统中有必须处理的紧急情况发生时用来通知CPU的中断信息。

### PC机键盘的处理过程

1. 键盘输入

   每个按键相当于一个开关，键盘上有一个芯片对键盘上的每一个键的开关状态进行扫描。按下时/松开时分别将通码（第7位0）和断码（第7位1）送到60h端口中。断码=通码+80h

2. 引发9号中断

   扫描码送到60h端口时，相关芯片就会向CPU发出中断类型码为9的可屏蔽中断信息。CPU检测到中断信息之后，如果IF=1，则响应中断，引发中断过程，转去执行int9中断例程

3. 执行int9中断例程

   BIOS提供了int9中断例程，用来进行基本的键盘输入处理，主要工作如下：

   1. 读出60h端口中的扫描码
   2. 如果是字符键，将扫描码和它对应的ASCII码送入内存中的BIOS键盘缓冲区；如果是控制键（比如Ctrl）和切换键（比如CapsLock）的扫描码，则将其转变为状态字节，写入内存中存储状态字节的单元
   3. 对键盘系统进行相关的控制，比如说，向相关芯片发出应答信息。

BIOS键盘缓冲区可以存储15个键盘输入，一个键盘输入对应一个字单元，高位字节存放扫描码，低位字节存放字符码。

0040:17单元存储键盘状态字节，该字节记录了控制键和切换键的状态。 

0：右shift状态，置1表示按下

1：左shift状态，置1表示按下

2：Ctrl状态，置1表示按下

3：Alt状态，置1表示按下

4：ScrollLock状态，置1表示Sroll指示灯亮

5：NumLock状态，置1表示小键盘输入的数字

6：CapsLock状态，置1表示输入大写字母

7：Insert状态，置1表示处于删除态



### 编写int9中断例程

编程：在屏幕中间依次显示"a"～"z"，并可以让人看清。先显示过程中，按Esc键后，改变显示的颜色。

完整的键盘中断的处理程序，设计硬件细节，我们这里只是包装了BIOS提供的int9中断例程。

字母的显示比较简单，但是要看清的话需要加入一定的延迟，我们通过delay函数来实现。

```assembly
delay:
            push ax
            push dx
            mov dx, 0010h
            mov ax, 0
s1:
            sub ax, 1
            sbb dx, 0
            cmp ax, 0
            jne s1
            cmp dx, 0
            jne s1

            pop dx
            pop ax
            ret
```



程序的主要逻辑如下：

```pseudocode
保存原来的int9中断向量
替换int9中断向量为新的中断例程地址
显示a~z
恢复int9中断向量原先的地址
```

主程序部分代码：

```assembly
assume cs:code, ss:stack, ds:data
stack segment
    db 128 dup (0)
stack ends

data segment
    dw 0, 0
data ends

code segment
start:      mov ax, stack
            mov ss, ax
            mov sp, 128
            mov ax, data
            mov ds, ax

            mov ax, 0
            mov es, ax      ; 中断向量表
            push es:[9*4]
            pop ds:[0]
            push es:[9*4+2]
            pop ds:[2]      ; 保存原来的中断例程的入口地址

            pushf
            cli             ; 屏蔽中断，防止中断向量出现非法状态
            mov word ptr es:[9*4], offset int9
            mov es:[9*4+2], cs
            popf

            ; 依次显示a~z
            mov ax, 0b800h
            mov es, ax
            mov ah, 'a'
s:
            mov es:[160*12+40*2], ah
            call delay
            inc ah
            cmp ah, 'z'
            jna s
            mov ax, 0
            mov es, ax

            pushf
            cli             ; 屏蔽中断，防止向量出现非法状态
            push ds:[0]
            pop es:[9*4]
            push ds:[2]
            pop es:[9*4+2]  ; int9中断例程恢复为原来的地址
            popf

            mov ax, 4c00h
            int 21h
            
code ends
end start
```

注意在修改中断向量时，我们使用cli屏蔽了中断，防止修改了一半发生了int9键盘中断，导致发生错误。

新的中断例程代码：

```assembly
; 新的int9中断例程
int9:
            push ax
            push bx
            push es
            in al, 60h      ; 从60h端口读取扫描码

            pushf           ; 中断过程：标志寄存器入栈

            pushf           ; 中断过程：IF=0, TF=0，可以省略，外面已经设置了
            pop bx
            and bh, 11111100b
            push bx
            popf

            call dword ptr ds:[0]   ; 中断过程：模拟int9

            cmp al, 1
            jne int9ret     ; 不是Esc键，跳过

            mov ax, 0b800h
            mov es, ax
            inc byte ptr es:[160*12+40*2+1] ; 属性值+1

int9ret:
            pop es
            pop bx
            pop ax
            iret
```

对原来的int9中断例程进行包装，并模拟了int指令的操作。

### 安装新的int9中断例程

安装一个新的int9中断例程，按下F1键后改变当前屏幕的显示颜色，其他键照常处理。

中断例程的安装方式跟内中断类似，我们将中断例程代码放到0:204h开始的位置，0:200~0:203存放原来的int9中断例程地址。完整的程序如下：

```assembly
assume cs:code, ss:stack
stack segment
    db 128 dup (0)
stack ends

code segment
start:      mov ax, stack
            mov ss, ax
            mov sp, 128

            push cs
            pop ds

            mov ax, 0
            mov es, ax      ; 中断向量表

            mov si, offset int9     ; ds:si 指向源地址
            mov di, 204h            ; es:di 指向目的地址
            mov cx, offset int9end-offset int9 ; 长度
            cld                     ; 传输方向为正
            rep movsb

            push es:[9*4]
            pop es:[200h]
            push es:[9*4+2]
            pop es:[202h]      ; 保存原来的中断例程的入口地址

            pushf
            cli             ; 屏蔽中断，防止中断向量出现非法状态
            mov word ptr es:[9*4], 204h
            mov es:[9*4+2], 0
            popf

            mov ax, 4c00h
            int 21h
            
; 新的int9中断例程
int9:
            push ax
            push bx
            push cx
            push es
            in al, 60h      ; 从60h端口读取扫描码

            pushf           ; 中断过程：标志寄存器入栈

            call dword ptr cs:[200h]   ; 中断过程：模拟int9

            cmp al, 3bh     ; F1的扫描码
            jne int9ret     ; 不是Esc键，跳过

            mov ax, 0b800h
            mov es, ax
            mov bx, 1
            mov cx, 2000
s:
            inc byte ptr es:[bx] ; 属性值+1
            add bx, 2
            loop s

int9ret:
            pop es
            pop cx
            pop bx
            pop ax
            iret

int9end:    nop

code ends
end start
```

### 实验 15

安装一个新的int9中断例程。按下“A”键后，除非不再松开，如果松开，就显示满屏幕的“A”；其他的键照常处理。（断码=通码+80h）

新的中断例程稍作修改即可：

```assembly
; 新的int9中断例程
int9:
            push ax
            push bx
            push cx
            push es
            in al, 60h      ; 从60h端口读取扫描码

            pushf           ; 中断过程：标志寄存器入栈

            call dword ptr cs:[200h]   ; 中断过程：模拟int9

            cmp al, 9eh     ; a的扫描码1eh + 80h
            jne int9ret     ; 不是a键，跳过

            mov ax, 40h
            mov es, ax
            mov cx, 0
            mov cl, es:[17h]  ; 键盘状态字节
            and cl, 01000011b   ; shift或CapsLock
            jcxz int9ret    ; 不是大写，跳过

            mov ax, 0b800h
            mov es, ax
            mov bx, 0
            mov cx, 2000
s:
            mov byte ptr es:[bx], 'A'
            add bx, 2
            loop s

int9ret:
            pop es
            pop cx
            pop bx
            pop ax
            iret

int9end:    nop
```

严格来说，如果只要按下的时候是大写A，松开的时候不需要是大写A的话，需要记录A的通码状态。

### 8086指令总结

#### 数据传送指令

如mov、push、pop、pushf、popf、xchg等，这些指令实现寄存器和内核、寄存器和寄存器之间的单个数据传送

#### 算术运算指令

如add、sub、adc、sbb、inc、dec、cmp、imul、idiv、aaa等，这些指令实现了寄存器和内存中的数据的算术运算。它们的执行结果影响标志寄存器位sf、zf、of、cf、pf、af位。

inc不影响cf？

#### 逻辑指令

如and、or、not、xor、test、shl、shr、sal、sar、rol、ror、rcl、rcr等。除了not指令外，它们的执行结果都影响标志寄存器的相关标志位。

#### 转移指令

可以修改IP，或同时修改CS和IP的指令统称为转移指令。分为以下几类：

1. 无条件转移指令，如jmp
2. 条件转移指令，如jcxz、je、jb、ja、jnb、jna等
3. 循环指令，如loop
4. 过程，比如call、ret、retf
5. 中断，比如int、iret

#### 处理机控制指令

这些指令对标志寄存器或其他处理机状态进行设置，比如cld、std、cli、sti、nop、clc、cmc、stc、htl、wait、esc、lock等

#### 串处理指令

这些指令对批量数据进行处理，比如movsb、movsw、cmps、scas、lods、stos等。若要使用这些指令方便地进行批量数据处理，则需要和rep、repe、repne等前缀指令配置使用。



## 16 直接定址表

这一章，讨论如何有效合理地组织数据，以及相关的编程技术。

### 数据标号

地址标号仅仅表示地址，数据标号标记了存储数据的单元的地址和长度。

```assembly
assume cs:code
code segment
	a db 1, 2, 3, 4, 5, 6, 7, 8
	b dw 0
code ends
start:
		mov si, 0
		mov cx, 8
s:		
		mov al, a[si]
		mov ah, 0
		add b, ax
		inc si
		loop s
		mov ax, 4c00h
		int 21h
		
end start
```

对于程序中的b dw 0

```assembly
mov ax, b;	相当于mov ax, cs:[8]
mov b, 2;	相当于mov word ptr cs:[8], 2
inc b;		相当于inc word ptr cs:[8]
```

### 在其他段中使用数据标号

一般来说，我们不在代码段中定义数据，而是将数据定义到其他段中。

注意，在后面加冒号“:”的地址标号，只能在代码段中使用。注意，如果想在代码段中直接用数据标号访问数据，则需要用伪指令assume将标号所在的段和一个段寄存器联系起来。否则编译器在编译的时候，无法确定标号的段地址在哪一个寄存器中。

可以将标号当作数据来定义，此时编译器将标号所表示的地址当作数据的值

```assembly
data segment
	a db 1,2,3,4,5,6,7,8
	b dw 0
	c dw a, b
	; 相当于c dw offset a, offset b
data ends

data segment
	a db 1,2,3,4,5,6,7,8
	b dw 0
	c dd a, b
	; 相当于c dw offset a, seg a, offset b, seg b
data ends
```

### 直接定址表

用查表的方式编写相关程序的技巧。

编写子程序，以十六进制的形式在屏幕中断显示给定的字节型数据。

数值0～9：+30h=对应的ASCII值

数值10～15：+37h=对应的ASCII值

没有一致的映射关系，需要进行一些比较，我们希望更便捷的算法，具体做法是建立一张表

```assembly
; ax存放要显示的word数据
showbyte:
        jmp short show
        table db '0123456789ABCDEF' ; 字符表
show:
        push bx
        push cx
        push ex

        mov ah, al
        mov cl, 4
        shr ah, cl          ; 高4位
        and al, 00001111b   ; 低4位

        mov bl, ah
        mov bh, 0
        mov ah, table[bx]   ; 高4位对应字符

        mov bl, al
        mov bh, 0
        mov al, table[bx]   ; 低4位对应字符

        mov bx, 0b800h
        mov es, bx
        mov es:[160*12+40*2], ah
        mov es:[160*12+40*2+2], al

        pop es
        pop cx
        pop bx
        ret
```



通过数据，直接计算出所要找的元素的位置的表，我们称之为：直接定址表。



### 程序入口地址的直接定址表

可以在直接定址表中存储子程序的地址，从而方便地实现不同子程序的调用。

安装一个新的int 7ch中断例程，为显示输出提供如下功能子程序：

1. 清屏
2. 设置前景色
3. 设置背景色
4. 向上滚动一行

使用ah寄存器传递功能号，对于设置颜色的功能，用al传送颜色（0～7）。

1. 清屏：将显存中当前屏幕中的字符设为空格符
2. 设置前景色：设置显存中当前屏幕中处于奇地址的属性字节的第0、1、2位
3. 设置背景色：设置显存中当前屏幕中处于奇地址的属性字节的第4、5、6位
4. 向上滚动一行：依次将第n+1行的内容复制到第n行；最后一行为空



**注意：**标号的偏移是在安装程序代码段中的偏移，并不是中断例程安装了之后所在的偏移，所以需要进行修正。

4个sub暂时省略

```assembly
; 安装一个新的int 7ch中断例程，为显示输出提供如下功能子程序：
;
; 1. 清屏
; 2. 设置前景色
; 3. 设置背景色
; 4. 向上滚动一行
;
; 使用ah寄存器传递功能号，对于设置颜色的功能，用al传送颜色（0～7）。
assume cs:code
code segment
; 我们将中断例程放到代码段的开头，方便计算偏移
setscreen:
        jmp short set
 ; 因为我们将中断例程安装到了0:200h位置，所以+200h修正偏移
 table  dw sub1+200h, sub2+200h, sub3+200h, sub4+200h
set:
        push bx

        cmp ah, 3       ; 判断是否大于3
        ja sret
        mov bl, ah
        mov bh, 0
        add bx, bx      ; 计算在table中的偏移

        ; 注意table的偏移是在安装程序代码段中的偏移
        ; 因为我们将中断例程安装到了0:200h位置
        ; 所以+200h修正到实际安装的位置
        call word ptr table.200h[bx]     ; 调用对应的功能子程序

sret:
        pop bx
        iret
        
sub1:
sub2:
sub3:
sub4:

setscreenend:
        nop

start:
        mov ax, cs
        mov ds, ax
        mov si, offset setscreen; ds:si指向源地址

        mov ax, 0
        mov es, ax
        mov di, 200h            ; es:si指向目的地址

        mov cx, offset setscreenend - offset setscreen

        cld                     ; df=0
        rep movsb

        mov word ptr es:[7ch*4], 200h ; 设置中断向量
        mov word ptr es:[7ch*4+2], 0

        mov ax, 4c00h
        int 21h

code ends
end start
```



4个子程序代码如下：

```assembly
; 清屏
sub1:
        push bx
        push cx
        push es
        mov bx, 0b800h
        mov es, bx
        mov bx, 0
        mov cx, 2000
sub1s:
        mov byte ptr es:[bx], ' '
        add bx, 2
        loop sub1s

        pop es
        pop cx
        pop bx
        ret

; 设置前景色
sub2:
        push bx
        push cx
        push es
        mov bx, 0b800h
        mov es, bx
        mov bx, 1
        mov cx, 2000
sub2s:
        and byte ptr es:[bx], 11111000b
        or es:[bx], al
        add bx, 2
        loop sub2s

        pop es
        pop cx
        pop bx
        ret
        
; 设置背景色
sub3:
        push bx
        push cx
        push es
        mov cl, 4
        shl al, cl

        mov bx, 0b800h
        mov es, bx
        mov bx, 1
        mov cx, 2000
sub3s:
        and byte ptr es:[bx], 10001111b
        or es:[bx], al
        add bx, 2
        loop sub3s

        pop es
        pop cx
        pop bx
        ret

; 向上滚动一行
sub4:
        push cx
        push si
        push di
        push es
        push ds

        mov si, 0b800h
        mov es, si
        mov ds, si
        mov si, 160     ; ds:si 指向n+1行
        mov di, 0       ; es:di 指向n行
        cld             ; 正向
        mov cx, 24      ; 共复制24行
        
sub4s:
        push cx
        mov cx, 160
        rep movsb       ; 复制
        pop cx
        loop sub4s

        mov cx, 80
        mov si, 0
sub4s1:
        mov byte ptr [160*24+si], ' ' ; 最后一行清空
        add si, 2
        loop sub4s1

        pop ds
        pop es
        pop di
        pop si
        pop cx
        ret
```



## 17 使用BIOS进行键盘输入和磁盘读写

BIOS为这两种外设的I/O提供了最基本的中断例程。

### int9中断例程对键盘输入的处理

键盘输入将引发9号中断，BIOS提供了int9中断例程。int9中断发生后，指向int9中断例程，从60h端口读出扫描码，并将其转化为相应的ASCII码或状态信息，存储在内存的指定空间（键盘缓冲区或状态字节）中。

键盘缓冲区中有16个字单元，可以存储15个按键的扫描码和对应的ASCII码，高位字节扫描码，低位字节ASCII码。

读到字符，会检测状态字节，看见是否有shift、Ctrl等切换按键按下，根据状态字节ASCII码可能会不同。

读到状态信息，设置0040:17处的状态字节。

### 使用int16h中断例程读取键盘缓冲区

BIOS提供了int 16h中断例程供程序员调用，其中最重要的功能是从键盘缓冲区中读取一个键盘输入，该功能的编号为0。

```assembly
mov ah, 0
int 16h		; (ah)=扫描码，(al)=ASCII码
```

0号功能的工作如下：

1. 检测键盘缓冲区中是否有数据
2. 没有则继续做第1步
3. 读取缓冲区第一个字单元中的键盘输入
4. 将读取的扫描码送入ah，ASCII码送入al
5. 将已读取的键盘输入从缓冲区中删除



可见，int9负责向键盘缓冲区写入，int16h从缓冲区读出。不过它们写入和读出的时机不同，int9中断例程是在有键按下的是写入；int16h是在应用程序对其进行调用的时候读出。

编程：接收用户键盘输入，输入“r”、“g”、“b”分别将屏幕前景色设置为红、绿、蓝。

```assembly
assume cs:code
code segment
start:
        mov ah, 0
        int 16h

        mov ah, 1       ; blue
        cmp al, 'r'
        je red
        cmp al, 'g'
        je green
        cmp al, 'b'
        je blue
        jmp short sret

red:    shl ah, 1
green:  shl ah, 1
blue:   mov bx, 0b800h
        mov es, bx
        mov bx, 1
        mov cx, 2000
s:
        and byte ptr es:[bx], 11111000b
        or es:[bx], ah
        add bx, 2
        loop s

sret:
        mov ax, 4c00h
        int 21h

code ends
end start
```

### 字符串的输入

最基本的字符串输入程序：

1. 在输入的同时需要显示这个字符串
2. 一般在输入回车符后，字符串输入结束
3. 能够删除已经输入的字符

处理流程如下：

1. 调用int 16h读取键盘输入
2. 如果是字符，进入字符栈，显示字符栈中的所有字符；继续执行1
3. 如果是退格键，从字符栈中弹出一个字符，显示字符栈中的所有字符；继续执行1
4. 如果是Enter键，向字符栈中压入0，返回



我们将字符栈的入栈、出栈和显示栈中的内容，写成子程序。

```assembly
; 字符栈的入栈、出栈和显示
; 参数说明: (ah)=功能号，0入栈，1出栈，2显示
;           ds:si 指向字符栈空间
;           对于0号功能: (al)=入栈字符
;           对于1号功能: (al)=返回的字符
;           对于2号功能: (dh)和(dl)=字符串在屏幕上显示的行、列
charstack:
            jmp short charstart
table       dw charpush, charpop, charshow
top         dw 0                ; 栈顶，空的

charstart:
            push bx
            push dx
            push di
            push es
            cmp ah, 2
            ja charret
            mov bl, ah
            mov bh, 0
            add bx, bx
            jmp word ptr table[bx]  ; 跳到子程序

charpush:
            mov bx, top
            mov [si][bx], al
            inc top
            jmp charret
charpop:
            cmp top, 0
            je charret
            dec top
            mov bx, top
            mov al, [si][bx]
            jmp charret

charshow:
            mov bx, 0b800h
            mov es, bx
            mov al, 160
            mov ah, 0
            mul dh          ; 行号*160
            mov di, ax
            add dl, dl      ; 列号*2
            mov dh, 0
            add di, dx

            mov bx, 0
charshows:
            cmp bx, top
            jne noempty
            mov byte ptr es:[di], ' '
            jmp charret
noempty:
            mov al, [si][bx]
            mov es:[di], al
            mov byte ptr es:[di+2], ' '
            inc bx
            add di, 2
            jmp charshows

charret:
            pop es
            pop di
            pop dx
            pop bx
            ret
```

完整的子程序如下

```assembly
getstr:
            push ax
getstrs:
            mov ah, 0
            int 16h
            cmp al, 20h     ; ASCII码小于20h，说明不是字符
            jb nochar
            mov ah, 0
            call charstack  ; 字符入栈
            mov ah, 2
            call charstack  ; 显示栈中的字符
            jmp getstrs
nochar:
            cmp ah, 0eh     ; 退格键的扫描码
            je backspace
            cmp ah, 1ch     ; Enter键的扫描码
            je enter
            jmp getstrs
backspace:
            mov ah, 1
            call charstack  ; 字符出栈
            mov ah, 2
            call charstack  ; 显示栈中的字符
            jmp getstrs
enter:
            mov al, 0
            mov ah, 0
            call charstack  ; 0入栈
            mov ah, 2
            call charstack  ; 显示栈中的字符
            pop ax
            ret
```

最后我们写个外部的测试程序

```assembly
assume cs:code, ds:data
data segment
    db 32 dup (0)
data ends
code segment
; ...
; ...
start:
            mov ax, data
            mov ds, ax
            mov si, 0
            mov dh, 3       ; 3行
            mov dl, 20      ; 20列
            call getstr

            mov ax, 4c00h
            int 21h
code ends
end start
```

### 应用int13h中断例程对磁盘进行读写

以3.5英寸软盘为例，它有上下两面，每面有80个磁道，每个磁道分为18个扇区，每个扇区大小为512字节。

2\*80\*18\*512=1440KB~=1.44MB

磁盘的实际访问由磁盘控制器进行，只能以扇区为单位进行读写。面号和磁道号从0开始，而扇区号从1开始。

BIOS提供了访问磁盘的中断例程为int13h。读取0面0道1扇区的内容道0:200的程序

```assembly
mov ax, 0
mov es, ax
mov bx, 200h	; es:bx 指向接收从扇区读如数据的内存区

mov al, 1	; 读取的扇区数
mov ch, 0	; 磁道号
mov cl, 1	; 扇区号
mov dl, 0	; 驱动器号	软驱从0开始，0:软驱A，1:软驱B；硬盘从80h开始，80h：硬盘C，81h：硬盘D
mov dh, 0	; 磁头号（对于软盘即面号）
mov ah, 2	; 功能号，2表示读扇区，3表示写扇区
int 13h
; 返回参数
; 操作成功：(ah)=0, (al)=读如的扇区数
; 操作失败：(ah)=出错代码
```

写磁盘参数类似，功能号为3。



### 编写包含多个功能子程序的中断例程

对所有扇区进行统一编号

逻辑扇区号=（面号\*80+磁道号）\*18+扇区号 - 1

面号=int(逻辑扇区号/1440)

磁道号=int(rem(逻辑扇区号/1440)/18)

扇区号=rem(rem(逻辑扇区号/1440)/18)+1



### 课程设计2

计算机开机之后，CPU自动进入FFFF:0单元处执行，此处有一条跳转指令，CPU转去执行BIOS中的硬件系统检测和初始化程序。

初始化程序将建立中断向量表。硬件系统检测和初始化完成之后，调用int 19h进行操作系统引导。从软盘或硬盘第一个扇区读取内容道0:7c00，然后CS:IP指向0:7c00开始执行。

课程设计的任务是编写一个可以自行启动计算机，不需要在现有操作系统环境中运行的程序，功能如下：

1. 列出功能选项，让用户通过键盘进行选择，界面如下。
   1. reset pc			; 重新启动计算机
   2. start system                ; 引导现有的操作系统
   3. clock                             ; 进入时钟程序
   4. set clock                       ; 设置时间
2. 用户输入1后重新启动计算机（提示：考虑ffff:0单元）
3. 用户输入2后引导现有的操作系统（提示：考虑硬盘C的0道0面1扇区）
4. 用户输入3后，执行动态显示当前日期、时间的程序，格式：年/月/日 时:分:秒。（循环读取CMOS），按下F1键后，改变显示颜色；按下Esc键后，返回到主选单（提示：利用键盘中断）
5. 用户输入4后，可更改当前的日期、时间，更改后返回到主选单（提示：输入字符串）



建议：

1. 在DOS下编写安装程序，在安装程序中包含任务程序
2. 运行安装程序，将任务程序写到软盘上
3. 若要任务程序可以在开机后自行执行，要将它写到软盘的0道0面1扇区上。如果程序长度大于512字节，则需要用多个扇区存放，这种情况下，处于软盘0道0面1扇区中的程序就必须负责将其他扇区中的内容读取内存



完整程序见[assembly-course-design2](https://github.com/catbro666/assembly-course-design2)



x86汇编扩展https://www.cs.virginia.edu/~evans/cs216/guides/x86.html

