+++
title = "ls clone using getdents syscall"
date = "2021-11-15T18:20:39+05:30"
author = ""
authorTwitter = "" #do not include @
cover = ""
description = "This post focuses on building a small,minimal directory listing utility with the getdents syscall in x86asm"
showFullContent = false
readingTime = false
+++


# Introduction

In this short post, we’re building a barebones,toy implementation of the `ls`-like directory listing program in x86-64 assembly. We achieve this by using the `getdents` syscall.

# getdents() syscall

The `getdents` syscall takes in 3 parameters. So the prototype of the syscall looks like: `ssize_t getdents(int fd, void *dirp, size_t count)`, where:

1. `fd` : is the file descriptor of the directory (which is also just another file in Unix)
2. `dirp` : is where all the `dirent` structs are copied during the syscall.
3. `count` : is the size of the buffer that `dirp` points to.

# Dirent structs
A *dirent* or *linux_dirent* struct is defined as follows.

```c++
struct linux_dirent {
       unsigned long  d_ino;     /* File inode*/
       unsigned long  d_off;     /* Offset of the next linux_dirent */
       unsigned short d_reclen;  /* Size of the current dirent struct*/
       char           d_name[];  // The filename of the file

       char           pad;       // Zero padding byte
       char           d_type;    // File types such as block devices, named pipes, sockets etc.
   }

```

So anytime we call the `getdents` syscall, we are populating with `dirent` structures, the buffer that `dirp` points to. Notice how the struct also involves the filename of the file that the struct is for. We can use this for directory listing!

# The program
The program that we are building lists the directory contents of the current directory. We allocate a 512 bytes buffer for storing the dirent structs. (Which is sufficient for a toy implementation)

Based on the info above, we define the `.bss` and the `.data` sections as follows:

```x86asm
[SECTION .data]
dirname: db '.'

[SECTION .bss]
msgbuf:
	resb 512
```

Then in the `.text` section, we write the actual program, which consists of 2 functions called `main` and `next_dirent`. We populate the `msgbuf` in the `main` function and then process the dirent structs in the `next_dirent` function, which handles the buffer struct by struct.

The `main` function goes like:

```x86asm
[SECTION .text]
extern puts
global main

main:
	;Setup
	push rbp
	mov rbp, rsp
	and rsp, 0xfffffffffffffff0

	;Open the current directory using the open() syscall
	mov rax, 2
	mov rdi, dirname
	mov rsi, 0x10000
	syscall

	;getdents(fd, buf, 64) syscall
	mov rdi, rax  ;The open syscall returns the fd for the current directory
	mov rax,78
	mov rsi, msgbuf
	mov rdx, 512
	syscall
	xchg rax,rdx  ;Syscall returns total size, save it somewhere safe

	;setup registers for loop
	xor rcx, rcx
	xor rbx, rbx
```

Now that we have the structs we need in the `msgbuf`, we can iterate over them. Heading into the `next_dirent` function:

```x86asm
next_dirent:
	;We move to the 0th position of msgbuf every loop
	;and absolute position of every struct from it
	mov rax, msgbuf
	add rax, rcx		

	;Grab the length of current struct
	;i.e reclen with offset adjustments
	add rax, 0x10
	movzx bx, byte [rax]

	;puts modifies state of registers so save
	;them on the stack
	push rcx
	push rdx

	;Grab filename from its offset and print it
	;to stdout
	add rax, 0x2
	mov rdi, rax
	call puts

	;Restore register states
	pop rdx				
	pop rcx

	;Move to next struct if
	;index(curr_struct) < total structs
	add cx, bx			
	cmp cx, dx			
	jl next_dirent

	;Leave
	mov rsp, rbp
	pop rbp
	ret
```

Offset calculation is:

   -  The struct size sits at `struct_index + 8*2 = 0x10` bytes. 2 `unsigned long`s are considered for.
   -  The filename sits at `struct_index + 8*2 + 2 = 0x12`bytes. 1 `unsigned short` is considered for. Note that we just add `0x2` to the struct size offset when outputting the filename.

# Results

```bash
>=> ls
file1.txt  file2.txt  main
>=> ./main
.
file2.txt
main
..
file1.txt
```

Thus we get a barebones directory listing.

The compile script and code is available : [Here](https://github.com/CrystalSage/Direntries)

# P.S

I originally intended to write some malware in assembly with this, but got sidetracked and this was made ;)


