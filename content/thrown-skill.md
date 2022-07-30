Title:  Thrown Missile Skill
Date:   2020-06-01 16:16:47
Category: Exile

Motivation
----------------

Thrown weapons are common in Exile, and throwing skill is cheap in both gold and
skill points. Unfortunately, in all versions of Exile, throwing skill does not
actually do anything. The chance to hit with a thrown missile attack depends
strangely on the PC's defense skill. It would be nice if throwing skill actually
increased the chance to hit.

Remedy
----------------

Make the chance to hit with thrown missiles depend on throwing skill instead of
defense.

Method
----------------

We just need to swap `defense` for `thrown` in the code that calculates the
chance to hit with a thrown missile. It is a matter of subtracting 4 from the
value of a single byte in the executable[^1].

### Windows Versions

Use any hex editor to replace the byte as shown at the specified offset in the
specified file.

| File               | Version | Offset (Decimal)     | Change (subtract 4)      |
| :----------------- | :------ | :------------------- | :----------------------- |
| BLADES.EXE         | v1.0.1  | 0x8B5DA (570842)     | 0x7a should be 0x76      |
| EXILE3.EXE         | v1.0b   | 0x7AF17 (503575)     | 0x4c should be 0x48      |
| EXILE2.EXE         | v2.0.1  | 0xC25DD (796125)     | 0xd8 should be 0xd4      |
| EXILE.EXE          | v2.0    | 0x5ABD9 (371673)     | 0x26 should be 0x22      |

### Macintosh Versions (68k[^2] )

Use a resource editor such as *ResEdit* to replace the byte `0x26` with the byte
`0x22` at the specified offset of the specified `CODE` resource in the resource
fork of the specified file.

| File                         | CODE resource | Offset |
| :--------------------------- | :------------ | :----- |
| Blades of Exile (fat) v1.0.2 | CODE 7        | 0x3a0d |
| Exile III (fat) v1.0.3       | CODE 7        | 0x363b |
| Exile II v2.0.3              | CODE 1        | 0x4e91 |
| Exile v2.0.1                 | CODE 3        | 0x82e1 |

#### Alternative Method without *ResEdit*

Normally we would use a resource editor such as *ResEdit* to modify the resource
fork of any Macintosh files. However, since we are only changing the value of a
constant, it is okay to use a plain hex editor in this case. Search the resource
fork of the executable for the hex bytes `3d 70 08 26`. The sequence will only
appear once in each file. Change the `0x26` at the end of the sequence to
`0x22`.

Conclusion
-----------------

Now when we spend skill points on throwing skill, it's not a total waste.

Happy Adventuring!

-----------------

Footnotes:

[^1]: These are the free registered versions available for download from
      [Spiderweb Software](http://spiderwebsoftware.com/productsOld.html).

[^2]: Although some of the executables are FAT, these patches only change the 68k parts.

