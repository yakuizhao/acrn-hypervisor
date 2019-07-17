Prepare ChromeOS image:
#######################

Prepare the build environment and download the source code by following the guide: https://www.chromium.org/chromium-os/quick-start-guide "Prerequisites" and "Get the Source" sections.
Go to the ChromeOS source folder, for exampe: chrome, and then run::

# cros_sdk

Then cd to the overlays folder and then apply the patch: `overlay.patch`_ ::

# cd ~/trunk/src/overlays
# git apply overlay.patch

Go to script folder and run build command, which may take some time::

# cd ~/trunk/src/scripts
# ./build_packages --board=amd64-generic
# ./build_image --board=amd64-generic

Exit from ChromeOS build environment::

# exit

Now copy this image onto a usb drive.  Insert the usb stick you’d like to use and run::

# cros_sdk -- cros flash --board=amd64-generic usb://

This will prompt you for which usb device you’d like to use. (Note that auto-mounting of USB devices should be turned off as it may corrupt the disk image while it's being written.)

Mount the EFI partition of ChromeOS, please find the disk node of your USB driver, for example: sdb. And replace [chrome source] with your source patch, for example: ~/chrome/ ::

# sudo mount /dev/sdb12 /mnt
# cd /mnt/efi/boot
# sudo mkdir x86_64-efi
# sudo cp [chrome source]/chroot/lib/grub/x86_64-efi/multiboot.mod x86_64-efi/

Integrate Acrn Hypervisor:
##########################

Copy the pre-built `acrn.32.out`_ into the /mnt/efi/boot folder::

# sudo cp acrn.32.out /mnt/efi/boot/

Then edit the grub.cfg in /mnt/efi/boot/ folder, and insert below menuentry::

  menuentry "acrn" {
     insmod multiboot
     multiboot --quirk-modules-after-kernel /EFI/boot/acrn.32.out
     module /syslinux/vmlinuz.A Linux_bzImage init=/sbin/init boot=local rootwait ro noresume noswap loglevel=7 noinitrd console=ttyS0  i915.modeset=1 cros_efi cros_debug  root=PARTUUID=073875AA-4F53-B64F-BB31-B6CB4E3C0B32
   }

The kernel parameter after "Linux_bzImage" should be copied from the ChromeOS kernel commandline in "local image A", like below::

  init=/sbin/init boot=local rootwait ro noresume noswap loglevel=7 noinitrd console=ttyS0  i915.modeset=1 cros_efi cros_debug  root=PARTUUID=073875AA-4F53-B64F-BB31-B6CB4E3C0B32

Umount the USB driver::

# sudo umount /mnt

Now the USB thumb driver with Acrn Hypervisor + ChromeOS ServiceOS is ready. You can plug it onto your KBL-NUC machine and select boot from USB EFI mode

Build the Acrn Hypervisor from Source:
######################################

Please prepare the acrn build environment by following the guide:
https://projectacrn.github.io/latest/getting-started/building-from-source.html#introduction

And then clone this repo and switch to the test branch. After that, go to the acrn hypervisor source folder and build the code::

# cd hypervisor
# make defconfig BOARD=nuc7i7bnh
# make menuconfig

Then choose "Hybrid VMs" in "ACRN Scenario" option and save::

# make

After build, you will get the acrn.32.out binary. 


.. _overlay.patch: https://github.com/minhe1/acrn-hypervisor/blob/test/overlay.patch
.. _acrn.32.out: https://github.com/minhe1/acrn-hypervisor/blob/test/acrn.32.out
