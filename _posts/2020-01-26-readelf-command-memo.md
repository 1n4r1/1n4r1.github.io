---
layout: post
title: Summary - ELF format
categories: Reversing
---

# Explanation
To understand the ELF (Executable and Linkable Format) executable more, take a memo with readelf command.

# Last Update
2020/01/26

# Environment
* OS: Kali linux 2019.4

# Solution
### 1. Preparation
In this memo, we use the following binary
#### Source code
{% highlight shell %}
root@kali:~# cat helloworld.c 
#include <stdio.h>

int main() {
   printf("Hello World!");
   return 0;
}
{% endhighlight %}

#### Compile
{% highlight shell %}
root@kali:~# gcc helloworld.c -o helloworld

root@kali:~# file helloworld
helloworld: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=5bf916655a810f6b92eb7a40ec1d293619d5f278, for GNU/Linux 3.2.0, not stripped
{% endhighlight %}

### 2. Structure
The ELF format object file consists of the following parts.
1. ELF header
2. Program headers
3. Segments to be used for the linker to allow execution
4. Section headers
5. Section for categorizing instruction and data
6. Data to execute

#### Showing ELF header
> The ELF header defines whether to use 32- or 64-bit addresses. The header contains three fields that are affected by this setting and offset other fields that follow them. The ELF header is 52 or 64 bytes long for 32-bit and 64-bit binaries respectively.

{% highlight shell %}
root@kali:~# readelf -h helloworld
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Shared object file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x1050
  Start of program headers:          64 (bytes into file)
  Start of section headers:          14704 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         11
  Size of section headers:           64 (bytes)
  Number of section headers:         30
  Section header string table index: 29
{% endhighlight %}

* ABI: ABI is short for Application Binary Interface and specifies a low-level interface between the operating system and a piece of executable code.

Also, the following is the definition of ELF header.<br>
We can find it with the command "man elf"$
{% highlight shell %}
ELF header (Ehdr)
       The ELF header is described by the type Elf32_Ehdr or Elf64_Ehdr:

           #define EI_NIDENT 16

           typedef struct {
               unsigned char e_ident[EI_NIDENT];
               uint16_t      e_type;
               uint16_t      e_machine;
               uint32_t      e_version;
               ElfN_Addr     e_entry;
               ElfN_Off      e_phoff;
               ElfN_Off      e_shoff;
               uint32_t      e_flags;
               uint16_t      e_ehsize;
               uint16_t      e_phentsize;
               uint16_t      e_phnum;
               uint16_t      e_shentsize;
               uint16_t      e_shnum;
               uint16_t      e_shstrndx;
           } ElfN_Ehdr;
{% endhighlight %}


#### Showing program header
> For the kernel to map segments into virtual address space with mmap(2) system call for runtime execution.
> Mandatory for execution

1. -l: for showing program header
2. -W: for showing more than 80 chars in one line
{% highlight shell %}
root@kali:~# readelf -l -W helloworld

Elf file type is DYN (Shared object file)
Entry point 0x1050
There are 11 program headers, starting at offset 64

Program Headers:
  Type           Offset   VirtAddr           PhysAddr           FileSiz  MemSiz   Flg Align
  PHDR           0x000040 0x0000000000000040 0x0000000000000040 0x000268 0x000268 R   0x8
  INTERP         0x0002a8 0x00000000000002a8 0x00000000000002a8 0x00001c 0x00001c R   0x1
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
  LOAD           0x000000 0x0000000000000000 0x0000000000000000 0x000568 0x000568 R   0x1000
  LOAD           0x001000 0x0000000000001000 0x0000000000001000 0x0001cd 0x0001cd R E 0x1000
  LOAD           0x002000 0x0000000000002000 0x0000000000002000 0x000158 0x000158 R   0x1000
  LOAD           0x002de8 0x0000000000003de8 0x0000000000003de8 0x000248 0x000250 RW  0x1000
  DYNAMIC        0x002df8 0x0000000000003df8 0x0000000000003df8 0x0001e0 0x0001e0 RW  0x8
  NOTE           0x0002c4 0x00000000000002c4 0x00000000000002c4 0x000044 0x000044 R   0x4
  GNU_EH_FRAME   0x002014 0x0000000000002014 0x0000000000002014 0x00003c 0x00003c R   0x4
  GNU_STACK      0x000000 0x0000000000000000 0x0000000000000000 0x000000 0x000000 RW  0x10
  GNU_RELRO      0x002de8 0x0000000000003de8 0x0000000000003de8 0x000218 0x000218 R   0x1

 Section to Segment mapping:
  Segment Sections...
   00     
   01     .interp 
   02     .interp .note.gnu.build-id .note.ABI-tag .gnu.hash .dynsym .dynstr .gnu.version .gnu.version_r .rela.dyn .rela.plt 
   03     .init .plt .plt.got .text .fini 
   04     .rodata .eh_frame_hdr .eh_frame 
   05     .init_array .fini_array .dynamic .got .got.plt .data .bss 
   06     .dynamic 
   07     .note.gnu.build-id .note.ABI-tag 
   08     .eh_frame_hdr 
   09     
   10     .init_array .fini_array .dynamic .got 
{% endhighlight %}

#### Showing section header
> The section headers define all the sections in the file for linking and relocation. 
> Actually not mandatory to execute the ELF file.

1. -S: for showing section header
2. -W: for showing more than 80 chars in one line
{% highlight shell %}
root@kali:~# readelf -S -W helloworld
There are 30 section headers, starting at offset 0x3970:

Section Headers:
  [Nr] Name              Type            Address          Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            0000000000000000 000000 000000 00      0   0  0
  [ 1] .interp           PROGBITS        00000000000002a8 0002a8 00001c 00   A  0   0  1
  [ 2] .note.gnu.build-id NOTE            00000000000002c4 0002c4 000024 00   A  0   0  4
  [ 3] .note.ABI-tag     NOTE            00000000000002e8 0002e8 000020 00   A  0   0  4
  [ 4] .gnu.hash         GNU_HASH        0000000000000308 000308 000024 00   A  5   0  8
  [ 5] .dynsym           DYNSYM          0000000000000330 000330 0000a8 18   A  6   1  8
  [ 6] .dynstr           STRTAB          00000000000003d8 0003d8 000084 00   A  0   0  1
  [ 7] .gnu.version      VERSYM          000000000000045c 00045c 00000e 02   A  5   0  2
  [ 8] .gnu.version_r    VERNEED         0000000000000470 000470 000020 00   A  6   1  8
  [ 9] .rela.dyn         RELA            0000000000000490 000490 0000c0 18   A  5   0  8
  [10] .rela.plt         RELA            0000000000000550 000550 000018 18  AI  5  23  8
  [11] .init             PROGBITS        0000000000001000 001000 000017 00  AX  0   0  4
  [12] .plt              PROGBITS        0000000000001020 001020 000020 10  AX  0   0 16
  [13] .plt.got          PROGBITS        0000000000001040 001040 000008 08  AX  0   0  8
  [14] .text             PROGBITS        0000000000001050 001050 000171 00  AX  0   0 16
  [15] .fini             PROGBITS        00000000000011c4 0011c4 000009 00  AX  0   0  4
  [16] .rodata           PROGBITS        0000000000002000 002000 000011 00   A  0   0  4
  [17] .eh_frame_hdr     PROGBITS        0000000000002014 002014 00003c 00   A  0   0  4
  [18] .eh_frame         PROGBITS        0000000000002050 002050 000108 00   A  0   0  8
  [19] .init_array       INIT_ARRAY      0000000000003de8 002de8 000008 08  WA  0   0  8
  [20] .fini_array       FINI_ARRAY      0000000000003df0 002df0 000008 08  WA  0   0  8
  [21] .dynamic          DYNAMIC         0000000000003df8 002df8 0001e0 10  WA  6   0  8
  [22] .got              PROGBITS        0000000000003fd8 002fd8 000028 08  WA  0   0  8
  [23] .got.plt          PROGBITS        0000000000004000 003000 000020 08  WA  0   0  8
  [24] .data             PROGBITS        0000000000004020 003020 000010 00  WA  0   0  8
  [25] .bss              NOBITS          0000000000004030 003030 000008 00  WA  0   0  1
  [26] .comment          PROGBITS        0000000000000000 003030 000026 01  MS  0   0  1
  [27] .symtab           SYMTAB          0000000000000000 003058 000600 18     28  45  8
  [28] .strtab           STRTAB          0000000000000000 003658 00020a 00      0   0  1
  [29] .shstrtab         STRTAB          0000000000000000 003862 000107 00      0   0  1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  l (large), p (processor specific)
{% endhighlight %}

### 3. Reference

* <a href="http://man7.org/linux/man-pages/man5/elf.5.html">http://man7.org/linux/man-pages/man5/elf.5.html</a>
* <a href="https://en.wikipedia.org/wiki/Executable_and_Linkable_Format">https://en.wikipedia.org/wiki/Executable_and_Linkable_Format</a>
* <a href="https://linux-audit.com/elf-binaries-on-linux-understanding-and-analysis/">https://linux-audit.com/elf-binaries-on-linux-understanding-and-analysis/</a>
