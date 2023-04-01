---
title: Core Principles of Reverse Engineering --Note2
tags: Reverse
categories: Reverse
top: false
---

# PE文件格式

## 种类

| 种类         | 主扩展名           |
| ------------ | ------------------ |
| 可执行系列   | EXE，SCR           |
| 库系列       | DLL，OCX，CPL，DRV |
| 驱动程序系列 | SYS，VXD           |
| 对象文件系列 | OBJ                |

## 零碎点

1. PE文件分PE头与PE体
2. PE头由各种结构体构成，说明了文件大小、执行地址等等信息，PE体分多个节区，如代码段、数据段、资源段等。
3. PE文件在内存中和在磁盘中大小不同
4. RVA(相对虚拟地址) VA(虚拟内存绝对地址) RAW(文件偏移)

## PE头分解

### DOS头

核心参数：

- e_magic:DOS签名，“MZ”（DOS可执行文件设计者）
- e_lfanew:指示NT头的偏移

### DOS存根

一个可忽略项，可理解为DOS下执行的数据

> 例如notepad.exe中的提示语:This program cannot be run in DOS mode

### NT头

```
typedef struct _IMAGE_NT_HEADERS {
    DWORD Signature;                             //签名“PE”
    IMAGE_FILE_HEADER FileHeader;                //文件头
    IMAGE_OPTIONAL_HEADER32 OptionalHeader;      //可选头
} IMAGE_NT_HEADERS32, *PIMAGE_NT_HEADERS32;
```

#### 文件头

```
typedef struct _IMAGE_FILE_HEADER {
    WORD    Machine;                  //机器码（每种CPU都有各自的固定值，如intel x86的14C）
    WORD    NumberOfSections;         //节区数量
    DWORD   TimeDateStamp;
    DWORD   PointerToSymbolTable;
    DWORD   NumberOfSymbols;
    WORD    SizeOfOptionalHeader;     //指示可选头的大小
    WORD    Characteristics;          //标识文件属性（是否为DLL等等）
} IMAGE_FILE_HEADER, *PIMAGE_FILE_HEADER
```

#### 可选头(直接把64位的搬过来了)

```
typedef struct _IMAGE_OPTIONAL_HEADER64 {
    WORD        Magic;                       //32-10B， 64-20B
    BYTE        MajorLinkerVersion;          
    BYTE        MinorLinkerVersion;
    DWORD       SizeOfCode;
    DWORD       SizeOfInitializedData;
    DWORD       SizeOfUninitializedData;
    DWORD       AddressOfEntryPoint;         //PE的RVA值，指出代码起始位置
    DWORD       BaseOfCode;
    ULONGLONG   ImageBase;                   //基准，VA = RVA + ImageBase
    DWORD       SectionAlignment;            //节区内存最小单位
    DWORD       FileAlignment;               //节区磁盘最小单位
    WORD        MajorOperatingSystemVersion;
    WORD        MinorOperatingSystemVersion;
    WORD        MajorImageVersion;
    WORD        MinorImageVersion;
    WORD        MajorSubsystemVersion;
    WORD        MinorSubsystemVersion;
    DWORD       Win32VersionValue;
    DWORD       SizeOfImage;                 //指定在虚拟内存中所占空间大小
    DWORD       SizeOfHeaders;               //PE头大小（第一节区所在位置与该值距文件开始偏移量相同）
    DWORD       CheckSum;
    WORD        Subsystem;                   //区分文件
                                              1 Driver文件（系统驱动）
                                              2 GUI文件（窗口应用程序）
                                              3 CUI文件（控制台应用程序）
    WORD        DllCharacteristics;
    ULONGLONG   SizeOfStackReserve;
    ULONGLONG   SizeOfStackCommit;
    ULONGLONG   SizeOfHeapReserve;
    ULONGLONG   SizeOfHeapCommit;
    DWORD       LoaderFlags;
    DWORD       NumberOfRvaAndSizes;         //
    IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES];
} IMAGE_OPTIONAL_HEADER64, *PIMAGE_OPTIONAL_HEADER64;
```

### 节区头

#### 属性

| 类别     | 访问权限         |
| -------- | ---------------- |
| code     | 执行，读取权限   |
| data     | 非执行，读写权限 |
| resource | 非执行，读取权限 |

#### 内容

```
#define IMAGE_SIZEOF_SHORT_NAME              8

typedef struct _IMAGE_SECTION_HEADER {
    BYTE    Name[IMAGE_SIZEOF_SHORT_NAME];
    union {
            DWORD   PhysicalAddress;
            DWORD   VirtualSize;            //内存中节区大小
    } Misc;
    DWORD   VirtualAddress;                 //内存中节区起始地址（RVA）
    DWORD   SizeOfRawData;                  //磁盘中节区大小
    DWORD   PointerToRawData;               //磁盘中节区起始地址
    DWORD   PointerToRelocations;
    DWORD   PointerToLinenumbers;
    WORD    NumberOfRelocations;
    WORD    NumberOfLinenumbers;
    DWORD   Characteristics;                //节区属性
} IMAGE_SECTION_HEADER, *PIMAGE_SECTION_HEADER;
```

## 计算

- RAW = RVA - (VA)VirtualAddress + PointerToRawData


# DLL文件

- （Dynamic Link Library）动态链接库
- 目的：解决在多进程时一个系统API可能会被多次调用，多次加载到内存造成空间浪费的问题
- 描述：
  - 库封装，单独组成DLL文件
  - 内存映射技术，实现多进程共享同一个文件
  - 更新库时替换对应DLL文件
- 加载方式：
  - 显式链接：使用时加载，用完后释放内存
  - 隐式链接：加载程序时将DLL全部加载，程序终止后释放内存

# 节区删除

1. 删除节区头，PEview中查看节区头起始地址及长度，在hex编辑器中将该区域用0覆盖
2. 删除节区，查看节区起始偏移及长度，hex编辑器中删除
3. 修改IMAGE_FILE_HEADER[Number Of Sections -= 1]
4. 修改IMAGE_OPTIONAL_HEADER[Size Of Image减少对应长度]

# 附表：

## DataDirectory结构体数组(64)[01239为重点]

```
#define IMAGE_DIRECTORY_ENTRY_EXPORT          0   // Export Directory
#define IMAGE_DIRECTORY_ENTRY_IMPORT          1   // Import Directory
#define IMAGE_DIRECTORY_ENTRY_RESOURCE        2   // Resource Directory
#define IMAGE_DIRECTORY_ENTRY_EXCEPTION       3   // Exception Directory
#define IMAGE_DIRECTORY_ENTRY_SECURITY        4   // Security Directory
#define IMAGE_DIRECTORY_ENTRY_BASERELOC       5   // Base Relocation Table
#define IMAGE_DIRECTORY_ENTRY_DEBUG           6   // Debug Directory
        IMAGE_DIRECTORY_ENTRY_COPYRIGHT       7   // (X86 usage)
#define IMAGE_DIRECTORY_ENTRY_ARCHITECTURE    7   // Architecture Specific Data
#define IMAGE_DIRECTORY_ENTRY_GLOBALPTR       8   // RVA of GP
#define IMAGE_DIRECTORY_ENTRY_TLS             9   // TLS Directory
#define IMAGE_DIRECTORY_ENTRY_LOAD_CONFIG    10   // Load Configuration Directory
#define IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT   11   // Bound Import Directory in headers
#define IMAGE_DIRECTORY_ENTRY_IAT            12   // Import Address Table
#define IMAGE_DIRECTORY_ENTRY_DELAY_IMPORT   13   // Delay Load Import Descriptors
#define IMAGE_DIRECTORY_ENTRY_COM_DESCRIPTOR 14   // COM Runtime descriptor
```

## Machine值

```
#define IMAGE_FILE_MACHINE_UNKNOWN           0
#define IMAGE_FILE_MACHINE_I386              0x014c  // Intel 386.
#define IMAGE_FILE_MACHINE_R3000             0x0162  // MIPS little-endian, 0x160 big-endian
#define IMAGE_FILE_MACHINE_R4000             0x0166  // MIPS little-endian
#define IMAGE_FILE_MACHINE_R10000            0x0168  // MIPS little-endian
#define IMAGE_FILE_MACHINE_WCEMIPSV2         0x0169  // MIPS little-endian WCE v2
#define IMAGE_FILE_MACHINE_ALPHA             0x0184  // Alpha_AXP
#define IMAGE_FILE_MACHINE_SH3               0x01a2  // SH3 little-endian
#define IMAGE_FILE_MACHINE_SH3DSP            0x01a3
#define IMAGE_FILE_MACHINE_SH3E              0x01a4  // SH3E little-endian
#define IMAGE_FILE_MACHINE_SH4               0x01a6  // SH4 little-endian
#define IMAGE_FILE_MACHINE_SH5               0x01a8  // SH5
#define IMAGE_FILE_MACHINE_ARM               0x01c0  // ARM Little-Endian
#define IMAGE_FILE_MACHINE_THUMB             0x01c2
#define IMAGE_FILE_MACHINE_AM33              0x01d3
#define IMAGE_FILE_MACHINE_POWERPC           0x01F0  // IBM PowerPC Little-Endian
#define IMAGE_FILE_MACHINE_POWERPCFP         0x01f1
#define IMAGE_FILE_MACHINE_IA64              0x0200  // Intel 64
#define IMAGE_FILE_MACHINE_MIPS16            0x0266  // MIPS
#define IMAGE_FILE_MACHINE_ALPHA64           0x0284  // ALPHA64
#define IMAGE_FILE_MACHINE_MIPSFPU           0x0366  // MIPS
#define IMAGE_FILE_MACHINE_MIPSFPU16         0x0466  // MIPS
#define IMAGE_FILE_MACHINE_AXP64             IMAGE_FILE_MACHINE_ALPHA64
#define IMAGE_FILE_MACHINE_TRICORE           0x0520  // Infineon
#define IMAGE_FILE_MACHINE_CEF               0x0CEF
#define IMAGE_FILE_MACHINE_EBC               0x0EBC  // EFI Byte Code
#define IMAGE_FILE_MACHINE_AMD64             0x8664  // AMD64 (K8)
#define IMAGE_FILE_MACHINE_M32R              0x9041  // M32R little-endian
#define IMAGE_FILE_MACHINE_CEE               0xC0EE
```

## Characteristics值

```
#define IMAGE_FILE_RELOCS_STRIPPED           0x0001  // Relocation info stripped from file.
#define IMAGE_FILE_EXECUTABLE_IMAGE          0x0002  // File is executable  (i.e. no unresolved externel references).
#define IMAGE_FILE_LINE_NUMS_STRIPPED        0x0004  // Line nunbers stripped from file.
#define IMAGE_FILE_LOCAL_SYMS_STRIPPED       0x0008  // Local symbols stripped from file.
#define IMAGE_FILE_AGGRESIVE_WS_TRIM         0x0010  // Agressively trim working set
#define IMAGE_FILE_LARGE_ADDRESS_AWARE       0x0020  // App can handle >2gb addresses
#define IMAGE_FILE_BYTES_REVERSED_LO         0x0080  // Bytes of machine word are reversed.
#define IMAGE_FILE_32BIT_MACHINE             0x0100  // 32 bit word machine.
#define IMAGE_FILE_DEBUG_STRIPPED            0x0200  // Debugging info stripped from file in .DBG file
#define IMAGE_FILE_REMOVABLE_RUN_FROM_SWAP   0x0400  // If Image is on removable media, copy and run from the swap file.
#define IMAGE_FILE_NET_RUN_FROM_SWAP         0x0800  // If Image is on Net, copy and run from the swap file.
#define IMAGE_FILE_SYSTEM                    0x1000  // System File.
#define IMAGE_FILE_DLL                       0x2000  // File is a DLL.
#define IMAGE_FILE_UP_SYSTEM_ONLY            0x4000  // File should only be run on a UP machine
#define IMAGE_FILE_BYTES_REVERSED_HI         0x8000  // Bytes of machine word are reversed.
```