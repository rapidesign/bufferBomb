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

I set a breakpoint at test+12 to examine the register state so I could see what is being passed from %eax to -0xC(%ebp) via the instruction mov. In this instance the value being mov'd from %eax to the offset -0xC(%ebp) is 0x6cb0949a. However, with each execution of the program this value is different. But the relative offset of where it is stored is always the same.

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

currentAddress%ebp - 0xC = addressOfOffsetOnStack
    0x55683380     - 0xC = 0x55683374
