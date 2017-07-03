# Internal flashing with coreboot

Since you're using coreboot this procedure is much easier than the non-coreboot case.
First get a copy of your current firmware (and save it somewhere safe, in case something goes wrong) and check the ME region protections with ifdtool; if you have a laptop run

     $ flashrom -p internal:laptop=force_I_want_a_brick -c <CHIP MODEL> -r dump.bin

otherwise run

     $ flashrom -p internal -c <CHIP MODEL> -r dump.bin

Now run `ifdtool -d dump.bin` and check the section

     FLMSTR1:   0x........ (Host CPU/BIOS)

If the lines `Intel ME Region Write Access` and `Intel ME Region Read Access` are `disabled` you're out of luck, you have to use an external programmer to apply me_cleaner (or you can directly unlock the read/write access to the ME region with `ifdtool -u dump.bin`, but you still need an external programmer to flash the new permissions). Otherwise, if you have read/write access to the ME regions, you can proceed with this guide.

## Neutralize Intel ME

If you don't need to recover the space freed by me_cleaner just change the coreboot build option `Strip down the Intel ME/TXE firmware` (`CONFIG_USE_ME_CLEANER`) to `y`, rebuild coreboot and flash it with the usual methods.

## ME neutralization and shrinking

If you instead want to recover the extra ROM space (which is a considerable amount of space, ~1 MB or ~5 MB, depending on the firmware type):

     $ ifdtool -f layout.txt original_dump.bin
     $ python me_cleaner.py -O modified_shrinked_image.bin -r original_dump.bin

me_cleaner should print some output; copy the new ME region line

     The ME region can be reduced up to:
      00003000:00019fff me

into the file `layout.txt`, so that it changes from something like this

<pre>
00000000:00000fff fd
<b>00500000</b>:007fffff bios
00003000:<b>004fffff</b> me
00001000:00002fff gbe
</pre>

to something like this

<pre>
00000000:00000fff fd
<b>00020000</b>:007fffff bios
00003000:<b>00019fff</b> me
00001000:00002fff gbe
</pre>

This correspond to moving from this layout

![before](http://oi65.tinypic.com/10rn12d.jpg)

to this one

![after](http://oi67.tinypic.com/2nkrkoi.jpg)

Note that I've changed both the ME and the BIOS regions, as the ending address of the ME region has changed (from 0x4fffff to 0x1ffff), but also the starting address of the BIOS region has changed (from 0x500000 to 0x20000, the byte after the end of the ME region).

Now run

     $ ifdtool -n layout.txt modified_shrinked_image.bin

modified_shrinked_image.bin.new contains now a valid ME image which occupies much less space than before (for example, for ME version 7, the ME region size went from ~5 MB to ~84 kB). You can now extract the descriptor and the ME image from modified_shrinked_image.bin.new and use them for the next build of coreboot (IFD_BIN_PATH and ME_BIN_PATH)

     $ ifdtool -x modified_shrinked_image.bin.new

Don't forget to change CONFIG_CBFS_SIZE to increase the size of cbfs. Rebuild coreboot with the new options and files and you're ready to go.

##  It works!!

Great! If you want to check the status of Intel ME you can use `intelmetool` in coreboot/util/intelmetool:

     $ cd coreboot/util/intelmetool
     $ make
     # ./intelmetool -s

The relevant lines are

     ME: Error Code              : Image Failure

and (on pre-Skylake platforms)

     ME: Progress Phase State    : M0 kernel load

If it shows an error like

     Error mapping physical memory 0x..... [0x4000] ERRNO=1 Operation not permitted
     Could not map MEI PCI device memory

you just have to [run your kernel with the `iomem=relaxed` option](https://github.com/corna/me_cleaner/issues/30#issuecomment-301193328).

##  It doesn't work...

Don't panic! Just use an external programmer to flash back a working firmware:

     $ flashrom -p linux_spi:dev=/dev/spidev0.0,spispeed=10000 -c <CHIP MODEL> -w dump.bin

Everything should be now exactly as before, and the system should boot again. If you really want to have a deblobbed Intel ME firmware you can open an issue on GitHub and look for possible solutions.