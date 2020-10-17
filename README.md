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
  
  
