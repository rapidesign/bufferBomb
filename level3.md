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
   0x08048c93 <+20>:    mov    %eax,%ebx
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

At getbuf+17 the mov instruction places the value 1 in the register %eax. 

