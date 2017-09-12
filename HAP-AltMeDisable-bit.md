_me_cleaner_ supports two ways to disable Intel ME:
 * by removing the non-fundamental partitions and modules from the Intel ME firmware
 * by setting the HAP (Intel ME >= 11) or the AltMeDisable (Intel ME < 11) bit in the flash descriptor

The former is on by default and it is the older one; it has been tested on many platforms and it seems to work quite well. However it is not a "clean" solution, as it forces Intel ME to hang after the minimal necessary hardware initialization (and wasn't probably planned by Intel).

A new way to disable Intel ME has been discovered by [Positive Technologies](https://www.ptsecurity.com) (as explained in [this blog post](http://blog.ptsecurity.com/2017/08/disabling-intel-me.html)): they found out that Intel ME (>= 11, Skylake or newer) has a "HAP" bit which acts like a kill-switch, telling Intel ME to hang after the initialization. Igor Skochinsky discovered a similar bit, the AltMeDisable bit, which does the same on Intel ME < 11. Essentially, they achieves exactly the same result as the "old" mode (as both modes stops the execution of Intel ME after the hardware initialization), however they have the advantage of:
 * being something introduced by Intel, which report a nice `Alt Disable Mode` status to the BIOS (that should be handled better than the old `Normal` status but with an `Image Failure`)
 * setting an alternate mode of Intel Boot Guard (however the outcome of this is currently unknown)

_me_cleaner_ sets this HAP/AltMeDisable bit when the `-s` (enable only the kill-switch, but don't remove the extra code from the firmware) or the `-S` (enable the kill-switch **and** remove the extra code from the firmware) are passed.

Now the question is: which mode should I use? It depends... Some BIOS seem to work better with the code removal mode, some others with the kill-switch, it's up to you to test which one works better on your platform. Usually, if one mode works, the other follows (but may introduce delays in the boot): my suggestion is to try first without any `-s`/`-S` flags and:
 * if it doesn't work, try with `-s`
 * if it works (and you want to dig deeper) you can try with the `-S` option