# External flashing

The prerequisites for the external flashing are:
 * A Linux board with a SPI interface (Raspberry Pi, Beaglebone, C.H.I.P, ...) or a SPI programmer
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

Download flashrom and build it (see [here](https://www.flashrom.org/Downloads) for more info):

     $ wget https://download.flashrom.org/releases/flashrom-0.9.9.tar.bz2
     $ tar -xf flashrom-0.9.9.tar.bz2
     $ cd flashrom-0.9.9/
     $ make

Turn off the PC, disconnect it from the power supply and, if it has a removable battery, remove it. Locate the ROM chip on your board and connect it to the programmer; don't forget to pullup the /WP and /HOLD lines (especially if you've detached the chip from its socket).

Now it's time to get the dump of your original firmware (change the programmer/path and the spispeed parameter accordingly):

     $ flashrom -p linux_spi:dev=/dev/spidev0.0,spispeed=10000 -c <CHIP MODEL> -r original_dump.bin

Do it twice, and check that they're identical:

     $ flashrom -p linux_spi:dev=/dev/spidev0.0,spispeed=10000 -c <CHIP MODEL> -r original_dump_2.bin
     $ diff original_dump.bin original_dump_2.bin

If they're the same you can proceed, otherwise you should check if you've connected the chip correctly and you can try to reduce the length of the cables.

Just to be extra sure, you can also check if the dumped binary is correct. Download and build ifdtool from coreboot:

     $ git clone --depth=1 http://review.coreboot.org/p/coreboot
     $ cd coreboot/util/ifdtool
     $ make

Now check if the dumped image has the correct structure:

     $ ifdtool -d original_dump.bin

It should print something like [this](https://gist.github.com/corna/66322fb938dedd93d2aaa1d59b27341d).

Now to check if the dumped ME image is valid just run:

     $ python me_cleaner.py -c original_dump.bin

It should pass all the checks like [this one](https://gist.github.com/corna/92df16e65248c63a258fdbdac5cb0923).

Save the dump somewhere safe, in case something goes wrong.

## Neutralize Intel ME

If you only want to neutralize Intel ME you can just use me_cleaner on it:

     $ python me_cleaner.py -O modified_image.bin original_dump.bin

This will create an image with a stripped ME.

## ME neutralization and shrinking

If you want to recover the space freed by me_cleaner (for example, if you use coreboot and you want that extra space on the ROM chip) you can shrink the ME image with the following commands:

     $ ifdtool -f layout.txt original_dump.bin
     $ python me_cleaner.py -O modified_shrinked_image.bin -r original_dump.bin

me_cleaner should print some output; copy the new ME region line

     The ME region can be reduced up to:
      00003000:00017fff me

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
<b>00018000</b>:007fffff bios
00003000:<b>00017fff</b> me
00001000:00002fff gbe
</pre>

Note that I've changed both the ME and the BIOS regions. Now run

     $ ifdtool -n layout.txt modified_shrinked_image.bin

modified_shrinked_image.bin.new contains now a valid ME image which occupies much less space than before (for example, for ME version 7, the ME region size went from 5 MB to 84 kB). You can now extract the descriptor and the ME image from modified_shrinked_image.bin.new and use them for the next build of coreboot (IFD_BIN_PATH and ME_BIN_PATH)

     $ ifdtool -x modified_shrinked_image.bin.new

Don't forget to change CONFIG_CBFS_SIZE to increase the size of cbfs. Rebuild coreboot with the new options and files and you're ready to go.

## Flash back the modified image

Now that we have a stripped image we can flash it back (and hope that everything works well). To flash it just connect again the programmer to the chip and run the same command used during the dump, just changing `-r` with `-w` and selecting the correct image:

     $ flashrom -p linux_spi:dev=/dev/spidev0.0,spispeed=10000 -c <CHIP MODEL> -w modified_image.bin

Detach the programmer, put the chip back into the system (if it is socketed) and power on the PC.

## It works!!

Great! If you want to check the status of Intel ME you can download `intelmetool` from coreboot and run it (note that, on kernels >= 4.9 you might need the `iomem=relaxed` option):

     $ cd coreboot/util/intelmetool
     $ make
     # ./intelmetool -s

The relevant lines are

     ME: Error Code              : Image Failure

and (on pre-Skylake platforms)

     ME: Progress Phase State    : M0 kernel load

For example, on my Lenovo X220t with coreboot and a shrinked and deblobbed ME, [this](https://gist.github.com/corna/d637a7c3279f41e9be65b43b673d54d3) is the output.

## It doesn't work...

Don't panic! Since you're using an external programmer you can easily rollback the modifications; power off the PC, connect back the flash chip to the programmer and run.

     $ flashrom -p linux_spi:dev=/dev/spidev0.0,spispeed=10000 -c <CHIP MODEL> -w original_dump.bin

Everything should be now exactly as before, and the system should boot again. If you really want to have a deblobbed Intel ME firmware you can open an issue on GitHub and look for possible solutions.