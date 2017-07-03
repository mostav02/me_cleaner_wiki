# Intel Boot Guard

Intel Boot Guard is a technology introduced by Intel in the 4th Intel Core generation (Haswell) to verify the boot process. This is accomplished by flashing the public key of the BIOS signature into the field programmable fuses (FPFs), a one-time programmable memory inside Intel ME, during the manufacturing process; in this way the CPU contains the public key of the BIOS and it can verify its correct signature during the boot. Obviously, once enabled by the manufacturer, Intel Boot Guard can't be disabled anymore.

Unfortunately for us Intel Boot Guard is not compatible with _me_cleaner_ as the machine will not power on if Intel ME has been disabled, **even if the BIOS hasn't been modified**.

Now the question is: do you have Intel Boot Guard on your computer?
 * If you have a CPU older than the 4th generation (Haswell), **no**
 * If you don't have a preassembled computer, **no**, as the CPU and the chipset must be physically connected during the manufacturing process to fuse the public key into the CPU and enable Intel Boot Guard
 * If your PC doesn't have _vPro_ (there's usually a sticker somewhere on the case), **probably no**

So, if you have a preassembled PC, with the _vPro_ sticker and at least Haswell (or newer) you may have Intel Boot Guard enabled.

Anyways, if you want to check whether you have or not Intel Boot Guard enabled you can use `intelmetool` with [this](https://review.coreboot.org/#/c/16328/) patch applied (if not merged yet):

     $ git clone --depth=1 http://review.coreboot.org/p/coreboot
     $ cd util/intelmetool/

Go [here](https://review.coreboot.org/#/c/16328/), click on "Download" (top right corner), copy the line "Cherry pick" (which should be something like `git fetch https://review.coreboot.org/coreboot refs/changes/28/16328/21 && git cherry-pick FETCH_HEAD`) and paste it in the shell. Then run

     $ make
     # modprobe msr
     # ./intelmetool -b

and check the output.

To remove the patch run

     $ git reset --hard origin/master

If you want to understand more about Intel Boot Guard you can have a look at [this](https://2016.zeronights.ru/wp-content/uploads/2017/03/Intel-BootGuard.pdf) presentation by Alexander Ermolov at the ZERONIGHTS 2016.