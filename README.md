# WHAT IS THIS PAGE

I love when things are shortly explained. Reverse engineering and assembly code can be difficult to learn. 

It is just a reminder of some analysis stuff. It has not an educational purpose. 

I will use **x86 instruction** **set** during my analysis. Compilation is made with mingw-gcc (i686-w64-mingw32-gcc).

Notice that some of the examples are taken from the book made by "Dennis Yuriken" ( Reverse Engineering for beginner/Understand assembly langage).

# What is the point of reverse

- Discover how the program works
- Locate hidden functionality
- Search for vulnerabilities

# Step of analysis

- Trace the code to understand how it works,
- Observe/Understand function call tree,
- Understand data type

One of the best way to determine data types is to look for calls to know functions (C standard library / API calls). Observe the parameters and name them accordingly

# Heap and Stack

<u>The "stack"</u> :

- It is a block of memory,
- it is located in the space of a processus,
- It is used by the ESP register which is used as a pointer in this memory space.
- The instruction of access in the stack are PUSH and POP (sometimes move is used instead of PUSH)

<u>Why the "stack"</u> ?

1. <u>Save the **return address of the function**</u>,

![image-20220601165715844](/home/ad/.config/Typora/typora-user-images/image-20220601165715844.png)![image-20220601172847652](/home/ad/.config/Typora/typora-user-images/image-20220601172847652.png)



in this case, when we enter into the "stack" function, the address of the "main" function is saved into the stack :

![image-20220601173032523](/home/ad/.config/Typora/typora-user-images/image-20220601173032523.png)

The address of the "main" function will be called at the "retn".

2) <u>To **add arguments of a function**</u> :

![image-20220601173603035](/home/ad/.config/Typora/typora-user-images/image-20220601173603035.png)![image-20220601173625000](/home/ad/.config/Typora/typora-user-images/image-20220601173625000.png)

The number 5 and 10 are mov to the stack (esp and esp+4) before the call of the "stack function". We can see it in dynamic analysis :

![image-20220601173808398](/home/ad/.config/Typora/typora-user-images/image-20220601173808398.png)

The PUSH instruction is often used to push the arguments into the stack.

3) This is a good representation of the stack made by "Varonis"  (https://www.varonis.com/blog/stack-memory-3) :

![image-20220601165351164](/home/ad/.config/Typora/typora-user-images/image-20220601165351164.png)

4) It is important to notice that the first arguments is push at the last position in the stack (first in last out) :

   ![image-20220601174628996](/home/ad/.config/Typora/typora-user-images/image-20220601174628996.png)

   ![image-20220601174618453](/home/ad/.config/Typora/typora-user-images/image-20220601174618453.png)

   

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

# A simple scanf

![image-20220601175434646](/home/ad/.config/Typora/typora-user-images/image-20220601175434646.png)![image-20220601180251361](/home/ad/.config/Typora/typora-user-images/image-20220601180251361.png)

The "scanf" has two arguments :

- %d = pointer to the "string"containing %d,
- &x = Address of the variable x.

1) with lea we "create" and address (in the stack at esp+1C) that we store in eax,

2) We save this address in esp+4 :

   ![image-20220601181351346](/home/ad/.config/Typora/typora-user-images/image-20220601181351346.png)

3) We push "%d" into the stack,

4) mov eax, [esp+1C] : we put the value located in esp+1C (view the lea instruction) in eax. We put this value in esp+4 (second argument),

5) We push the "string" value "You entered.." into the top of the stack,

6) We call the printf function which will use the two arguments ("You entered..." + The scanf result).

# GLOBAL VARIABLE

![image-20220602090424332](/home/ad/.config/Typora/typora-user-images/image-20220602090424332.png)![image-20220602090531391](/home/ad/.config/Typora/typora-user-images/image-20220602090531391.png)![image-20220602090445880](/home/ad/.config/Typora/typora-user-images/image-20220602090445880.png)

In this case, the variable x is defined into the _DATA section and there is no memory allocated into the stack.

# Access to the arguments

![image-20220602102015750](/home/ad/.config/Typora/typora-user-images/image-20220602102015750.png)

![image-20220602101535459](/home/ad/.config/Typora/typora-user-images/image-20220602101535459.png)

Arguments are put into the stack (first in last out). After that we call the _f function :

![image-20220602101943083](/home/ad/.config/Typora/typora-user-images/image-20220602101943083.png)

<u>Just a quick word about the arguments of the main function</u> :

![image-20220602103110589](/home/ad/.config/Typora/typora-user-images/image-20220602103110589.png)

- argc = Number of arguments,
- argv = List of arguments,
- envp = List of environment variables.

# WHAT HAPPEN TO THE STRUCT 

![image-20220602103804965](/home/ad/.config/Typora/typora-user-images/image-20220602103804965.png)

![image-20220602135701370](/home/ad/.config/Typora/typora-user-images/image-20220602135701370.png)

<u>'aMyNumberD' value</u> :

![image-20220602104909831](/home/ad/.config/Typora/typora-user-images/image-20220602104909831.png)



<u>MOVZX</u> : Move a register with an inferior size into a bigger one. It fill the gap with 0.



# Array

![image-20220602142041169](/home/ad/.config/Typora/typora-user-images/image-20220602142041169.png)

![image-20220602175249350](/home/ad/.config/Typora/typora-user-images/image-20220602175249350.png)

mov [esp+eax*4+1Ch], edx : Here there is a construction of an array into the stack.

each time eax is incremented the data contain in edx will be put at the offset of esp+1Ch (base of the stack array) + 4 bytes (because EAX increase by 1 each times in the loop).

![image-20220602175914064](/home/ad/.config/Typora/typora-user-images/image-20220602175914064.png)

<u>Example of jump</u> :

![image-20220602142505398](/home/ad/.config/Typora/typora-user-images/image-20220602142505398.png)



# Pre multiplication of a register

One register can be pre-multiplied by 2,4 or 8 (word align, double align, quadword-align).

ex : mov	[esp+eax*4+1Ch]



# Les syscall

<u>What is a syscall ?</u> 

It is the link between the usermod and the kernel  mode.

<img src="/home/ad/.config/Typora/typora-user-images/image-20220602120009505.png" alt="image-20220602120009505" style="zoom:33%;" />



<u>A little reminder about the different rings</u> :

x86 CPU hardware provides protections rings. Rings 0 (kernel) and 3 (user) are the most used.

<img src="/home/ad/.config/Typora/typora-user-images/image-20220602121215715.png" alt="image-20220602121215715" style="zoom:50%;" />

- Ring ‑3: Management Engine (ME) {Highest Privilege}
- Ring ‑2: System Management Mode (SMM)
- Ring ‑1: Hypervisor
- Ring 0: Kernel
- Ring 1: Device Drivers
- Ring 2: Device Drivers
- Ring 3: User Applications {Lowest Privilege}

Read this great paper about rings protection : https://medium.com/swlh/negative-rings-in-intel-architecture-the-security-threats-youve-probably-never-heard-of-d725a4b6f831.

<u>How to make a syscall</u> :

- Linux : 0x80.

<u>General usage of syscall</u> :

- Shellcode ( allow for smallest possible shellcode),
- Statically linked code : make code independent of installed librairies.

<u>Usefull linux syscall</u> :

![image-20220602114639229](/home/ad/.config/Typora/typora-user-images/image-20220602114639229.png)