# Introduction
This is the **5th** level in the Microcorruption series.
This is the first level that requires knowledge of buffer overflows.
It's the first level that actually requires to look at the memory dump.
# Vulnerability Title
Buffer overflow to override the return address.
# Walkthrough
The `main` function just calls the `login` function:
```asm
4438 <main>
4438:  b012 0045      call	#0x4500 <login>
```
So, let's take a look at `login`:
```asm
4500 <login>
4500:  3150 f0ff      add	#0xfff0, sp
4504:  3f40 7c44      mov	#0x447c "Enter the password to continue.", r15
4508:  b012 a645      call	#0x45a6 <puts>
450c:  3f40 9c44      mov	#0x449c "Remember: passwords are between 8 and 16 characters.", r15
4510:  b012 a645      call	#0x45a6 <puts>
4514:  3e40 3000      mov	#0x30, r14
4518:  0f41           mov	sp, r15
451a:  b012 9645      call	#0x4596 <getsn>
451e:  0f41           mov	sp, r15
4520:  b012 5244      call	#0x4452 <test_password_valid>
4524:  0f93           tst	r15
4526:  0524           jz	$+0xc <login+0x32>
4528:  b012 4644      call	#0x4446 <unlock_door>
452c:  3f40 d144      mov	#0x44d1 "Access granted.", r15
4530:  023c           jmp	$+0x6 <login+0x36>
4532:  3f40 e144      mov	#0x44e1 "That password is not correct.", r15
4536:  b012 a645      call	#0x45a6 <puts>
453a:  3150 1000      add	#0x10, sp
453e:  3041           ret
```
This function follows a very clear progression.
First, it prompts the user for a password:
```asm
4504:  3f40 7c44      mov	#0x447c "Enter the password to continue.", r15
4508:  b012 a645      call	#0x45a6 <puts>
450c:  3f40 9c44      mov	#0x449c "Remember: passwords are between 8 and 16 characters.", r15
4510:  b012 a645      call	#0x45a6 <puts>
```
Second, it reads in the password (`0x30` bytes into the stack):
```asm
4514:  3e40 3000      mov	#0x30, r14
4518:  0f41           mov	sp, r15
451a:  b012 9645      call	#0x4596 <getsn>
```
Third, it checks the password:
```asm
451e:  0f41           mov	sp, r15
4520:  b012 5244      call	#0x4452 <test_password_valid>
4524:  0f93           tst	r15
4526:  0524           jz	$+0xc <login+0x32>
```
Now, if the password is correct, it unlocks the door:
```asm
4528:  b012 4644      call	#0x4446 <unlock_door>
452c:  3f40 d144      mov	#0x44d1 "Access granted.", r15
4530:  023c           jmp	$+0x6 <login+0x36>
4536:  b012 a645      call	#0x45a6 <puts>
453a:  3150 1000      add	#0x10, sp
453e:  3041           ret
```
But, if the password is incorrect, this part gets executed:
```asm
4532:  3f40 e144      mov	#0x44e1 "That password is not correct.", r15
4536:  b012 a645      call	#0x45a6 <puts>
453a:  3150 1000      add	#0x10, sp
453e:  3041           ret
```
# Vulnerability
This level's vulnerability is in the `login` function.
Essentially, there is a possibility of a **buffer overflow** on the stack, **overriding the return address** at the end of `login`.
Looking at the **memory dump** of the program while it's executing, we can see that the input starts at address `0x43ee`, and that the return address is at address `0x43fe`.
This means that we need only 17 bytes to **override** the return address, and 18 bytes to **override** it **completely**, replacing it with our own.
Fortunately for us (and unfortunately for the developers), we have `0x30` bytes for the input.
# Exploit
Let's first find what to replace the current return address with.
Of course, the easiest one to replace with is the address of the `unlock_door` function.
The address of this function is `0x4446`.
To pad the 16 bytes leading to the address, we can use whatever bytes we want.
I'll go with `1`'s - so 32 `1`'s in total before the new return address: `11111111111111111111111111111111`.
Now, the return address has to be in **little endian**, so we'll flip it: `0x4644`.
And then put it together with the padding: `111111111111111111111111111111114644`.
Using this exploit, the `login` function **will** display to the user that the password is incorrect, **but** it will jump to `unlock_door` anyway at the end.
# Solution
Hex: `111111111111111111111111111111114644`