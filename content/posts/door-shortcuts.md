Title:  BoE Locked Door Keyboard Shortcuts
Date:   2020-07-15 16:16:47
Category: Exile

Motivation
----------------

![Blades of Exile "Door is Locked" dialog box]({static}/images/locked_door.png){: style="width:100%;"}
{: style="float:right;padding-left:5px;width:66%;" }
In both *Exile 3* and *Blades of Exile*, walking into a locked door triggers a
dialog that presents `Bash Door`, `Pick Lock`, and `Leave` buttons.

In *Exile 3*, the desired action can quickly be selected from the keyboard with
`b` and `p` shortcut keys. However in *Blades of Exile*, these handy shortcuts
are inexplicably missing, and opening locked doors requires reaching for the
mouse to click the dialog buttons.


Remedy
----------------

Make locked door dialogs in *Blades of Exile* respond to `b` and `p` key presses the
way *Exile 3* does.

Method
----------------

In order to restore the keyboard shortcuts in *Blades of Exile*, we simply insert
the ascii bytes `b` and `p` at the appropriate location in the executable[^1].

### Windows Version

Patching the Windows version of *Blades of Exile v1.0.1* requires modifying two
bytes in the file *BLADES.EXE*. The shortcut keys for the `Bash Door` and `Pick
Lock` buttons are at offset `0x72912` and `0x72913`. Use a hex editor to replace
the two `0x00` bytes at those offsets with the two bytes `0x62` and `0x70`
(ascii for `b` and `p`). Alternatively, use `xxd` or `dd` as a command line hex
editor. For example:

~~~bash
echo -n bp |xxd -o 0x72912 |xxd -r -c 2 '-' BLADES.EXE
-or-
echo -n bp |dd iflag=count_bytes oflag=seek_bytes count=2 seek=$((0x72912)) conv=notrunc of=BLADES.EXE
~~~

### Macintosh Version (68k[^2])

Patching the Mac version is a bit more complicated because the data are in a
compressed format within the *Blades of Exile (fat) v1.0.2* file. For the 68k
executable, the resource of type `DATA` with id `0` must be modified using a
resource editor[^3] such as *ResEdit*. Two changes are required:

1. at offset `0x0003`, the byte `0x7b` must be changed to `0x7f` to account
   for the increased size of the resource in step 2.
2. at offset `0x777b`, the byte `0x4c` must be replaced with the five bytes
   `0x46 0x81 0x62 0x70 0x43` This will cause everything after `0x777b` to
   shift forward increasing the size of the resource by four bytes.

Conclusion
----------------

Now we can open doors without reaching for the mouse.

Happy Adventuring!

----------------
Footnotes:

[^1]: These are the free registered versions available for download from
      [Spiderweb Software](http://spiderwebsoftware.com/productsOld.html).

[^2]: Although the executable is FAT, this patch only changes the 68k part.
      Patching the PPC part requires similar shenanigans to the data fork.
      I don't currently have any way to run PPC code, so I won't speculate on
      the procedure.

[^3]: Be sure to use a resource-aware editor and not a simple hex editor when
      modifying resources.
