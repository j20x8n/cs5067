java cLab 3: The Attack Lab:
Understanding Buffer Overffow Bugs
1 Introduction
This assignment involves generating a total of ffve attacks on two programs having different security vulnerabilities.
 Outcomes you will gain from this lab include:
• You will learn different ways that attackers can exploit security vulnerabilities when programs do not
safeguard themselves well enough against buffer overffows.
• Through this, you will get a better understanding of how to write programs that are more secure, as
well as some of the features provided by compilers and operating systems to make programs less
vulnerable.
• You will gain a deeper understanding of the stack and parameter-passing mechanisms of x86-64
machine code.
• You will gain a deeper understanding of how x86-64 instructions are encoded.
• You will gain more experience with debugging tools such as GDB and OBJDUMP.
Note: In this lab, you will gain ffrsthand experience with methods used to exploit security weaknesses in
operating systems and network servers. Our purpose is to help you learn about the runtime operation of
programs and to understand the nature of these security weaknesses so that you can avoid them when you
write system code. We do not condone the use of any other form of attack to gain unauthorized access to
any system resources.
2 Logistics
As usual, this is an individual project. You will generate attacks for target programs that are custom generated
 for you.
2.1 Getting Files
You just need to switch ”lab3” branch
1git checkout lab3
The ffles will include:
README.txt: A ffle describing the contents of the directory
ctarget: An executable program vulnerable to code-injection attacks
rtarget: An executable program vulnerable to return-oriented-programming attacks
cookie.txt: An 8-digit hex code that you will use as a unique identiffer in your attacks.
farm.c: The source code of your target’s “gadget farm,” which you will use in generating return-oriented
programming attacks.
hex2raw: A utility to generate attack strings.
printf.so: A self-made ffle that adds an abstraction layer, providing a adaptive printf interface, to avoid
SIGSEGV.
In the following instructions, we will assume that you have copied the ffles to a protected local directory,
and that you are executing the programs in that local directory.
And the scoreboard is available at:
http://202.120.40.8:10623/scoreboard
2.2 Important Points
Here is a summary of some important rules regarding valid solutions for this lab. These points will not make
much sense when you read this document for the ffrst time. They are presented here as a central reference
of rules once you get started.
When you meet ”SIGSEGV” or ”Segmentation fault” in this lab, it means your glibc version is not matched
with the one we used to generate the target. So please use the provided ”printf.so” to avoid this problem.
Like:
LD_PRELOAD=./printf.so ./ctarget
• You must do the assignment on a machine that is similar to the one that generated your targets.
• Your solutions may not use attacks to circumvent the validation code in the programs. Speciffcally,
any address you incorporate into an attack string for use by a ret instruction should be to one of the
following destinations:
– The addresses for functions touch1, touch2, or touch3.
– The address of your injected code
– The address of one of your gadgets from the gadget farm.
• You may only construct gadgets from ffle rtarget with addresses ranging between those for functions
start_farm and end_farm.
23 Target Programs
Both CTARGET and RTARGET read strings from standard input. They do so with the function getbuf
deffned below:
1 unsigned getbuf()
2 {
3 char buf[BUFFER_SIZE];
4 Gets(buf);
5 return 1;
6 }
The function Gets is similar to the standard library function gets—it reads a string from standard input
(terminated by ‘\n’ or end-of-ffle) and stores it (along with a null terminator) at the speciffed destination.
In this code, you can see that the destination is an array buf, declared as having BUFFER_SIZE bytes. At
the time your targets were generated, BUFFER_SIZE was a compile-time constant speciffc to your version
of the programs.
Functions Gets() and gets() have no way to determine whether their destination buffers are large
enough to store the string they read. They simply copy sequences of bytes, possibly overrunning the bounds
of the storage allocated at the destinations.
If the string typed by the user and read by getbuf is sufffciently short, it is clear that getbuf will return
1, as shown by the following execution examples:
unix> ./ctarget
Cookie: 0x1a7dd803
Type string: Keep it short!
No exploit. Getbuf returned 0x1
Normal return
Typically an error occurs if you type a long string:
unix> ./ctarget
Cookie: 0x1a7dd803
Type string: This is not a very interesting string, but it has the property ...
Ouch!: You caused a segmentation fault!
Better luck next time
(Note that the value of the cookie shown will differ from yours.) Program RTARGET will have the same
behavior. As the error message indicates, overrunning the buffer typically causes the program state to be
corrupted, leading to a memory access error. Your task is to be more clever with the strings you feed
CTARGET and RTARGET so that they do more interesting things. These are called exploit strings.
Both CTARGET and RTARGET take several different command line arguments:
-h: Print list of possible command line arguments
 3-q: Don’t send results to the grading server
-i FILE: Supply input from a ffle, rather than from standard input
Your exploit strings will typically contain byte values that do not correspond to the ASCII values for printing
characters. The program HEX2RAW will enable you to generate these raw strings. See Appendix A for more
information on how to use HEX2RAW.
Important points:
• Your exploit string must not contain byte value 0x0a at any intermediate position, since this is the
ASCII code for newline (‘\n’). When Gets encounters this byte, it will assume you intended to
terminate the string.
• HEX2RAW expects two-digit hex values separated by one or more white spaces. So if you want to
create a byte with a hex value of 0, you need to write it as 00. To create the word 0xdeadbeef
you should pass “ef be ad de” to HEX2RAW (note the reversal required for little-endian byte
ordering).
When you have correctly solved one of the levels, your target program will automatically send a notiffcation
to the grading server. For example:
unix> ./hex2raw :
400f15: c7 07 d4 48 89 c7 movl $0xc78948d4,(%rdi)
400f1b: c3 retq
The byte sequence 48 89 c7 encodes the instruction movq %rax, %rdi. (See Figure 3A for the
encodings of useful movq instructions.) This sequence is followed by byte value c3, which encodes the
ret instruction. The function starts at address 0x400f15, and the sequence starts on the fourth byte of
the function. Thus, this code contains a gadget, having a starting address of 0x400f18, that will copy the
64-bit value in register %rax to register %rdi.
Your code for RTARGET contains a number of functions similar to the setval_210 function shown above
in a region we refer to as the gadget farm. Your job will be to identify useful gadgets in the gadget farm and
use these to perform attacks similar to those you did in Phases 2 and 3.
Important: The gadget farm is demarcated by functions start_farm and end_farm in your copy of
rtarget. Do not attempt to construct gadgets from other portions of the program code.
5.1 Level 2
For Phase 4, you will repeat the attack of Phase 2, but do so on program RTARGET using gadgets from your
gadget farm. You can construct your solution using gadgets consisting of the following instruction types,
and using only the first eight x86-64 registers (%rax–%rdi).
movq : The codes for these are shown in Figure 3A.
popq : The codes f代 写program、C/C++
代做程序编程语言or these are shown in Figure 3B.
ret : This instruction is encoded by the single byte 0xc3.
nop : This instruction (pronounced “no op,” which is short for “no operation”) is encoded by the single
byte 0x90. Its only effect is to cause the program counter to be incremented by 1.
9A. Encodings of movq instructions
movq S, D
Source Destination D
S %rax %rcx %rdx %rbx %rsp %rbp %rsi %rdi
%rax 48 89 c0 48 89 c1 48 89 c2 48 89 c3 48 89 c4 48 89 c5 48 89 c6 48 89 c7
%rcx 48 89 c8 48 89 c9 48 89 ca 48 89 cb 48 89 cc 48 89 cd 48 89 ce 48 89 cf
%rdx 48 89 d0 48 89 d1 48 89 d2 48 89 d3 48 89 d4 48 89 d5 48 89 d6 48 89 d7
%rbx 48 89 d8 48 89 d9 48 89 da 48 89 db 48 89 dc 48 89 dd 48 89 de 48 89 df
%rsp 48 89 e0 48 89 e1 48 89 e2 48 89 e3 48 89 e4 48 89 e5 48 89 e6 48 89 e7
%rbp 48 89 e8 48 89 e9 48 89 ea 48 89 eb 48 89 ec 48 89 ed 48 89 ee 48 89 ef
%rsi 48 89 f0 48 89 f1 48 89 f2 48 89 f3 48 89 f4 48 89 f5 48 89 f6 48 89 f7
%rdi 48 89 f8 48 89 f9 48 89 fa 48 89 fb 48 89 fc 48 89 fd 48 89 fe 48 89 ff
B. Encodings of popq instructions
Operation Register R
%rax %rcx %rdx %rbx %rsp %rbp %rsi %rdi
popq R 58 59 5a 5b 5c 5d 5e 5f
C. Encodings of movl instructions
movl S, D
Source Destination D
S %eax %ecx %edx %ebx %esp %ebp %esi %edi
%eax 89 c0 89 c1 89 c2 89 c3 89 c4 89 c5 89 c6 89 c7
%ecx 89 c8 89 c9 89 ca 89 cb 89 cc 89 cd 89 ce 89 cf
%edx 89 d0 89 d1 89 d2 89 d3 89 d4 89 d5 89 d6 89 d7
%ebx 89 d8 89 d9 89 da 89 db 89 dc 89 dd 89 de 89 df
%esp 89 e0 89 e1 89 e2 89 e3 89 e4 89 e5 89 e6 89 e7
%ebp 89 e8 89 e9 89 ea 89 eb 89 ec 89 ed 89 ee 89 ef
%esi 89 f0 89 f1 89 f2 89 f3 89 f4 89 f5 89 f6 89 f7
%edi 89 f8 89 f9 89 fa 89 fb 89 fc 89 fd 89 fe 89 ff
D. Encodings of 2-byte functional nop instructions
Operation Register R
%al %cl %dl %bl
andb R, R 20 c0 20 c9 20 d2 20 db
orb R, R 08 c0 08 c9 08 d2 08 db
cmpb R, R 38 c0 38 c9 38 d2 38 db
testb R, R 84 c0 84 c9 84 d2 84 db
Figure 3: Byte encodings of instructions. All values are shown in hexadecimal.
10Some Advice:
• All the gadgets you need can be found in the region of the code for rtarget demarcated by the
functions start_farm and mid_farm.
• You can do this attack with just two gadgets.
• When a gadget uses a popq instruction, it will pop data from the stack. As a result, your exploit
string will contain a combination of gadget addresses and data.
5.2 Level 3
Before you take on the Phase 5, pause to consider what you have accomplished so far. In Phases 2 and 3,
you caused a program to execute machine code of your own design. If CTARGET had been a network server,
you could have injected your own code into a distant machine. In Phase 4, you circumvented two of the
main devices modern systems use to thwart buffer overflow attacks. Although you did not inject your own
code, you were able inject a type of program that operates by stitching together sequences of existing code.
You have also gotten 95/100 points for the lab. That’s a good score. If you have other pressing obligations
consider stopping right now.
Phase 5 requires you to do an ROP attack on RTARGET to invoke function touch3 with a pointer to a string
representation of your cookie. That may not seem significantly more difficult than using an ROP attack to
invoke touch2, except that we have made it so. Moreover, Phase 5 counts for only 5 points, which is not a
true measure of the effort it will require. Think of it as more an extra credit problem for those who want to
go beyond the normal expectations for the course.
To solve Phase 5, you can use gadgets in the region of the code in rtarget demarcated by functions
start_farm and end_farm. In addition to the gadgets used in Phase 4, this expanded farm includes
the encodings of different movl instructions, as shown in Figure 3C. The byte sequences in this part of the
farm also contain 2-byte instructions that serve as functional nops, i.e., they do not change any register or
memory values. These include instructions, shown in Figure 3D, such as andb %al,%al, that operate on
the low-order bytes of some of the registers but do not change their values.
Some Advice:
• You’ll want to review the effect a movl instruction has on the upper 4 bytes of a register, as is
described on page 183 of the text.
• The official solution requires eight gadgets (not all of which are unique).
Good luck and have fun!
A Using HEX2RAW
HEX2RAW takes as input a hex-formatted string. In this format, each byte value is represented by two hex
digits. For example, the string “012345” could be entered in hex format as “30 31 32 33 34 35
1100.” (Recall that the ASCII code for decimal digit x is 0x3x, and that the end of a string is indicated by a
null byte.)
The hex characters you pass to HEX2RAW should be separated by whitespace (blanks or newlines). We
recommend separating different parts of your exploit string with newlines while you’re working on it.
HEX2RAW supports C-style block comments, so you can mark off sections of your exploit string. For
example:
48 c7 c1 f0 11 40 00 /* mov $0x40011f0,%rcx */
Be sure to leave space around both the starting and ending comment strings (“/*”, “*/”), so that the
comments will be properly ignored.
If you generate a hex-formatted exploit string in the file exploit.txt, you can apply the raw string to
CTARGET or RTARGET in several different ways:
1. You can set up a series of pipes to pass the string through HEX2RAW.
unix> cat exploit.txt | ./hex2raw | ./ctarget
2. You can store the raw string in a file and use I/O redirection:
unix> ./hex2raw  exploit-raw.txt
unix> ./ctarget  gdb ctarget
(gdb) run  ./hex2raw  exploit-raw.txt
unix> ./ctarget -i exploit-raw.txt
This approach also can be used when running from within GDB.
B Generating Byte Codes
Using GCC as an assembler and OBJDUMP as a disassembler makes it convenient to generate the byte codes
for instruction sequences. For example, suppose you write a file example.s containing the following
assembly code:
# Example of hand-generated assembly code
pushq $0xabcdef # Push value onto stack
addq $17,%rax # Add 17 to %rax
movl %eax,%edx # Copy lower 32 bits to %edx
12The code can contain a mixture of instructions and data. Anything to the right of a ‘#’ character is a
comment.
You can now assemble and disassemble this file:
unix> gcc -c example.s
unix> objdump -d example.o > example.d
The generated file example.d contains the following:
example.o: file format elf64-x86-64
Disassembly of section .text:
0000000000000000 :
0: 68 ef cd ab 00 push $0xabcdef
5: 48 83 c0 11 add $0x11,%rax
9: 89 c2 mov %eax,%edx
The lines at the bottom show the machine code generated from the assembly language instructions. Each
line has a hexadecimal number on the left indicating the instruction’s starting address (starting with 0), while
the hex digits after the ‘:’ character indicate the byte codes for the instruction. Thus, we can see that the
instruction push $0xABCDEF has hex-formatted byte code 68 ef cd ab 00.
From this file, you can get the byte sequence for the code:
68 ef cd ab 00 48 83 c0 11 89 c2
This string can then be passed through HEX2RAW to generate an input string for the target programs.. Alternatively,
you can edit example.d to omit extraneous values and to contain C-style comments for readability,
yielding:
68 ef cd ab 00 /* pushq $0xabcdef */
48 83 c0 11 /* add $0x11,%rax */
89 c2 /* mov %eax,%edx */
This is also a valid input you can pass through HEX2RAW before sending to one of the target programs.
References
[1] R. Roemer, E. Buchanan, H. Shacham, and S. Savage. Return-oriented programming: Systems, languages,
and applications. ACM Transactions on Information System Security, 15(1):2:1–2:34, March
2012.
13[2] E. J. Schwartz, T. Avgerinos, and D. Brumley. Q: Exploit hardening made easy. In USENIX Security
Symposium, 2011.
14

         
加QQ：99515681  WX：codinghelp  Email: 99515681@qq.com
