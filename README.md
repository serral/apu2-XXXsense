# APU2e4 pfSense to OPNsense Migration Guide

Complete documentation for upgrading and migrating an APU2e4 system board from pfSense to OPNsense, including BIOS flashing and serial console setup for macOS.

## Table of Contents

- [Hardware Specifications](#hardware-specifications)
- [BIOS Upgrade](#bios-upgrade)
- [Serial Console Setup](#serial-console-setup)
- [pfSense to OPNsense Migration](#pfsense-to-opnsense-migration)
- [Troubleshooting](#troubleshooting)
- [References](#references)

---

## Hardware Specifications

### System Board
- **Model**: APU2e4
- **Processor**: AMD G-series (quad-core)
- **RAM**: Typically 4GB (varies by configuration)
- **Storage**: CompactFlash or mSATA

### Serial Console Adapter
- **Brand**: StarTech.com
- **Model**: USB to Serial Adapter (6ft/1.8m)
- **Connector**: FTDI DB9 Serial DCE Adapter Cable
- **Type**: Null Modem
- **Interface**: USB 2.0
- **TAA Compliant**: Yes
- **Typical Device**: `/dev/cu.usbserial-AR0JJ3WE` (macOS)

---

## BIOS Upgrade

### Overview
Upgrade the APU2e4 firmware to the latest mainline version for improved stability and security.

### Prerequisites
- Latest firmware from [PC Engines GitHub](https://github.com/pcengines/apu2-documentation)
- Current recommended version: **v4.19.0.1** or later (check [PC Engines downloads](https://pcengines.github.io/))
- Serial console access
- Linux system with `flashrom` installed (or access via serial console)

### Step-by-Step Instructions

1. **Download the latest firmware**
   ```bash
   # Get firmware from PC Engines
   # Current recommended: apu2_v4.19.0.1.rom or later
   ```

2. **Flash the BIOS**
   ```bash
   flashrom -p internal -w apu2_v4.19.0.1.rom --fmap -i COREBOOT -c W25Q64BV/W25Q64CV/W25Q64FV
   ```

   > **Important**: If you encounter a board mismatch error, use the `boardmismatch=force` flag:
   > ```bash
   > flashrom -w coreboot.rom -p internal:boardmismatch=force
   > ```

3. **Power cycle the system**
   - After flashing completes, power down the APU2e4
   - Wait 30 seconds
   - Power back on to apply the new firmware

### References
- [PC Engines BIOS Flashing Guide](https://github.com/pcengines/apu2-documentation/blob/master/docs/firmware_flashing.md)
- [PC Engines Downloads](https://pcengines.github.io/)

---

## Serial Console Setup

### Overview
Access the APU2e4 serial console from macOS for system administration and troubleshooting.

### Hardware Setup

1. **Connect the StarTech USB-to-Serial cable**
   - Connect USB end to macOS system
   - Connect DB9 end to APU2e4 serial port

2. **Identify the serial device**
   ```bash
   ls -la /dev/cu.usbserial*
   ```

   Example output:
   ```
   /dev/cu.usbserial-AR0JJ3WE
   ```

### Software Setup

1. **Install minicom (macOS)**
   ```bash
   # Using homebrew
   brew install minicom
   
   # Or manually via MacPorts
   # See: https://pbxbook.com/other/mac-tty.html#minicom
   ```

2. **Configure minicom**
   ```bash
   minicom -s
   ```

   Configure the following settings:
   - **Serial port setup**: Replace default with your device (e.g., `/dev/cu.usbserial-AR0JJ3WE`)
   - **Baud rate**: 115200 (default)
   - **Data bits**: 8
   - **Stop bits**: 1
   - **Parity**: None
   - **Flow control**: None
   - **Save setup as dfl** to make it the default

3. **Launch minicom**
   ```bash
   minicom
   ```

### Troubleshooting Serial Connection

| Issue | Solution |
|-------|----------|
| Device not found | Verify cable is connected; run `ls -la /dev/cu.usbserial*` |
| No output | Check baud rate is 115200; verify serial port setting |
| Garbled text | Confirm baud rate and flow control settings match APU2e4 defaults |

---

## pfSense to OPNsense Migration

### Overview
Migrate from pfSense to OPNsense on APU2e4 hardware using a bootable TinyCore USB drive.

### Prerequisites
- USB drive (minimum 2GB)
- OPNsense installation images (download from [OPNsense](https://opnsense.org))
- TinyCore Linux image
- Serial console access (recommended for troubleshooting)

### Step 1: Prepare USB Drive

1. **Format USB drive as FAT**
   ```bash
   # Identify your USB drive
   diskutil list
   
   # Example: disk3 is the USB drive
   diskutil secureErase 0 JHFS+ USB disk3
   diskutil eraseDisk FAT32 USB /dev/disk3
   ```

2. **Make USB drive bootable**
   ```bash
   # Use fdisk to mark partition as bootable
   sudo fdisk -e /dev/disk3s2
   ```

   Then in fdisk prompt:
   ```
   fdisk> f 1
   fdisk> write
   fdisk> exit
   ```

3. **Write TinyCore image to USB**
   ```bash
   # Using pv for progress visualization
   sudo dd if=apu2-tinycore6.4.img | pv -s 1.8G | dd of=/dev/disk3 bs=64k
   
   # Without pv (fallback)
   sudo dd if=apu2-tinycore6.4.img of=/dev/disk3 bs=64k
   ```

### Step 2: Prepare OPNsense Images

1. **Download OPNsense images**
   - Download `amd64-serial` image (for serial console support)
   - Alternatively, download `amd64-nano` if serial image has compatibility issues

   > **⚠️ Known Issue**: The APU2e4 may have compatibility problems with the serial OPNsense image (AHCI errors, disk read failures). In such cases, use the `amd64-nano` image instead. See [OPNsense Forum Discussion](https://forum.opnsense.org/index.php?topic=6942.msg35847#msg35847).

2. **Copy images to USB FAT partition**
   ```bash
   # Mount the USB FAT partition
   mkdir -p /tmp/usb-mount
   sudo mount -t msdos /dev/disk3s1 /tmp/usb-mount
   
   # Copy OPNsense image(s)
   cp OPNsense-20.7-OpenSSL-serial-amd64.img.bz2 /tmp/usb-mount/
   
   # Or use nano image if serial has issues
   cp OPNsense-20.7-OpenSSL-nano-amd64.img.bz2 /tmp/usb-mount/
   
   # Unmount
   sudo umount /tmp/usb-mount
   ```

### Step 3: Boot and Install OPNsense

1. **Boot APU2e4 from USB**
   - Insert USB drive
   - Connect to serial console (optional but recommended)
   - Power on APU2e4
   - Press `F12` during boot to access boot menu
   - Select USB drive

2. **Write OPNsense image in TinyCore**
   ```bash
   # Decompress and write to disk (option 1: serial image)
   bunzip2 -dc OPNsense-20.7-OpenSSL-serial-amd64.img.bz2 | dd of=/dev/sda bs=1M
   
   # Or write nano image if serial has issues
   bunzip2 -dc OPNsense-20.7-OpenSSL-nano-amd64.img.bz2 | dd of=/dev/sda bs=1M
   ```

3. **Power cycle and verify**
   - OPNsense should boot and prompt for initial configuration
   - Follow the OPNsense setup wizard

### Troubleshooting Installation

| Issue | Solution |
|-------|----------|
| TinyCore won't boot | Verify USB is marked bootable; check BIOS boot order |
| `dd` command fails | Use serial console to check device name; may be `/dev/ada0` or `/dev/sda` |
| OPNsense shows AHCI errors | Switch to `amd64-nano` image instead of serial image |
| Disk read errors | Clean partition table with `gpart destroy -F ada0` (before writing image) |
| Blank/no output | Check serial console baud rate (115200); verify cable connection |

### Cleanup Commands (if needed)

```bash
# Destroy existing partition table
gpart destroy -F ada0

# Blank the disk completely
dd if=/dev/zero of=/dev/ada0 bs=64k
```

---

## Troubleshooting

### BIOS Flashing Issues
- **Board mismatch error**: Use `boardmismatch=force` flag with flashrom
- **No serial output after flash**: Power cycle the system completely (30 seconds)
- **Flash verification failed**: Retry with `--verify` flag after initial write

### Serial Console Issues
- **No output after connecting**: Check minicom baud rate (should be 115200)
- **Garbled characters**: Verify flow control is set to "None"
- **Device disappears**: Check USB cable and try different USB port

### OPNsense Installation Issues
- **Serial image compatibility**: Use nano image as fallback
- **Boot hangs**: Verify USB boot priority in BIOS
- **Network not working after install**: Reconfigure network interfaces during OPNsense setup

---

## References

### Hardware Documentation
- [PC Engines APU2 Documentation](https://www.pcengines.ch/pdf/apu2.pdf)
- [PC Engines GitHub Repository](https://github.com/pcengines/apu2-documentation)

### Firmware & BIOS
- [PC Engines Downloads](https://pcengines.github.io/)
- [BIOS Flashing Guide](https://github.com/pcengines/apu2-documentation/blob/master/docs/firmware_flashing.md)
- [MR-63 BIOS Release Notes](https://pcengines.github.io/#mr-63)

### OPNsense Documentation
- [OPNsense Official Website](https://opnsense.org)
- [OPNsense Installation Guide](https://docs.opnsense.org/manual/install.html)
- [OPNsense Forum - APU2 Migration Discussion](https://forum.opnsense.org/index.php?topic=6942.msg35847#msg35847)

### pfSense Documentation
- [pfSense Official Website](https://www.pfsense.org)

### Serial Console Setup
- [macOS Serial Console Tutorial](https://pbxbook.com/other/mac-tty.html#minicom)
- [StarTech USB Serial Adapter](https://www.startech.com)

### Tools
- [flashrom - BIOS Flashing Tool](https://www.flashrom.org/)
- [minicom - Terminal Emulator](https://alioth-lists.debian.net/pipermail/minicom-devel/)
- [TinyCore Linux](http://tinycorelinux.net/)

---

## Notes

This guide is based on practical experience migrating an APU2e4 from pfSense to OPNsense on macOS. The original reference material was provided by [Tristan Greaves](https://extricate.org/2020/05/10/home-firewall-pc-engines-apu2-e2-pfsense-and-opnsense-build-courtesy-of-linitx/).

Last updated: October 2024

