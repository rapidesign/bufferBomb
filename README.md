Buffer Bomb
===========
5 phases where you exploit the getbuf() function in C and manipulate the outcome of the bufbomb executable file.

<h1>Level 0</h1>

Goal = provide a string longer than getbuf can handle causing an overflow and pushing the rest of the string onto the stack where you can then control where the function getbuf returns after exection.

Main Steps:

1. locate the address of the function ```Smoke()```
2. calculate the length of the string size to cause an overflow
3. create the buffer overflow string 
4. force the function getbuf to return to fucntion ```Smoke()``` instead of returning the value 1.
5. success!!!

1. To find the address of function ```Smoke()``` type:
   ```unix> objdump -d bufbomb | less```

   My address for ```Smoke()``` = 080490aa <smoke>:

