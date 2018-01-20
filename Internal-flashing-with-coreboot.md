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

## Neutralize and shrink Intel ME

If you instead want to recover the extra ROM space (which is a considerable amount of space, ~1 MB or ~5 MB, depending on the firmware type):

     $ python me_cleaner.py -r -t -d -O out.bin -D ifd_shrinked.bin -M me_shrinked.bin original_dump.bin

me_cleaner should print some output; note the lines

```
Modifying the regions of the extracted descriptor...
 00003000:004fffff me   --> 00003000:0001ffff me
 00500000:007fffff bios --> 00020000:007fffff bios
```

This means that _me_cleaner_ has modified the original descriptor with this layout

![before](http://oi65.tinypic.com/10rn12d.jpg)

to this new one, where the BIOS region is 4.9 MB bigger than before.

![after](http://oi67.tinypic.com/2nkrkoi.jpg)

Three files should have been generated:
 * `out.bin`, a flashable full image with the original layout (which we don't need)
 * `ifd_shrinked.bin`, an Intel Flash Descriptor with the new layout
 * `me_shrinked.bin`, a modified Intel ME image, truncated to fit in the ME region of the new layout

Rebuild coreboot selecting `ifd_shrinked.bin` as `IFD_BIN_PATH`, `me_shrinked.bin` as `ME_BIN_PATH` and increasing `CBFS_SIZE` accordingly and flash the resulting image.

##  It works!!

Great! If you want to check the status of Intel ME you can use `intelmetool` in coreboot/util/intelmetool:

     $ cd coreboot/util/intelmetool
     $ make
     # ./intelmetool -m

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

Everything should be now exactly as before, and the system should boot again. If you really want to have a deblobbed Intel ME firmware you can take a look at [this page](https://github.com/corna/me_cleaner/wiki/HAP-AltMeDisable-bit) and, if nothing works, open an issue on GitHub and look for possible solutions.