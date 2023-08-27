---
title:       "编译和链接"
subtitle:    ""
description: ""
date:        2023-08-27T15:08:40+08:00 
author:      ""
image:       ""
tags:        ["cpp"]
categories:  ["Tech" ]
katex:       false
---

## 什么是编译

我们先创建两个源文件  
**main.c** 
```c
#include <iostream>

int add(int a, int b);

int main(){
        printf("Hello, World!\n");
        int result = add(5, 5);
        return 0;
}
```  

**math.c**
```c
int add(int a, int b) {
    return a + b;
}
```

然后我们通过`g++ -c`命令将代码进行编译，随后文件目录下会出现两个`.o`文件，也叫目标文件
```
g++ -c main.c
g++ -c math.c

main.o math.o
```

**注意：编译永远以单个源文件为单位**, 在实际开发过程中，我们通常会把不同功能的代码分散到不同的源文件中。一方面方便阅读和维护源文件，另一方面也提升了软件构建的速度。比如你修改了一个源文件，只需要将其重新编译，不需要再编译其他源文件。

说回来， 目标文件是一个二进制的文件，文件的格式是ELF(liux)，可以使用`file main.o查看`
```
main.o: ELF 64-bit LSB relocatable, x86-64, version 1 (SYSV), not stripped
```

我们也能在文件头部找到可执行文件的基本信息 `readelf -h main.o`

```
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V                // 支持的操作系统   
  ABI Version:                       0
  Type:                              REL (Relocatable file)
  Machine:                           Advanced Micro Devices X86-64  // 机器类型
  Version:                           0x1
  Entry point address:               0x0
  Start of program headers:          0 (bytes into file)
  Start of section headers:          1496 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           0 (bytes)
  Number of program headers:         0
  Size of section headers:           64 (bytes)
  Number of section headers:         16
  Section header string table index: 15```
``` 

后面则是一系列的区块/段，被叫做sections, 里面有程序的代码和数据等 `readelf -S main.o`

```
There are 16 section headers, starting at offset 0x5d8:

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .text             PROGBITS         0000000000000000  00000040
       00000000000000a3  0000000000000000  AX       0     0     1
  [ 2] .rela.text        RELA             0000000000000000  00000418
       00000000000000d8  0000000000000018   I      13     1     8
  [ 3] .data             PROGBITS         0000000000000000  000000e3
       0000000000000000  0000000000000000  WA       0     0     1
  [ 4] .bss              NOBITS           0000000000000000  000000e3
       0000000000000001  0000000000000000  WA       0     0     1
  [ 5] .rodata           PROGBITS         0000000000000000  000000e3
       000000000000000e  0000000000000000   A       0     0     1
  [ 6] .init_array       INIT_ARRAY       0000000000000000  000000f8
       0000000000000008  0000000000000008  WA       0     0     8
  [ 7] .rela.init_array  RELA             0000000000000000  000004f0
       0000000000000018  0000000000000018   I      13     6     8
  [ 8] .comment          PROGBITS         0000000000000000  00000100
       000000000000002c  0000000000000001  MS       0     0     1
  [ 9] .note.GNU-stack   PROGBITS         0000000000000000  0000012c
       0000000000000000  0000000000000000           0     0     1
  [10] .note.gnu.pr[...] NOTE             0000000000000000  00000130
       0000000000000020  0000000000000000   A       0     0     8
  [11] .eh_frame         PROGBITS         0000000000000000  00000150
       0000000000000078  0000000000000000   A       0     0     8
  [12] .rela.eh_frame    RELA             0000000000000000  00000508
       0000000000000048  0000000000000018   I      13    11     8
  [13] .symtab           SYMTAB           0000000000000000  000001c8
       0000000000000180  0000000000000018          14     8     8
  [14] .strtab           STRTAB           0000000000000000  00000348
       00000000000000c9  0000000000000000           0     0     1
  [15] .shstrtab         STRTAB           0000000000000000  00000550
       0000000000000085  0000000000000000           0     0     1
```
 注意，目标文件虽然包含了编译之后的机器代码，但他不能直接执行，因为在编译过程中我们用到了尚未定义的函数`add(int a, int b)`， 主程序中的add只是个声明而已。也就是在编译的时候，编译器完全不知道add的存在的，比如它位于内存的哪个区块，代码长什么样都不值，因此编译器只能暂时将此函数的挑战地址变为0，随后再去修正它。

 比如看看main.o的目标文件的内容, `objdump -s -d main.o`, 就是main编译后的内容， 左边是机器代码，右边是汇编
 ```
 Disassembly of section .text:

0000000000000000 <main>:
   0:	f3 0f 1e fa          	endbr64 
   4:	55                   	push   %rbp
   5:	48 89 e5             	mov    %rsp,%rbp
   8:	48 83 ec 10          	sub    $0x10,%rsp
   c:	48 8d 05 00 00 00 00 	lea    0x0(%rip),%rax        # 13 <main+0x13>
  13:	48 89 c7             	mov    %rax,%rdi
  16:	e8 00 00 00 00       	call   1b <main+0x1b>  //printf
  1b:	be 05 00 00 00       	mov    $0x5,%esi
  20:	bf 05 00 00 00       	mov    $0x5,%edi
  25:	e8 00 00 00 00       	call   2a <main+0x2a>  //add
  2a:	89 45 fc             	mov    %eax,-0x4(%rbp)
  2d:	b8 00 00 00 00       	mov    $0x0,%eax
  32:	c9                   	leave  
  33:	c3                   	ret    

 ```

 注意printf,add 函数的跳转地址都是0，这里的0会在后面的链接被修正  

 而为了让链接器能够定位到这些需要被修正的地址，在代码块中我们还可以找到一个重定位表 `objdump -r main.o`

```
main.o:     file format elf64-x86-64

RELOCATION RECORDS FOR [.text]:
OFFSET           TYPE              VALUE 
000000000000000f R_X86_64_PC32     .rodata-0x0000000000000004
0000000000000017 R_X86_64_PLT32    puts-0x0000000000000004     //prinf
0000000000000026 R_X86_64_PLT32    _Z3addii-0x0000000000000004 //call
0000000000000058 R_X86_64_PC32     .bss-0x0000000000000004
0000000000000060 R_X86_64_PLT32    _ZNSt8ios_base4InitC1Ev-0x0000000000000004
0000000000000067 R_X86_64_PC32     __dso_handle-0x0000000000000004
0000000000000071 R_X86_64_PC32     .bss-0x0000000000000004
000000000000007b R_X86_64_REX_GOTPCRELX  _ZNSt8ios_base4InitD1Ev-0x0000000000000004
0000000000000083 R_X86_64_PLT32    __cxa_atexit-0x0000000000000004


RELOCATION RECORDS FOR [.init_array]:
OFFSET           TYPE              VALUE 
0000000000000000 R_X86_64_64       .text+0x000000000000008a


RELOCATION RECORDS FOR [.eh_frame]:
OFFSET           TYPE              VALUE 
0000000000000020 R_X86_64_PC32     .text
0000000000000040 R_X86_64_PC32     .text+0x0000000000000034
0000000000000060 R_X86_64_PC32     .text+0x000000000000008a
```

##  链接

我们用`g++ main.o math.o -o main`将两个目标文件链接起来形成一个可执行文件；所以链接其实就是将编译好的目标文件，连同用到的一些静态库、运行时库，组合拼装成一个独立的可执行文件，其中就包括之前提到的地址修正。  

在这个时候，链接器会根据我们的目标文件或者静态库中的重定位表，找到那些需要被重定位的函数、全局变量，从而修正他们的地址。  
[重定向表](/img/redirect_table.png))  

如果我们在链接的时候，忘记提供必需的目标文件。  
```
g++ main.o -o main

/usr/bin/ld: main.o: in function `main':
main.c:(.text+0x26): undefined reference to `add(int, int)'
collect2: error: ld returned 1 exit status
```  

由于链接器找不到add函数的实现，于是报错引用未定义（符号未定义），意思就是我们用到add函数，链接器却无法找到它的定义