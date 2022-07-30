Title:  Movement Keys Without a Numeric Keypad
Date:   2021-07-22 11:16:47
Tags: exile, patches
Category: Exile

## Motivation

Movement in the Exile games is controlled either by clicking on the terrain in
the desired direction, or by using the keys on the numeric keypad. Clicking with
the mouse seems tedious to me, so I have always used the keypad. In the past, I
have even specifically bought a laptop with a numeric keypad just to play Exile.
However, for my newest laptop, there were not a lot of choices with a keypad,
and I ended up getting something without one.

So now it is time to modify the game to use different keys for movement.

## Remedy

I have decided to substitute `[m,.][jkl][uio]` for the keypad's `1-9` keys. This
will make them easy to find without looking by placing my right hand on the home
row with my index finger on `j`, just like the normal touch typing position that
I am used to. Of course, this means I will have to choose substitutes for the
`l` and `m` keys, which are used in Exile for *L*ooking and casting *M*age
spells. I will choose `h` and `n` for these, just because they are convenient
and unused.

## Method

I will start with the free, registered version of [Exile I for
Macintosh](http://www.spiderwebsoftware.com/ftp/installers/mac/ExileInstaller.zip)
that is available for download from Spiderweb Software. After copying the
Installer to a virtual HFS volume and running it in BasiliskII, the Exile
program file is ready to play. I want to get a copy of the executable code on my
native Linux filesystem so that I can use modern unix tools to examine and
modify it. From BasiliskII, I used a program to convert the Exile game file to
MacBinary format. This combines the data and resource forks into a file with a
single fork that unix understands.

Now I have the game code on my Linux filesystem as a MacBinary encoded file
called [`e1orig.bin`]({static}/exile/e1orig.bin). Let's find those keycodes.

The Exile games were written in C and compiled with _Metrowerk's Code Warrior_, a
popular Macintosh IDE from the 1990s. Code Warrior inserts so called `macsbug`
strings into the executable code it generates. These strings contain the names
of the functions that were given in the original C source code. This makes it
very easy to locate particular sections of code in an executable. Let's search
the `macsbug` strings for the word `keystroke`.

```sh
$ python3 -c "print(open('e1orig.bin','rb').read().find(b'keystroke'))"
2425898
$ xxd -s 2425888 -l 32 e1orig.bin
00250420: 4e75 9068 616e 646c 655f 6b65 7973 7472  Nu.handle_keystr
00250430: 6f6b 6500 0000 4e56 0000 2f0a 594f 3f3c  oke...NV../.YO?<
```

We found a procedure called `handle_keystroke`. The name immediately follows the
end of the function, so will search backwards from the name to find the start of
the function. Macintosh procedures start with a `LINK A6` (`4e56`) instruction
and end with a `RTS` instruction (`4e75` as seen above).

Disassembling the `handle_keystroke` function from the beginning yields:

```sh
$ d68 e1orig.bin 2424198 128
00000000 : 4e56 ffac      LINK     A6,#-84
00000004 : 48e7 1f30      MOVEM.L  D4,D5,A0-A4,-(A7)
00000008 : 1a2e 0009      MOVE.B   +9(A6),D5
0000000c : 1c2e 000b      MOVE.B   +11(A6),D6
00000010 : 7800           MOVEQ    #$00,D4
00000012 : 42ae fff8      CLR.L    -8(A6)
00000016 : 2d79 0001 094c ffb4 MOVE.L   $0001094c,-76(A6)
0000001e : 2d79 0001 0948 ffb0 MOVE.L   $00010948,-80(A6)
00000026 : 2d79 0001 093e ffc6 MOVE.L   $0001093e,-58(A6)
0000002e : 2d79 0001 0942 ffca MOVE.L   $00010942,-54(A6)
00000036 : 3d79 0001 0946 ffce MOVE.W   $00010946,-50(A6)
0000003e : 41f9 0001 0916 LEA      $00010916,A0
00000044 : 43ee ffd0      LEA      -48(A6),A1
00000048 : 7009           MOVEQ    #$09,D0
0000004a : 22d8           MOVE.L   (A0)+,(A1)+
0000004c : 51c8 fffc      DBF      D0,$0000004a
00000050 : 594f           SUBQ.W   #4,A7
00000052 : a924           ???
00000054 : 205f           MOVEA.L  (A7)+,A0
... output truncated ...
```

A-trap instructions such as `a924` are not yet recognized by the dis68k disassembler.
Maybe someday I will add them. Let's look a little farther into the procedure:

```sh
$ d68 e1orig.bin $((0x24ff60)) 192
00000000 : 1005           MOVE.B   D5,D0
00000002 : 4880           EXT.W    D0
00000004 : 0440 0030      SUBI.W   #$0030,D0	0
00000008 : 6700 0134      BEQ      $0000013e
0000000c : 5340           SUBQ.W   #1,D0	1
0000000e : 6700 00b0      BEQ      $000000c0
00000012 : 5340           SUBQ.W   #1,D0	2
00000014 : 6700 00aa      BEQ      $000000c0
00000018 : 5340           SUBQ.W   #1,D0	3
0000001a : 6700 00a4      BEQ      $000000c0
0000001e : 5340           SUBQ.W   #1,D0	4
00000020 : 6700 009e      BEQ      $000000c0
00000024 : 5340           SUBQ.W   #1,D0	5
00000026 : 6700 0098      BEQ      $000000c0
0000002a : 5340           SUBQ.W   #1,D0	6
0000002c : 6700 0092      BEQ      $000000c0
00000030 : 5740           SUBQ.W   #3,D0	9
00000032 : 6700 00ee      BEQ      $00000122
00000036 : 5940           SUBQ.W   #4,D0	=
00000038 : 6700 0104      BEQ      $0000013e
0000003c : 5940           SUBQ.W   #4,D0	A
0000003e : 6700 023a      BEQ      $0000027a
00000042 : 5b40           SUBQ.W   #5,D0	F
00000044 : 6700 0188      BEQ      $000001ce
00000048 : 5340           SUBQ.W   #1,D0	G
0000004a : 6700 01b4      BEQ      $00000200
0000004e : 5b40           SUBQ.W   #5,D0	L -> H subq.w #1,d0	5340
00000050 : 6700 0228      BEQ      $0000027a
00000054 : 5340           SUBQ.W   #1,D0	M -> N subq.w #6,d0	5d40
00000056 : 6700 02d6      BEQ      $0000032e
0000005a : 5740           SUBQ.W   #3,D0	P -> P subq.w #2,d0	5540
0000005c : 6700 02d0      BEQ      $0000032e
00000060 : 0440 0011      SUBI.W   #$0011,D0	a
00000064 : 6700 01c6      BEQ      $0000022c
00000068 : 5340           SUBQ.W   #1,D0	b
0000006a : 6700 020e      BEQ      $0000027a
0000006e : 5540           SUBQ.W   #2,D0	d
00000070 : 6700 02bc      BEQ      $0000032e
00000074 : 5340           SUBQ.W   #1,D0	e
00000076 : 6700 025c      BEQ      $000002d4
0000007a : 5340           SUBQ.W   #1,D0	f
0000007c : 6700 02b0      BEQ      $0000032e
00000080 : 5340           SUBQ.W   #1,D0	g
00000082 : 6700 02aa      BEQ      $0000032e
00000086 : 5b40           SUBQ.W   #5,D0	l -> h subq.w #1,d0	5340
00000088 : 6700 02a4      BEQ      $0000032e
0000008c : 5340           SUBQ.W   #1,D0	m -> n subq.w #6,d0	5d40
0000008e : 6700 029e      BEQ      $0000032e
00000092 : 5740           SUBQ.W   #3,D0	p -> p subq.w #2,d0	5540
00000094 : 6700 0298      BEQ      $0000032e
00000098 : 5540           SUBQ.W   #2,D0	r
0000009a : 6700 0292      BEQ      $0000032e
0000009e : 5340           SUBQ.W   #1,D0	s
000000a0 : 6700 0232      BEQ      $000002d4
000000a4 : 5340           SUBQ.W   #1,D0	t
000000a6 : 6700 0286      BEQ      $0000032e
000000aa : 5740           SUBQ.W   #3,D0	w
000000ac : 6700 0280      BEQ      $0000032e
000000b0 : 5340           SUBQ.W   #1,D0	x
000000b2 : 6700 0220      BEQ      $000002d4
000000b6 : 5540           SUBQ.W   #2,D0	z
000000b8 : 6700 009e      BEQ      $00000158
```

I added the letters for the keycodes in the rightmost column to the listing
above. We can see that the routine reads the keycode of the key that was pressed
from register D5 and then compares it to a series of values and branches to the
corresponding code when a match is found. The first keycode that is compared is
`0x30` (regular, non-numpad `0`), followed by the (regular, non-numpad) digits
`1` through `6`. Farther down, the letters `g`, `l`, `m`, and `p` are checked.
This is were the code should be modified to substitute `h` and `n` instead. The
modification is accomplished simply by changing the immediate value of the
`subq.w #_,d0` instructions as shown above. Following the branches above for the
`l` and `m` keys, there are two more locations that need similar modifications:

```sh
0000034c : 1005           MOVE.B   D5,D0
0000034e : 4880           EXT.W    D0
00000350 : 0440 004d      SUBI.W   #$004D,D0	M -> N	0440 004e
00000354 : 6734           BEQ      $0000038a
00000356 : 5740           SUBQ.W   #3,D0		P -> P	5540
00000358 : 6744           BEQ      $0000039e
0000035a : 0440 0014      SUBI.W   #$0014,D0	d
0000035e : 6700 00de      BEQ      $0000043e
00000362 : 5540           SUBQ.W   #2,D0		f
00000364 : 6700 00fa      BEQ      $00000460
00000368 : 5340           SUBQ.W   #1,D0		g
0000036a : 6700 00e4      BEQ      $00000450
0000036e : 5b40           SUBQ.W   #5,D0		l -> h subq.w #1,d0	5340
00000370 : 6740           BEQ      $000003b2
00000372 : 5340           SUBQ.W   #1,D0		m -> n subq.w #6,d0	5d40
00000374 : 6722           BEQ      $00000398
00000376 : 5740           SUBQ.W   #3,D0		p -> p subq.w #2,d0	5540
00000378 : 6732           BEQ      $000003ac
0000037a : 5540           SUBQ.W   #2,D0		r
0000037c : 673a           BEQ      $000003b8
0000037e : 5540           SUBQ.W   #2,D0		t
00000380 : 674a           BEQ      $000003cc
00000382 : 5740           SUBQ.W   #3,D0		w
00000384 : 675c           BEQ      $000003e2
```

That takes care of substituting `l` and `m` with `h` and `n`. Now we can work on
changing the numeric keypad motion keys. This is actually much easier to change
because the motion keys are stored as an array of keycodes in a statically
initialized data block.

The listing below shows that the keycode is compared to each value in a ten
element array, and if a match is found, another routine is called with arguments
based on which element was matched.

```sh
$ d68 e1orig.bin 2424502 92
00000000 : 7600           MOVEQ    #$00,D3        ;d3 is the array index
00000002 : 6052           BRA      $00000056
00000004 : 41ee ffc6      LEA      -58(A6),A0
00000008 : bc30 3000      CMP.B    +0(A0,D3.W),D6 ;compare d6 to each value in the array
0000000c : 6646           BNE      $00000054
0000000e : 4a43           TST      D3
00000010 : 6604           BNE      $00000016
00000012 : 7a7a           MOVEQ    #$7A,D5
00000014 : 603e           BRA      $00000054
00000016 : 3443           MOVEA.W  D3,A2          ;lookup arguments based on matched index
00000018 : 200a           MOVE.L   A2,D0
0000001a : e588           LSL.L    #2,D0
0000001c : 45ee ffd0      LEA      -48(A6),A2
00000020 : d5c0           ADDA.L   D0,A0
00000022 : 3d52 fffe      MOVE.W   (A2),-2(A6)
00000026 : 3d6a 0002 fffc MOVE.W   +2(A2),-4(A6)
0000002c : 486e fffc      PEA      -4(A6)
00000030 : a870           ???
00000032 : 2d6e fffc 0016 MOVE.L   -4(A6),+22(A6)
00000038 : 41ee 001c      LEA      +28(A6),A0
0000003c : 2f20           MOVE.L   -(A0),-(A7)    ;push args on stack and call new routine
0000003e : 2f20           MOVE.L   -(A0),-(A7)
00000040 : 2f20           MOVE.L   -(A0),-(A7)
00000042 : 2f20           MOVE.L   -(A0),-(A7)
00000044 : 4eba df0e      JSR      -8434(PC) {$4294958932}
00000048 : 1800           MOVE.B   D0,D4
0000004a : 1004           MOVE.B   D4,D0
0000004c : 4fef 0010      LEA      +16(A7),A7
00000050 : 6000 0512      BRA      $00000564
00000054 : 5243           ADDQ.W   #1,D3
00000056 : 0c43 000a      CMPI.W   #$000A,D3
0000005a : 6da8           BLT      $00000004
```

The only thing left to do is locate the array of keypad codes. The Macintosh's
keyboard encodes the numeric keypad keys as `0x52-0x5c` for `0-9` with `0x5a`
being skipped. Searching the game file for an array of those values yields:

```sh
$ python3 -c "print(open('e1orig.bin','rb').read().find(b'\x52\x53\x54\x55\x56\x57\x58\x59\x5b\x5c'))"
2371181
$ xxd -s 2371181 -l 16 e1orig.bin
00242e6d: 5253 5455 5657 5859 5b5c 2325 0100 2125  RSTUVWXY[\#%..!%
```

The Macintosh key codes for the replacement keys (`[m,.][jkl][uio]`) are
`[2e2b2f][262825][20221f]`. Finally let's substitute numeric keypad `0` with `y`
(`0x10` on the Mac). I don't actually know what keypad `0` does in the game, but we
might as well grab it while we're here just in case.

Putting it all together, the patch file for the Mac version of Exile I looks
like this:

```sh
$ cat e1-keypad-patch.txt
00242e6d: 102e 2b2f 2628 2520 221f 2325 0100 2125
0024ffae: 5340 6700 0228 5d40 6700 02d6 5540 6700
0024ffe6: 5340 6700 02a4 5d40 6700 029e 5540 6700
002502b0: 0440 004e 6734 5540 6744 0440 0014 6700
002502ce: 5340 6740 5d40 6722 5540 6732 5540 673a
$ cp e1orig.bin e1-no-keypad.bin
$ xxd -r e1-keypad-patch.txt e1-no-keypad.bin
```

## Conclusion

We can copy the [patched game file]({static}/exile/keypad/e1-no-keypad.bin) back over to BasiliskII, extract the
application program from the MacBinary format, and test out our new motion keys.

Good luck and have fun!
