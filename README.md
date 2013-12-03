Buffer Bomb
===========
5 phases where you exploit the getbuf() function in C and manipulate the outcome of the bufbomb executable file.

<h1>Level 0</h1>

Goal = provide a string longer than getbuf can handle causing an overflow and pushing the rest of the string onto the stack where you can then control where the function getbuf returns after exection.

Main Steps:

1. locate the address of the function ```Smoke()```

To find the address of function ```Smoke()``` type:
   ```unix> objdump -d bufbomb | less```

   My address for ```Smoke()``` = 080490aa <smoke>:


2. calculate the length of the string size to cause an overflow
```
unix> gdb bufbomb
(gdb) break getbuf
(gdb) run -u quinnliu
(gdb) break *getbuf+17
(gdb) disas
(gdb) info registers
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
(gdb) info registers
eax            0x45789ae        72845742
ecx            0x45789ae        72845742
edx            0x3c1048 3936328
ebx            0x0      0
esp            0x55683318       0x55683318
ebp            0x55683350       0x55683350
esi            0x55686018       1432903704
edi            0xc60    3168
eip            0x8048c0a        0x8048c0a <getbuf+6>
eflags         0x216    [ PF AF IF ]
cs             0x23     35
ss             0x2b     43
ds             0x2b     43
es             0x2b     43
fs             0x0      0
gs             0x63     99
(gdb)
```

Remember that the goal is to push a string that goes one address beyond the location of %ebp. 

Write down %ebp address as you will be using it later. In my case:

%ebp = 0x55683350 

getBuf() will take 32 characters. To modify the return address, we will need the buffer to extend to the address space right after %ebp on the stack or (%ebp)+4. So 32 bytes will be allocated to the buf variable for getbuf(). We need to look at the stack to determine the rest of the buffer size.

Now quit out
```
(gdb) q
A debugging session is active.

        Inferior 1 [process 29088] will be killed.

Quit anyway? (y or n) y
```

Now create a test input file called "hex" and type a few commands
```
unix> perl -e 'print "A"x32 '> hex
unix> gdb bufbomb
(gdb) break getbuf
(gdb) break *getbuf+17
(gdb) run -u quinnliu < hex
(gdb) x/20x $esp
(gdb) disas
```

Gives you:

```
(gdb) x/20x $esp
0x55683318 <_reserved+1037080>: 0x55683330      0x00263b36      0x003c132c      0x55683328
0x55683328 <_reserved+1037096>: 0x41723de3      0x00000000      0x55683350      0x08048b48
0x55683338 <_reserved+1037112>: 0x00007399      0x5568338c      0x00000001      0x00000000
0x55683348 <_reserved+1037128>: 0x55683370      0x003c0ff4      0x55683380      0x08048c93
0x55683358 <_reserved+1037144>: 0x55683380      0x0027e610      0x55686018      0x00000c60
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

Next type these commands:
```
(gdb) continue
(gdb) disas # notice that we are now at <+17>
(gdb) x/20x $esp
```

Gives you:
```
(gdb) continue
Continuing.

Breakpoint 2, 0x08048c15 in getbuf ()
(gdb) disas
Dump of assembler code for function getbuf:
   0x08048c04 <+0>:     push   %ebp
   0x08048c05 <+1>:     mov    %esp,%ebp
   0x08048c07 <+3>:     sub    $0x38,%esp
   0x08048c0a <+6>:     lea    -0x28(%ebp),%eax
   0x08048c0d <+9>:     mov    %eax,(%esp)
   0x08048c10 <+12>:    call   0x8048b4a <Gets>
=> 0x08048c15 <+17>:    mov    $0x1,%eax
   0x08048c1a <+22>:    leave
   0x08048c1b <+23>:    ret
End of assembler dump.
(gdb) x/20x $esp
0x55683318 <_reserved+1037080>: 0x55683328      0x00263b36      0x003c132c      0x55683328
0x55683328 <_reserved+1037096>: 0x41414141      0x41414141      0x41414141      0x41414141
0x55683338 <_reserved+1037112>: 0x41414141      0x41414141      0x41414141      0x41414141
0x55683348 <_reserved+1037128>: 0x55683300      0x003c0ff4      0x55683380      0x08048c93
0x55683358 <_reserved+1037144>: 0x55683380      0x0027e610      0x55686018      0x00000c60
```

We can see that the "buf" variable stores its 32 byte string on the stack. Starting at address 0x55683328 and ending right before 0x55683348.

startingAddressOfBufVariable = 0x55683328
%ebp = 0x55683350 # I told you to write this down earlier

buffer size = addressAt%ebp+4 - startingAddressOfBufVariable
            = 0x55683350   +4 - 0x55683328
            # make sure to use a hex calculator like this one: http://www.squarebox.com/legacy/hcalc.html
            = 




3. create the buffer overflow string 


4. force the function getbuf to return to fucntion ```Smoke()``` instead of returning the value 1.


5. success!!!
