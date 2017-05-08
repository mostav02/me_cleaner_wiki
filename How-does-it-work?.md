Since me_cleaner modifies a very fundamental part of an Intel system, it is reasonable to be worried about such modification. Here I'm explaining what this script does, and how.

## Sources

The basic informations about Intel ME firmwares were provided by:
 * [me.bios.info](http://me.bios.io/ME_blob_format)
 * [Igor Skochinsky - Rootkit in your laptop - Breakpoint 2012](http://me.bios.io/images/c/ca/Rootkit_in_your_laptop.pdf)
 * [Igor Skochinsky - Intel ME Secrets - RECON 2014](https://recon.cx/2014/slides/Recon%202014%20Skochinsky.pdf)
 * [me-tools](https://github.com/skochinsky/me-tools)
 * [MEAnalyzer](https://github.com/platomav/MEAnalyzer)

Our work started when we found, on the coreboot Mailing List, a message from Trammel Hudson in which he shared his [recent discovers about Intel ME firmwares](https://www.coreboot.org/pipermail/coreboot/2016-September/082016.html).
Then we tried by ourself and replicated the results: to make things simpler I created a Python script to modify the firmware automatically and we published it on the [coreboot Mailing List](https://www.coreboot.org/pipermail/coreboot/2016-November/082331.html) and later on [GitHub](https://github.com/corna/me_cleaner).

## What does this tool do?

As you can see in [me.bios.info](http://me.bios.io/ME_blob_format), the firmware is divided in partitions and every partition can contain one or more modules. At the beginning of the image there is the [FPT](http://me.bios.io/ME_blob_format#.24FPT_Partition_table_header) (Firmware Partition Table), a region where all the partitions are indexed. The FPT region is **NOT** signed and can be freely modified (but it has a checksum, offset 0x1b).

[Each partition](http://me.bios.io/ME_blob_format#.24FPT_Partition_table_header) is individually signed, therefore all the non-fundamental partitions can be removed without breaking anything (as we are removing the partitions, but also the signatures).

It seems that the only fundamental partition is the FTPR (Factory Partition), as every other partition (with the respective entry in the FPT) can be removed. The firmware still contains some code, but:
 * The network stack (partition NFTP) has been removed
 * The PAVP (partition MDMV, module JCOM) has been removed
 * And more...

Based on the results on Trammel Hudson (again, on the coreboot ML) I improved me_cleaner by removing also the LZMA-compressed modules in the FTPR partition (more or less half of the modules in the partition). This also removes the Intel Antitheft module (module TDT), and leaves only 4-5 Huffman-compressed modules. According to [Igor's 2014 presentation, slide 17](https://recon.cx/2014/slides/Recon%202014%20Skochinsky.pdf), these modules do the basic ME initialization:
 * ROMP (not always present)
 * BUP - Bringup (hardware initialization/configuration)
 * KERNEL - Scheduler, low-level APIs for other modules
 * POLICY - Secondary init tasks, some high-level APIs
 * FTCS

After a while I updated me_cleaner to remove also most of the Huffman-compressed modules, leaving only:
 * ROMP (not always present)
 * BUP - Bringup (hardware initialization/configuration)

Since the TXE firmwares are very similar to the ME ones, I also adapted me_cleaner to work on them.

A month later I finally realized how to move the partitions; in this way the empty space before the FTPR partition can be recovered, and the final size of the Intel ME image is ~80 kB.

## How can I be sure that all the bad stuff is not in a ROM inside the CPU?

We can never be sure about this, but, thanks to the [Igor's 2014 presentation](https://recon.cx/2014/slides/Recon%202014%20Skochinsky.pdf) (slide 18), we can suppose that the code inside the internal ROM is just the code responsible for the very basic initialization of Intel ME.

## How does it work?

Since there isn't any official documentation of the Intel ME firmwares, a short explanation of how me_cleaner works is mandatory.
At first it detects if the image is a "pure" ME image (magic **$FPT** at offset 0x10) or a full BIOS dump (magic **5a a5 f0 0f** at offset 0x10). If it is a "pure" ME image it uses the whole file as ME image, otherwise it finds the ME region by reading the values inside the Intel Flash Descriptor.

Then it searches for the basic ME informations: presence of the FPT region, version of the firmware, number of partitions and offset and size of the FTPR partition. Now it has all the informations it needs, and it can start the "cleaning" process by:
 * dumping the original FTPR entry in the FPT
 * filling the whole image with 0xff, except for the first 0x30 bytes (the start of the FPT) and the FTPR partition
 * restoring the FTPR entry in the FPT
 * removing the EFFS presence flag
 * correcting the FPT checksum

Now the ME image contains only the FTPR partition, as every other partition has been overwritten by 0xff.

For pre-Skylake images, the internal structure of the partitions is known, and also most of the modules can be removed.

The LZMA modules are placed after the Huffman data (after the [LLUT](http://me.bios.io/ME_blob_format#LLUT_Breakdown)) and their positions are clearly saved inside the manifests, so they can easily be removed.

The Huffman modules are more tricky to remove, as the module manifests does not contain directly an offset. The Huffman modules are indexed by "chunks": each "chunk" represent a fixed-size block (usually 1024 bytes of uncompressed data) and it contains an offset to the Huffman stream and a flag. Each module manifest contains the base loading address of that module (that sets the starting chunk, `first_chunk = (module_base - overall_base) / chunk_size`) and the uncompressed module size (that sets the number of chunks, `last_chunk = first_chunk + module_size / chunk_size`). Once we have the chunk indexes we can construct a whitelist of unremovable blocks and remove the others.

Moving a partition is not straightforward, as additional steps are needed:
 * correct the partition's FPT offset ([bytes 0x8:0xc](http://me.bios.io/ME_blob_format#Partition_table_entry))
 * fix the LLUT absolute offset (bytes 0xc:0x10 of the LLUT, minus bytes 0x9:0xb)
 * adjust the absolute Huffman offset in the LLUT ([bytes 0x14:0x18](http://me.bios.io/ME_blob_format#LLUT_Breakdown))
 * correct the offset of each Huffman chunk

## Why does it work? Aren't the partitions signed? How can you modify them?

This is the simplified structure of a partition header:

![Partition structure](http://oi64.tinypic.com/bhlkdg.jpg)

As you can see the partition is not signed directly, instead all the modules are hashed and the modules manifests (that contain the hashes) are signed. This means that modify a module doesn't invalidate the signature, but only its hash.

Luckily for us, Intel ME doesn't check all the hashes at once, but only when it needs to execute them. Moreover the 30-min watchdog that forcefully shuts off the system is disabled in the BUP module.

Therefore Intel ME:
 * checks the partition signature: valid
 * finds the BUP module and checks its hash: valid
 * executes BUP
   * at this point the system can boot correctly, without the 30-min window
 * finds the following module (probably KERNEL) and checks its hash: **INVALID**
 * stops the execution

## But I still don't trust you

You're right, you should never trust a random guy on the Internet, but you don't have to trust me, you can check the [code](https://github.com/corna/me_cleaner/blob/master/me_cleaner.py) by yourself.

And, by the way
 * most of the `me_cleaner` code is "find _x_ and overwrite it with 0xff".
 * Intel ME doesn't accept unsigned code and, unfortunately, I don't own the Intel ME key

## Cool, how can I apply it?

Before flashing the modified image you should understand the implications of such modification.
First you should understand that this tool does not reimplement **anything**, it only wipes parts of a basic component of your processor, so keep in mind:
 * Currently `me_cleaner` **DOES NOT** work on platforms with Intel Boot Guard in `verified boot` mode, see [here](https://github.com/corna/me_cleaner/issues/6)
 * Even if this tool has been tested with your system, it does not mean that this modification is safe, so be prepared for a brick!
 * Intel ME doesn't only provide some services (that you may or may not use), but it also does low-level stuff (like silicon workaround, thermal management, fan control...). Luckily, no user has reported any side effect so far (as many of these features aren't used anymore or they are implemented by something else).
 * Often the ME region is not writeable by software: in these cases you need an external programmer to write the modified firmware.

Even if it sounds dangerous, once you have a valid backup of your ROM and a way to reprogram it (external flasher, dual BIOS...), you should be safe.

## Ok, I'm not scared, I want to try it!

Great! You can follow [this](https://github.com/corna/me_cleaner/wiki/How-to-apply-me_cleaner) guide if you want to try it.

Please report the success on [#3](https://github.com/corna/me_cleaner/issues/3) (even if it's already listed in the [status page](https://github.com/corna/me_cleaner/wiki/me_cleaner-status)).

## I've applied me_cleaner and my PC still works well: how can I check the status of Intel ME?

You can use `intelmetool` from the coreboot repository:

     $ git clone --depth=1 http://review.coreboot.org/p/coreboot
     $ cd coreboot/util/intelmetool
     $ make
     $ sudo ./intelmetool -s

On my platform (Thinkpad X220T with coreboot) it shows [this](https://gist.github.com/corna/d637a7c3279f41e9be65b43b673d54d3).

The interesting lines are (in Skylake and newer some lines can differ, as less code is removed):
 * `ME: FW Partition Table : OK` → the checksum of the FPT is correct
 * `ME: Firmware Init Complete : NO` → the firmware init hasn't been completed
 * `ME: Current Working State : Recovery` → anything but `Platform Disable Wait` (which means that the device will forcefully turn off in 30 mins) is OK here
 * `ME: Error Code : Image Failure` → Intel ME had problems loading the firmware image
 * `ME: Progress Phase : BUP Phase` → the initialization process is stuck at the bring-up phase
 * `ME: Progress Phase State : M0 kernel load` → the initialization has been interrupted during the kernel loading