# Booting the installation USB drive
## BIOS/UEFI
You'll want to go into your computer's BIOS and change the boot order, or boot off the USB directly.  
If this won't work for whatever reason, try the DD mode in Rufus as described in [install-usb.md](install-usb.md).
## Begin of the installation
You should be put in a terminal session, automatically logged in as root.  
If like me, you have a HiDPI screen, you'll want to make the text a bit more readable.  
You can do this with `setfont latarcyrheb-sun32`.  
If you'd like to make this permanent, add `FONT=latarcyrheb-sun32` to `/etc/vconsole.conf`

Let's start with setting up some basics.
### Keyboard layout
You can list all the available layouts with `ls /usr/share/kbd/keymaps/**/*.map.gz`. Load the keymap for your device with `loadkeys`. 
If you have a normal US keyboard, just go with `loadkeys us`, which I believe is already the default, so you probably don't need to do this :)
### Internet
Plug in your ethernet cable, or run `wifi-menu`, and connect to a WiFi network.
Verify that you're connected: `ping archlinux.org` or any other site you want to ping to :)
### Set system clock
Synchronize your clock using NTP by running `timedatectl set-ntp true`.
### Partitioning
Identify the disk you want to install Arch Linux with `fdisk -l`.
In my case, I want to install to `/dev/sda`, which is the SSD.

Now you'll want to set up your partition table.
Run `fdisk /dev/disk` where `disk` is the disk you want to install on, so in my case, I run `fdisk /dev/sda`.  
Enter `g` to create a new GPT partition table. 
Now press `n` to create a new partition. We'll start with the bootloader partition.
```
Partition number: default (press enter)
First sector: default (press enter)
Last sector: +512M
```
Now change the type of this partition. Enter `t`, then enter `1` for "EFI System" type.  
Let's create the root partition now. Enter `n` again.
```
Partition number: default (press enter)
First sector: default (press enter)
Last sector: default (press enter)
```
I don't create a separate partition for swap, since you could just as easily use a swapfile.
### Filesystems
To see the partitions you made, run `lsblk`.  
We'll now want to format the partitions with the correct filesystem.  
For the boot partition (the partition that's 512M large), we'll want a FAT32 filesystem.  
Format the partition with `mkfs.vfat -F32 /dev/bootpartition` where `bootpartition` is the boot partition.  
The root partition is formatted using `mkfs.ext4 /dev/rootpartition` where `rootpartition` is the partition which has all the disk space.  
In my case, I run `mkfs.vfat -F32 /dev/sda1` and `mkfs.ext4 /dev/sda2`.  
### Mounting the filesystems
Once you've partitioned and created the filesystems, you can mount the filesystems so we can start actually installing Arch Linux.  
Mount the root partition:  
```
mount /dev/rootpartition /mnt
```    
Now create the boot folder and mount the boot partition to it:
```
mkdir /mnt/boot
mount /dev/bootpartition /mnt/boot
```
### Installing the base packages
Cool, now we have our filesystems mounted we can actually install the packages!  
Run `pacstrap /mnt base` to install the base packages. I actually recommend you also install base-devel. It'll save you time later on.
For my machine, I'd run `pacstrap /mnt base base-devel`.  
On my Macbook, I also install `wireless-tools`, so I can still run `wifi-menu` once I've installed Arch.  
The `base` packages does not contain all the packages that the live image (your install USB) has.
