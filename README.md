# Libreboot_T480


Edit 1/5/2025:
Added Compatibility Log



EDIT 1/1/2025:
Added more images showing how to wire and place the test clip
Added extra info about the different rom choices


Great guides I used for help along the way

<https://libreboot.org/docs/install/t480.html>

<https://libreboot.org/docs/install/spi.html>

<https://ezntek.com/posts/librebooting-the-thinkpad-t480-20241207t0933/>

<https://www.cyberciti.biz/faq/update-lenovo-bios-from-linux-usb-stick-pen/>

NOTE: Some of these steps are likely to change very soon. One has already changed before I posted (this will be mentioned below). When the scripts/official documents change, blindly following these steps will likely cause problems and headaches. Please use alongside the official documentation
Tested Features (This is only my personal testing):

Working:

   * trackpad
   * keyboard (with backlight function - only options for brightness are 0%, 50%, or 100%)
   * speakers
   * Rj45 ethernet port
   * wifi card
   * USB ports (thunderbolt untested - I did not apply the thunderbolt patch)
   * dual battery power bridge
   * camera

Not working:

   * headphone jack
   * display backlight function keys

Things I needed:

  *  ThinkPad T480 or T480s

   * Another PC running Linux (without an ecryptfs filesystem. If you have an encrypted home using ecryptfs directory, you will want to use a live Linux USB for your 2nd machine, as the vendor flash script will fail. This is a common issue when using Linux Mint with an encrypted /home)

 *   Raspberry Pi Pico (with pins soldered)

 *   SOIC 8 test clip

 *   jumper wires to go from the test clip to the rpi

  *  internet connection

 *   usb stick that can be wiped

 *   basic Linux knowledge

 *   many prayers

Upgrade/Downgrade Factory BIOS

It is worth noting that it is recommended to take a few steps before updating/downgrading the bios:

    dump/backup the current bios

    deal with the Thunderbolt Issue

However, I did not do either of these steps, so we will move on to the upgrading/downgrading

If you wish to backup your current BIOS before attempting to upgrade/downgrade, please skip down to "Prepare Test clip" as well as the following step, "Backup the current BIOS"

Before upgrading/downgrading, first make sure you check the current BIOS version you have. We will need V1.52. You can check this by shutting down the T480, powering back on, and pressing F1 repeatedly until it says "entering setup". In the first page of the BIOS, you should see a version number for your BIOS (1.xx)

If you are already using 1.52, you can skip this section and jump to "Prepare Firmware/Flash Vendor Blobs"

V1.52 is necessary for EC UART support. This can be accomplished by taking the bios image from this site or here's a direct link to the iso. You will want to download the n24ur39w version.

These links are only for the T480. If you have the T480s, go to the same page but for your model.

After you have the iso image, unfortunately you cannot just write it to a usb stick. You must instead use a tool called geteltorito. This is an excellent guide that I used. If something goes wrong with the install or the process, refer to the guide linked as it is more detailed, as well as contains arch-specific instructions. But I did not use arch, so they will not be mentioned here. To install:

Debian/Ubuntu:

sudo apt install genisoimage

Fedora:

sudo dnf install geteltorito genisoimage

Next, we will want to convert our iso to an img:

geteltorito -o {output-file.img {Bootable-CD.iso}

in the case of the T480, an example would be:

geteltorito -o T480.img n24ur39w.iso #assuming the n24ur39w.iso is in your home directory

After the img has been saved, you can then write this file to a usb stick using the following command:

sudo dd if=T480.img of=/dev/sdx bs=4M

You will need to replace "sdx" with your usb stick's location. You can find it by running lsblk and looking for a device with the same size as your usb. If you have a 128GB usb stick it may say 119.5G, and it will be labeled sda, sdb, sdc, etc. DO be sure to double check this and ensure you are not selecting the wrong drive. Using the wrong drive here will wipe it

Once this is done, we are ready to boot into our newly created USB stick.

To boot from the USB, you will need to shut down the laptop, power it back on, and repeatedly press F12 when the Lenovo logo appears until it says "entering boot menu"

Select your usb stick

If the laptop will not boot from USB, you will need to change some BIOS settings. Bios can be accessed by repeatedly pressing F1 when the Lenovo logo appears

As per the libreboot.org documentation linked above:

    You must disable SecureBoot, and enable legacy/CSM boot, and boot it in BIOS mode, not UEFI mode. Make sure your battery is well-charged, and boot it with a battery and with the power supply plugged in. Select option 2 in the menu, to update your BIOS, which also updates the EC firmware. This is the Lenovo BIOS/UEFI updater. Once you’ve updated, you can flash Libreboot.

There is also an option in the "Security" settings of the BIOS that says UEFI flashing or UEFI downgrade protection. You want to make sure that downgrade protection is disabled if you are downgrading

After this you should be good to boot from the USB

When you boot from the USB, you will want to select option 2 and follow the instructions

Once the machine reboots, it will continue the flashing process. Please be patient

After the flashing is completely done, go back into the bios and confirm that it is V1.52

If it says 1.52, you are good to go
Prepare Firmware/Flash Vendor Blobs

Next you will want to download the libreboot image for the T480/T480s

https://libreboot.org/download.html < This is where you can find the download link (HTTPS Mirrors are recommended. Just scroll down to HTTPS Mirrors and select any link)

You will want to download from stable > 20241206 > Roms > libreboot-20241206rev6_t480_fsp_16mb.tar.xz

Make sure to get the correct one. If you have T480s, download the libreboot-20241206rev6_t480s_fsp_16mb.tar.xz

This is the current version as of 12/30/2024. The next revision is expected to come early January 2025. At that point, you should use the next revision, as it will include bug fixes and other improvements. I have not noticed any major issues so far with Rev 6, except for the display's backlight. It works fine, but the function keys do not natively work for adjusting the brightness. You will need to adjust the display brightness in your OS, or remap the function buttons. I didn't bother with any of this since I can adjust the backlight from the main panel in my distro

Now that we have the firmware, we need to modify the roms with vendor blobs. Without this step, your bios chip will be bricked and the laptop will be a paperweight

To flash the vendor blobs, we need more software. Download the lbmk repo through your terminal >

>git clone https://notabug.org/libreboot/lbmk.git

Now we want to copy the archive with our T480 roms to this folder, or take note of it's current location. IMO it is easier to work with if you copy the archive to the lbmk folder

Now,

cd lbmk
sudo ./mk dependencies debian (if you use debian - recommended)
sudo ./mk dependencies mint (if you use mint - not recommended)
sudo ./mk dependencies arch (if you use arch)

and finally

./vendor inject libreboot-20241206rev6_t480_fsp_16mb.tar.xz

If you run into issues injecting the blobs, be sure you are not running in an encryptfs encrypted directory, as some of the filenames will be too long to process and the script will fail. If this is the case, I recommend using a live linux distro like debian for this part of the process. I got it to work on a Linux Mint live ISO, but debian is recommended

If you are using an old version of lbmk, the modified roms will be stored in your lbmk folder > bin > release

I heard that the script is being updated to replace the archive with a modified one, instead of creating a new directory entirely. If this is the case, be sure to use the appropriate rom files. I suggest checking the bin folder anyways just in case. Again this is based on my experience as an end user, and I found the roms in the bin file. But the script is supposed to be updated now to instead, replace the archive
Setup RPI Pico

Now you will want to prepare your flasher. Libreboot.org recommends the Raspberry Pi pico, so this is what I have used. You can attempt to compile the RPI firmware yourself using this repo, but I could not get it to work. Instead, I used the pre-compiled firmware from this fork. Your goal is to end up with a pico_serprog.uf2 file. This is the file you need. Whether you compile it yourself, or download it from the fork. Personally, I used the one from the fork

Now, hold down the "Bootsel" button on the pico while connecting the usb cable to your pc. It should mount like any other usb flash drive

Simply drag the uf2 file to the raspberry pi, and it will automatically disconnect as the update applies. Give it a few seconds, and then unplug the pico's usb from your pc

Now, it is time to prepare for flashing. In a terminal, we will want to run "dmesg -wh" and AFTER running this command, plug in your newly flashed raspberry pi pico (do not hold the button this time). You will see the terminal update when the usb device is recognized. We want to look for line similar to this:

[  +1.391500] usb 7-3: new full-speed USB device number 12 using ohci-pci
[  +0.171110] usb 7-3: New USB device found, idVendor=cafe, idProduct=4001, bcdDevice= 1.00
[  +0.000015] usb 7-3: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[  +0.000006] usb 7-3: Product: TinyUSB Device
[  +0.000005] usb 7-3: Manufacturer: TinyUSB
[  +0.000005] usb 7-3: SerialNumber: 123456
[  +0.006042] cdc_acm 7-3:1.0: ttyACM0: USB ACM device #<< THIS IS WHAT WE WANT

Take note of the value "ttyACM0" ^^^
Backup the current BIOS

Now, before we begin flashing the new firmware, we need to backup the old one in case anything goes wrong. This, I did do, and I did need to restore the backups. Hopefully you won't have to restore them, but they are indeed very good to have handy. So let's connect the test clip correctly. First we need to disassemble the laptop, and disconnect all batteries/power sources (including the CMOS battery). After this you will find an 8 pin chip near the RAM labeled as Winbond (there is also a Winbond chip near the hinges. This is the thunderbolt controller, ignore this chip - unless you need to apply the fix as discussed earlier). You may also have a macronix BIOS chip instead of winbond. Be sure to inspect your chip. Be sure to connect the clip correctly BEFORE connecting power/usb to the pico. Always connect/disconnect the clip while NO POWER IS ATTACHED

Your clip should look similar to the image here when it is seated correctly:
r/libreboot - A guide for flashing the T480
r/libreboot - A guide for flashing the T480

These next images were taken directly from libreboot documentation as well as ezntek and I highly recommend doing your own research in these documents as well. Messing this up can and will cause issues. Make sure the test clip is flush with the board, and the pin 1 indicator (either a different colored wire or some indicator on the plastic clip) matches up with pin1 on the board shown below
r/libreboot - A guide for flashing the T480
r/libreboot - A guide for flashing the T480

This is the command we will run after successfully connecting the test clip, to create a dump of the chip:

flashrom -p yourprogrammer -c "your chip model" -r t480_stockbios_1.bin

In my case:

flashrom -p serprog:dev=/dev/ttyACM0 -c "W25Q128.V" -r t480_stockbios_1.bin

We want to repeat this 3 times, renaming the file to t480_stockbios_2.bin, and so on (it will take up to 15-20 minutes to read the chip each time. This is a long process)

Once you have 3 dumps, run this command:

sha256sum t480_stockbios*.bin

Check the output. There should be 3 lines, and all lines should be identical. If they are not identical, run the flashrom command again until you have 3 identical dumps
Selecting the right ROM for you

After this is done, we are ready to wipe the chip and replace it with libreboot. But before we do so, we need to figure out which rom to use. If we check our rom folder, we see several different roms to choose. Although I cannot explain what all the various options do, I can give info for a few of them:

    seabios_t480_fsp_16mb_libgfxinit_corebootfb.rom > This is the standard seabios rom. It will immediately try to boot from the first disk available, unless you stop it and select grub (in my case with full disk encryption, the only way I can boot with this rom is by going to boot menu (esc) and then selecting option 2 for grub.

    Because of this, I chose to re-flash with seagrub instead (seagrub_t480_fsp_16mb_libgfxinit_corebootfb_usqwerty.rom)

This way, it boots straight to grub without any user input

The below image shows the only difference I can find between the two. This is seagrub. Notice how boot option 1 is grub2 payload, and option 2 is my NVMe. This is reversed if you flash the seabios rom. In seabios rom, option 1 will be the NVMe, and option 2 will be the grub2 payload
r/libreboot - A guide for flashing the T480

If you have an encrypted drive, or dual drives, going straight to grub (seagrub rom) will be the best option, as it will not require you to interrupt the boot process to get into grub. If you flash a seabios rom with an encrypted drive, it will get stuck forever trying to "boot from hdd". You will need to press "esc" before the hdd starts booting, to enter the menu above, and select option 2 for the grub2 payload

If you have a different keyboard layout, you can select the corresponding seagrub version (ukqwerty if you have a uk layout, etc)

There are also variants of both seabios and seagrub that are configured as txtmode. This is the info I could find in the libreboot docs:

    Configurations with libgfxinit will use coreboot’s native graphics init code if available on that board. If the file name has txtmode in it, coreboot will be configured to start in text mode, when setting up the display. If the file name has corebootfb in it, coreboot will be configured to set up a high resolution frame buffer, when initializing the display.

    NOTE: If the configuration file is libgfxinit_txtmode, the SeaBIOS payload can still run external VGA option ROMs on graphics cards, and this is the recommended setup (SeaBIOS in text mode) if you have a board with both onboard and an add-on graphics card (e.g. PCI express slot) installed.

There are also some strange layouts like Dvorak and Colemak. This is what I could find regarding these two layouts:
r/libreboot - A guide for flashing the T480
Time to flash!

Now that we have our rom picked out, we can flash it with one easy command:

sudo flashrom -p serprog:dev=/dev/ttyACM0 -c "W25Q128.V" -w /bin/release/t480_fsp_16mb/seabios_t480_fsp_16mb_libgfxinit_corebootfb.rom

Remember to substitute the info here with your programmer, your chip, and your rom of choice

This process will take a long time, be prepared to spend 40 minutes or so waiting for it to finish. Under any circumstances, DO NOT interrupt this process. Flashing time will depend on your hardware.

Eventually you should see the following output:

Reading old flash chip contents... done.
Erasing and writing flash chip... Erase/write done.
Verifying flash... VERIFIED.

After successful verification, you can disconnect the pico's usb cable (do this before removing the test clip), re-assemble, and enjoy your new BIOS!
r/libreboot - A guide for flashing the T480

