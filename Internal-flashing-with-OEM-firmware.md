# Internal flashing with OEM firmware

We need two things:
 * A copy of the original firmware
 * A flashing tool

The flashing tool is often provided as a Windows executable and it's usually available on the vendor's website. Alternatively, the OEM BIOS may provide a way to flash a new firmware.

The original firmware can be obtained in two ways:
 * Dumped from the PC with the flashing tool (as they often offer a way to "backup" the current firmware)
 * Downloaded from the vendor's website

However the firmware on the vendor's website is usually a generic firmware, which doesn't contain things like MAC address of your integrated ethernet controller and serial code.

## Check the format of the original firmware

_me_cleaner_ needs a plain dump of the firmware (1:1 copy of the flash content, often called _.bin_ or _.rom_ files), while the vendor's tools sometimes work in different formats. To check whether the obtained firmware is in the correct format you can use coreboot's `ifdtool`.

     $ git clone --depth=1 https://review.coreboot.org/p/coreboot
     $ cd coreboot/util/ifdtool
     $ make

Now check if the firmware has the correct structure:

     $ ifdtool -d original_firmware.bin

It should print something like [this](https://gist.github.com/corna/66322fb938dedd93d2aaa1d59b27341d).

If not it means that either the firmware is corrupted or it's not in the "plain" format. The vendors sometimes offer  a way to convert their firmware to the standard "plain" format, check their website or Google it.

Sometimes `ifdtool` prints a resonable output even in case of an invalid image (like [this](https://gist.github.com/corna/e7d08c23049a325ef3222ac765d1c5cf)); to exclude these cases you should check at least that:
 * The region sizes in the [`FLREGn` section](https://gist.github.com/corna/66322fb938dedd93d2aaa1d59b27341d#file-gistfile1-txt-L147-L157) make sense (4 KiB for the descriptor, some MiB for the BIOS, some MiB for the ME, some KiB for the GbE, if present)
 * Each region in [`FLMSTRn` section](https://gist.github.com/corna/66322fb938dedd93d2aaa1d59b27341d#file-gistfile1-txt-L197-L235) has RW access at least to itself

## Check the validity of the original firmware

Ok, now we have a valid firmware in the correct format, it's time to check if it contains a ME image. Run

     $ python me_cleaner.py -c original_dump.bin

If it passes all the checks like [this one](https://gist.github.com/corna/92df16e65248c63a258fdbdac5cb0923) it means that the firmware is in the correct format and it contains a valid ME image, and we can now proceed.

## Neutralize Intel ME

Now just use me_cleaner on the firmware:

     $ python me_cleaner.py -S -O modified_firmware.bin original_firmware.bin

This will create a modified_firmware.bin file which contains the untouched OEM BIOS, a modified ME image and a flash descriptor with the HAP/AltMeDisable bit set.

## Flash the modified image

Until now every step was safe, as no modification has been done on your hardware, now comes the dangerous part.

**You do have a recovery solution, [don't you](https://github.com/corna/me_cleaner/wiki/How-to-apply-me_cleaner)?**

If so, use the OEM flashing tool to flash the modified firmware, power off the PC, remove any power source (batteries included) for a couple of seconds and power it on.

## It works!!

Great! If you want to check the status of Intel ME you can download `intelmetool` from coreboot and run it:

     $ cd coreboot/util/intelmetool
     $ make
     # ./intelmetool -m

The relevant lines are

     ME: Error Code              : Image Failure

and (on pre-Skylake platforms)

     ME: Progress Phase State    : M0 kernel load

For example, on my Lenovo X220t with coreboot and a shrinked and deblobbed ME, [this](https://gist.github.com/corna/d637a7c3279f41e9be65b43b673d54d3) is the output.

If it shows an error like

     Error mapping physical memory 0x..... [0x4000] ERRNO=1 Operation not permitted
     Could not map MEI PCI device memory

you just have to [run your kernel with the `iomem=relaxed` option](https://github.com/corna/me_cleaner/issues/30#issuecomment-301193328).

##  It doesn't work...

Don't panic! Just use an external programmer (or any recovery solution you have) to flash back the original:
everything should be now exactly as before, and the system should boot again. If you really want to have a deblobbed Intel ME firmware you can take a look at [this page](https://github.com/corna/me_cleaner/wiki/HAP-AltMeDisable-bit) and, if nothing works, open an issue on GitHub and look for possible solutions.