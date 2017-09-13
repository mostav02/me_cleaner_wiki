As a reference, I'll explain here the full process to apply _me_cleaner_ on my MSI H110M ECO.

<img src="https://www.msi.com/asset/resize/image/global/product/product_10_20151204135719_56612b3fe255a.png62405b38c58fe0f07fcef2367d8a9ba1/1024.png" width="600">

This board has a Sunrise Point PCH and comes with Intel ME version 11.6.1.1142 CON (2 MB). It is sold without a CPU, therefore we are sure that Intel Boot Guard is not set in Verified Mode, and there are good chances that it will work with a disabled Intel ME.

# Step 1: locate the SPI flash chip

As usual, you can flash the modified firmware in two ways:
 * External flashing, heavily recommended (and used in this example)
 * Internal flashing, on this board using the UEFI flash tool

Even if you want to use the internal flashing method, you should locate the flash chip and check whether it is supported by flashrom, to unbrick the board in case something goes wrong.

Recent motherboards (especially the desktop ones) uses SOIC-8 SPI flash chips, which are quite easy to locate and deal with. On this board it is here:

<img src="https://image.ibb.co/nnziTa/product_10_20151204135720_56612b4026997.png" width="600">

[Searching on Google the code written on it](https://www.google.com/search?q=25L6473E) confirms that it is a SPI flash chip. On page 4 of [its datasheet](http://www.macronix.com/Lists/Datasheet/Attachments/6207/MX25L6473E,%203V,%2064Mb,%20v1.4.pdf) we can see that it accepts 3.3 V as power supply, we can therefore use a common Raspberry Pi as a programmer. On page 7 there is the pinout, however I discovered that the manufacturer of the motherboard has routed the programming pins to the pin header next to the chip, which easies the flashing.

<img src="https://image.ibb.co/grBYuF/IMG_20170913_161943.jpg" width="600">
<img src="https://image.ibb.co/cexHMv/pinout.png" width="400">

This chip is fully supported by `flashrom` (**P**robe, **R**ead, **E**rase and **W**rite):

<pre>
$ flashrom -L | grep -C 4 25L6473E
              MX25L6408E
Macronix      MX25L6436E/            PREW           8192  SPI       
              MX25L6445E/
              MX25L6465E/
              <b>MX25L6473E</b>
Macronix      MX25L12805D            PREW          16384  SPI       
Macronix      MX25L12835F/           PREW          16384  SPI       
              MX25L12845E/
              MX25L12865E
</pre>

# Step 2: install the required tools

Power on the Raspberry Pi, open a shell and install the required tools:

     $ sudo apt-get update
     $ sudo apt-get install git flashrom
     $ git clone --depth=1 http://review.coreboot.org/p/coreboot
     $ make -C coreboot/util/ifdtool
     $ git clone https://github.com/corna/me_cleaner.git

Open the file `/boot/config.txt` and uncomment the line `dtparam=spi=on`, to make the SPI interface available in userspace; reboot and you should see a file `/dev/spidev0.0`.

# Step 3: dump the original firmware

Turn off the Raspberry Pi, connect [these pins of the Raspberry Pi](https://pinout.xyz)

 * +3.3 V
 * GND
 * MOSI
 * MISO
 * SCK
 * CE0

to the corresponding pins on the header and turn the Raspberry Pi on.
You can also use a SOIC clip if you prefer

<img src="https://image.ibb.co/eJqhoa/IMG_20170913_160928.jpg" width="600">

Open a shell and run:

     $ flashrom -p linux_spi:dev=/dev/spidev0.0,spispeed=8000 -c MX25L6436E/MX25L6445E/MX25L6465E/MX25L6473E -r original.rom

Twice:

     $ flashrom -p linux_spi:dev=/dev/spidev0.0,spispeed=8000 -c MX25L6436E/MX25L6445E/MX25L6465E/MX25L6473E -r original2.rom

Now check if they are identical:

     $ diff original.rom original2.rom

It should print nothing (which means that they're identical); if not try to reduce the spispeed parameter, the length of the cables and check if they're connected firmly. Check also if they a valid flash descriptor

     $ coreboot/util/ifdtool/ifdtool -d original.rom

and if they contain a valid Intel ME firmware

     $ python me_cleaner/me_cleaner.py -c original.rom

If they succeeds all the tests, copy one of them to a safe place.

# Step 4: modify the Intel ME firmware

Just run

     $ python me_cleaner/me_cleaner.py -S -d original.rom -O modified.rom

`modified.rom` now contains:
 * A descriptor with the HAP bit set and a restricted set of permissions for the flash access of Intel ME
 * An unmodified BIOS
 * An Intel ME image with the minimal required modules

# Step 5: flash back the modified Intel ME firmware

With the same connections as before, run

     $ flashrom -p linux_spi:dev=/dev/spidev0.0,spispeed=8000 -c MX25L6436E/MX25L6445E/MX25L6465E/MX25L6473E -w modified.rom

# Step 6: verify that it works

Disconnect the Raspberry Pi, connect the power supply to the PC and turn it on. It will turn ON correctly, however:

 * the boot takes longer, and the debug LEDs are stuck on "RAM" for some seconds
 * the PC displays a warning message "The ME FW of system was found abnormal" at the boot

Success!