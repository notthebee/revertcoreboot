# Reverting coreboot installation on Thinkpad X220 without original SPI dump
Congratulations! You forgot to back up your original BIOS dump, and now you're stuck with coreboot forever...right? Not so fast.  
Even without the original BIOS dump, you can still install the original BIOS, which is really useful if you experience some problems or decide to install macOS.  
### Requirements
* .FL1 file from the modified BIOS v1.46 - http://www.mcdonnelltech.com/X220_v1.46_Modified_BIOS.zip
* X220 running Linux with `flashrom` package installed, as well as kernel option `iomem=relaxed` set in bootloader
### WARNING
Only do this if you know what you're doing, be sure to have a Raspberry Pi and a Pomona clip handy to re-flash your SPI chip in case you mess up. I am not responsible for any damage your computer may suffer during this process.
### The process
Take the *.FL1 file from the modified BIOS v1,46, e.g. $01C9100.FL1, and truncate it to exactly 8MByte:  
```
dd if=\$01C9100.FL1 of=x220_spi.bin bs=$((0x800000)) count=1
```
Backup the old content:  
```
flashrom -p internal:laptop=force_I_want_a_brick -r file.rom
```
You may need to tell the tool which flash IC it is with the "-c" option:
```
flashrom -p internal:laptop=force_I_want_a_brick -c MX25L6405 -r file.rom
```
As we want to write only the BIOS part in the SPI flash starting at offset 0x500000, we need a layout file:
```
echo -e "000000:4fffff dummy\n500000:7fffff bios" > x220.layout
```
Update the BIOS portion in the SPI flash
```
flashrom -p internal:laptop=force_I_want_a_brick -c MX25L6405 --layout t520.layout -i bios -w x220_spi.bin
```
Done!
  
  
This guide is mostly based on this wiki page: http://thinkwiki.de/UEFI_BIOS_T420_BIOS_Structure
