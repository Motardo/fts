Title:  Let Thrown Weapons Be Poisoned
Date:   2021-07-22 11:16:47
Category: Exile



## Motivation

In Exile, thrown weapons are pretty ineffective. They do very little damage,
have a short range, miss a lot, are consumed in a single use, heavy, and
overpriced. As if that wasn't bad enough, for some strange reason, they are the
only type of weapons in Exile which are unable to be poisoned. The only good
thing about thrown weapons is that throwing skill is cheaper than other weapon
skills. Let's make thrown weapons a little more interesting by allowing them to
be poisoned.

## Remedy

Looking through the source code for Blades (Windows version[1]), we find that,
when a PC uses poison, the game calls the `poison_weapon` function in
`PARTY.CPP` This function loops over the PC's inventory finding the first item
that is equipped and is also a weapon. Then it applies the poison to that item.

```c
Boolean poison_weapon( short pc_num, short how_much,short safe)
//short safe; // 1 - always succeeds
{
    short i,weap = 24,p_level,r1;
    short p_chance[21] = {40,72,81,85,88,89,90,
                            91,92,93,94,94,95,95,96,97,98,100,100,100,100};

    for (i = 0; i < 24; i++)
        if ((adven[pc_num].equip[i] == TRUE) && (is_weapon(pc_num,i) == TRUE)) {
            weap = i;
            i = 30;
            }
    if (weap == 24) {
        add_string_to_buf("  No weapon equipped.       ");
    ...
```
Interestingly, that `i = 30` statement is used to break out of the loop after a weapon is found. 

The `is_weapon` function returns true for one and two-handed melee weapons
(varieties 1 and 2), bows (5), and crossbows (24), and it returns false
otherwise.

```c
Boolean is_weapon(short pc_num,short item)
{
    if ((adven[pc_num].items[item].variety  == 1) ||  // one-handed melee
        (adven[pc_num].items[item].variety  == 2) ||  // two-handed melee
        (adven[pc_num].items[item].variety  == 5) ||  // bows
        (adven[pc_num].items[item].variety  == 24))   // crossbows
            return TRUE;
            else return FALSE;
}
```

So in order to allow thrown weapons to be poisoned, we just need to patch
`is_weapon` to return `true` for thrown weapons, which are item variety `6` (By
the way, item variety `3` is gold pieces and `4` is arrows). Next, let's look at the
`BLADES.EXE` binary to think about how we might proceed.

Notice the string "No weapon equipped" in the `poison_weapon` function above.
Strings like these will help us find the function in the game's binary. I used a
hex editor to search for this string in BLADES.EXE and located it at around
offset `0xdabb0.`

![wxHexEditor finding text in BLADES.EXE]({static}/exile/poison/search_nick.png){: style="width:100%;" }

Let's find the next routine after `0xdabb0` and disassemble it. We can tell by
the inclusion of the file `BC450RTL.DLL` in the game directory that `Blades`
(and indeed all of the Exile games for Windows) was compiled with *Borland Turbo C++
4.5*. This compiler begins C subroutines with the bytes `8c d0 90 45`, so we can
search for those bytes after `0xdabb0` and locate a subroutine at `0xdac60`.
Let's examine the disassembly.

```sh
$ ndisasm -k0,0xdac60 BLADES.EXE |head -n 50 |nl
     1	00000000  skipping 0xDAC60 bytes
     2	000DAC60  8CD0              mov ax,ss
     3	000DAC62  90                nop
     4	000DAC63  45                inc bp
     5	000DAC64  55                push bp
     6	000DAC65  8BEC              mov bp,sp
     7	000DAC67  1E                push ds
     8	000DAC68  8ED8              mov ds,ax
     9	000DAC6A  83EC30            sub sp,byte +0x30
    10	000DAC6D  56                push si
    11	000DAC6E  57                push di
    12	000DAC6F  8B7606            mov si,[bp+0x6]
    13	000DAC72  C746FC1800        mov word [bp-0x4],0x18
    14	000DAC77  8D46CE            lea ax,[bp-0x32]
    15	000DAC7A  16                push ss
    16	000DAC7B  50                push ax
    17	000DAC7C  1E                push ds
    18	000DAC7D  68BA3A            push word 0x3aba
    19	000DAC80  B92A00            mov cx,0x2a
    20	000DAC83  9AFFFF0000        call 0x0:0xffff
    21	000DAC88  33FF              xor di,di
    22	000DAC8A  EB2D              jmp short 0xacb9
    23	000DAC8C  8BDE              mov bx,si
    24	000DAC8E  69DB6A07          imul bx,bx,word 0x76a
    25	000DAC92  B80000            mov ax,0x0
    26	000DAC95  8EC0              mov es,ax
    27	000DAC97  268A8102BA        mov al,[es:bx+di-0x45fe]
    28	000DAC9C  98                cbw
    29	000DAC9D  3D0100            cmp ax,0x1
    30	000DACA0  7516              jnz 0xacb8
    31	000DACA2  57                push di
    32	000DACA3  56                push si
    33	000DACA4  90                nop
    34	000DACA5  0E                push cs
    35	000DACA6  E85001            call 0xadf9
    36	000DACA9  83C404            add sp,byte +0x4
    37	000DACAC  98                cbw
    38	000DACAD  3D0100            cmp ax,0x1
    39	000DACB0  7506              jnz 0xacb8
    40	000DACB2  897EFC            mov [bp-0x4],di
    41	000DACB5  BF1E00            mov di,0x1e
    42	000DACB8  47                inc di
    43	000DACB9  83FF18            cmp di,byte +0x18
    44	000DACBC  7CCE              jl 0xac8c
    45	000DACBE  837EFC18          cmp word [bp-0x4],byte +0x18
    46	000DACC2  7514              jnz 0xacd8
    47	000DACC4  0E                push cs
    48	000DACC5  68E339            push word 0x39e3
    49	000DACC8  9AFFFF0000        call 0x0:0xffff
    50	000DACCD  83C404            add sp,byte +0x4
```

Lines 2-8 are the standard function prolog which sets up the stack frame. Line 9
makes room on the stack for the function's local variables, and lines 10-21
initialize the local variables and save registers. The function begins execution
at line 22. Note that lines 22-44 make a loop which executes 24 times (line 43).
Also note how line 41 breaks the loop early by injecting a value of 30. This is
clearly the `poison_weapon` function we saw in the source code earlier. Line 35
calls the `is_weapon` function that we want to change. We can see the call is to
offset `0xadf9`, but it's not necessarily in the same segment. Although it is in
this case.

In the source code, `is_weapon` comes right after `poison_weapon` in the same
`PARTY.CPP` file, so there's a good chance it does in the binary as well. When
we search forward for the bytes `8c d0 90 45`, we do indeed find the next
routine at offset `0xadf9`. Let's examine the disassembly.

```sh
$ ndisasm -k0,0xdadf9 BLADES.EXE |head -n 55 |nl
     1	00000000  skipping 0xDADF9 bytes
     2	000DADF9  8CD0              mov ax,ss
     3	000DADFB  90                nop
     4	000DADFC  45                inc bp
     5	000DADFD  55                push bp
     6	000DADFE  8BEC              mov bp,sp
     7	000DAE00  1E                push ds
     8	000DAE01  8ED8              mov ds,ax
     9	000DAE03  8B5606            mov dx,[bp+0x6]
    10	000DAE06  8B4E08            mov cx,[bp+0x8]
    11	000DAE09  8BDA              mov bx,dx
    12	000DAE0B  69DB6A07          imul bx,bx,word 0x76a
    13	000DAE0F  8BC1              mov ax,cx
    14	000DAE11  6BC042            imul ax,ax,byte +0x42
    15	000DAE14  03D8              add bx,ax
    16	000DAE16  B80000            mov ax,0x0
    17	000DAE19  8EC0              mov es,ax
    18	000DAE1B  2683BFD2B301      cmp word [es:bx-0x4c2e],byte +0x1
    19	000DAE21  744E              jz 0xae71
    20	000DAE23  8BDA              mov bx,dx
    21	000DAE25  69DB6A07          imul bx,bx,word 0x76a
    22	000DAE29  8BC1              mov ax,cx
    23	000DAE2B  6BC042            imul ax,ax,byte +0x42
    24	000DAE2E  03D8              add bx,ax
    25	000DAE30  B80000            mov ax,0x0
    26	000DAE33  8EC0              mov es,ax
    27	000DAE35  2683BFD2B302      cmp word [es:bx-0x4c2e],byte +0x2
    28	000DAE3B  7434              jz 0xae71
    29	000DAE3D  8BDA              mov bx,dx
    30	000DAE3F  69DB6A07          imul bx,bx,word 0x76a
    31	000DAE43  8BC1              mov ax,cx
    32	000DAE45  6BC042            imul ax,ax,byte +0x42
    33	000DAE48  03D8              add bx,ax
    34	000DAE4A  B80000            mov ax,0x0
    35	000DAE4D  8EC0              mov es,ax
    36	000DAE4F  2683BFD2B305      cmp word [es:bx-0x4c2e],byte +0x5
    37	000DAE55  741A              jz 0xae71
    38	000DAE57  8BDA              mov bx,dx
    39	000DAE59  69DB6A07          imul bx,bx,word 0x76a
    40	000DAE5D  8BC1              mov ax,cx
    41	000DAE5F  6BC042            imul ax,ax,byte +0x42
    42	000DAE62  03D8              add bx,ax
    43	000DAE64  B80000            mov ax,0x0
    44	000DAE67  8EC0              mov es,ax
    45	000DAE69  2683BFD2B318      cmp word [es:bx-0x4c2e],byte +0x18
    46	000DAE6F  7506              jnz 0xae77
    47	000DAE71  B001              mov al,0x1
    48	000DAE73  EB06              jmp short 0xae7b
    49	000DAE75  EB04              jmp short 0xae7b
    50	000DAE77  B000              mov al,0x0
    51	000DAE79  EB00              jmp short 0xae7b
    52	000DAE7B  1F                pop ds
    53	000DAE7C  5D                pop bp
    54	000DAE7D  4D                dec bp
    55	000DAE7E  CB                retf
```

<aside style="font-size:80%;">
<h5>Aside</h5>
<p>Wow, that's really inefficient! Every pair of compare and branch instructions is
preceded by 7 identical instructions to compute the same value. The value could
have just been computed once, which would have made the code both shorter and
faster by 4.5 times. I wonder how different the whole Exile experience would
have been in the 90's if the games had been compiled with optimizations turned
on.</p>
</aside>

## Method

So, we have four comparisons for item varieties 1, 2, 5, and 24, and we want to
squeeze an extra comparison for variety 6 in somehow without taking up more
space and with minimal disruption to the existing structure. Since we also know
that variety 0 (nothing) and variety 3 (gold) can never be equipped, we don't
need to worry about filtering those values. So for any variety less than or
equal to 6, we can safely return `true` for everything except 4 (arrows), which
should return false. And if it's 7 or greater, we only need to check for variety
24 (crossbows). So we can actually get away with just 3 comparisons to implement
our new game logic (6, 4, and 24). Let's throw in an unnecessary comparison for
variety 0 just to maintain the original number of four comparisons. So our patch
will look like this.

<a href="{static}/exile/poison/poison_asm_diff.png" target="_blank">
![vimdiff showing new assembly code for is_weapon]({static}/exile/poison/poison_asm_diff.png){: style="width:100%;" }
</a>

```
$ cat patch.txt 
000dae20: 0074 54  <- 0174 4e
000dae3a: 0474 3a  <- 0274 34
000dae54: 067e 1a  <- 0574 1a
```
We just changed the first three compare and jump pairs. The fourth one for
crossbows stays the same.

Let's apply the patch to a copy of `BLADES.EXE`

```sh
$ nasm -o wb-poison wb-poison.asm
$ ndisasm -o 0xdae1b wb-poison
000DAE1B  2683BFD2B300      cmp word [es:bx-0x4c2e],byte +0x0
000DAE21  7454              jz 0xae77
000DAE23  89D3              mov bx,dx
000DAE25  69DB6A07          imul bx,bx,word 0x76a
000DAE29  89C8              mov ax,cx
000DAE2B  6BC042            imul ax,ax,byte +0x42
000DAE2E  01C3              add bx,ax
000DAE30  B80000            mov ax,0x0
000DAE33  8EC0              mov es,ax
000DAE35  2683BFD2B304      cmp word [es:bx-0x4c2e],byte +0x4
000DAE3B  743A              jz 0xae77
000DAE3D  89D3              mov bx,dx
000DAE3F  69DB6A07          imul bx,bx,word 0x76a
000DAE43  89C8              mov ax,cx
000DAE45  6BC042            imul ax,ax,byte +0x42
000DAE48  01C3              add bx,ax
000DAE4A  B80000            mov ax,0x0
000DAE4D  8EC0              mov es,ax
000DAE4F  2683BFD2B306      cmp word [es:bx-0x4c2e],byte +0x6
000DAE55  7E1A              jng 0xae71
000DAE57  89D3              mov bx,dx

$ cp BLADES.EXE BLADESPT.EXE
$ alias dd='dd iflag=count_bytes oflag=seek_bytes conv=notrunc'
$ dd count=64 seek=$((0xdae1b)) if=wb-poison of=BLADESPT.EXE 
0+1 records in
0+1 records out
64 bytes copied, 0.000366593 s, 175 kB/s
$ cmp B*
BLADES.EXE BLADESPT.EXE differ: byte 896545, line 2844
$ ndisasm -k 0,0xdae1b BLADESPT.EXE |head -n 30
00000000  skipping 0xDAE1B bytes
000DAE1B  2683BFD2B300      cmp word [es:bx-0x4c2e],byte +0x0
000DAE21  7454              jz 0xae77
000DAE23  89D3              mov bx,dx
000DAE25  69DB6A07          imul bx,bx,word 0x76a
000DAE29  89C8              mov ax,cx
000DAE2B  6BC042            imul ax,ax,byte +0x42
000DAE2E  01C3              add bx,ax
000DAE30  B80000            mov ax,0x0
000DAE33  8EC0              mov es,ax
000DAE35  2683BFD2B304      cmp word [es:bx-0x4c2e],byte +0x4
000DAE3B  743A              jz 0xae77
000DAE3D  89D3              mov bx,dx
000DAE3F  69DB6A07          imul bx,bx,word 0x76a
000DAE43  89C8              mov ax,cx
000DAE45  6BC042            imul ax,ax,byte +0x42
000DAE48  01C3              add bx,ax
000DAE4A  B80000            mov ax,0x0
000DAE4D  8EC0              mov es,ax
000DAE4F  2683BFD2B306      cmp word [es:bx-0x4c2e],byte +0x6
000DAE55  7E1A              jng 0xae71
000DAE57  89D3              mov bx,dx
000DAE59  69DB6A07          imul bx,bx,word 0x76a
000DAE5D  8BC1              mov ax,cx
000DAE5F  6BC042            imul ax,ax,byte +0x42
000DAE62  03D8              add bx,ax
000DAE64  B80000            mov ax,0x0
000DAE67  8EC0              mov es,ax
000DAE69  2683BFD2B318      cmp word [es:bx-0x4c2e],byte +0x18
000DAE6F  7506              jnz 0xae77
```

Everything looks good. Let's fire it up in DOSBox and see if it works.

![Frrr with darts equiped and poisoned]({static}/exile/poison/poison-darts.png){: style="width:100%;" }

Success!

## Macintosh Versions

I prefer to play the Mac versions of the Exile games using the BasiliskII emulator,
so let's make the same modification to them. Here is the `is_weapon` function
in 68k disassembly from the Mac version of Blades of Exile:

```sh
3430 087E	move.w    $7E(a0,d0.l),d2
0C42 0001	cmpi.w    #$01,d2
6712		beq.s     True
0C42 0002	cmpi.w    #$02,d2
670C		beq.s     True
0C42 0005	cmpi.w    #$05,d2
6706		beq.s     True
0C42 0018	cmpi.w    #$18,d2
6604		bne.s     False
True:   ; return true
7001		moveq     #$01,d0
6002		bra.s     Continue
False:  ; return false
7000		moveq     #$00,d0
Continue:
```

Just like the x86 version, it returns true for item varieties `1`, `2`, `5`, and `24`.
The new code will return `false` for varieties `0` and `4`, or return `true` for 
anything less than `7` or `24`. It will look like this:

```sh
3430 087E	move.w    $7E(a0,d0.l),d2
0C42 0000	cmpi.w    #$00,d2
6716		beq.s     False
0C42 0004	cmpi.w    #$04,d2
6710		beq.s     False
0C42 0006	cmpi.w    #$06,d2
6F06		ble.s     True
0C42 0018	cmpi.w    #$18,d2
6604		bne.s     False
True:   ; return true
7001		moveq     #$01,d0
6002		bra.s     Continue
False:  ; return false
7000		moveq     #$00,d0
Continue:
```

In _Exile III_, the original code and patch are identical to _Blades of Exile_.
For _Exile II_ and _Exile I_, there were no crossbows, so the code is a little bit
different. 

```sh
Original Exile I and Exile II:
3430 087E	move.w    $7E(a0,d0.l),d2
0C42 0001	cmpi.w    #$01,d2
670C		beq.s     True
0C42 0002	cmpi.w    #$02,d2
6706		beq.s     True
0C42 0005	cmpi.w    #$05,d2
6604		bne.s     False
True:
7001		moveq     #$01,d0
6002		bra.s     Continue
False:
7000		moveq     #$00,d0
Continue:

New Exile I and Exile II:
3430 087E	move.w    $7E(a0,d0.l),d2
0C42 0000	cmpi.w    #$00,d2
6710		beq.s     False
0C42 0004	cmpi.w    #$04,d2
670a		beq.s     False
0C42 0006	cmpi.w    #$06,d2
6E04		bgt.s     False
True:
7001		moveq     #$01,d0
6002		bra.s     Continue
False:
7000		moveq     #$00,d0
Continue:
```

So the patches for the Mac versions look like this:

_Blades_ and _Exile III_:

```sh
Original: 0001 6712 0c42 0002 670c 0c42 0005 6706
Patched:  0000 6716 0c42 0004 6710 0c42 0006 6f06
```

_Exile I_ and _Exile II_:

```
Original: 0001 670c 0c42 0002 6706 0c42 0005 6604  ..g..B..g..B..f.
Patched:  0000 6710 0c42 0004 670a 0c42 0006 6e04  ..g..B..g..B..f.
```

Here is a screenshot of the Mac version of Exile I showing John having 
darts successfully envenomed.

![Darts equiped and poisoned in the Mac version of Exile I]({static}/exile/poison/ExilePoisonThrown.png){: style="width:100%;" }

## Conclusion

Allowing thrown weapons to be poisoned should make them slightly less useless.
It might be just the nudge needed to try playing a throwing-based character. In
fact, a hasted character with 8 action points could hit an enemy with 4 poison
darts in one turn. I'm not sure if poison stacks like that, but that could
potentially be quite effective.

Happy Adventuring!
