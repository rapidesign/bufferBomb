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
1) at 8049065 the value at %esp+8 is being stored into %eax by the mov instruction.
2) the value stored in %eax is compared to the value at address 0x804908e.
   
0x804908e is where the cookie for id "quinnliu" has been stored.

When the comp instruction executes, cmp will set an EFLAG based on the results of the compare.

3) the jne(jump if not equal) instruction jumps to 0x804908e depending the the state of the
EFlag. When the cmp instruction compares 2 values that are equal then the ZF flag = 1 within EFLAGS. 
and the jump will not occur. 

More specific goal = push the address of fizz onto the stack, place the cookie produced by
the id "quinnliu" at the location 0x8(%ebp) 





```
unix> gdb bufbomb
(gdb) break fizz
(gdb) run -u quinnliu
(gdb) disas
```