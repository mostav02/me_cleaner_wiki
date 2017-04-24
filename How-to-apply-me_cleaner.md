# How to apply me_cleaner

There are two methods to apply me_cleaner to your platform:
 * External flashing (recommended)
 * Internal flashing

The external flashing requires a flash programmer and the opening of the PC to reach the flash chip. Luckily, the ROM chips are often in SOIC-8 or DIP-8 packages (and sometimes they're even socketed), so they're big enough to be flashed without problems with a DIP/SOIC clip. Moreover, the ROM chips are standard SPI chips, so a common Raspberry Pi, a Beaglebone or any Linux device with a SPI interface can be used to dump and flash the chip. This procedure has the advantage of the safety: even if you mess completely the firmware of your board you can always flash back the original firmware and restore your device (unless you've broken the hardware). Moreover any flash read/write protections set by the manufacturer are bypassed by this technique. The problems are the required skills to locate the chip and flash it.

On the other side you can flash the modified firmware internally. This procedure is easier and faster but depends heavily on the firmware and the flash tools provided and it can be harder to recover from a brick. If you really want to flash it internally, you definitely want a recover solution (external programmer, dual BIOS chips, internal recovery flasher...). 

Make your choice:
 * [External flashing](https://github.com/corna/me_cleaner/wiki/External-flashing)
 * Internal flashing
   * [with coreboot](https://github.com/corna/me_cleaner/wiki/Internal-flashing-with-coreboot)
   * [with OEM firmware](https://github.com/corna/me_cleaner/wiki/Internal-flashing-with-OEM-firmware)

As a reference, you should take a look at [this guide](https://hardenedlinux.github.io/firmware/2016/11/17/neutralize_ME_firmware_on_sandybridge_and_ivybridge.html) where an external programmer (Raspberry Pi) is used on a Lenovo X220 with coreboot.