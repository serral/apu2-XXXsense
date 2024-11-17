# apu2-XXXsense
apu2e4 system board and pfSense to OPNsense documentation

There is a great webpage by Tristan Greaves that outlines how to:
1- upgrade bios
2- use serial console to connect to the system board
3- references migration from pfSense to OPNsense (but lack details here)
and can be found here:
https://extricate.org/2020/05/10/home-firewall-pc-engines-apu2-e2-pfsense-and-opnsense-build-courtesy-of-linitx/

Decided to add some details that I had to go throught to make it work on my specific setup.

1- upgrade bios
  As of writing, the "recommended firmware version is latest mainline v4.11.0.x". It can be found here:
  https://pcengines.github.io/
  and just need to follow https://github.com/pcengines/apu2-documentation/blob/master/docs/firmware_flashing.md, taking into accout that, instead of flashrom -w coreboot.rom -p internal you have to do flashrom -w coreboot.rom -p internal:boardmismatch=force (know bug, included in this page)
  
2- use serial console to connect to the system board
  In my particular case, I use macos so, after instaling the StarTech USB null modem cable (https://www.amazon.co.uk/gp/product/B008634VJY), I just needed to find witch device I should use (my case, ls -la /dev/cu.usbserial-AR0JJ3WE). 
  I than installed minicom for macos (https://pbxbook.com/other/mac-tty.html#minicom) and just needed to run minicom -s (/opt/minicom/current/bin/minicom -s), configure the "Serial port setup" (replacing the existing with /dev/cu.usbserial-AR0JJ3WE), "Save setup as dfl" and it worked directly like a charm.
  
3- migration from pfSense to OPNsense
  Lots of confusing documentation here. I just followed the document https://www.pcengines.ch/pdf/apu2.pdf (pfSense section) but adapted to the following steps:
    - formated a USB drive as FAT
    - made it bootable:
      sudo fdisk -e /dev/disk3s2
      f 1
      write
      exit
    - write the image to disk:
      sudo dd if=apu2-tinycore6.4.img | pv -s 1.8G | dd of=/dev/disk3 bs=64k
    - copied the OPNsense software image (amd64 ,serial (for serial console support) to the FAT partition of the TinyCore USB stick
    !!! - I did struggle to get this part to work. It seems that APU2 and the serial image do not work (AHCI,disk errors, etc). If that is the case with you, jjust copy the OPNsense nano image to the USB drive, instead of the serial.
For more details, follow: https://forum.opnsense.org/index.php?topic=6942.msg35847#msg35847


## These are notes on how to:
  Write the OPNSense image to the disk (when in tinycore):
    bunzip2 -dc OPNsense-20.7-OpenSSL-serial-amd64.img.bz2 | dd of=/dev/sda bs=1M
  Clean the partition tables (when struggling with the serial version of OPNSense and probably no longer needed):
    gpart destroy -F ada0
  Blank the disk:
    dd if=/dev/zero of=/dev/ada0 bs=64k
    
This process took a lot longer than planned and I've just completed it. I will have to repeat it and will update this documentation then.
Andre Serralheiro



#### 20241117 ####

https://pcengines.github.io/#mr-63
HOW TO https://github.com/pcengines/apu2-documentation/blob/master/docs/firmware_flashing.md

https://github.com/pcengines/apu2-documentation/tree/master/scripts/apu_fw_updater_opnsense.sh !!! download fails but the logic is the same as in the howto (plus reboot which should be powerdown)

flashrom -p internal -w apu2_v4.19.0.1.rom --fmap -i COREBOOT -c W25Q64BV/W25Q64CV/W25Q64FV

