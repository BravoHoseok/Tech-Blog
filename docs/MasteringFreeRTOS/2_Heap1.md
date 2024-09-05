---
layout: default
title: 2_Heap1
parent: Topic_FreeRTOS
nav_order: 8
---

# Heap1 Overview
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

# 1. Alignment
Before diving into Heap1, we first look at what is Alignment in terms of C and processor. As mentioned in [Alignment in C], great care should be given to working with memory, for it causes the most time consuming task in the processor. To handle memory, the modern processor normally use 'Memory Addressing' which determines the computation speed of it. Computers commonly address their memory in workd-sized chunks. A word is a computer's natural unit for data. Its size is defined by the computers architecture. Modern general purpose computers generally have a word-size of either 4 byte (32bit) or 8 byte (64 bit). In other word, the word-sized data gurantee the most fast operation speed such as fetch and add.

<b>[Pic.1]</b> shows 4 bytes Alignment in an memory region. Each cell represent 1 byte memory space and 4 bytes belong to each 4 byte address. With such visualization, I will explain why we need alignment. 

<p align="center">
    <img src="../../../asset/images/FreeRTOS/Heap1_4BytesAlignment.png" width="500"/>
    <br><b>[Pic.1] 4 bytes Alignment</b>
</p>

Let's say, as shown in <b>[Pic.2]</b>, we put 4 byte <b>int</b>, the green color, at 0x00000000 of the memory. It makes the 4 byte integer properly algined without doing any special work, because an int on this memory architecture exactly fits 4 bytes into the first slot.

<p align="center">
    <img src="../../../asset/images/FreeRTOS/Heap1_4BytesAlignment_Int.png" width="500"/>
    <br><b>[Pic.2] Memory with an int</b>
</p>

If we decdied to put a char (Red color), a short (Blue Color), and an int (Green Color) into the memory region like <b>[Pic.3]</b>, it will make a problem.

<p align="center">
    <img src="../../../asset/images/FreeRTOS/Heap1_AlignmentProblem.png" width="500"/>
    <br><b>[Pic.3] Memory with an char, an short, and an int</b>
</p>

Why this is problem? Let take into a situation where we get the 4 bytes int. In <b>[Pic3]</b>, we would like to fetch the int. Then we first access 0x00000000 and extract the last green byte. Next, we access 0x00000004 and extract the three green bytes. Fianlly, we assemble the fetched green blocks with bit-shifiting to make an complete int. Compared to <b>[Pic.2]</b>, this case requires additional actions that comsume CPU clock cycle. This is the reason why we need 'Data Alignment' to speed up fetching the int in the memory.

To solve this problem, We put a padding byte (Gray color) between the char and short to make the int aligned. So, we can just access 0x00000004 to fetch the int at one time. <b>[Pic.4]</b> represents the aligned int.

<p align="center">
    <img src="../../../asset/images/FreeRTOS/Heap1_AlignmentPadding.png" width="500"/>
    <br><b>[Pic.4] Memory with an char, an short, and an aligned int</b>
</p>

But we still have problem, when fetching the char and short, because we need more actions to extract them such as bit-shifiting and & operation. Thus, we insert more padding bytes as shown in <b>[Pic.5] </b>. Imagine a situation where we fetch the 3 data and sum of them. We only need to access 0x00000000, 0x00000004, and 0x00000008, fetch and sum of them. The char and short will considered as 4 bytes but still they have its value with their size (1 byte and 2 bytes). 

<p align="center">
    <img src="../../../asset/images/FreeRTOS/Heap1_AlignmentPadding2.png" width="500"/>
    <br><b>[Pic.5] Memory with an char, an short, and an int, which are all aligned</b>
</p>

# 2. Structure Alignment
Because of Alignment, the size of data structure would be different than you might think. The code snippet below an simple real world example of structs. A programming beginner is likely to consider the size of structure below as 7 bytes.

```c
struct MyStruct
{
    char a;  // 1 byte
    short b; // 2 byte
    int c;   // 4 byte
};
```

As you learned about Alignment above, the size of the struct shall be 24 bytes. The elements of the structure shall be aligned and padded based on an element that is the largest one in it.

```c
struct MyStruct
{
    char a;  // 4 byte
    short b; // 2 byte
    int c;   // 4 byte
};
```

Padding bytes actually consumes the more memory space, but this is tread-off between speed and space. If the data are not aligned, the consequences of data structure misalignment vary widely between architectures. For exmaple, Some RISC, ARM and MIPS processors will respond with an alignment fault if an attempt is made to access a misaligned address. Specialized processors such as DSPs usually don’t support accessing misaligned locations. Most modern general purpose processors are capable of accessing misaligned addresses, albeit at a steep performance hit of at least two times the aligned access time.

# 3. Anlaysis of Heap1 Source Code


# Reference
- [Alignment in C]

[Alignment in C]: chrome-extension://efaidnbmnnnibpcajpcglclefindmkaj/https://hps.vi4io.org/_media/teaching/wintersemester_2013_2014/epc-14-haase-svenhendrik-alignmentinc-paper.pdf