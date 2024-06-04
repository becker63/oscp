https://ctf101.org/binary-exploitation/what-is-the-got/

https://systemoverlord.com/2017/03/19/got-and-plt-for-pwning.html

* The Global Offset Table (or GOT) is a section inside of programs that holds addresses of functions that are [[Dynamic Vs Static linking#Dynamic linking|Dynamically linked]]. 
* Each time the library changes, the addresses change
* ASLR changes these addresses everytime a program is ran 
* The addresses for these functions are calculated through the linker through a process called **relocation**
* As the GOT is part of the binary, it will always be a constant offset away from the base.
	* Therefore, if PIE is disabled or you somehow leak the binary base, you know the exact address that contains a `libc` function's address. 
	* If you perhaps have an arbitrary read, it's trivial to leak the real address of the `libc` function and therefore bypass ASLR.

# Relocations
* ## `.got`
	* This is the GOT, or ***Global Offset Table***. This is the actual table of offsets as filled in by the linker for external symbols.
* ## `.plt`
	* This is the PLT, or Procedure Linkage Table. These are stubs that look up the addresses in the `.got.plt` section, and either jump to the right address, or trigger the code in the linker to look up the address. (If the address has not been filled in to `.got.plt` yet.)
* ## `.got.plt`
	* This is the GOT for the PLT. It contains the target addresses (after they have been looked up) or an address back in the `.plt` to trigger the lookup. Classically, this data was part of the `.got` section.
* ## `.plt.got`
	* It seems like they wanted every combination of PLT and GOT! This just seems to contain code to jump to the first entry of the .got. I’m not actually sure what uses this. (If you know, please reach out and let me know! In testing a couple of programs, this code is not hit, but maybe there’s some obscure case for this.)

TL;DR: Those starting with `.plt` contain stubs to jump to the target, those starting with `.got` are tables of the target addresses.

The GOT is a _massive_ table of addresses; these addresses are the actual locations in memory of the `libc` functions. `puts@got`, for example, will contain the address of `puts` in memory. When the PLT gets called, it reads the GOT address and redirects execution there. If the address is empty, it coordinates with the `ld.so` (also called the **dynamic linker/loader**) to get the function address and stores it in the GOT.



