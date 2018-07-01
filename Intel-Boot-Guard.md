# Intel Boot Guard

Intel Boot Guard is a technology introduced by Intel in the 4th Intel Core generation (Haswell) to verify the boot process. This is accomplished by flashing the public key of the BIOS signature into the field programmable fuses (FPFs), a one-time programmable memory inside Intel ME, during the manufacturing process; in this way it has the public key of the BIOS and it can verify the correct signature during every subsequent boot. Obviously, once enabled by the manufacturer, Intel Boot Guard can't be disabled anymore.

~~Unfortunately for us Intel Boot Guard is not compatible with _me_cleaner_ as the machine will not power on if Intel ME has been disabled, **even if the BIOS hasn't been modified**.~~

me_cleaner should work with Intel Boot Guard when run with the `-s` flag and might work without flags or with the `-S` option. The interaction between Intel Boot Guard and me_cleaner is not yet fully understood, there is work in progress on it.

Now the question is: do you have Intel Boot Guard on your computer?
 * If you have a CPU older than the 4th generation (Haswell), **no**
 * If you don't have a preassembled computer, **no**, as the CPU and the chipset must be physically connected during the manufacturing process to fuse the public key into the CPU and enable Intel Boot Guard
 * If your PC doesn't have _vPro_ (there's usually a sticker somewhere on the case), **probably no**

So, if you have a preassembled PC, with the _vPro_ sticker and at least Haswell (or newer) you may have Intel Boot Guard enabled.

Anyways, if you want to check whether you have or not Intel Boot Guard enabled you can use `intelmetool`:

     $ git clone --depth=1 http://review.coreboot.org/p/coreboot
     $ cd util/intelmetool/
     $ make
     # modprobe msr
     # ./intelmetool -b

and check the output.

If you want to understand more about Intel Boot Guard you can have a look at [this](https://2016.zeronights.ru/wp-content/uploads/2017/03/Intel-BootGuard.pdf) presentation by Alexander Ermolov at the ZERONIGHTS 2016.