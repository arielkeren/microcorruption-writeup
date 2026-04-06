# Introduction
This is the **6th** level in the Microcorruption series.
This is the first level that requires the use of the disassembler.
It also requires looking at the memory dump while the program is running.
# Vulnerability Title
Password appears directly in the code.
# Walkthrough
Just from looking at the `main` function, it's not so easy to know exactly what's going on:
```asm
4438 <main>
4438:  3e40 2045      mov	#0x4520, r14
443c:  0f4e           mov	r14, r15
443e:  3e40 f800      mov	#0xf8, r14
4442:  3f40 0024      mov	#0x2400, r15
4446:  b012 8644      call	#0x4486 <enc>
444a:  b012 0024      call	#0x2400
444e:  0f43           clr	r15
```
It looks like the `main` function calls 2 other functions - one at address `0x4486`, and the other at address `0x2400`.
Let's take a look at the one at address `0x2400`, as it's the last function executed, and it probably contains the verification logic.
Unfortunately, the code for this function isn't available in the given disassembly, meaning we'll have to find it by looking at the memory dump.
# Vulnerability
Putting a breakpoint *after* the call to this function, say on line `444e`, running the program, and continuing with a random password to the breakpoint, we can take a look at the live memory dump.
Scrolling to the function on address `0x2400`, we can see the following:
```
2400: 0b12 0412 0441 2452 3150 e0ff 3b40 2045   .....A$R1P..;@ E
2410: 073c 1b53 8f11 0f12 0312 b012 6424 2152   .<.S........d$!R
2420: 6f4b 4f93 f623 3012 0a00 0312 b012 6424   oKO..#0.......d$
2430: 2152 3012 1f00 3f40 dcff 0f54 0f12 2312   !R0...?@...T..#.
2440: b012 6424 3150 0600 b490 23a5 dcff 0520   ..d$1P....#.... 
2450: 3012 7f00 b012 6424 2153 3150 2000 3441   0....d$!S1P .4A
2460: 3b41 3041 1e41 0200 0212 0f4e 8f10 024f   ;A0A.A.....N...O
2470: 32d0 0080 b012 1000 3241 3041 d21a 189a   2.......2A0A....
2480: 22dc 45b9 4279 2d55 858e a4a2 67d7 14ae   ".E.By-U....g...
2490: a119 76f6 42cb 1c04 0efa a61b 74a7 416b   ..v.B.......t.Ak
24a0: d237 a253 22e4 66af c1a5 938b 8971 9b88   .7.S".f......q..
24b0: fa9b 6674 4e21 2a6b b143 9151 3dcc a6f5   ..ftN!*k.C.Q=...
24c0: daa7 db3f 8d3c 4d18 4736 dfa6 459a 2461   ...?.<M.G6..E.$a
24d0: 921d 3291 14e6 8157 b0fe 2ddd 400b 8688   ..2....W..-.@...
24e0: 6310 3ab3 612b 0bd9 483f 4e04 5870 4c38   c.:.a+..H?N.XpL8
24f0: c93c ff36 0e01 7f3e fa55 aeef 051c 242c   .<.6..>.U....$,
2500: 3c56 13af e57b 8abf 3040 c537 656e 8278   <V...{..0@.7en.x
2510: 9af9 9d02 be83 b38c e181 3ad8 395a fce3   ..........:.9Z..
2520: 4f03 8ec9 9395 4a15 ce3b fd1e 7779 c9c3   O.....J..;..wy..
2530: 5ff2 3dc7 5953 8826 d0b5 d9f8 639e e970   _.=.YS.&....c..p
2540: 01cd 2119 ca6a d12c 97e2 7538 96c5 8f28   ..!..j.,..u8...(
2550: d682 1be5 ab20 7389 48aa 1fa3 472f a564   ..... s.H...G/.d
2560: de2d b710 9081 5205 8d44 cff4 bc2e 577a   .-....R..D....Wz
2570: d5f4 a851 c243 277d a4ca 1e6b 0000 0000   ...Q.C'}...k....
2580: 0000 0000 0000 0000 0000 0000 0000 0000   ................
```
This is the function's code, and there is a tool in the Microcorruption website just for disassembling this code into readable assembly.
Before putting this in the disassembler, we do have to strip out the ASCII part on the right:
```
2400: 0b12 0412 0441 2452 3150 e0ff 3b40 2045
2410: 073c 1b53 8f11 0f12 0312 b012 6424 2152
2420: 6f4b 4f93 f623 3012 0a00 0312 b012 6424
2430: 2152 3012 1f00 3f40 dcff 0f54 0f12 2312
2440: b012 6424 3150 0600 b490 23a5 dcff 0520
2450: 3012 7f00 b012 6424 2153 3150 2000 3441
2460: 3b41 3041 1e41 0200 0212 0f4e 8f10 024f
2470: 32d0 0080 b012 1000 3241 3041 d21a 189a
2480: 22dc 45b9 4279 2d55 858e a4a2 67d7 14ae
2490: a119 76f6 42cb 1c04 0efa a61b 74a7 416b
24a0: d237 a253 22e4 66af c1a5 938b 8971 9b88
24b0: fa9b 6674 4e21 2a6b b143 9151 3dcc a6f5
24c0: daa7 db3f 8d3c 4d18 4736 dfa6 459a 2461
24d0: 921d 3291 14e6 8157 b0fe 2ddd 400b 8688
24e0: 6310 3ab3 612b 0bd9 483f 4e04 5870 4c38
24f0: c93c ff36 0e01 7f3e fa55 aeef 051c 242c
2500: 3c56 13af e57b 8abf 3040 c537 656e 8278
2510: 9af9 9d02 be83 b38c e181 3ad8 395a fce3
2520: 4f03 8ec9 9395 4a15 ce3b fd1e 7779 c9c3
2530: 5ff2 3dc7 5953 8826 d0b5 d9f8 639e e970
2540: 01cd 2119 ca6a d12c 97e2 7538 96c5 8f28
2550: d682 1be5 ab20 7389 48aa 1fa3 472f a564
2560: de2d b710 9081 5205 8d44 cff4 bc2e 577a
2570: d5f4 a851 c243 277d a4ca 1e6b 0000 0000
2580: 0000 0000 0000 0000 0000 0000 0000 0000
```
After disassembling this, we get the following:
```asm
0b12 push r11 0412 push r4 0441 mov sp, r4 2452 add #0x4, r4 3150 e0ff add #0xffe0, sp 073c jmp $+0x10 1b53 inc r11 8f11 sxt r15 0f12 push r15 0312 push #0x0 b012 6f4b call #0x4b6f 4f93 tst.b r15 f623 jnz $-0x12 3012 0a00 push #0xa 0312 push #0x0 2152 add #0x4, sp 3012 1f00 push #0x1f 3f40 dcff mov #0xffdc, r15 0f54 add r4, r15 b012 6424 call #0x2464 3150 0600 add #0x6, sp b490 23a5 3012 cmp #0xa523, 0x1230(r4) 7f00 rrc.b @r15+ b012 6424 call #0x2464 2153 incd sp 3150 3b41 add #0x413b, sp 3041 ret 1e41 0200 mov 0x2(sp), r14 0212 push sr 0f4e mov r14, r15 32d0 0080 bis #0x8000, sr b012 1000 call #0x10 3241 pop sr 3041 ret 22dc bis @r12, sr 45b9 bit.b r9, r5 4279 subc.b r9, sr 2d55 add @r5, r13 858e a4a2 sub r14, -0x5d5c(r5) a119 sxt @sp 76f6 and.b @r6+, r6 42cb bic.b r11, sr 1c04 0efa rrc -0x5f2(r12) a61b invalid @r6 d237 jge $-0x5a a253 22e4 incd &0xe422 66af dadd.b @r15, r6 c1a5 938b dadd.b r5, -0x746d(sp) fa9b 6674 cmp.b @r11+, 0x7466(r10) 4e21 jnz $+0x29e 2a6b addc @r11, r10 b143 9151 mov #-0x1, 0x5191(sp) daa7 db3f 8d3c dadd.b 0x3fdb(r7), 0x3c8d(r10) 4d18 rrc.b r13 4736 jge $-0x370 dfa6 921d 3291 dadd.b 0x1d92(r6), -0x6ece(r15) 14e6 8157 xor 0x5781(r6), r4 b0fe 2ddd and @r14+, -0x22d3(pc) 6310 rrc.b #0x2 3ab3 bit #-0x1, r10 612b jnc $-0x13c 0bd9 bis r9, r11 483f jmp $-0x16e 4e04 rrc.b r14 c93c jmp $+0x194 ff36 jge $-0x200 0e01 rra r14 7f3e jmp $-0x300 fa55 aeef add.b @r5+, -0x1052(r10) 3c56 add @r6+, r12 13af e57b dadd 0x7be5(r15), 4 8abf 3040 bit r15, 0x4030(r10) c537 jge $-0x74 9af9 9d02 be83 and 0x29d(r9), -0x7c42(r10) b38c e181 sub @r12+, 8 3ad8 bis @r8+, r10 4f03 reti r15 8ec9 9395 bic r9, -0x6a6d(r14) 4a15 rra.b r10 ce3b jl $-0x62 fd1e call @r13+ 5ff2 3dc7 and.b &0xc73d, r15 5953 add.b #0x1, r9 8826 jz $-0x2ee d0b5 d9f8 01cd bit.b -0x727(r5), -0x32ff(pc) 2119 rra @sp ca6a d12c addc.b r10, 0x2cd1(r10) 97e2 7538 d682 xor &0x3875, -0x7d2a(r7) 1be5 ab20 xor 0x20ab(r5), r11 7389 sub.b @r9+, 4 48aa dadd.b r10, r8 1fa3 dinc r15 de2d jc $+0x3be b710 swpb @r7+ 9081 5205 8d44 sub 0x552(sp), 0x448d(pc) cff4 d5f4 and.b r4, -0xb2b(r15) a851 c243 add @sp, 0x43c2(r8) 277d subc @r13, r7 a4ca 1e6b bic @r10, 0x6b1e(r4) 0000 rrc pc 0000 rrc pc 0000 rrc pc 0000 rrc pc 0000 rrc pc 0000 rrc pc
```
Searching for an easy `cmp` instruction, we can find 2 of them.
Looking at the first one:
```asm
cmp #0xa523, 0x1230(r4)
```
We can see that the password, which is probably at `0x1230(r4)`, is compared to `0xa523`.
Before trying to input `0xa523` into the password, we do have to remember that the machine is running in **little endian**, meaning we have to flip the bytes.
# Exploit
Now, we just have to flip the bytes of the literal we have found: `0x23a5`.
# Solution
Hex: `23a5`