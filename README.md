# A Personal Arch Installation Guide

This is a personal guide so if you are lost and just found this guide from somewhere, I recommend you to read the official [`wiki`](https://wiki.archlinux.org/index.php/Installation_guide)!  This guide will focus on `systemd-boot`, `UEFI` and a guide if you want to encrypt your partition with `LUKS/LVM`. This guide exists so that I can remember a bunch of things when reinstalling `Archlinux`.

## Pre-installation

Before installing, make sure to:

+ Read the [official wiki](https://wiki.archlinux.org/index.php/installation_guide). It is advisable to read that instead. I wrote this guide for myself.
+ Acquire an installation image from [here](https://www.archlinux.org/download/).
+ Verify signature.
+ Prepare an installation medium.
+ Boot the live environment.

## Set the keyboard layout

The default console keymap is US. Available layouts can be listed with:

```
# ls /usr/share/kbd/keymaps/**/*.map.gz
```

To modify the layout, append a corresponding file name to loadkeys, omitting path and file extension. For example, to set a US keyboard layout:  

```
# loadkeys us
```

## Verify the boot mode

If UEFI mode is enabled on an UEFI motherboard, Archiso will boot Arch Linux accordingly via systemd-boot. To verify this, list the efivars directory:  

```
# ls /sys/firmware/efi/efivars
```

If the command shows the directory without error, then the system is booted in UEFI mode. If the directory does not exist, the system may be booted in **BIOS** (or **CSM**) mode.

## Connect to the internet

We need to make sure that we are connected to the internet to be able to install Arch Linux `base` and `linux` packages. Let’s see the names of our interfaces.

```
# ip link
```

You should see something like this:

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
		link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN mode DEFAULT group default qlen 1000
		link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff
3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DORMANT group default qlen 1000
		link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff permaddr 00:00:00:00:00:00
```

+ `enp0s0` is the wired interface  
+ `wlan0` is the wireless interface  

### Wired Connection

If you are on a wired connection, you can enable your wired interface by systemctl start `dhcpcd@<interface>`.  

```
# systemctl start dhcpcd@enp0s0
```

### Wireless Connection

If you are on a laptop, you can connect to a wireless access point using `iwctl` command from `iwd`. Note that it's already enabled by default. Also make sure the wireless card is not blocked with `rfkill`.

Scan for network.

```
# iwctl station wlan0 scan
```

Get the list of scanned networks by:

```
# iwctl station wlan0 get-networks
```

Connect to your network.

```
# iwctl -P "PASSPHRASE" station wlan0 connect "NETWORKNAME"
```

Ping archlinux website to make sure we are online:

```
# ping archlinux.org
``` 

If you receive Unknown host or Destination host unreachable response, means you are not online yet. Review your network configuration and redo the steps above.

## Update the system clock

Use `timedatectl` to ensure the system clock is accurate:

```
# timedatectl set-ntp true
```

To check the service status, use `timedatectl status`.

## Partition the disks

When recognized by the live system, disks are assigned to a block device such as `/dev/sda`, `/dev/nvme0n1` or `/dev/mmcblk0`. To identify these devices, use lsblk or fdisk.  The most common main drive is **sda**.

```
# lsblk
```

Results ending in `rom`, `loop` or `airoot` may be ignored.

In this guide, I'll create a two different ways to partition a drive. One for a normal installation, the other one is setting up with an encryption(LUKS/LVM). Let's start with the unecrypted one:

### Unencrypted filesystem

+ Let’s clean up our main drive to create new partitions for our installation. And yeah, in this guide, we will use `/dev/sda` as our disk.

	```
	# gdisk /dev/sda
	```

+ Press <kbd>x</kbd> to enter **expert mode**. Then press <kbd>z</kbd> to *zap* our drive. Then hit <kbd>y</kbd> when prompted about wiping out GPT and blanking out MBR. Note that this will ***zap*** your entire drive so your data will be gone - reduced to atoms after doing this. THIS. CANNOT. BE. UNDONE.

+ Open `cgdisk` to start partitioning our filesystem

	```
	# cgdisk /dev/sda
	```

+ Press <kbd>Return</kbd> when warned about damaged GPT.

	Now we should be presented with our main drive showing the partition number, partition size, partition type, and partition name. If you see list of partitions, delete all those first.

+ Create the `boot` partition

	- Hit New from the options at the bottom.
	- Just hit enter to select the default option for the first sector.
	- Now the partion size - Arch wiki recommends 200-300 MB for the boot + size. Let’s make 1GiB in case we need to add more OS to our machine. I’m gonna assign mine with 1024MiB. Hit enter.
	- Set GUID to `EF00`. Hit enter.
	- Set name to `boot`. Hit enter.
	- Now you should see the new partition in the partitions list with a partition type of EFI System and a partition name of boot. You will also notice there is 1007KB above the created partition. That is the MBR. Don’t worry about that and just leave it there.

+ Create the `swap` partition

	- Hit New again from the options at the bottom of partition list.
	- Just hit enter to select the default option for the first sector.
	- For the swap partition size, I always assign mine with 1GiB. Hit enter.
	- Set GUID to `8200`. Hit enter.
	- Set name to `swap`. Hit enter.

+ Create the `root` partition

	- Hit New again.
	- Hit enter to select the default option for the first sector.
	- Hit enter again to input your root size.
	- Also hit enter for the GUID to select default(`8300`).
	- Then set name of the partition to `root`.

+ Create the `root` partition

	- Hit New again.
	- Hit enter to select the default option for the first sector.
	- Hit enter again to use the remainder of the disk.
	- Also hit enter for the GUID to select default.
	- Then set name of the partition to `home`.

+ Lastly, hit `Write` at the bottom of the patitions list to *write the changes* to the disk. Type `yes` to *confirm* the write command. Now we are done partitioning the disk. Hit `Quit` *to exit cgdisk*. Go to the [next section](#formatting-partitions).

### Encrypted filesystem with `LUKS/LVM`

+ Let’s clean up our main drive to create new partitions for our installation. And yeah, in this guide, we will use `/dev/sda` as our disk.

	```
	# gdisk /dev/sda
	```

+ Press <kbd>x</kbd> to enter **expert mode**. Then press <kbd>z</kbd> to *zap* our drive. Then hit <kbd>y</kbd> when prompted about wiping out GPT and blanking out MBR. Note that this will ***zap*** your entire drive so your data will be gone - reduced to atoms after doing this. THIS. CANNOT. BE. UNDONE.

+ Create our partitions by running `cgdisk /dev/sda`

	```
	# cgdisk /dev/sda
	```

+ Just press <kbd>Return</kbd> when warned about damaged GPT.

	Now we should be presented with our main drive showing the partition number, partition size, partition type, and partition name. If you see list of partitions, delete all those first.

+ Create the `LVM` partition

	- Hit New again.
	- Hit enter to select the default option for the first sector.
	- Hit enter again to use the remainder of the disk.
	- Set GUID to `8e00`. Hit enter.
	- Set name to `lvm`. Hit enter.

+ Lastly, hit `Write` at the bottom of the patitions list to *write the changes* to the disk. Type `yes` to *confirm* the write command. Now we are done partitioning the disk. Hit `Quit` *to exit cgdisk*. Go to the [next section](#formatting-partitions).


## Verifying the partitions

Use `lsblk` again to check the partitions we created. *We? I thought I'm doing this guide for myself lol*

```
# lsblk
```

You should see *something like this*:

### Unencrypted filesystem

| NAME | MAJ:MIN | RM | SIZE | RO | TYPE | MOUNTPOINT |
| --- | --- | --- | --- | --- | --- | --- |
| sda | 8:0 | 0 | 477G | 0 |   |   |
| sda1 | 8:1 | 0 | 1 | 0 | part |   |
| sda2 | 8:2 | 0 | 1 | 0 | part |   |
| sda3 | 8:3 | 0 | 175G | 0 | part |   |
| sda4 | 8:4 | 0 | 300G | 0 | part |   |

**`sda`** is the main disk  
**`sda1`** is the boot partition  
**`sda2`** is the swap partition  
**`sda3`** is the home partition  
**`sda4`** is the root partition

### Encrypted filesystem

| NAME | MAJ:MIN | RM | SIZE | RO | TYPE | MOUNTPOINT |
| --- | --- | --- | --- | --- | --- | --- |
| sda | 8:0 | 0 | 477G | 0 | disk |   |
| sda1 | 8:1 | 0 | 1 | 0 | part |   |
| sda2 | 8:2 | 0 | 1 | 0 | part |   |

**`sda`** is the main disk  
**`sda1`** is the boot partition  
**`sda2`** is the LVM partition

**Surprise! Surprise!** We will **not** encrypt the `/boot` partition.

## Format the partitions

### Unencrypted filesystem

+ Format `/dev/sda1` partition as `FAT32`. This will be our `/boot`.

	```
	# mkfs.fat -F32 /dev/sda1
	```

+ Create and enable our `swap` under the `/dev/sda2` partition.

	```
	# mkswap /dev/sda2
	# swapon /dev/sda2
	```

+ Format `/dev/sda3` and `/dev/sda4` partition as `EXT4`. This will be our `root` and `home`  partition.

	```
	# mkfs.ext4 /dev/sda3
	# mkfs.ext4 /dev/sda4
	```

### Encrypted filesystem

+ Format `/dev/sda1` partition as `FAT32`. This will be our `/boot`.

	```
	# mkfs.fat -F32 /dev/sda1
	```

+ Create the LUKS encrypted container.

	```
	# cryptsetup luksFormat /dev/sda2
	```

+ Enter your passphrase twice. Don't forget this!

+ Open the created container and name it whatever you want. In this guide I'll just use `cryptlvm`.

	```
	# cryptsetup open --type luks /dev/sda2 cryptlvm
	```

+ Enter your passphrase and verify it.

+ The decrypted container is now available at `/dev/mapper/cryptlvm`.

+ Create a physical volume on top of the opened LUKS container:

	```
	# pvcreate /dev/mapper/cryptlvm
	```

+ Create the volume group and name it `volume` (or whatever you want), adding the previously created physical volume to it:

	In this guide, I'll just use `volume` as the volume group name.

	```
	# vgcreate volume /dev/mapper/cryptlvm
	```

+ Create all your needed logical volumes on the volume group. We will create a `swap`, `root`, and `home` logical volumes. Note that the `volume` is the name of the volume we just created.

	- Create our `swap`. I'll assign 1GB to it.

		```
		# lvcreate -L 1G volume -n swap
		```

		This will create `/dev/mapper/volume-swap`.

	- Create our `root`. In this guide, I'll use 100GB.

		```
		# lvcreate -L 100G volume -n root
		```

		This will create `/dev/mapper/volume-root`.

	- Create our home sweet `home`. I'll just assign the remaining space to it.

		```
		# lvcreate -l 100%FREE volume -n home
		```

	This will create `/dev/mapper/volume-home`.

+ Format the logical partitions under the LVM volume.

	- Format and create our `swap`.

		```
		# mkswap /dev/mapper/volume-swap  
		# swapon /dev/mapper/volume-swap
		```

	- Format our `root` and `home` partitions.

		```
		# mkfs.ext4 /dev/mapper/volume-root
		# mkfs.ext4 /dev/mapper/volume-home
		```

## Mount the filesystems

### Unencryped partition

+ Mount the `/dev/sda` partition to `/mnt`. This is our `/`:

	```
	# mount /dev/sda3 /mnt
	```

+ Create a `/boot` mountpoint:

	```
	# mkdir /mnt/boot  
	```

+ Mount `/dev/sda1` to `/mnt/boot` partition. This is will be our `/boot`:

	```
	# mount /dev/sda1 /mnt/boot
	```

+ Create a `/home` mountpoint:

	```
	# mkdir /mnt/home  
	```

+ Mount `/dev/sda4` to `/mnt/home` partition. This is will be our `/home`:

	```
	# mount /dev/sda1 /mnt/home
	```

	We don’t need to mount `swap` since it is already enabled.  

### Encrypted partition

+ Mount the `/dev/mapper/volume-root` partition to `/mnt`. This is our `/`:

	```
	# mount /dev/mapper/volume-root /mnt
	```

+ Create a `/boot` mountpoint:

	```
	# mkdir /mnt/boot  
	```

+ Mount `/dev/sda1` to `/mnt/boot` partition. This is will be our `/boot`:

	```
	# mount /dev/sda1 /mnt/boot
	```

+ Create a `/home` mountpoint:

	```
	# mkdir /mnt/home  
	```

+ Mount `/dev/mapper/volume-home` to `/mnt/home` partition. This is will be our `/home`:

	```
	# mount /dev/mapper/volume-home /mnt/home
	```

	 We don’t need to mount `swap` since it is already enabled.

## Installation

Now let’s go ahead and install `base`, `linux`, `linux-firmware`, and `base-devel` packages into our system. 

```
# pacstrap /mnt base base-devel linux linux-firmware
```

The `base` package does not include all tools from the live installation, so installing other packages may be necessary for a fully functional base system. In particular, consider installing: 

+ userspace utilities for the management of file systems that will be used on the system,
	
	- `ntfs-3g`: NTFS filesystem driver and utilities
	- `unrar`: The RAR uncompression program
	- `unzip`: For extracting and viewing files in `.zip` archives
	- `p7zip`: Command-line file archiver with high compression ratio
	- `unarchiver`: `unar` and `lsar`: Objective-C tools for uncompressing archive files
	- `gvfs-mtp`: Virtual filesystem implementation for `GIO` (`MTP` backend; Android, media player)
	- `libmtp`: Library implementation of the Media Transfer Protocol
	- `android-udev`: Udev rules to connect Android devices to your linux box
	- `mtpfs`: A FUSE filesystem that supports reading and writing from any MTP devic
	- `xdg-user-dirs`: Manage user directories like `~/Desktop` and `~/Music`

+ utilities for accessing `RAID` or `LVM` partitions,

	- `lvm2`: Logical Volume Manager 2 utilities (*if you are setting up an encrypted filesystem with LUKS/LVM, include this on pacstrap*)

+ specific firmware for other devices not included in `linux-firmware`,
	
+ software necessary for networking,

	- `dhcpcd`: RFC2131 compliant DHCP client daemon
	- `iwd`: Internet Wireless Daemon
	- `inetutils`: A collection of common network programs
	- `iputils`: Network monitoring tools, including `ping`

+ a text editor(s),

	- `nano`
	- `vim`
	- `vi`

+ packages for accessing documentation in man and info pages,

	- `man-db`
	- `man-pages`

+ and more useful tools:

	- `git`: the fast distributed version control system
	- `tmux`: A terminal multiplexer
	- `less`: A terminal based program for viewing text files
	- `usbutils`: USB Device Utilities
	- `bash-completion`: Programmable completion for the bash shell

These tools will be useful later. So **future me**, install these.

## Generating the fstab

```
# genfstab -U /mnt >> /mnt/etc/fstab
```

Check the resulting `/mnt/etc/fstab` file, and edit it in case of errors. 

## Chroot

Now, change root into the newly installed system  

```
# arch-chroot /mnt /bin/bash
```

## Time zone

A selection of timezones can be found under `/usr/share/zoneinfo/`. Since I am in the Philippines, I will be using `/usr/share/zoneinfo/Asia/Manila`. Select the appropriate timezone for your country:

```
# ln -sf /usr/share/zoneinfo/Asia/Manila /etc/localtime
```

Run `hwclock` to generate `/etc/adjtime`: 

```
# hwclock --systohc
```

This command assumes the hardware clock is set to UTC.

## Localization

The `locale` defines which language the system uses, and other regional considerations such as currency denomination, numerology, and character sets. Possible values are listed in `/etc/locale.gen`. Uncomment `en_US.UTF-8`, as well as other needed localisations.

**Uncomment** `en_US.UTF-8 UTF-8` and other needed locales in `/etc/locale.gen`, **save**, and generate them with:  

```
# locale-gen
```

Create the `locale.conf` file, and set the LANG variable accordingly:  

```
# locale > /etc/locale.conf
```

If you set the keyboard layout earlier, make the changes persistent in `vconsole.conf`:

```
# echo "KEYMAP=us" > /etc/vconsole.conf
```

Not using `us` layout? Replace it, stoopid.

## Network configuration

Create the hostname file. In this guide I'll just use `MYHOSTNAME` as hostname. Hostname is the host name of the host. Every 60 seconds, a minute passes in Africa.

```
# echo "MYHOSTNAME" > /etc/hostname
```

Open `/etc/hosts` to add matching entries to `hosts`:

```
127.0.0.1    localhost  
::1          localhost  
127.0.1.1    MYHOSTNAME.localdomain	  MYHOSTNAME
```

If the system has a permanent IP address, it should be used instead of `127.0.1.1`.

## Initramfs  

Creating a new initramfs is usually not required, because mkinitcpio was run on installation of the kernel package with pacstrap. **This is important** if you are setting up a system with encryption!

### Unencrypted filesystem

	```
	# mkinitcpio -p linux
	```

### Encrypted filesystem with LVM/LUKS

+ Open `/etc/mkinitcpio.conf` with an editor:

+ In this guide, there are two ways to setting up initramfs, `udev` (default) and `systemd`. If you are planning to use `plymouth`(splashcreen), it is advisable to use a `systemd`-based initramfs.

	- udev-based initramfs (default).

		Find the `HOOKS` array, then change it to something like this:

		```
		HOOKS=(base udev autodetect keyboard modconf block encrypt lvm2 filesystems fsck)
		```

	- systemd-based initramfs.

		Find the `HOOKS` array, then change it to something like this:

		```
		HOOKS=(base systemd autodetect keyboard sd-vconsole modconf block sd-encrypt sd-lvm2 filesystems fsck)
		```

	- Regenerate initramfs image:

		```
		# mkinitcpio -p linux
		```

## Adding Repositories - `multilib` and `AUR`

Enable multilib and AUR repositories in `/etc/pacman.conf`. Open it with your editor of choice:

### Adding multilib repository

Uncomment `multilib` (remove # from the beginning of the lines). It should look like this:  

```
[multilib]
Include = /etc/pacman.d/mirrorlist
```

### Adding the AUR repository

Add the following lines at the end of your `/etc/pacman.conf` to enable the AUR repo:  

```
[archlinuxfr]
SigLevel = Never
Server = http://repo.archlinux.fr/$arch
```

### `pacman` easter eggs

You can enable the "easter-eggs" in `pacman`, the package manager of archlinux.

Open `/etc/pacman.conf`, then find `# Misc options`. 

To add colors to `pacman`, uncomment `Color`. Then add `Pac-Man` to `pacman` by adding `ILoveCandy` under the `Color` string:

```
Color
ILoveCandy
```

### Update repositories and packages

To check if you successfully added the repositories and enable the easter-eggs, run:

```
# pacman -Syu
```

If updating returns an error, open the `pacman.conf` again and check for human errors. Yes, you f'ed up big time.

## Root password

Set the `root` password:  

```
# passwd
```

## Add a user account

Add a new user account. In this guide, I'll just use `MYUSERNAME` as the username of the new user aside from `root` account. (My phrasing seems redundant, eh?) Of course, change the example username with your own:  

```
# useradd -m -g users -G wheel,storage,power,video,audio,rfkill,input -s /bin/bash MYUSERNAME
```

This will create a new user and its `home` folder.

Set the password of user `MYUSERNAME`:  

```
# passwd MYUSERNAME
```

## Add the new user to sudoers:

If you want a root privilege in the future by using the `sudo` command, you should grant one yourself:

```
# EDITOR=vim visudo
```

Uncomment the line (Remove #):

```
# %wheel ALL=(ALL) ALL
```

## Install the boot loader

Yeah, this is where we install the bootloader. We will be using `systemd-boot`, so no need for `grub2`. 

+ Install bootloader:
	
	We will install it in `/boot` mountpoint (`/dev/sda1` partition).

	```
	# bootctl --path=/boot install
	```

+ Create a boot entry `/boot/loader/entries/arch.conf`, then add these lines:

### Unencrypted filesystem

	```
	title Arch Linux  
	linux /vmlinuz-linux  
	initrd  /initramfs-linux.img  
	options root=/dev/sda3 rw
	```

	If your `/` is not in `/dev/sda3`, make sure to change it. 

	Save and exit.

### Encrypted filesystem

Remember the two-types of initramfs earlier? Each type needs a specific kernel parameters. So there's also a two type of entries here. Remember that `volume` is the volume group name and `/dev/mapper/volume-root` is the path to `/`.

+ udev-based initramfs

	```
	title Arch Linux  
	linux /vmlinuz-linux  
	initrd  /initramfs-linux.img  
	options cryptdevice=UUID=/DEV/SDA2/UUID/HERE:volume root=/dev/mapper/volume-root rw
	```

	Replace `/DEV/SDA2/UUID/HERE` with the UUID of your `LVM` partition. You can check it by running `blkid /dev/sda2`. Note that `cryptdevice` parameter  is unsupported by plymouth so it's advisable to use systemd-based initramfs if you are planning to use it.

	Tip: If you are using `vim`, you can write the UUID easier by typing `:read ! blkid /dev/sda2` then hit enter. Then manipulate the output by using visual mode.

+ systemd-based initramfs

	```
	title Arch Linux
	linux /vmlinuz-linux
	initrd /intel-ucode.img
	initrd /initramfs-linux.img
	options rd.luks.name=/DEV/SDA2/UUID/HERE=volume root=/dev/mapper/volume-root rw
	```

	Replace `/DEV/SDA2/UUID/HERE` with the UUID of your `LVM` partition. You can check it by running `blkid /dev/sda2`.

	Tip: If you are using `vim`, you can write the UUID easier by typing `:read ! blkid /dev/sda2` then hit enter. Then manipulate the output by using visual mode.

### Update boot loader configuration

Update bootloader configuration

```
# vim /boot/loader/loader.conf
```

Delete all of its content, then replaced it by:

```
default arch.conf
timeout 0
console-mode max
editor no
```

## Enable internet connection for the next boot

To enable the network daemons on your next reboot, you need to enable `dhcpcd.service` for wired connection and `iwd.service` for a wireless one.

```
# systemctl enable dhcpcd iwd
```

## Exit chroot and reboot:  

Exit the chroot environment by typing `exit` or pressing <kbd>Ctrl + d</kbd>. You can also unmount all mounted partition after this. 

Finally, `reboot`.

##  Finale

If your installation is a success, then ***yay!!!*** If not, you should start questioning your own existence. Are your parents proud of you? 

## [[POST INSTALLATION]](./POST.md)		[[EXTRAS]](./EXTRAS.md)
