# Introduction
This is the second level in the Microcorruption series.
This level is a bit more difficult than the first level (Tutorial).
But, it's still relatively easy to solve, not requiring any special exploiting techniques.
# Vulnerability
The lock's password is right in the code.
Let's take a look at the `create_password` function:
```asm
447e <create_password>
447e:  3f40 0024      mov	#0x2400, r15
4482:  ff40 3000 0000 mov.b	#0x30, 0x0(r15)
4488:  ff40 6500 0100 mov.b	#0x65, 0x1(r15)
448e:  ff40 6e00 0200 mov.b	#0x6e, 0x2(r15)
4494:  ff40 6a00 0300 mov.b	#0x6a, 0x3(r15)
449a:  ff40 2700 0400 mov.b	#0x27, 0x4(r15)
44a0:  ff40 5000 0500 mov.b	#0x50, 0x5(r15)
44a6:  ff40 3c00 0600 mov.b	#0x3c, 0x6(r15)
44ac:  cf43 0700      mov.b	#0x0, 0x7(r15)
44b0:  3041           ret
```
We can see that the lock's password gets put byte-by-byte, starting at address `0x2400`.
Putting all the bytes together, we get this password: `30656e6a27503c00`.
# Exploit
From the explanation above, the password is just `30656e6a27503c00`.
So, all there is to do is to input it exactly as the password, and the door will be unlocked.
# Solution
Hex: `30656e6a27503c00`
# Lesson from the Level
Don't **ever** put your password (or any secret) directly in the code...
That is, unless you *really* want to get hacked.