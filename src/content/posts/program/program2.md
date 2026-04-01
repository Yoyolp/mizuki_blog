---
title: Program02
published: 2025-11-12
description: "程序员的自我修养, ELF 文件格式描述"
tags: ["linux"]
category: linux
draft: false
---

## 文件头

ELF 目标文件格式的最前部是ELF 文件头，它包含了描述整个文件的基本属性，比如ELF 文件版本，目标机器型号，程序入口地址等。

我们可以使用readelf 命令来详细查看ELF 文件

```bash
readelf -h SimpleSection.o
```

ELF 文件头中**定义了ELF 魔数，文件机器字节长度。数据存储方式，版本，运行平台，ABI版本，ELF 重定位类型，硬件平台，硬件平台版本，入口地址，程序入口和长度，段表的位置以及短的数量**

。 这些数值中有关描述ELF 目标平台的部分，与我们常见的32位Intel 的硬件平台基本一样， ELF 文件由结构及相关数据常数被定义在 `/usr/include/elf.h` 中，因为elf文件在各种平台下都通用，分别叫做 `Elf32_Ehdr` & `Elf32_Ehdr` 32 位版本和 64位版本的ELF 文件的文件头内容是一样的，是不过有些成员大小不一样。为了对每一个成员大小做出明确的规定以便于在不同的编译环境下都有相同的字段长度，"elf.h" 使用 typedef 定义了一套自己的变量体系

| 自定义类型  | 描述                   | 原始类型 | 长度（BYTE） |
| ----------- | ---------------------- | -------- | ------------ |
| Elf32_Addr  | 32 位版本程序地址      | uint32_t | 4            |
| Elf32_Half  | 32位版本的无符号短整型 | uint16_t | 2            |
| Elf32_Off   | 32位版本的偏移地址     | uint32_t | 4            |
| Elf32_Sword | 32位版本有符号整型     | uint32_t | 4            |
| Elf32_Word  | 32位版本无符号整型     | uint32_t | 4            |
| Elf64_Addr  | 64版本程序地址         | uint64_t | 8            |
| Elf64_Half  | 64版本的无符号短整型   | uint16_t | 2            |
| Elf64_Off   | 64版本的偏移地址       | uint64_t | 8            |
| Elf32_Sword | 64版有符号整型         | uint32_t | 4            |
| Elf32_Word  | 64版本无符号整型       | uint32_t | 4            |

```c
typedef struct
{
  unsigned char	e_ident[EI_NIDENT];	/* Magic number and other info */
  Elf32_Half	e_type;			/* Object file type */
  Elf32_Half	e_machine;		/* Architecture */
  Elf32_Word	e_version;		/* Object file version */
  Elf32_Addr	e_entry;		/* Entry point virtual address */
  Elf32_Off	e_phoff;		/* Program header table file offset */
  Elf32_Off	e_shoff;		/* Section header table file offset */
  Elf32_Word	e_flags;		/* Processor-specific flags */
  Elf32_Half	e_ehsize;		/* ELF header size in bytes */
  Elf32_Half	e_phentsize;		/* Program header table entry size */
  Elf32_Half	e_phnum;		/* Program header table entry count */
  Elf32_Half	e_shentsize;		/* Section header table entry size */
  Elf32_Half	e_shnum;		/* Section header table entry count */
  Elf32_Half	e_shstrndx;		/* Section header string table index */
} Elf32_Ehdr;
```

### ELF魔数

最前面的Magic 的16个字节 刚好对应 Elf32_Ehdr -> e_ident 这个成员, 这16个字节刚好对应ELF 文件的平台，最开始的4个字节是所有ELF 文件都必须相同的标识码 `0x7F 0x45 0x4c 0x46` 第一个字节对应ASCII 字符里面的DEL 控制符，后面3个字节刚好是ELF 这3个字母的ASCII码 这4个字节被称为ELF 文件的魔数，几乎所有可执行文件的最开始几个字节都是魔数， 这种魔术被用来确认文件类型，操作系统在加载可执行文件的时候会确认魔数是否正确，乳沟不正确会拒绝加载

### 文件类型

***e_type*** 成员表示ELF 文件类型， 每一个文件类型对应一个常量。系统通过这个常量来判断ELF 的真正文件类型，而不是通过文件的扩展名。相关常量以 `ET_` 开头

| 常量    | 值 | 含义                         |
| ------- | -- | ---------------------------- |
| ET_REL  | 1  | 可重定位文件，一般为.o 文件  |
| ET_EXEC | 2  | 可执行文件                   |
| ET_DYN  | 3  | 共享目标文件，一般为.so 文件 |

### 机器类型

ELF 文件格式被设计成可以在多个平台下使用。这并不表示同一个ELF 文件可以在不同平台下使用，而是表示不同平台下ELF 文件都遵循同一套ELF文件标准，

***e_machine*** 成员就表示该ELF文件的平台属性，比如3表示的ELF文件只能在Intel x86 机器下使用，相关的量以 `EM_ `开头

| 常量     | 值 | 含义           |
| -------- | -- | -------------- |
| EM_M32   | 1  | AT&T WE 32100  |
| EM_SPARC | 2  | SPARC          |
| EM_386   | 3  | Intel x86      |
| EM_68K   | 4  | Motorola 68000 |
| EM_88K   | 5  | Motorola 88000 |
| EM_860   | 6  | Intel 80860    |

### 段表

ELF 文件中具有许多的段，段表就是保存这些段的基本属性的结构。段表是ELF 文件中除了文件头以外最重要的文件结构，它描述了ELF 的各个段的星系，比如每一个段的段名，段的长度，在文件中的偏移，读写权限及段的其他属性，段表在ELF 文件头的 ***e_shoff*** 成员决定

我们使用的 `objdump -h` 来查看 ELF 文件中所包含的段 但是 这个命令只是把ELF 文件中关键的段显示了出来，而省略了其他的辅助性的段，比如 符号表， 字符串表，段名字符串表，重定位表etc。
我们可以使用 readelf 工具来查看看ELF 文件的段，他显示出来的结果才是真正的段表结构

```bash
readelf -S SimpleSection.o
```

readelf 输出结果就是ELF 文件段表的内容，那么就让我们对照这个输出来看看段表的结构。段表的结构比较简单，他是一个以 ***Elf32_Shdr* **又被称为段描述符

```c
typedef struct
{
  Elf32_Word	sh_name;		/* Section name (string tbl index) */
  Elf32_Word	sh_type;		/* Section type */
  Elf32_Word	sh_flags;		/* Section flags */
  Elf32_Addr	sh_addr;		/* Section virtual addr at execution */
  Elf32_Off	sh_offset;		/* Section file offset */      // **段地址对齐**
  Elf32_Word	sh_size;		/* Section size in bytes */
  Elf32_Word	sh_link;		/* Link to another section */
  Elf32_Word	sh_info;		/* Additional section information */
  Elf32_Word	sh_addralign;		/* Section alignment */
  Elf32_Word	sh_entsize;		/* Entry size if section holds table */
} Elf32_Shdr;

typedef struct
{
  Elf64_Word	sh_name;		/* Section name (string tbl index) */
  Elf64_Word	sh_type;		/* Section type */
  Elf64_Xword	sh_flags;		/* Section flags */
  Elf64_Addr	sh_addr;		/* Section virtual addr at execution */
  Elf64_Off	sh_offset;		/* Section file offset */
  Elf64_Xword	sh_size;		/* Section size in bytes */
  Elf64_Word	sh_link;		/* Link to another section */
  Elf64_Word	sh_info;		/* Additional section information */
  Elf64_Xword	sh_addralign;		/* Section alignment */
  Elf64_Xword	sh_entsize;		/* Entry size if section holds table */
} Elf64_Shdr;
```

### 段地址对齐

有些段对段地址的对齐有要求，比如我们假设有一个段刚开始的位置包含了一个double 变量，因为Intel x86 系统要求浮点数的存储地址必须是本身的整数倍，也就是说保存的 double 变量的地址必须是 8 字节的整数倍，这样对一个段来说，它的 sh_addr 必须是8字节的整数倍。由于地址对齐的数量都是2的指数倍，sh_addralign 表示地址对齐数量中的质数，即sh_addrlign = 3 表示对齐位$2^3$倍，一此类推

所以一个段的地址必须满足下面的条件

$$
\text{sh\_addr} \mod (2^{\text{sh\_addralign}}) = 0
$$

如果 sh_addralign = 0 | 1 则表示该段没有对齐要求

### 段的类型

`sh_type` 段的名字只是在链接和编译过程中有意义，但是它不能真正表示段的类型 我们也可以将一个数据段的命名位 `.text` 对于编译器和连接器来说主要决定段的属性的是段的类型和段的标志位 ***sh_type*** & ***sh_flags***
相关常量以 `SHT_` 开头

| 常量         | 值 | 含义                                                         |
| ------------ | -- | ------------------------------------------------------------ |
| SHT_NULL     | 0  | 无效段                                                       |
| SHT_PROGBITS | 1  | 程序段。代码段，数据段都是这种类型的                         |
| SHT_SMTAB    | 2  | 表示该段的内容位符号表                                       |
| SHT_STRAB    | 3  | 表示该段的内容位字符串表                                     |
| SHT_RELA     | 4  | 重定位表，该段包含了重定位信息，具体参考静态地址决议和重定位 |
| SHT_HASH     | 5  | 符号表和哈希表                                               |
| SHT_DYNAMIC  | 6  | 动态链接信息                                                 |
| SHT_NOTE     | 7  | 提示性信息                                                   |
| SHT_NOBITS   | 8  | 表示该段在文本中没有内容， .bss 段                           |
| SHT_REL      | 9  | 该段包含重定位信息                                           |
| SHT_SHLIB    | 10 | 保留                                                         |
| SHT_DNYSYM   | 11 | 动态链接的符号表                                             |

### 段的标志位

表示该段在进程虚拟地址空间中的属性，比如是否可惜，是否可执行等。相关常量以 `SHF_` 开头

| 常量          | 值 | 含义                                                                       |
| ------------- | -- | -------------------------------------------------------------------------- |
| SHF_WRITE     | 1  | 表示该段在进程空间中可写                                                   |
| SHF_ALLOC     | 2  | 表示该段在进程空间中需要分配空间，有些包含只是或者控制信息的段不需要在进程 |
| SHF_EXECINSTR | 4  | 表示该段在进程空间中可以被执行，一般指代码段                               |

### 段的链接信息

如果段的类型是与链接相关的（无论是动态链接或者是静态链接），比如重定位表，符号表等 则 ***sh_line*** & ***sh_info** *

而对于其他类型的段，这两个成员没有意义

| sh_type     | sh_link                                            | sh_info                            |
| ----------- | -------------------------------------------------- | ---------------------------------- |
| SHT_DYNAMIC | 这个段使用的***字符串表***在段表中的下标   | 0                                  |
| SHT_HASH    | 该段所使用的***符号表***在段表中下标       | 0                                  |
| SHT_REL     | 该段所使用的***相应符号表***在段表中的下表 | 该重定位表所作用的段在段表中的下标 |
| SHT_RELA    | 该段所使用的***相应符号表***在段表中的下表 | 该重定位表所作用的段在段表中的下标 |
| SHT_SYMTAB  | 操作系统相关的                                     | 操作系统相关的                     |
| SHT_DYNSYM  | 操作系统相关的                                     | 操作系统相关的                     |
| other       | SHN_UNDEF                                          | 0                                  |

### 重定位表

链接器在处理目标文件的时，须要对目标文件中某些部位进行重定位，即代码段和数据段中那些对绝对地址的引用的位置，这些重定位星系都记录在ELF 文件的重定位表中，对于每个需要重定位的代码段或者数据段，都会有相应的重定位表

### 字符串表

ELF 文件中用到了很多字符串，比如段名，变量名等，因为字符串长度往往是不定的，所以用固定的结构来表示它比较困难，一种很常见的做法就是把字符串集中起来存放到一个表，然后使用字符串在表中的偏移来引用字符串

## 链接的接口

链接的本质就是要把多个不同的目标文件之间相互“粘”到一起，但是为了使得不同的目标文件之间能够相互拼合，必须要有固定的规则。

在连接中，目标文件之间相拼合实际上就是目标文件之间对地址的引用，即对函数和变量的地址的引用，在连接中我们将函数和变量统称为符号（Symbol），**函数名或者变量名**就是**符号名**

**在连接中整个连接的过程正是基于符号才能够正确完成** 整个连接过程中很关键的就是每一个目标文件都有一个相应的符号表，这个表记录了目标文件中所用到的所有符号，每一个定义的符号有一个定义的值，叫做符号值，对于变量和函数来说符号值就是他们的**地址**。除了函数和变量之外，还有其他几种不常用的符号

+ 定义在本目标文件的全局符号，可以被其他目标文件引用。
+ 在本目标文件中引用的全局符号，却没有定义在本目标文件中，这一般叫做外部符号（printf）
+ 段名 这种符号往往由编译器产生，它的值就是该段的起始地址
+ 局部符号，这类符号旨在编译单元内部可见，调试器可以使用这些符号来分析程序或者崩溃时的核心转储文件，这些局部符号对于链接过程中没有作用
+ 行号信息，即目标文件指令与源代码中代码行的对应关系，也是可选的

我们可以使用 `nm` 来查看目标文件中的符号表，

```bash
nm SimpleSection.o
```

### ELF 符号表的结构

ELF 文件表在往往是文件中的一个段，段名一般叫'.symtab' 符号表的结构是一个Elf32_Sym 结构的数组每个Elf32_Sym 结构因对一个符号

```c
typedef struct
{
  Elf32_Word	st_name;		/* Symbol name (string tbl index) */
  Elf32_Addr	st_value;		/* Symbol value */
  Elf32_Word	st_size;		/* Symbol size */
  unsigned char	st_info;		/* Symbol type and binding */
  unsigned char	st_other;		/* Symbol visibility */
  Elf32_Section	st_shndx;		/* Section index */
} Elf32_Sym;

typedef struct
{
  Elf64_Word	st_name;		/* Symbol name (string tbl index) */
  unsigned char	st_info;		/* Symbol type and binding */
  unsigned char st_other;		/* Symbol visibility */
  Elf64_Section	st_shndx;		/* Section index */
  Elf64_Addr	st_value;		/* Symbol value */
  Elf64_Xword	st_size;		/* Symbol size */
} Elf64_Sym;
```

### 符号类型和绑定信息

`si_info`该成员低4位表示符号的类型，高28位表示符号的绑定信息

这是符号的绑定信息

| 宏定义名字 | 值 | 说明                                 |
| ---------- | -- | ------------------------------------ |
| STB_LOCAL  | 0  | 局部符号，对于目标文件的外部都不可见 |
| STB_GLOBAL | 1  | 局部符号，外部可见                   |
| STB_WEAK   | 2  | 弱引用，详见“弱符号与强符号”       |

这是符号类型

| 宏定义名    | 值 | 说明                                                                                                                 |
| ----------- | -- | -------------------------------------------------------------------------------------------------------------------- |
| STT_NOTYPE  | 0  | 未知类型符号                                                                                                         |
| STT_OBJECT  | 1  | 这个符号是一个数据对象，比如变量，数组等                                                                             |
| STT_FUNC    | 2  | 这个符号是个函数或其他可执行代码                                                                                     |
| STT_SECTION | 3  | 这个符号表示一个段买这种符号必须是STB_LOCAL的                                                                        |
| STT_FILE    | 4  | 这个符号表示文件名，一般都是该文件所对应的源文件名，他一定是STB_LOCAL 类型的<br />，并且他的 st_shndx 一定是 SHN_ABS |

### 符号所在段

st_shndx 如果符号定义在本目标文件中，那么这个成员表示符号所在段表中的下标；但是如果符号不是定义在本目标文件中，或者对于有些特殊符号，`sh_shndx` 的值有些特殊

符号表所在段的特殊常量

| 宏定义名   | 值     | 说明                                                                                      |
| ---------- | ------ | ----------------------------------------------------------------------------------------- |
| SHN_ABS    | 0xfff1 | 表示该符号包含了一个绝对值，比如表示文件名的符号就属于这种类型的                          |
| SHN_COMMON | 0xfff2 | 表示该符号是一个"COMMON" 块，一般来说，***未初始化的全局符号定义就是这种类型的*** |
| SHN_UNDEF  | 0      | 表示这个符号没有定义，这个符号表表示该符号在本目标文件被引用到但是，定义在其他目标文件中  |

### 符号值

每一个符号都有一个对应的值，需要按照这几种情况区别对待

+ 在目标文件中，如果是符号的定义并且该符号不是'COMMON'块，则st_value 表示该符号在段中的偏移。即符号所对应的函数或变量位于由 st_shndx 指定的段，偏移 st_value 的位置
+ 在可执行文件中，st_value 表示富豪的虚拟地址。这个虚拟地址对于动态连接器来所十分有用

我们可以使用 `readelf` 工具来查看ELF 文件的符号

```bash
readelf -s SimpleSection.o
```

***输出结果中的 Ndx列就是表示该符号所表示的段***
