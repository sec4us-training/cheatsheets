---
title: Shellcoding creation
category: Shellcoding
layout: 2017/sheet
tags: [Featured]
updated: 2020-07-30
keywords:
  - Shellcoding
  - Assembly
  - GDB
  - Disassemble
  - ObjDump
  - m4v3r1ck
---

Getting started
---------------
{: .-three-column}

### Introduction
{: .-intro}

This is a quick reference to getting started with Shellcoding.

- [Shellcoding Training](https://sec4us.com.br/shellcoding-for-exploitation-pres/){:target="_blank"} _(learn from scratch show to create your own shellcode)_
- [Author page](https://www.helviojunior.com.br/){:target="_blank"} _(Hélvio Junior - M4v3r1ck)_
- [LinkedIn](https://www.linkedin.com/in/helviojunior/){:target="_blank"} _(Hélvio Junior - LinkedIn)_
- [PDF Version](http://s.sec4us.com.br/shellcoding_cheatseet.pdf){:target="_blank"} _(Hélvio Junior - M4v3r1ck)_

### Shellcode Tester
Installing
```bash
git clone https://github.com/helviojunior/shellcodetester.git
cd shellcodetester/Linux
make
```

Assembling using shellcode tester
```bash
shellcodetester arquivo.asm
```

Adding a break-point before shellcode
```bash
shellcodetester arquivo.asm --break-point
```

For more information, windows version and releases see: 
[Shellcode Tester](https://github.com/helviojunior/shellcodetester.git){:target="_blank"} 


GDB Commands
--------------------
{: .-three-column}


### Open an application
```bash
gdb -q [path_da_aplicação]
```

Using his coredump
```bash
gdb -q [path_da_aplicação] [path_core_dump]
```

### Command help
```bash
(gdb) help [comando]
(gdb) help run
```

### Run app without parameter
```bash
(gdb) run
```

### Check permissions
```bash
(gdb) checksec
```

### Run app with parameter
Hard-coded parameter
```bash
(gdb) run AAAAA
```

Parameter from an external command
```bash
(gdb) run <(echo -n “AAAA”)
(gdb) run <(python -c 'print "A" * 500 ')
```

Passing data to stdin from external command
```bash
(gdb) run < <(echo -n “AAAA”)
(gdb) run < <(python -c 'print "A" * 500 ')
```

### Breakpoints
Adding an breakpoint
```bash
(gdb) b *main
(gdb) b *0x01020304
```

Adding conditional breakpoint
```bash
(gdb) b main.c:260 if (resp_pool->first==0x4141414141414141)
```

List previous breakpoints
```bash
(gdb) info breakpoints
```

Delete one
```bash
(gdb) del 1  # 1 is the breakpoint id
(gdb) del 5  # 5 is the breakpoint id
```

Delete all breakpoints
```bash
(gdb) del
```

### Disassemble
Using function name
```bash
(gdb) disassemble main
```

Using memory position
```bash
(gdb) disassemble 0x000011e9
(gdb) disassemble 0x000011e9,+100
```

Displaying opcodes 
```bash
(gdb) disassemble/r main
```

### Displaying registers
```bash
(gdb) info registers
(gdb) info registers eax
```

### Displaying memory data
Using memory address
```bash
(gdb) print 0x01020304
```
Using debug symbol of variable name
```bash
(gdb) print &nome_variavel
```

### Struct
View struct members
```bash
(gdb) ptype struct [nome_da_struct]
```

View struct members and offset
```bash
(gdb) ptype/o struct [nome_da_struct]
```

Parsing memory data with struct
```bash
(gdb) print/x *(struct sockaddr_in *) &nome_variavel
(gdb) print/x *(struct sockaddr_in *) 0x01020304
```


Assembly
--------------------
{: .-two-column}

### Registers

Register is a small space used by CPU to store information.

### Registers - 32 bits

| 32 bits  | 16 bits | 8 bits    | |
|          |         | High | Low  |
| -------- | ------- | ---- | ---- |
| eax    | ax    | ah | al |
| ecx    | ax    | ch | cl |
| edx    | dx    | dh | dl |
| ebx    | bx    | bh | bl |
| esp    | sp    |      | spl|
| ebp    | bp    |      | bpl|
| esi    | si    |      | sil|
| edi    | di    |      | dil|

### Registers - 64 bits

| 64 bits  | 32 bits  | 16 bits | 8 bits    | |
|          |          |         | High | Low  |
| -------- | -------- | ------- | ---- | ---- |
| eax / r0 | eax / r0d | ax / r0w | ah | al / r0b |
| ecx / r1 | ecx / r1d | cx / r1w | ch | cl / r1b |
| rdx / r2 | edx / r2d | dx / r2w | dh | dl / r2b |
| rbx / r3 | ebx / r3d | bx / r3w | bh | bl / r3b |
| rsp / r4 | esp / r4d | sp / r4w |   | spl / r4b |
| rbp / r5 | ebp / r5d | bp / r5w |  | bpl / r5b |
| rsi / r6 | esi / r6d | si / r6w |  | sil / r6b |
| rdi / r7 | edi / r7d | di / r7w |  | dil / r7b |
| r8 | r8d | r8w |  | r8b |
| r9 | r9d | r9w |  | r9b |
| r10 | r10d | r10w |  | r10b |
| r11 | r11d | r11w |  | r11b |
| r12 | r12d | r12w |  | r12b |
| r13 | r13d | r13w |  | r13b |
| r14 | r14d | r14w |  | r14b |
| r15 | r15d | r15w |  | r15b |


### IP - Instruction Pointer

Points to next instruction to be executed


| Register   | Description         |
| ---------- | ------------------- |
| `ip`         | 16 bits           |
| `eip`        | 32 bits           |
| `rip`        | 64 bits           |


### SP - Stack Pointer

Points to top of stack


| Register   | Description         |
| ---------- | ------------------- |
| `sp`         | 16 bits           |
| `esp`        | 32 bits           |
| `rsp`        | 64 bits           |



### Assembly insctuctions

This table shows the main ASM instructions used to create shellcode

| Instruction   | Description         |
| ---------- | ------------------- |
| `INT3`     | Generate breakpoint trap           |
| `CALL`     | Call a procedure           |
| `CLD`      | Clear Direction Flag           |
| `DEC`      | Decrement by 1           |
| `INC`      | Increment by 1           |
| `JMP`      | Jump           |
| `LEA`      | Load Effective Address           |
| `MOV`      | Move           |
| `NOP`      | No Operation           |
| `POP`      | Pop a Value from the Stack           |
| `PUSH`     | Push Word, Doubleword or Quadword Onto the Stack           |
| `RET`      | Return from Procedure           |
| `SHL`      | Shift Left           |
| `SHR`      | Shift Right           |
| `XOR`      | Logical Exclusive OR           |


See: [Intel® 64 and IA-32 Architectures Software Developer Manuals](https://software.intel.com/content/www/us/en/develop/articles/intel-sdm.html){:target="_blank"}

### ASM Code sample for 32 bits
This sample can be used to generate: Shellcode or ELF
```assembly
[BITS 32]
global _start
section .text
_start:
    ; instructions
    nop
    nop
```

This sample can be used to generate: Windows PE
```assembly
[BITS 32]
global WinMain
section .text
WinMain:
    ; instructions
    nop
    nop
```

### ASM Code sample for 64 bits
This sample can be used to generate: Shellcode or ELF
```assembly
[BITS 64]
global _start
section .text
_start:
    ; instructions
    nop
    nop
```

This sample can be used to generate: Windows PE
```assembly
[BITS 64]
global WinMain
section .text
WinMain:
    ; instructions
    nop
    nop
```


### Assembling for Shellcoding
Using shellcode tester
```bash
shellcodetester input_file.asm
shellcodetester input_file.asm --break-point
```

In order to assembly to Shellcode we must have only our instrunctions without ELF or PE structure.
```bash
nasm input_file.asm -o output_file.o
```

Display raw data in HEX format (does not matter if it is an 32 bit or 64 bits code)
```bash
cat output_file.o | msfvenom -p - -a x86 --platform win -e generic/none -f hex
```

Display raw data in Python array format (does not matter if it is an 32 bit or 64 bits code)
```bash
cat output_file.o | msfvenom -p - -a x86 --platform win -e generic/none -f python
```

### Assembling for Execution
Linux 32 bits
```bash
nasm -f elf32 input_file.asm -o output_file.o
ld -o executable_file output_file.o -m elf_i386
```

Linux 64 bits
```bash
nasm -f elf64 input_file.asm -o output_file.o
ld -o executable_file output_file.o -m elf_x86_64
```

Windows 32 bits
```bash
nasm -f win32 input_file.asm -o output_file.o
gcc -o executable_file.exe output_file.o
```

Windows 64 bits
```bash
nasm -f win64 input_file.asm -o output_file.o
gcc -o executable_file.exe output_file.o
```

Tricks
--------------------
{: .-two-column}

### Reverse text using python
```python
texto = "Treinamento Shellcoding\n"
texto[::-1]
len(texto[::-1])
texto[::-1].encode('hex')
```

### ASM to JMP, CALL, POP
This sample can be used to generate: Shellcode or ELF
```assembly
[BITS 32]
global _start
section .text
_start:
    jmp step1
step2:
    pop ecx ; Save text addr at ECX
    nop
    ; Other instructions
step1:
    call step2                  
    db "Treinamento Shellcoding", 0x0a, 0x00
```

### Debug Symbols
Extract debug symbol from an binary executable file
```bash
objcopy --only-keep-debug [elf_file] symbols.debug
```

Use debug symbol insed of GDB
```bash
(gdb) symbol-file symbols.debug
```

Calling convention
--------------------
{: .-two-column}

### Syscall Linux 32 bits
When using syscall the parameters must be passed using registers. Return value, when exists, will be stored at `EAX`.

Follows pseudo-function:
```jsx
eax = func1(ebx, ecx, edx, esi, edi)
```

| Register   | Description         |
| ---------- | ------------------- |
| `eax`      | Syscall number      |
| `ebx`      | 1st parameter       |
| `ecx`      | 2nd parameter       |
| `edx`      | 3rd parameter       |
| `esi`      | 4th parameter       |
| `edi`      | 5th parameter       |

Syscall number can be found at: /usr/include/x86_64-linux-gnu/asm/unistd_32.h
Function desription can be fount with command `man 2 [function_name]`

### Syscall Linux 64 bits
When using syscall the parameters must be passed using registers. Return value, when exists, will be stored at `RAX`.

Follows pseudo-function:
```jsx
eax = func1(rdi, rsi, rdx, r10, r8, r9)
```

| Register   | Description         |
| ---------- | ------------------- |
| `rax`      | Syscall number      |
| `rdi`      | 1st parameter       |
| `rsi`      | 2nd parameter       |
| `rdx`      | 3rd parameter       |
| `r10`      | 4th parameter       |
| `r8`       | 5th parameter       |
| `r9`       | 6th parameter       |

Syscall number can be found at: /usr/include/x86_64-linux-gnu/asm/unistd_64.h
Function desription can be fount with command `man 2 [function_name]`

### Windows and Linux 32 bits stack
All parameters must be pushed onto stack. Remember that parameters must be pushed in reverse order. 

Return value, when exists, will be stored at `EAX`.

Follows pseudo-function:
```jsx
eax = func1(ESP, ESP + 0x04, ESP + 0x08, ...)
```

| Stack Position          | Description         |
| ----------------- | ------------------- |
| `esp + 0x00`      | 1st parameter       |
| `esp + 0x04`      | 2nd parameter       |
| `esp + 0x08`      | 3rd parameter       |
| `esp + 0x0C`      | 4th parameter       |
| `esp + 0x10`      | 5th parameter       |
| ...               | Other parameter     |

### Windows and Linux 64 bits
When using syscall the parameters must be passed using registers. Return value, when exists, will be stored at `RAX`.

Follows pseudo-function:
```jsx
eax = func1(int a, int b, int c, int d, int e)
// a on RCX, b on RDX, c on R8, d on R9, e pushed onto stack
```
Where: 

| Register / Stack Position  | Description         |
| ---------- | ------------------- |
| `rcx`      | Syscall number      |
| `rdx`      | 1st parameter       |
| `r8`       | 2nd parameter       |
| `r9`       | 3rd parameter       |
| `esp + 0x00`      | 4th parameter       |
| `esp + 0x04`      | 5th parameter       |
| ...               | Other parameter     |


### AMD64 Application Binary Interface (ABI)
This section describes the standard processes and conventions that one function (the caller) uses to make calls into another function (the callee) in x64 code

- Alignment
  - The stack must be 16-byte aligned
- Argument passing
  - RCX, RDX, R8, R9, remaining arguments get pushed on the stack in right-to-left order
- Shadow space
  - Must exists an strict one-to-one correspondence between a function call's arguments and the registers used for those arguments
- Return
  - A scalar return value that can fit into 64 bits is returned through RAX

See: [x64 calling convention](https://docs.microsoft.com/en-us/cpp/build/x64-calling-convention?view=vs-2019){:target="_blank"}

### ASM 64 bits tricks
```assembly
[BITS 64]
global _start
section .text
_start:
    ; This 2 instructions must be always present at start of an shellcode
    cld                         ; Clear direction flag
    and rsp, 0xFFFFFFFFFFFFFFF0 ; Align stack at 16-byte

    ; Creating one shadow space (without NULL Byte)
    xor eax,eax                 ; Fill EAX with Zero
    push eax                    ; Push 0x00 onto stack
```

## Also see
{: .-one-column}

* [Intel® 64 and IA-32 Architectures Software Developer Manuals](https://software.intel.com/content/www/us/en/develop/articles/intel-sdm.html){:target="_blank"} _(software.intel.com)_
* [Free Training Shellcoding for 64 bits - Brazilian Language](https://www.youtube.com/watch?v=ySKEF8MHcZA){:target="_blank"} _(YouTube)_

