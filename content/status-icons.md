Title:  PC Status Icons
Date:   2020-08-07 16:16:47
Categories: exile patches

Motivation
----------------

There are many effects in _Exile_ which modify the PCs status. For example, a PC
could be `slowed`, `hasted`, `blessed`, `cursed`, or `poisoned`. These effects
are indicated by the presence of small icons next to the PC's name.

![PC status area of game screen]({static}/exile/staticons/orig-staticons-w3.png){: style="width:100%;" }

The `poisoned` icon takes two different forms depending on the level of
poisoning. The other effects icons only have one form regardless of intensity.
It would be useful if the other effects icons also varied with the strength of
their effects. For example, when a PC is `diseased`, it would be helpful to know
the severity of their illness. Should you cast an expensive `Cleanse` to cure
them, or would a less costly `Cure Disease` suffice?

Remedy
----------------

Make all status icons vary with intensity.

Method
----------------

We will create new icons that indicate the level of their effects using pie
charts like this:

![Status icons with pie charts in background]({static}/exile/staticons/staticons.png){: style="width:100%;" }

We will append the new icons to the graphics file. And we will patch the code
that draws the status icons to use the appropriate new icons.

### Patches for Windows Versions[^1]

| Patched Executable | Version | Patch File           | Modified Graphics        |
| :----------------- | :------ | :------------------- | :----------------------- |
| [BLADES.EXE][wbe]  | v1.0.1  | [BoE patch][wbp]     | [MIXED.BMP][wbg]         |
| [EXILE3.EXE][w3e]  | v1.0b   | [Exile 3 patch][w3p] | [MIXED.BMP][w3g]         |
| [EXILE2.EXE][w2e]  | v2.0.1  | [Exile 2 patch][w2p] | [MIXED.BMP][w2g]         |
| EXILE.EXE (tbd)    | v2.0    | tbd                  | tbd                      |

[w2g]: /assets/exile/staticons/w2/MIXED.BMP
[w3g]: /assets/exile/staticons/w3/MIXED.BMP
[wbg]: /assets/exile/staticons/wb/MIXED.BMP
[w2e]: /assets/exile/staticons/w2/EXILE2.EXE
[w3e]: /assets/exile/staticons/w3/EXILE3.EXE
[wbe]: /assets/exile/staticons/wb/BLADES.EXE
[w2p]: /assets/exile/staticons/w2-staticons-patch.txt
[w3p]: /assets/exile/staticons/w3-staticons-patch.txt
[wbp]: /assets/exile/staticons/wb-staticons-patch.txt

### Simple Install

Replace the original executable and graphics file with the patched executable
and modified graphics files from the table above. For example, replace `EXILE3.EXE`
and `MIXED.BMP` files in the original `EXILE3` directory with the new versions
in the links above.

### Custom Install

If your copy of the executable already contains other patches or modifications
that you want to keep, you can install just the status icons patch onto your
executable. The patch files are in hex dump text format. For example, to install
the Exile 3 patch file onto your executable at `C/EXILE3/EXILE3.EXE` using
`xxd`:

``` bash
xxd -r staticons-patch-w3.txt C/EXILE3/EXILE3.EXE
```

You will also need to either install the modified graphics file, or add the new
icons in the modified graphics file into your existing graphics file. If the x,y
positions of the new icons change, you will need to update these positions in
the patch file to match. The offsets of the icons are given in the first few
lines of each patch file.

### Patches for Macintosh Versions

TBD

Discussion
-----------------

Here is an example of the new status icons in _Exile 2_. I never knew that the
`Invisibility` spell gave 8 levels of `Magic Resistance` and eight levels of
`Hidden` for a cost of only two spell points.

![PC status area of Exile 2]({static}/exile/staticons/new-staticons-w2.png){: style="width:100%;" }

And here is an example of the new status icons in _Exile 3_.

![PC status area of Exile 3]({static}/exile/staticons/new-staticons-w3.png){: style="width:100%;" }

Above, we can see that Jenneke and Thissa are heavily `cursed` while Michael is
only slightly `cursed`. Adrianna is almost fully `hasted`, and Frrrrrr is
completely `webbed` and will get no action points on the next round.

Contributing
-----------------

I encourage you to customize the new status icons to suite your taste and needs,
and to share your creations. Suggestions for improvements are welcome.

Conclusion
-----------------

Now we can answer questions such as whether three `Minor Haste`s are more or
less effective than one regular `Haste`, and how many `Envenom`s does it
take to max out the `poisoned weapon` effect[^2].

Happy Adventuring!

-----------------

Footnotes:

[^1]: These are the free registered versions available for download from
      [Spiderweb Software](http://spiderwebsoftware.com/productsOld.html).

[^2]: This is a trick question. `Envenom` does not stack.
