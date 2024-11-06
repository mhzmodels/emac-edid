# Display Fix for Apple eMacs running PowerPC Linux

**Special thanks to Edward J C Robbins over at the Linux on PowerPC Macs facebook group for making this all possible! This tutorial is based off of Edward's orginal post for installing Debian 10 on an eMac.**
### Why is this necessary?
The Linux DRM  (Direct Rendering Manager) has issues configuring the eMacs internal display automatically. These set of instructions show you how to provide the necessary display information early on in the boot process.
### Why not just share the original post?
There were a number of steps that were no longer necessary while some have changed completely, so I thought it would be helpful to others to make the notes and necessary files more accessable and available in one place.

Link to the original post.
https://www.facebook.com/groups/ppclinux/posts/437344776940132/

**Debian 12 installer ISO Used**
- http://cdimage.debian.org/cdimage/ports/12.0/powerpc/debian-12.0.0-powerpc-NETINST-1.iso

Boot, install and restart!

***VERY IMPORTANT***

- **When your eMac boots into your fresh install, be at the ready to choose `Advanced Options...` before it boots automatically.**
- Highlight the 6.1.0-9 kernel, then press `e` to edit boot options.
- Find the line begining with `linux:` and add `nomodeset` to the end of the line.
- Boot with `Control-X`

_Failing to do this will lead grub to automatically boot from the newest kernel, which for some reason doesn't work until apt upgrades your software._
_It will freeze while Open Firmware begins the boot process, forcing you to perform a hard shut-down._

__________________________

The eMac will now begin to boot. However it will remain stuck with a flashing text cursor when trying to load the graphical environment. 

Not a problem! Press `Control-Alt-F2` to login.

Once logged in:
```
$ su - root
# nano /etc/apt/sources.list
```

Add `non-free non-free-firmware` to the end of the first entry (after `sid main`) Save and exit.

Update all your packages with:
```
# apt update
# apt upgrade
```
Note: Ignore any error messages about any entries being mispelt. If you are not planning on compiling any software then you can comment out the last entry of sources.list to resolve the error.

## Installing firmware
```
# apt install firmware-linux-free firmware-linux-nonfree
```
## Installing the EDID file and initramfs-tools script:
```
# mkdir -p /lib/firmware/edid
```
Insert and mount a FAT formatted flash drive with the `1280x960.bin` and `edid-hooks` file. 

This assumes you have no other drive connected to your eMac besides the flash drive.
```
# mkdir /mnt/usb0
# mount /dev/sdb1 /mnt/usb0
# cp /mnt/usb0/1280x960.bin /lib/firmware/edid
# cp /mnt/usb0/edid-hooks /usr/share/initramfs-tools/hooks/edid
# chmod +x /usr/share/initramfs-tools/hooks/edid
# update-initramfs -u -k all
# reboot
```
Note: There are some error messages when it processes the script we added, probably due to some syntax changes from when it was created. Not sure how to resolve the error, but it looks like the error message points in the right direction. I'll make some attempts soon to fix this, but for now the original script works.

## Let's test it!
**Be at the ready to press the `e` key when the grub menu loads, before it boots automatically.** This time, we're going to boot using the newest kernel. Instead of adding `nomodeset`, we're going to add something different to the end of the line (after `quiet`).
```
drm.edid_firmware=edid/1280x960.bin
```
Press `Control-X` to boot.
Now you are greeted with the lightdm login screen!

## Making The Changes Permanent
```
# nano /etc/default/grub
```
Make the neccesary addition to the following line. Save and exit.
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet drm.edid_firmware=edid/1280x960.bin
```

Update the changes made to grub
```
# update-grub
```
You're all set. Thanks again to Edward for sharing your eMac knowledge!


----------------------
## Optional Changes

### Grub 
```
nano /etc/default/grub
```

Hiding OSX entries:
- Change GRUB_DISABLE_OS_PROBER=false to "true"

Faster Grub:
- Uncomment GRUB_TERMINAL=console

### Console mode (text only) at startup
```
# systemctl set-default multi-user.target
# reboot
```

### GUI mode (Xorg) at startup
```
# systemctl set-default graphical.target
# reboot
```
### Change default login wallpaper

Copy your wallpapers to /usr/share/wallpapers or a location lightdm can access them outside of your home folder

```
# nano /usr/share/lightdm/lightdm-gtk-greeter.conf.d/01_debian.conf
```
Make the neccesary changes to point to your wallpaper image instead of the default.

### Fix broken Mac OS 9 partition

The Debian installer will most likely render your Mac OS 9 installation unbootable. This can be fixed by booting from OS9 on another medium (MacOS9Lives CD) and running Drive Setup. Select "not mounted" from the list of drives and choose Update Driver from the Functions menu.

Restart and you'll find all is well again.
