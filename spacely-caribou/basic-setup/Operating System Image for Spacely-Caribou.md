# Operating System Image for Spacely-Caribou 

Spacely-Caribou uses a custom Petalinux-based operating system. 

The image for this operating system can be found here: https://drive.google.com/drive/folders/1Hcv0JJkTxV13foe6BdVN6iAZe5LIzP0Z?usp=sharing

To boot your ZCU102 with the OS image, you will first need to flash it onto an SD card following the steps below.

**NOTE:** These instructions assume that you are using a Linux machine to flash the SD card. Equivalent tools may be available for other operating systems.

## Partitioning the SD Card
If your SD Card is not already partitioned in the correct way for a PetaLinux image follow these instructions to re-partition it.

1. Delete old partitions (https://phoenixnap.com/kb/delete-partition-linux) 
2. Create new partitions following these instructions from AMD: https://docs.amd.com/r/en-US/ug1144-petalinux-tools-reference-guide/Partitioning-and-Formatting-an-SD-Card 
Your SD card should now have two partitions, a boot partition which has a “FAT32” file system, and an ext4 file system partition. (sudo fdisk -l /dev/sdb )


## Copy Files to the SD Card

First, mount your SD card as a media device with the following commands:
```
sudo mkdir /media/usb1
sudo mount /dev/sdb1 /media/usb1
```

Then, copy boot files to the FAT32 partition. 

*Boot files:*  BOOT.BIN, image.ub, and boot.scr

Note: the kernel and devicetree blob are included as part of these files. 

Next, Extract the rootfs into the second partition:

1. cd to mounted directory, then tar –xzvf /PATH/TO/rootfs.tar.gz
2. Optional: Add peary and cmake files to the rootfs to save time later.
Process for Mounting/unmounting an SD Card on Linux:

Finally, unmount the SD card:
```
umount
``` 

## Next Steps

Return your SD card to your ZCU102, and proceed to [ZCU102 Setup for Spacely-Caribou](</spacely-caribou/basic-setup/ZCU102 Setup for Spacely-Caribou.md>)