# Introduction
This is the first level in the Microcorruption series, so it's pretty easy to solve.
# Vulnerability
The vulnerability exists in the `check_password` function.
Essentially, the password-checking logic is very easy to understand and **break**.
Let's take a look at the function:
```asm
4484 <check_password>
4484:  6e4f           mov.b	@r15, r14
4486:  1f53           inc	r15
4488:  1c53           inc	r12
448a:  0e93           tst	r14
448c:  fb23           jnz	$-0x8 <check_password+0x0>
448e:  3c90 0900      cmp	#0x9, r12
4492:  0224           jz	$+0x6 <check_password+0x14>
4494:  0f43           clr	r15
4496:  3041           ret
4498:  1f43           mov	#0x1, r15
449a:  3041           ret
```
This part is just a loop going over each character in the input, and **incrementing** `r12` every iteration:
```asm
4484:  6e4f           mov.b	@r15, r14
4486:  1f53           inc	r15
4488:  1c53           inc	r12
448a:  0e93           tst	r14
448c:  fb23           jnz	$-0x8 <check_password+0x0>
```
**Notice** that this loop runs one more time than the total number of bytes in the input.
After the loop ends, there is a comparison between `r12` and `0x9`:
```asm
448e:  3c90 0900      cmp	#0x9, r12
4492:  0224           jz	$+0x6 <check_password+0x14>
```
If they're **equal**, execution jumps to this part:
```asm
4498:  1f43           mov	#0x1, r15
449a:  3041           ret
```
Otherwise, if they're **not** equal, execution continues to this part:
```asm
4494:  0f43           clr	r15
4496:  3041           ret
```
So, in order to get `r15` to be `1` (or `true`), we want the loop to increment `r12` exactly `9` times.
Remembering that `r12` gets incremented every iteration, and that there is always one more iteration than the amount of bytes in the input, we want to input $9-1=8$ bytes.
# Exploit
We want `r15` to be `1`, so that this comparison:
```asm
4450:  0f93           tst	r15
4452:  0520           jnz	$+0xc <main+0x26>
```
Will jump to this part:
```asm
445e:  3f40 e444      mov	#0x44e4 "Access Granted!", r15
4462:  b012 5845      call	#0x4558 <puts>
4466:  b012 9c44      call	#0x449c <unlock_door>
```
Which unlocks the door.
From the explanation above, for `r15` (the output of the `check_password`) to be `1`, all we have to do is to input exactly `8` bytes into the password.
# Solution
## All Solutions
Any input that is 8 bytes long will solve the level.
## Examples
ASCII: `AAAAAAAA`
Hex: `1111111111111111`
## Input Generation
To generate the ASCII input, I used this Python expression:
```python
'A' * 8
```
To generate the hex input, I used this Python expression:
```python
'1' * 16
```
# Lesson from the Level
Don't **ever** have simple password verification logic...
That is, unless you *really* want to get hacked.