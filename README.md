# WHAT IS THIS PAGE

I love when things are shortly explained. Reverse engineering and assembly code can be difficult to learn. 

It is just a reminder of some analysis stuff. It has not an educational purpose. May be it can be usefull for some poeple, so i keep the repo public.

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

![image](https://user-images.githubusercontent.com/89603689/171674269-b6f4a002-704b-4d5f-96f0-a74350bf059b.png)




in this case, when we enter into the "stack" function, the address of the "main" function is saved into the stack :

![image](https://user-images.githubusercontent.com/89603689/171674319-67cd7e44-b128-4e82-9ec2-211f9162c4a9.png)


The address of the "main" function will be called at the "retn".

2) <u>To **add arguments of a function**</u> :

![image](https://user-images.githubusercontent.com/89603689/171674369-3a19ede6-3c5d-4df0-b626-e025b418c6d8.png)


The number 5 and 10 are mov to the stack (esp and esp+4) before the call of the "stack function". We can see it in dynamic analysis :

![image](https://user-images.githubusercontent.com/89603689/171674415-671ee6a3-5261-46a6-9ab3-5b4bc519cf75.png)


The PUSH instruction is often used to push the arguments into the stack.

3) This is a good representation of the stack made by "Varonis"  (https://www.varonis.com/blog/stack-memory-3) :

![image](https://user-images.githubusercontent.com/89603689/171674472-1532346e-ca17-4044-b19b-af62ae9686a8.png)


4) It is important to notice that the first arguments is push at the last position in the stack (first in last out) :

 ![image](https://user-images.githubusercontent.com/89603689/171674521-1c573b3c-9b26-42dd-9130-04aba9961526.png)


   

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

![image](https://user-images.githubusercontent.com/89603689/171674560-eded7340-db64-4617-a0ff-36846538e7f2.png)


1 - Move the offset of "buffer" into the stack (at the offset of ESP).

![image](https://user-images.githubusercontent.com/89603689/171674592-7843e9b4-04dd-4b22-bfac-1afca96bc091.png)


The offset of "buffer" is in the section ".rdata" at the adress 0x00404000.

- **db** = define byte (reservation of 1 byte in memory),
- dw = define word (reservation of 2 byte in memory),
- dd = define double word (reservation of 4 bytes in memory).

**align** is a directive which means : "if this address is not a multiple of the given number, skip the necessary number of bytes to fulfill this address requirement for the next data".

**db 'hello, world'** is a the address 0040400. It occupies only 13 bytes (404000 to 40400D) and fills it with the value 'hello, world'. "align" begin at the offset 40400D". This address is skiped because it doesn't fulfill the request "a multiple of 10h (last sigit !=0).

# A simple scanf

![image](https://user-images.githubusercontent.com/89603689/171674644-3bf51214-976b-41bb-b779-ed2d2a93819e.png)


The "scanf" has two arguments :

- %d = pointer to the "string"containing %d,
- &x = Address of the variable x.

1) with lea we "create" and address (in the stack at esp+1C) that we store in eax,

2) We save this address in esp+4 :

   ![image](https://user-images.githubusercontent.com/89603689/171674703-73070e6d-f88a-4100-9b3b-e7d9a95d076f.png)


3) We push "%d" into the stack,

4) mov eax, [esp+1C] : we put the value located in esp+1C (view the lea instruction) in eax. We put this value in esp+4 (second argument),

5) We push the "string" value "You entered.." into the top of the stack,

6) We call the printf function which will use the two arguments ("You entered..." + The scanf result).

# GLOBAL VARIABLE

![image](https://user-images.githubusercontent.com/89603689/171674753-561ed159-f9a5-4815-8362-d484aed308f4.png)


In this case, the variable x is defined into the _DATA section and there is no memory allocated into the stack.

# Access to the arguments

![image](https://user-images.githubusercontent.com/89603689/171674819-9876d669-1c0c-40df-ac96-97a1a50ba9f1.png)


Arguments are put into the stack (first in last out). After that we call the _f function :

![image](https://user-images.githubusercontent.com/89603689/171674870-8ec8369d-e319-4a45-bcec-0134734a5857.png)


<u>Just a quick word about the arguments of the main function</u> :

![image-20220602103110589](/home/ad/.config/Typora/typora-user-images/image-20220602103110589.png)

- argc = Number of arguments,
- argv = List of arguments,
- envp = List of environment variables.

# WHAT HAPPEN TO THE STRUCT 

![image](https://user-images.githubusercontent.com/89603689/171674950-eba54849-e2c3-4821-90ad-2a88921677e5.png)


<u>'aMyNumberD' value</u> :

![image](https://user-images.githubusercontent.com/89603689/171674988-f0009207-28bc-44d2-9bb0-6c3631568809.png)




<u>MOVZX</u> : Move a register with an inferior size into a bigger one. It fill the gap with 0.



# Array
![image](https://user-images.githubusercontent.com/89603689/171675043-8f7a3626-5ca6-4bb4-9d57-e4750668f9ab.png)


mov [esp+eax*4+1Ch], edx : Here there is a construction of an array into the stack.

each time eax is incremented the data contain in edx will be put at the offset of esp+1Ch (base of the stack array) + 4 bytes (because EAX increase by 1 each times in the loop).
![image](https://user-images.githubusercontent.com/89603689/171675199-3ef6364b-c436-4d41-86e2-b94b91fbefca.png)


<u>Example of jump</u> :

![image](https://user-images.githubusercontent.com/89603689/171675227-062c6913-f8b2-4287-858e-db4cd5678f6b.png)


# Array of pointers

![image](https://user-images.githubusercontent.com/89603689/171804363-50135aab-2571-4ab1-baa6-5bb2322c81a2.png)![image](https://user-images.githubusercontent.com/89603689/171804793-d5be3f9d-d8c2-4e9a-b5cf-a64751cdd7ab.png)
![image](https://user-images.githubusercontent.com/89603689/171804834-e025376d-3898-4a9b-835c-10650ff0c9f2.png)

![image](https://user-images.githubusercontent.com/89603689/171804908-9ae79e13-28a5-43c9-b10c-7b9371fbcbc6.png)


# Pre multiplication of a register

One register can be pre-multiplied by 2,4 or 8 (word align, double align, quadword-align).

ex : mov	[esp+eax*4+1Ch]



# Syscall

<u>What is a syscall ?</u> 

It is the link between the usermod and the kernel  mode.

![image](https://user-images.githubusercontent.com/89603689/171675261-f085bb88-c78d-4b26-b659-f387402bd78b.png)!






<u>A little reminder about the different rings</u> :

x86 CPU hardware provides protections rings. Rings 0 (kernel) and 3 (user) are the most used.

![image](https://user-images.githubusercontent.com/89603689/171675328-a08a48b3-50dc-48e4-aeca-9a85f3ef1131.png)


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

![image](https://user-images.githubusercontent.com/89603689/171675368-ac561629-32af-4a3a-9b31-6c4c29f42ffd.png)
