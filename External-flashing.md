# External flashing

The prerequisites for the external flashing are:
 * A Linux board with a SPI interface (Raspberry Pi, Beaglebone, C.H.I.P, ...) or a SPI programmer
   * Alternatively you can use a USB SPI programmer, like the CH341A or the FT232H/FT2232H/FT4232H: in that case, replace the `-p linux_spi,...` option in _flashrom_ with the corresponding driver (see [this page](https://www.flashrom.org/Supported_hardware#USB_Devices) for more info).
   * You could also use an Arduino, but it's not covered bu this guide and it's definetly harder
 * A way to connect the ROM chip with the board (DIP/SOIC clip, SMD clips, socket in case of a detachable chip)
 * Good knowledge of the SPI chips and programmers

Usually the ROM chips are in the following packages:
 * SOIC
 * DIP
 * PLCC
 * WSON

The first two are easy to manage, as you can buy for cheap a SOIC clip, a DIP clip or some SMD clips (if you want something more versatile). The other two are more complex, and you have to find a solution to connect your chip to the programmer.

If the chip is socketed (which happens quite often in the desktop boards), you can buy an identical chip and keep the original one as "fallback", in order to have a backup solution in case something goes wrong.

## Dump of the original firmware

If you have flashrom in your distribution's repositories you can download it from there (for example on Raspbian, you can just get it with `sudo apt-get install flashrom`, otherwise you can download and build it (see [here](https://www.flashrom.org/Downloads) for more info):

     $ wget https://download.flashrom.org/releases/flashrom-0.9.9.tar.bz2
     $ tar -xf flashrom-0.9.9.tar.bz2
     $ cd flashrom-0.9.9/
     $ make
     $ sudo make install

Turn off the PC, disconnect it from the power supply and, if it has a removable battery, remove it. Locate the ROM chip on your board and connect it to the programmer; don't forget to pullup the /WP and /HOLD lines (especially if you've detached the chip from its socket).

Now it's time to get the dump of your original firmware (change the programmer/path and the spispeed parameter accordingly):

     $ flashrom -p linux_spi:dev=/dev/spidev0.0,spispeed=10000 -c <CHIP MODEL> -r original_dump.bin

Do it twice, and check that they're identical:

     $ flashrom -p linux_spi:dev=/dev/spidev0.0,spispeed=10000 -c <CHIP MODEL> -r original_dump_2.bin
     $ diff original_dump.bin original_dump_2.bin

If they're the same you can proceed, otherwise you should check if you've connected the chip correctly and you can try to reduce the length of the cables.

Just to be extra sure, you can also check if the dumped binary is correct. Download and build ifdtool from coreboot:

     $ git clone --depth=1 https://review.coreboot.org/p/coreboot
     $ cd coreboot/util/ifdtool
     $ make

Now check if the dumped image has the correct structure:

     $ ./ifdtool -d original_dump.bin

It should print something like [this](https://gist.github.com/corna/66322fb938dedd93d2aaa1d59b27341d).

Sometimes `ifdtool` prints a resonable output even in case of an invalid image (like [this](https://gist.github.com/corna/e7d08c23049a325ef3222ac765d1c5cf)); to exclude these cases you should check at least that:
 * The region sizes in the [`FLREGn` section](https://gist.github.com/corna/66322fb938dedd93d2aaa1d59b27341d#file-gistfile1-txt-L147-L157) make sense (4 KiB for the descriptor, some MiB for the BIOS, some MiB for the ME, some KiB for the GbE, if present)
 * Each region in [`FLMSTRn` section](https://gist.github.com/corna/66322fb938dedd93d2aaa1d59b27341d#file-gistfile1-txt-L197-L235) has RW access at least to itself

Now to check if the dumped ME image is valid just run:

     $ python me_cleaner.py -c original_dump.bin

It should pass all the checks like [this one](https://gist.github.com/corna/92df16e65248c63a258fdbdac5cb0923).

Save the dump somewhere safe, in case something goes wrong.

## Neutralize Intel ME

If you only want to neutralize Intel ME you can just use me_cleaner on it:

     $ python me_cleaner.py -S -O modified_image.bin original_dump.bin

This will create an image with a stripped ME and the HAP/AltMeDisable bit set.

## Neutralize and shrink Intel ME (useful only for coreboot)

If you instead want to recover the extra ROM space (which is a considerable amount of space, ~1 MB or ~5 MB, depending on the firmware type):

     $ python me_cleaner.py -S -r -t -d -O out.bin -D ifd_shrinked.bin -M me_shrinked.bin original_dump.bin

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

## Flash back the modified image

Now that we have a stripped image we can flash it back (and hope that everything works well). To flash it just connect again the programmer to the chip and run the same command used during the dump, just changing `-r` with `-w` and selecting the correct image:

     $ flashrom -p linux_spi:dev=/dev/spidev0.0,spispeed=10000 -c <CHIP MODEL> -w modified_image.bin

Detach the programmer, put the chip back into the system (if it is socketed) and power on the PC.

## It works!!

Great! If you want to check the status of Intel ME you can see [this](https://github.com/corna/me_cleaner/wiki/Get-the-status-of-Intel-ME) guide.

## It doesn't work...

Don't panic! Since you're using an external programmer you can easily rollback the modifications; power off the PC, connect back the flash chip to the programmer and run.

     $ flashrom -p linux_spi:dev=/dev/spidev0.0,spispeed=10000 -c <CHIP MODEL> -w original_dump.bin

Everything should be now exactly as before, and the system should boot again. If you really want to have a deblobbed Intel ME firmware you can take a look at [this page](https://github.com/corna/me_cleaner/wiki/HAP-AltMeDisable-bit) and, if nothing works, open an issue on GitHub and look for possible solutions.