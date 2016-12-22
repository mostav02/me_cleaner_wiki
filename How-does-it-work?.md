Since me_cleaner modifies a very fundamental part of an Intel system, it is reasonable to be worried about such modification. Here I'm explaining what this script does, and how.

## Sources

The basic informations about Intel ME firmwares were provided by:
 * [me.bios.info](http://me.bios.io/ME_blob_format)
 * [Igor Skochinsky - Rootkit in your laptop - Breakpoint 2012](http://me.bios.io/images/c/ca/Rootkit_in_your_laptop.pdf)
 * [Igor Skochinsky - Intel ME Secrets - RECON 2014](https://recon.cx/2014/slides/Recon%202014%20Skochinsky.pdf)

Our work started when we found, on the Coreboot Mailing List, a message from Trammel Hudson in which he shared his [recent discovers about Intel ME firmwares](https://www.coreboot.org/pipermail/coreboot/2016-September/082016.html).
Then we tried by ourself and we replicated the results: to make things simpler I created a Python script to modify the firmware automatically and we published it on the [Coreboot Mailing List](https://www.coreboot.org/pipermail/coreboot/2016-November/082331.html) and later on [GitHub](https://github.com/corna/me_cleaner).

## What does this tool do?

As you can see in [me.bios.info](http://me.bios.io/ME_blob_format), the firmware is divided in partitions and every partition can contain one or more modules. At the beginning of the image there is the [FPT](http://me.bios.io/ME_blob_format#.24FPT_Partition_table_header) (Firmware Partition Table), a region where all the partitions are indexed. The FPT region is **NOT** signed and can be freely modified (but it has a checksum, offset 0x1b).

[Each partition](http://me.bios.io/ME_blob_format#.24FPT_Partition_table_header) is individually signed, therefore all the non-fundamental partitions can be removed without breaking anything (as we are removing the partitions, but also the signatures).

It seems that the only fundamental partition is the FTPR (Factory Partition), as every other partition (with the respective entry in the FPT) can be removed. The firmware still contains some code, but:
 * The network stack (partition NFTP) has been removed
 * The PAVP (partition MDMV, module JCOM) has been removed
 * And more...

Based on the results on Trammel Hudson (again, on the Coreboot ML) I improved me_cleaner by removing also the LZMA-compressed modules in the FTPR partition (more or less half of the modules in the partition). This also removes the Intel Antitheft module (module TDT), and leaves only 4-5 Huffman-compressed modules. According to [Igor's 2014 presentation, slide 17](https://recon.cx/2014/slides/Recon%202014%20Skochinsky.pdf), these modules do the basic ME initialization:
 * ROMP (not always present)
 * BUP - Bringup (hardware initialization/configuration)
 * KERNEL - Scheduler, low-level APIs for other modules
 * POLICY - Secondary init tasks, some high-level APIs
 * FTCS

Some of you could ask: "Ok, but here you are tampering the partition code, aren't you invalidating the signature?"
The answer is: **yes**!

But why does the PC still work if the signature is not valid anymore? This has 3 possible reasons:
 1. Intel ME doesn't check the signature of the fundamental partition: **very** unlikely (but someone should confirm this sooner or later)
 2. The Huffman modules have an additional signature, stored somewhere inside the module: this could be verified with an entropy analysis, but it doesn't seem likely (why sign the same code twice and ignore one of the two signature?)
 3. No code is run, as no signed code is available. Intel ME allows the boot of the system if the structure (offsets, magic numbers, SHA-256 hashes...) is valid, but doesn't run the code of the Intel ME firmware if its signature is invalid.

The **3** is the most likely, and this means that, _maybe_, also the Huffman modules can be removed, as soon as I (or someone else) realizes how to remove them.

## How can I be sure that all the bad stuff is not in a ROM inside the CPU?

We can never be sure about this, but, thanks to the [Igor's 2014 presentation](https://recon.cx/2014/slides/Recon%202014%20Skochinsky.pdf) (slide 18), we can suppose that the code inside the internal ROM is just the code responsible for the very basic initialization of Intel ME.

## How?

Since there isn't any official documentation of the Intel ME firmwares, a short explanation of how me_cleaner works is mandatory.
At first it detects if the image is a "pure" ME image (magic **$FPT** at offset 0x10) or a full BIOS dump (magic **5a a5 f0 0f** at offset 0x10). If it is a "pure" ME image it uses the whole file as ME image, otherwise it finds the ME region by reading the values inside the Intel Flash Descriptor.

Then it searches for the basic ME informations: presence of the FPT region, version of the firmware, number of partitions and offset and size of the FTPR partition. Now it has all the informations it needs, and it can start the "cleaning" process by:
 * dumping the original FTPR entry in the FPT
 * filling the whole image with 0xff, except for the first 0x30 bytes (the start of the FPT) and the FTPR partition
 * restoring the FTPR entry in the FPT
 * removing the EFFS presence flag

Now the ME image contains only the FTPR partition, as every other partition has been overwritten by 0xff.

For pre-Skylake images, the internal structure of the partitions is known, and also the LZMA modules can be removed. The LZMA modules are placed after the Huffman data (after the [LLUT](http://me.bios.io/ME_blob_format#LLUT_Breakdown)), therefore any byte from the end of the Huffman section to the end of the FTPR partition is overwritten with 0xff. It seems that the SHA-256 hash of the LZMA modules is not checked, therefore the hashes are left unmodified.

In the end, the checksum of the FPT section is corrected.

## But I still don't trust you

You're right, you should never trust a random guy on the Internet, but you don't have to trust me, you can check the [code](https://github.com/corna/me_cleaner/blob/master/me_cleaner.py) by yourself.

And, by the way, most of the code is "find _x_ and overwrite it with 0xff".

## Cool, how can I apply it?

Whoa there cowboy, before flashing the modified image you should understand the implications of such modification.
First you should understand that this tool does not reimplements **anything**, it only wipes parts of a basic component of your processor, so keep in mind:
 * Bricking is likely to happen! Even if this tool has been tested with your system, it does not mean that this modification is safe, everything could go wrong
 * You are losing something. Intel ME doesn't only provides some services (useless and dangerous, IMHO), but it also does low-level stuff (like silicon workaround, thermal management, fan control...). Most of these things are often controlled by the OS, so they're not really needed, but who can be sure?
 * Bricking is **very** likely to happen! Just in case you didn't hear me the first time
 * Usually the ME region is not writeable by software, therefore you usually need an external programmer

If you have a motherboard with a socketed ROM chip you can test me_cleaner on a spare chip and leave the original one untouched, otherwise you can dump the original firmware and restore it with an external programmer in case of brick.

## Ok, I'm not scared, I want to try it!

Great! You can follow [this](http://hardenedlinux.org/firmware/2016/11/17/neutralize_ME_firmware_on_sandybridge_and_ivybridge.html) guide if you want to test it. Please report the success on [#3](https://github.com/corna/me_cleaner/issues/3) (even if it's already listed in the [status page](https://github.com/corna/me_cleaner/wiki/me_cleaner-status)).