# WHAT IS THIS PAGE

I love when things are shortly explained. Reverse engineering and assembly code can be difficult to learn. That is why i create this page in order to explain all the little things that i would love to know before to begin in the path of RE.

I will use **x86 instruction** **set** during my analysis. 

# Heap and Stack

<u>The "stack"</u> :

- It is a block of memory,
- it is located in the space of a processus,
- It is used by the ESP register which is used as a pointer in this memory space.
- The instruction of access in the stack are PUSH and POP







# Prolog and epilog

<u>Each function you have a **prolog**</u> :

<u>ex</u> :

`push	ebp           (save EBP value in the stack)`

`mov	ebp, esp    (save the actual value of ESP in the register EBP)`

`sub	 esp, X		 (reserved memory in the stack for the local variables)`

<u>the **epilog**</u> :

<u>ex</u>: 

`mov esp, ebp (free the memory allocated in the stack)`

`pop ebp 	      (restore the old value of ebp)`

`ret 0				  (exit the function)`



# A simple print

![image](https://user-images.githubusercontent.com/89603689/171406852-a0a0926d-54f7-43fe-bd64-c81fc1d76600.png)![image](https://user-images.githubusercontent.com/89603689/171408225-1e374675-c626-4b6e-ba63-7c15b2ca34a3.png)

1 - Move the offset of "buffer" into the stack (at the offset of ESP).

![image](https://user-images.githubusercontent.com/89603689/171410256-d3546aac-a4c2-4e01-aedc-357215bdda76.png)

The offset of "buffer" is in the section ".rdata" at the adress 0x00404000.

- **db** = define byte (reservation of 1 byte in memory),
- dw = define word (reservation of 2 byte in memory),
- dd = define double word (reservation of 4 bytes in memory).

**align** is a directive which means : "if this address is not a multiple of the given number, skip the necessary number of bytes to fulfill this address requirement for the next data".

**db 'hello, world'** is a the address 0040400. It occupies only 13 bytes (404000 to 40400D) and fills it with the value 'hello, world'. "align" begin at the offset 40400D". This address is skiped because it doesn't fulfill the request "a multiple of 10h (last sigit !=0).
