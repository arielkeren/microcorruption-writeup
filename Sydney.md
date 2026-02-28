# Introduction
This is the third level in the Microcorruption series.
This level is pretty much like the second level.
It's also relatively easy to solve, not requiring any special exploiting techniques.
# Vulnerability
The lock's password is (again) right in the code.
Let's take a look at the `check_password` function:
```asm
448a <check_password>
448a:  bf90 5b29 0000 cmp	#0x295b, 0x0(r15)
4490:  0d20           jnz	$+0x1c <check_password+0x22>
4492:  bf90 3f5b 0200 cmp	#0x5b3f, 0x2(r15)
4498:  0920           jnz	$+0x14 <check_password+0x22>
449a:  bf90 577d 0400 cmp	#0x7d57, 0x4(r15)
44a0:  0520           jnz	$+0xc <check_password+0x22>
44a2:  1e43           mov	#0x1, r14
44a4:  bf90 437b 0600 cmp	#0x7b43, 0x6(r15)
44aa:  0124           jz	$+0x4 <check_password+0x24>
44ac:  0e43           clr	r14
44ae:  0f4e           mov	r14, r15
44b0:  3041           ret
```
Each comparison in the `check_password` function checks equality of a 2-byte literal with 2 bytes of the input.
Passing all the comparisons will unlock the door.
# Exploit
From the explanation above, we need to pass all the comparisons, meaning we have to construct the password byte-by-byte from the 2-byte literals in the function.
Let's list all the 2-byte literals:
1. `0x295b` - Has to match bytes `0`-`1` in the input.
2. `0x5b3f` - Has to match bytes `2`-`3` in the input.
3. `0x7d57` - Has to match bytes `4`-`5` in the input.
4. `0x7b43` - Has to match bytes `6`-`7` in the input.
Before we put them all together, we have to remember that we're working in little endian, meaning we have to flip the bytes in every 2-byte literal.
Let's list all the 2-byte literals, but flipped:
5. `0x5b29`
6. `0x3f5b`
7. `0x577d`
8. `0x437b`
Now that they're all flipped correctly, we can put them all together for the final input: `5b293f5b577d437b`.
# Solution
Hex: `5b293f5b577d437b`
# Lesson from the Level
Don't **ever** put your password (or any secret) directly in the code...
That is, unless you *really* want to get hacked.