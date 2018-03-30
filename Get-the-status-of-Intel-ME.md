Once you've installed your modified Intel ME firmware you may want to verify if everything went as expected.
There are various ways to accomplish this:

## intelmetool

The easiest way is to use `intelmetool`, a tool able to get the current status of Intel ME. If not already done, clone the `coreboot` repository:

     $ git clone --depth=1 https://review.coreboot.org/p/coreboot

and run:

     $ cd coreboot/util/intelmetool
     $ make
     # ./intelmetool -m

The output depends on several factors (`-s`/`-S` flag, ME version, ...), see these these two references ([without options](https://gist.github.com/corna/d637a7c3279f41e9be65b43b673d54d3) and [with `-S`](https://gist.github.com/corna/6d8a24fdaca1afd0ae2a84ecde4573dd)).

The output of `intelmetool` should match the content of this table:

| `intelmetool` line       | Stock firmware                   | `me_cleaner` with `-S` or `-s` | `me_cleaner` with no options |
|--------------------------|----------------------------------|--------------------------------|------------------------------|
| `FW Partition Table`     | `OK`                             | `OK`                           | `OK`                         |
| `Firmware Init Complete` | `YES`                            | `NO`                           | `NO`                         |
| `Current Working State`  | `Normal`                         | (*)                            | (*)                          |
| `Error Code`             | `No Error`                       | `No Error`                     | `Image Failure`              |
| `Progress Phase`         | `Host Communication`             | `BUP Phase`                    | `BUP Phase`                  |
| `Progress Phase State`   | `Host communication established` | `[...] straps say ME DISABLED` | `M0 kernel load`             |

(*) anything but `Platform Disable Wait`

If it shows an error like

     Error mapping physical memory 0x..... [0x4000] ERRNO=1 Operation not permitted
     Could not map MEI PCI device memory

you just have to [run your kernel with the `iomem=relaxed` option](https://github.com/corna/me_cleaner/issues/30#issuecomment-301193328).

## `me_cleaner -c`

You can also dump again the Intel ME firmware again after the successful boot and run `me_cleaner.py -c` on it; the output should be similar to this one:

<pre>
Full image detected
The ME/TXE region goes from 0x3000 to 0x500000
Found FPT header at 0x3010
<b>Found 1 partition(s)</b>
Found FTPR header: FTPR partition spans from 0xcc000 to 0x142000
ME/TXE firmware version 7.1.86.1221
Public key match: Intel ME, firmware versions 7.x.x.x, 8.x.x.x
<b>The AltMeDisable bit is SET</b>
Checking the FTPR RSA signature... VALID
</pre>

Note the two bold lines, which means respectively that only a partition is left (`-S` or no options) and that the HAP/AltMeDisable bit is set (`-S`/`-s` options).