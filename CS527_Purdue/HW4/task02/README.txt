There is a canary but can be bypassed since the program allows arbitrary write. Since RELRO isn't enabled, I can potentially overwrite the GOT table (in this case my target is puts function in _echo). I first find the GOT address of puts (0x600f18), then the base address of libc by calculating the offset between its GOT pointed actual address (0x7fcb7c75f970 found using read on address 0x600f18) and its offset in libc (0x80970 found using libc.symbols.puts). I then use the base address of libc to find the location of system (libc.symbols.system = 0x7f90f85d4420), and finally overwrite the GOT-pointed address of puts with that of system. After inputting "/bin/sh" in "_echo" function, the system function will run instead.

Note that no code was written for this task since I did most of the work using the binary and python3 interface.

Reference: https://blog.pwntools.com/posts/got-overwrite/
