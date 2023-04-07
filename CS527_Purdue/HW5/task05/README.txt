The task has multiple layers of defense, however there are 3 features of this program that allows for exploit: 1. it allocates a read and writeable memory space at 0x200000 and copies the input buffer to that location, which could be used as the pseudo-stack for my ROP chain since its address is always the same, bypassing ASLR. 2. it allocates a space at 0x300000 and copies all 2-byte integers at that location. Since the memory protection bit (local_1f) can be overwritten with whichever number I sent it (bypassing NX), this could be modified into executable memory that contains 2-byte instructions. Its address never changes, thus bypassing PIE. 3. when the program receives 7, it leads us to a function pointer (local_27) that could be overwritten.

However, if I want to make an ROP chain, a ret instruction is needed at the end of each chain, meaning that I realistically have only access to syscall at the end of the chain and 1-byte instructions such as push and pop. Also, in order to make use of the readable/writable memory at 0x200000, I have to move my rsp to that location, and the only way to do that is to use local_27. 

One useful instruction I found is movsd (\xa5), which copies a 4-byte dword from esi to edi, which are set to 0x200000 and the address at local_27 respectively when local_27 is called. Thus I devised a plan to write the bytes '\xa5\x50\x5c\x59' (movsd; push rax; pop rsp; pop rcx;) at 0x200000 and copy it to the location that contains '\xa5\xc3', so that when the instruction is executed, rsp will be assigned rax's value (0x200000) and will be added 8 (pop rcx;), ending its location at 0x200008, where I will write my ROP chain. Note that a ret instruction will eventually be executed after a few filler instructions, leading rip to my ROP chain.

At the ROP chain I injected addresses of pop; ret; chains to input the right values into the right registers. Since I have enough space for only 1 syscall, I decided to perform execve('/bin/cat flag.txt') which does not drop the privilege. Note that due to flag.txt's name being too long, I had to link it to another file with shorter name so that my ROP chain does not go beyond the allowed 128 bytes.

Please see details in my code. 

Reference:
http://www.nacad.ufrj.br/online/intel/vtune/users_guide/mergedProjects/analyzer_ec/mergedProjects/reference_olh/mergedProjects/instructions/instruct32_hh/vc201.htm (movsd)

https://packetstormsecurity.com/files/161349/Linux-x64-execve-cat-etc-shadow-Shellcode.html (writing execve('bin/cat'))
