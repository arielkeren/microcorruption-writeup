# Introduction
This is the **8th** level in the Microcorruption series.
It's almost exactly the same as the last level, just with a `strcpy` performed on the input password.
# Vulnerability Title
Buffer overflow to override the return address, and to provide an argument to the new function.
# Walkthrough
The `main` function just calls the `login` function:
```asm
4438 <main>
4438:  b012 f444      call	#0x44f4 <login>
```
And this is the `login` function:
```asm
44f4 <login>
44f4:  3150 f0ff      add	#0xfff0, sp
44f8:  3f40 7044      mov	#0x4470 "Enter the password to continue.", r15
44fc:  b012 b045      call	#0x45b0 <puts>
4500:  3f40 9044      mov	#0x4490 "Remember: passwords are between 8 and 16 characters.", r15
4504:  b012 b045      call	#0x45b0 <puts>
4508:  3e40 3000      mov	#0x30, r14
450c:  3f40 0024      mov	#0x2400, r15
4510:  b012 a045      call	#0x45a0 <getsn>
4514:  3e40 0024      mov	#0x2400, r14
4518:  0f41           mov	sp, r15
451a:  b012 dc45      call	#0x45dc <strcpy>
451e:  3d40 6400      mov	#0x64, r13
4522:  0e43           clr	r14
4524:  3f40 0024      mov	#0x2400, r15
4528:  b012 f045      call	#0x45f0 <memset>
452c:  0f41           mov	sp, r15
452e:  b012 4644      call	#0x4446 <conditional_unlock_door>
4532:  0f93           tst	r15
4534:  0324           jz	$+0x8 <login+0x48>
4536:  3f40 c544      mov	#0x44c5 "Access granted.", r15
453a:  023c           jmp	$+0x6 <login+0x4c>
453c:  3f40 d544      mov	#0x44d5 "That password is not correct.", r15
4540:  b012 b045      call	#0x45b0 <puts>
4544:  3150 1000      add	#0x10, sp
4548:  3041           ret
```
First, it prompts the user to enter a password:
```asm
44f8:  3f40 7044      mov	#0x4470 "Enter the password to continue.", r15
44fc:  b012 b045      call	#0x45b0 <puts>
4500:  3f40 9044      mov	#0x4490 "Remember: passwords are between 8 and 16 characters.", r15
4504:  b012 b045      call	#0x45b0 <puts>
```
Second, it reads the password from the user (at most `0x30` bytes, into address `0x2400`):
```asm
4508:  3e40 3000      mov	#0x30, r14
450c:  3f40 0024      mov	#0x2400, r15
4510:  b012 a045      call	#0x45a0 <getsn>
```
Third, it copies the password from address `0x2400` onto the stack, using `strcpy`:
```asm
4514:  3e40 0024      mov	#0x2400, r14
4518:  0f41           mov	sp, r15
451a:  b012 dc45      call	#0x45dc <strcpy>
```
Fourth, it zeroes out the whole original input password, using `memset`:
```asm
451e:  3d40 6400      mov	#0x64, r13
4522:  0e43           clr	r14
4524:  3f40 0024      mov	#0x2400, r15
4528:  b012 f045      call	#0x45f0 <memset>
```
Fifth, it checks the password *and* unlocks the door if it's correct:
```asm
452c:  0f41           mov	sp, r15
452e:  b012 4644      call	#0x4446 <conditional_unlock_door>
4532:  0f93           tst	r15
4534:  0324           jz	$+0x8 <login+0x48>
```
Finally, it prints the decision to the user:
```asm
4536:  3f40 c544      mov	#0x44c5 "Access granted.", r15
453a:  023c           jmp	$+0x6 <login+0x4c>
453c:  3f40 d544      mov	#0x44d5 "That password is not correct.", r15
4540:  b012 b045      call	#0x45b0 <puts>
```
# Vulnerability
The vulnerability is in the `login` function.
Essentially, it's possible to overflow the password input to override the return address of the function.
Specifically, the program is reading `0x30` bytes of data into address `0x2400`, and then copying it into the stack, while the return address is only `0x10` bytes away from the starting address of the input.
# Exploit
We can craft a payload that will override the return address, by starting with 16 bytes (for example, 32 `1`'s), and then putting the new return address:
`11111111111111111111111111111111` + `return address`
While we can't jump to `conditional_unlock_door`, because it checks the password before unlocking the door, we can directly jump to the `INT` function, and perform an interrupt that will open the door anyway.
Looking at the given disassembly, the `INT` function is located at address `0x454c`.
We can also see that the interrupt code argument is taken from the stack, exactly 2 bytes after the current stack pointer's location.
From the lock's manual, the interrupt code that unlocks the door is `0x7f`.
Combining all of this, we have:
32 x `1` + `return address` + 4 x `1` + `interrupt code`.
Substituting the return address (flipping it because of little endian) and interrupt code, we have:
`111111111111111111111111111111114c4511117f`
# Solution
Hex: `111111111111111111111111111111114c4511117f`