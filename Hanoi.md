# Introduction
This is the **4th** level in the Microcorruption series.
This is the first level that requires crafting a specific payload to pass.
It's a bit more involved, memory-wise, than the previous levels.
# Vulnerability Title
Password verification logic is easy to figure out.
# Walkthrough
The `main` function just calls the `login` function:
```asm
4438 <main>
4438:  b012 2045      call	#0x4520 <login>
443c:  0f43           clr	r15
```
So, let's take a look at `login`:
```asm
4520 <login>
4520:  c243 1024      mov.b	#0x0, &0x2410
4524:  3f40 7e44      mov	#0x447e "Enter the password to continue.", r15
4528:  b012 de45      call	#0x45de <puts>
452c:  3f40 9e44      mov	#0x449e "Remember: passwords are between 8 and 16 characters.", r15
4530:  b012 de45      call	#0x45de <puts>
4534:  3e40 1c00      mov	#0x1c, r14
4538:  3f40 0024      mov	#0x2400, r15
453c:  b012 ce45      call	#0x45ce <getsn>
4540:  3f40 0024      mov	#0x2400, r15
4544:  b012 5444      call	#0x4454 <test_password_valid>
4548:  0f93           tst	r15
454a:  0324           jz	$+0x8 <login+0x32>
454c:  f240 6a00 1024 mov.b	#0x6a, &0x2410
4552:  3f40 d344      mov	#0x44d3 "Testing if password is valid.", r15
4556:  b012 de45      call	#0x45de <puts>
455a:  f290 9d00 1024 cmp.b	#0x9d, &0x2410
4560:  0720           jnz	$+0x10 <login+0x50>
4562:  3f40 f144      mov	#0x44f1 "Access granted.", r15
4566:  b012 de45      call	#0x45de <puts>
456a:  b012 4844      call	#0x4448 <unlock_door>
456e:  3041           ret
4570:  3f40 0145      mov	#0x4501 "That password is not correct.", r15
4574:  b012 de45      call	#0x45de <puts>
4578:  3041           ret
```
We can see that it first prompts the user for a password:
```asm
4520:  c243 1024      mov.b	#0x0, &0x2410
4524:  3f40 7e44      mov	#0x447e "Enter the password to continue.", r15
4528:  b012 de45      call	#0x45de <puts>
452c:  3f40 9e44      mov	#0x449e "Remember: passwords are between 8 and 16 characters.", r15
4530:  b012 de45      call	#0x45de <puts>
```
Then, it reads the password as input from the user - at most `0x1c` bytes (as in `r14`) - into address `0x2400`:
```asm
4534:  3e40 1c00      mov	#0x1c, r14
4538:  3f40 0024      mov	#0x2400, r15
453c:  b012 ce45      call	#0x45ce <getsn>
```
Skipping forward a bit, there is a comparison of the byte at address `0x2410` with the literal `0x9d`:
```asm
455a:  f290 9d00 1024 cmp.b	#0x9d, &0x2410
4560:  0720           jnz	$+0x10 <login+0x50>
4562:  3f40 f144      mov	#0x44f1 "Access granted.", r15
4566:  b012 de45      call	#0x45de <puts>
456a:  b012 4844      call	#0x4448 <unlock_door>
456e:  3041           ret
```
If they're equal, then program execution continues, eventually calling `unlock_door`.
What is at address `0x2410`, though? Well, it's the 17th byte of the input.
Why? Because `getsn` got the address `0x2400`, which is the starting address of the input.
# Vulnerability
This level's vulnerability is in the `login` function.
Essentially, the whole password verification here is that the 17th byte of the input has to be `0x9d`.
# Exploit
We need to craft a payload where the 17th byte is `0x9d`.
So, we can just put 16 random bytes (that don't really matter), and then the `0x9d` byte.
For example, each random byte can be `0x11`.
Then, the 16 bytes are: `11111111111111111111111111111111` (32 `1`'s).
Appending the 17th `0x9d` byte results in the following input: `111111111111111111111111111111119d`.
# Solution
Hex: `111111111111111111111111111111119d`