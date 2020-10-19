# A Personal Arch Installation Guide

This is a personal guide so if you're lost and just found this guide from somewhere, I recommend you to read the official [`wiki`](https://wiki.archlinux.org/index.php/Installation_guide)!  This guide will focus on `systemd-boot`, `UEFI` and a guide if you want to encrypt your partition with `LUKS/LVM`. This guide exists so that I can remember a bunch of things when reinstalling `Archlinux`.

### Set the keyboard layout

The default console keymap is US. Available layouts can be listed with:

```
# ls /usr/share/kbd/keymaps/**/*.map.gz
```

To modify the layout, append a corresponding file name to loadkeys, omitting path and file extension. For example, to set a US keyboard layout:  

```
# loadkeys us
```

### Verify the boot mode

If UEFI mode is enabled on an UEFI motherboard, Archiso will boot Arch Linux accordingly via systemd-boot. To verify this, list the efivars directory:  

```
# ls /sys/firmware/efi/efivars
```

### Connect to internet

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

##### Wired Connection

If you are on a wired connection, you can enable your wired interface by systemctl start `dhcpcd@<interface>`.  

```
# systemctl start dhcpcd@enp0s0
```

##### Wireless Connection

If you are on a laptop, you can connect to a wireless access point using `iwctl` command from `iwd`. Note that it's already enabled by default.

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

### Update the system clock

Use `timedatectl` to ensure the system clock is accurate:

```
# timedatectl set-ntp true
```

### Partition the disks

When recognized by the live system, disks are assigned to a block device such as `/dev/sda`, `/dev/nvme0n1` or `/dev/mmcblk0`. To identify these devices, use lsblk or fdisk.  The most common main drive is **sda**.

```
# lsblk
```

Results ending in `rom`, `loop` or `airoot` may be ignored.

In this guide, I'll create a two different ways to partition a drive. One for a normal installation, the other one is setting up with an encryption(LUKS/LVM). Let's start with the unecrypted one:

+ **Unencrypted filesystem**

	- Let’s clean up our main drive to create new partitions for our installation. And yeah, in this guide, we will use `/dev/sda` as our disk.

		```
		# gdisk /dev/sda
		```

		+ Press <kbd>x</kbd> to enter **expert mode**. Then press <kbd>z</kbd> to *zap* our drive. Then hit <kbd>y</kbd> when prompted about wiping out GPT and blanking out MBR. Note that this will ***zap*** your entire drive so your data will be gone - reduced to atoms after doing this. THIS. CANNOT. BE. UNDONE.

	- Partitioning

		```
		# cgdisk /dev/sda
		```

		+ Just press <kbd>Return</kbd> when warned about damaged GPT.

		Now we should be presented with our main drive showing the partition number, partition size, partition type, and partition name. If you see list of partitions, delete all those first.

	- Create the `boot` partition

		+   Hit New from the options at the bottom.
		+   Just hit enter to select the default option for the first sector.
		+   Now the partion size - Arch wiki recommends 200-300 MB for the boot + size. Let’s make 1GiB in case we need to add more OS to our machine. I’m gonna assign mine with 1024MiB. Hit enter.
		+   Set GUID to `EF00`. Hit enter.
		+   Set name to `boot`. Hit enter.
		+   Now you should see the new partition in the partitions list with a partition type of EFI System and a partition name of boot. You will also notice there is 1007KB above the created partition. That is the MBR. Don’t worry about that and just leave it there.

	- Create the `swap` partition

		+   Hit New again from the options at the bottom of partition list.
		+   Just hit enter to select the default option for the first sector.
		+   For the swap partition size, I always assign mine with 1GiB. Hit enter.
		+   Set GUID to `8200`. Hit enter.
		+   Set name to `swap`. Hit enter.

	- Create the `root` partition

		+   Hit New again.
		+   Hit enter to select the default option for the first sector.
		+   Hit enter again to input your root size.
		+   Also hit enter for the GUID to select default(`8300`).
		+   Then set name of the partition to `root`.

	- Create the `root` partition

		+   Hit New again.
		+   Hit enter to select the default option for the first sector.
		+   Hit enter again to use the remainder of the disk.
		+   Also hit enter for the GUID to select default.
		+   Then set name of the partition to `home`.

	- Lastly, hit `Write` at the bottom of the patitions list to *write the changes* to the disk. Type `yes` to *confirm* the write command. Now we are done partitioning the disk. Hit `Quit` *to exit cgdisk*. Go to the [next section](#formatting-partitions).

+ **Encrypted filesystem - `LVM`**

	- Let’s clean up our main drive to create new partitions for our installation. And yeah, in this guide, we will use `/dev/sda` as our disk.

		```
		# gdisk /dev/sda
		```

		+ Press <kbd>x</kbd> to enter **expert mode**. Then press <kbd>z</kbd> to *zap* our drive. Then hit <kbd>y</kbd> when prompted about wiping out GPT and blanking out MBR. Note that this will ***zap*** your entire drive so your data will be gone - reduced to atoms after doing this. THIS. CANNOT. BE. UNDONE.

	- Create our partitions by running `cgdisk /dev/sda`

		```
		# cgdisk /dev/sda
		```

		+ Just press <kbd>Return</kbd> when warned about damaged GPT.

		Now we should be presented with our main drive showing the partition number, partition size, partition type, and partition name. If you see list of partitions, delete all those first.

	- Create the `LVM` partition

		+   Hit New again.
		+   Hit enter to select the default option for the first sector.
		+   Hit enter again to use the remainder of the disk.
		+   Set GUID to `8e00`. Hit enter.
		+   Set name to `lvm`. Hit enter.

	- Lastly, hit `Write` at the bottom of the patitions list to *write the changes* to the disk. Type `yes` to *confirm* the write command. Now we are done partitioning the disk. Hit `Quit` *to exit cgdisk*. Go to the [next section](#formatting-partitions).


### Verifying the partitions

Use `lsblk` again to check the partitions we created. *We? I thought I'm doing this guide for myself lol*

```
# lsblk
```

You should see *something like this*:

+ Unencrypted filesystem

	| NAME | MAJ:MIN | RM | SIZE | RO | TYPE | MOUNTPOINT |
	| --- | --- | --- | --- | --- | --- | --- |
	| sda | 8:0 | 0 | 477G | 0 | disk |   |
	| sda1 | 8:1 | 0 | 1 | 0 | part | /boot |
	| sda2 | 8:2 | 0 | 1 | 0 | part | [SWAP] |
	| sda3 | 8:3 | 0 | 175G | 0 | part | / |
	| sda4 | 8:4 | 0 | 300G | 0 | part | /home  |

	**`sda`** is the main disk  
	**`sda1`** is the boot partition  
	**`sda2`** is the swap partition  
	**`sda3`** is the home partition  
	**`sda4`** is the root partition

+ Encrypted filesystem - `LVM`

	| NAME | MAJ:MIN | RM | SIZE | RO | TYPE | MOUNTPOINT |
	| --- | --- | --- | --- | --- | --- | --- |
	| sda | 8:0 | 0 | 477G | 0 | disk |   |
	| sda1 | 8:1 | 0 | 1 | 0 | part | /boot |
	| sda2 | 8:2 | 0 | 1 | 0 | part |   |

	**`sda`** is the main disk  
	**`sda1`** is the boot partition  
	**`sda2`** is the LVM partition

	**Surprise! Surprise!** We will **not** encrypt the `/boot` partition.

### Format the partitions

+ Unencrypted filesystem

	- Format `/dev/sda1` partition as `FAT32`. This will be our `/boot`.

		```
		# mkfs.fat -F32 /dev/sda1
		```

	- Create and enable our `swap` under the `/dev/sda2` partition.

		```
		# mkswap /dev/sda2
		# swapon /dev/sda2
		```

	- Format `/dev/sda3` and `/dev/sda4` partition as `EXT4`. This will be our `root` and `home`  partition.

		```
		# mkfs.ext4 /dev/sda3
		# mkfs.ext4 /dev/sda4
		```

		- Go to the next section.

+ Encrypted filesystem - LVM

	I know that this part is not supposed to be under this section. But whatever. This part will include both guide to setup the LVM partition and formatting it.

	- Format `/dev/sda1` partition as `FAT32`. This will be our `/boot`.

		```
		# mkfs.fat -F32 /dev/sda1
		```

	- Create the LUKS encrypted container.

		```
		# cryptsetup luksFormat /dev/sda2
		```

		- Enter your passphrase twice.

	- Open the created container

		```
		# cryptsetup open --type luks /dev/sda2 cryptlvm
		```

		- Enter your passphrase and verify it.
		- The decrypted container is now available at `/dev/mapper/cryptlvm`.

	- Create a physical volume on top of the opened LUKS container:

		```
		# pvcreate /dev/mapper/cryptlvm
		```

	- Create the volume group named `volume` (or whatever you want), adding the previously created physical volume to it:

		In this guide. I'll just use `volume` as the volume group name.

		```
		# vgcreate volume /dev/mapper/cryptlvm
		```

	- Create all your needed logical volumes on the volume group:

		We will create a `swap`, a `root`, and `home` logical volumes. Note that the `volume` is the name of the volume we just created.

		+ Create our `swap`. I'll assign 1GB to it.

			```
			# lvcreate -L 1G volume -n swap
			```

			This will create `/dev/mapper/volume-swap`.

		+ Create our `root`. In this guide, I'll use 100GB.

			```
			# lvcreate -L 100G volume -n root
			```

			This will create `/dev/mapper/volume-root`.

		+ Create our home sweet `home`. I'll just assign the remaining space to it.

			```
			# lvcreate -l 100%FREE volume -n home
			```

			This will create `/dev/mapper/volume-home`.

		+ Formatting logical partitions under the LVM volume.

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

			- Go to the next section.


### Mount the filesystems

+ Unencryped filesystem

	- Mount the `/dev/sda` partition to `/mnt`. This is our `/`:

		```
		# mount /dev/sda3 /mnt
		```

	- Create a `/boot` mountpoint:

		```
		# mkdir /mnt/boot  
		```

	- Mount `/dev/sda1` to `/mnt/boot` partition. This is will be our `/boot`:

		```
		# mount /dev/sda1 /mnt/boot
		```

	- Create a `/home` mountpoint:

		```
		# mkdir /mnt/home  
		```

	- Mount `/dev/sda4` to `/mnt/home` partition. This is will be our `/home`:

		```
		# mount /dev/sda1 /mnt/home
		```

	- We don’t need to mount `swap` since it is already enabled.  

+ Encrypted filesystem

	- Mount the `/dev/mapper/volume-root` partition to `/mnt`. This is our `/`:

		```
		# mount /dev/mapper/volume-root /mnt
		```

	- Create a `/boot` mountpoint:

		```
		# mkdir /mnt/boot  
		```

	- Mount `/dev/sda1` to `/mnt/boot` partition. This is will be our `/boot`:

		```
		# mount /dev/sda1 /mnt/boot
		```

	- Create a `/home` mountpoint:

		```
		# mkdir /mnt/home  
		```

	- Mount `/dev/mapper/volume-home` to `/mnt/home` partition. This is will be our `/home`:

		```
		# mount /dev/mapper/volume-home /mnt/home
		```

	- We don’t need to mount `swap` since it is already enabled.

### Installing the base and linux packages

Now let’s go ahead and install `base`, `linux`, `linux-firmware`, and `base-devel` packages into our system. 

```
# pacstrap /mnt base base-devel linux linux-firmware
```

It's also recommended to install these:

+ Encryption - Install `lvm` if you're planning to encrypt your filesystem

	```
	# pacstrap /mnt lvm2
	```
	
+ Networking

	```
	# pacstrap /mnt dhcpcd iwd iputils inetutils
	```

+ Manpages

	```
	# pacstrap /mnt man-db man-pages
	```

+ Text editors

	```
	# pacstrap /mnt nano vim
	```

+ More

	```
	# pacstrap /mnt less usbutils bash-completion
	```

	If you ignore these, it will bite you in the ass in the future, I swear*(No, this is not a threat)*. Again, it is recommended to install these. And of couse, it is still your decision.

### Generating the fstab

```
# genfstab -U /mnt >> /mnt/etc/fstab
```

### Chroot

Now, change root into the newly installed system  

```
# arch-chroot /mnt /bin/bash
```

### Configure time zone

A selection of timezones can be found under `/usr/share/zoneinfo/`. Since I am in the Philippines, I will be using `/usr/share/zoneinfo/Asia/Manila`. Select the appropriate timezone for your country:

```
# ln -s /usr/share/zoneinfo/Asia/Manila /etc/localtime
```

Run `hwclock` to generate `/etc/adjtime`: 

```
# hwclock --systohc
```

This command assumes the hardware clock is set to UTC.

### Localization

The Locale defines which language the system uses, and other regional considerations such as currency denomination, numerology, and character sets. Possible values are listed in `/etc/locale.gen`. Uncomment `en_US.UTF-8`, as well as other needed localisations.

**Uncomment** `en_US.UTF-8 UTF-8` and other needed locales in `/etc/locale.gen`, **save**, and generate them with:  

```
# locale-gen
```

Create the locale.conf file, and set the LANG variable accordingly:  

```
# locale > /etc/locale.conf
```

If you set the keyboard layout earlier, make the changes persistent in `vconsole.conf`:

```
# echo 'KEYMAP=us' > /etc/vconsole.conf
```

### Network configuration

Create the hostname file:

In this guide I'll just use `MYHOSTNAME` as hostname. Hostname is the host name. Every 60 seconds, a minute passes in Africa.

```
# echo "MYHOSTNAME" > /etc/hostname
```

Open hosts file to add matching entries to `hosts`:

```
# vim /etc/hosts
```

Add this:  

```
127.0.0.1    localhost  
::1          localhost  
127.0.1.1    MYHOSTNAME.localdomain	  MYHOSTNAME
```

If the system has a permanent IP address, it should be used instead of 127.0.1.1.

### Initramfs  

Creating a new initramfs is usually not required, because mkinitcpio was run on installation of the kernel package with pacstrap. **This is important** if you're on an encrypted filesystem!

+ Unencrypted filesystem

	```
	# mkinitcpio -p linux
	```

+ LVM/LUKS

	+ Open `/etc/mkinitcpio.conf`:

		```
		# vim /etc/mkinitcpio.conf
		```

	+ In this guide, I'll add a two different type of initramfs - `udev` (default) and `systemd`. If you're planning to use `plymouth`(splashcreen), it is advisable to use a `systemd` initramfs.

		- `udev`-based initramfs - default.

			Find `HOOKS`. Then change the array to:

			```
			HOOKS=(base udev autodetect keyboard modconf block encrypt lvm2 filesystems fsck)
			```

		- `systemd`-based initramfs.

			Find `HOOKS`. Then change the array to something like this:

			```
			HOOKS=(base systemd autodetect keyboard sd-vconsole modconf block sd-encrypt sd-lvm2 filesystems fsck)
			```

				Note that you've replaced `udev` with `systemd` and used `sd-encrypt` and `sd-lvm2` instead of `encrypt` and `lvm2`.

		- Regenerate initramfs image:

			```
			# mkinitcpio -p linux
			```

### Network Connection

To have an internet connection on your next reboot, you need to enable `dhcpcd.service` for wired connection and `iwd.service` for a wireless one.

```
# systemctl enable dhcpcd iwd
```

### Adding Repositories - `multilib` and `AUR`

Enable multilib and AUR repositories in `/etc/pacman.conf`:

```
# vim /etc/pacman.conf
```

#### Adding multilib repo

Uncomment `multilib` (remove # from the beginning of the lines). It should look like this:  

```
[multilib]
Include = /etc/pacman.d/mirrorlist
```

#### Adding the AUR repo

Add the following lines at the end of your `/etc/pacman.conf` to enable the AUR repo:  

```
[archlinuxfr]
SigLevel = Never
Server = http://repo.archlinux.fr/$arch
```

### `pacman` easter eggs

`pacman` is the package manager of archlinux. To "improve" it:

Open `/etc/pacman.conf`, then find `# Misc options`:

To add colors to `pacman`, uncomment:

```
Color
```

To add `Pac-Man` to `pacman`, add `ILoveCandy` under the `Color`:

```
ILoveCandy
```

### Root password

Set the root password:  

```
# passwd
```

### Add a user account

Add a new user account. In this guide, I'll just use `MYUSERNAME` as the username of the new user aside from `root` account. (My phrasing seems redundant, eh?) Of course, change the example username with your own:  

```
# useradd -m -g users -G wheel,storage,power,video,audio,rfkill,input -s /bin/bash MYUSERNAME
```

Set the password of user `MYUSERNAME`:  

```
# passwd MYUSERNAME
```

### Add the new user to sudoers:

If you want a root privilege in the future by using the `sudo` command, you should grant one yourself:

```
# EDITOR=vim visudo
```

Uncomment the line (Remove #):

```
# %wheel ALL=(ALL) ALL
```

### Install the boot loader:  

Yeah, this is where we install the bootloader. We'll be using `systemd-boot`, so no need for `grub2`. 

+ Install bootloader:
	
	Of course, we will install it in `/boot` mountpoint (`/dev/sda1` partition).

	```
	# bootctl --path=/boot/ install
	```

+ Create a boot entry:  

	```
	# vim /boot/loader/entries/arch.conf
	```

	- Add these lines:

		+ Unencrypted filesystem

			```
			title Arch Linux  
			linux /vmlinuz-linux  
			initrd  /initramfs-linux.img  
			options root=/dev/sda3 rw
			```

			If your `root` is not in `/dev/sda3`, make sure to change it. 

			Save and exit.

		+ Encrypted filesystem - LUKS/LVM

			Remember the two-types of initramfs earlier? Each type needs a specific kernel parameters. So there's also a two type of entries here:

			- `udev`-based initramfs

				```
				title Arch Linux  
				linux /vmlinuz-linux  
				initrd  /initramfs-linux.img  
				options cryptdevice=UUID=/DEV/SDA2/UUID/HERE:volume root=/dev/mapper/volume-root rw
				```

				Replace `/DEV/SDA2/UUID/HERE` with the UUID of your `LVM` partition. You can check it by running `blkid /dev/sda2`. Note that `cryptdevice` parameter  is unsupported by plymouth so it's advisable to use `systemd`-based initramfs if you're planning to use it.

				Tip: If you're using `vim`, you can write the UUID easier by typing `:read ! blkid /dev/sda2` then hit enter.

			- `systemd`-based initramfs

				```
				title Arch Linux
				linux /vmlinuz-linux
				initrd /intel-ucode.img
				initrd /initramfs-linux.img
				options rd.luks.name=/DEV/SDA2/UUID/HERE=volume root=/dev/mapper/volume-root rw
				```

				Replace `/DEV/SDA2/UUID/HERE` with the UUID of your `LVM` partition. You can check it by running `blkid /dev/sda2`.

				Tip: If you're using `vim`, you can write the UUID easier by typing `:read ! blkid /dev/sda2` then hit enter.

+ Change loader

	```
	# vim /boot/loader/entries/arch.conf
	```

	Delete all of its content, then replaced it by:

	```
	default arch.conf
	timeout 0
	```

### Exit chroot and reboot:  

Exit the chroot environment by typing `exit` or pressing <kbd>Ctrl + d</kbd>. You can also unmount all mounted partition after this. 

Finally, `reboot`.

###  Finale

If your installation is a success, then ***yay!!!*** If not, you should start questioning your own existence. Are your parents proud of you? 

## POST INSTALLATION

In post installation we will be using a lot of `sudo`. I'm not responsible if you broke your newly installed system. Remember that this guide is *for* ***future me***.

### Connect to the internet

We will be downloading stuff so we need a connection! There's a guide up there on how to do this.

### Check for updates

It's recommended to check for updates first before installing anything so:

```
# pacman -Syu
```

### Xorg

I will be using `X`, will switch to wayland if there's awesomewm in it. :V

After booting up, you will be greeted by TTY. Still, there's no sign of GUI everywhere. So we need to install it.

Now that you have an internet connection, you need to install a graphical server - `xorg` or `wayland`. I'm using `awesomewm` as my window manager and it only runs on `X`, sooo I'll install `X`.

```
# pacman -S xorg-server xorg-xrdb xorg-xinit xorg-xrandr xorg-xev xorg-xdpyinfo xorg-xprop
```

#### Install video drivers

After installing the graphical server, we need to install the video drivers. I'm only using an integrated intel graphics card. **Sobs**


```
# pacman -S xf86-video-intel vulkan-intel vulkan-icd-loader libva-intel-driver
```

Add your (kernel) graphics driver to your initramfs. For example, if using `intel` add `i915`: 

```
# sudoedit /etc/mkinitcpio.conf
```

Then add `i915` to your initramfs - `/etc/mkinitcpio.conf`:

```
MODULES=(i915 ...)
```

#### Install audio drivers

```
# pacman -S alsa-utils pulseaudio-alsa pulseaudio-bluetooth pulseaudio pavucontrol
```

#### Install GTK

GTK, or the GIMP Toolkit, is a multi-platform toolkit for creating graphical user interfaces.

1. Install GTK

	```
	# pacman -S gtk3
	```

2. Install GTK engines

	```
	# pacman -S gtk-engine-murrine gtk-engines gnome-theme-extra
	```

#### Install file system tools and file manager

File system tools

```
# pacman -S unrar unzip p7zip gvfs-mtp libmtp ntfs-3g \
android-udev mtpfs xdg-user-dirs
```

File managers

```
# pacman -S dolphin dolphin-plugins kde-cli-tools ranger
```

`xdg-user-dirs` is a tool to help manage "well known" user directories like the desktop folder and the music folder so generate XDG user directories by:

```
# xdg-user-dirs-update
```

#### Terminal Emulator

We need a terminal emulator. Every linux user's first partner. Note that we're still on the `TTY`.

```
# pacman -S kitty xterm
```

#### Install GUI and CLI web browser

```
# pacman -S firefox w3m
```

#### Install git

Git is the version control system (VCS) designed and developed by Linus Torvalds, the creator of the Linux kernel. Git is now used to maintain AUR packages, as well as many other projects, including sources for the Linux kernel. 

```
#pacman -S git
```

#### Install an AUR helper

`yay` will be our AUR helper.

Clone `yay-bin` from the AUR.

```
$ git clone https://aur.archlinux.org/yay-bin.git
$ cd yay
$ makepkg -sri
```

#### Install missing kernel modules.

```
$ yay -S wd719x-firmware aic94xx-firmware --removemake --noconfirm
# mkinitcpio -p linux
```

#### Backlight Control

We'll use `light` for this because this is a better version of `xorg-xbacklight`.

```
$ yay -S light-git
```

#### GUI Text Editors

Install a GUI editor.

```
$ yay -S sublime-text-dev
```

#### GUI Environment

Install your preferred desktop environment or window manager. 

I'm using a window manager and it is `awesomewm`:

```
$ yay -S awesome-git --noconfirm --removemake
```

If I will be using a desktop environment, it will be `KDE Plasma`.

Install the `plasma-meta` meta-package or the `plasma` group. For differences between `plasma-meta` and `plasma` reference Package group. Alternatively, for a more minimal Plasma installation, install the `plasma-desktop` package.

```
# sudo pacman -S plasma-desktop
```

#### Install a compositor

I will be using `picom-git` from AUR.

```
$ yay -S picom-git --noconfirm --removemake
```

#### Install an application launcher. 

I will use `rofi`.

```
# pacman -S rofi
```

#### Plymouth

Plymouth provides a flicker-free graphical boot process. A splash screen or a boot screen.

1. Install plymouth.

	```
	$ yay -S plymouth-git
	```

2. Add `plymouth` to the HOOKS array in mkinitcpio.conf. It must be added after base and udev/systemd for it to work: 

	```
	#edit /etc/mkinitcpio.conf
	```

	- Unencrypted partition

		Put `plymouth` after base and udev:

		```
		HOOKS=(base udev plymouth ...)
		```

	- Encrypted partition and `systemd`-based initramfs.

		Again, if you're using `udev`-based initramfs, you will encounter a problem if you reboot your PC **lol**.

		Put `sd-plymouth` after base and systemd:

		```
		HOOKS=(base systemd sd-plymouth ...)
		```

3. Set the theme.
	
	List all the installed plymouth theme:

	```
	# plymouth-set-default-theme -l
	```

	Select a theme then it will rebuild the initramfs image:

	```
	# plymouth-set-default-theme -R THEME_NAME
	```

4. You now need to append `splash` in the kernel parameters - inside boot entry  `/boot/loader/enable/arch.conf`.

	Example:

	```
	title Arch Linux
	linux /vmlinuz-linux
	initrd /initramfs-linux.img
	options rd.luks.name=/DEV/SDA2/UUID/HERE=volume root=/dev/mapper/volume-root rw
	options quiet splash
	```

#### Silent boot

This is for who prefer to limit the verbosity of their system to a strict minimum, either for aesthetics or other reasons. For me, it's aesthetics. 

1. Edit boot loader kernel parameters:

	```
	# sudoedit /boot/loader/entries/arch.conf
	```

2. Add these parameters `(options ... loglevel=3 vga=current rd.udev.log_priority=3 fbcon=nodefer ...)` in the `options`:

	```
	options quiet loglevel=3 vga=current rd.udev.log_priority=3 fbcon=nodefer
	```

	If you're using `plymouth` and its `splash` kernel parameter, put `splash` after the `quiet` parameter.

#### Microcode

Processor manufacturers release stability and security updates to the processor microcode. These updates provide bug fixes that can be critical to the stability of your system. Without them, you may experience spurious crashes or unexpected system halts that can be difficult to track down. 

1. Install microcode.

	For AMD processors:

	```
	# pacman -S amd-ucode
	```

	For Intel processors:

	```
	# pacman -S intel-ucode
	```

	If your Arch installation is on a removable drive that needs to have microcode for both manufacturer processors, install both packages. 


2. Load  microcode. For systemd-boot, use the `initrd` option to load the microcode, before the initial ramdisk, as follows:

	```
	#edit /boot/loader/entries/entry.conf
	```

	```
	title   Arch Linux
	linux   /vmlinuz-linux
	initrd  /CPU_MANUFACTURER-ucode.img
	initrd  /initramfs-linux.img
	...
	```

	Change `CPU_MANUFACTURER` with either `amd` or `intel` depending on your processor.

#### Display Manager

A display manager, or login manager, is typically a graphical user interface that is displayed at the end of the boot process in place of the default shell.

There's a ton of display manager out there. I'm using `lightdm` with `lightdm-webkit2-greeter`, so this guide will cover that. 

1. First, install it:

	```
	# pacman -S lightdm lightdm-webkit2-greeter
	```

2. To enable graphical login, enable the appropriate systemd service. For example, for Lightdn, enable `lightdm.service`. Just because we're using plymouth, we will enable `lightdm-plymouth.service` to have a smooth transition from plymouth to lightdm.

	```
	# systemctl enable lightdm-plymouth
	```

3. Install a theme. I create my own lightdm-webkit2 theme and it's called [glorious](https://github.com/manilarome/lightdm-webkit2-theme-glorious).

	```
	$ yay -S lightdm-webkit2-theme-glorious
	```

4. Configure the lightdm to use lightdm-webkit2-greeter by:

	+ Set lightdm greeter session to webkit2.

		```
		#edit /etc/lightdm/lightdm.conf
		# Find `greeter-session` under the `[Seat:*]` section, uncomment it, then set its value to `lightdm-webkit2-greeter`.
		```

	+ Set it as the lightdm-webkit2 theme then enable `debug_mode` by setting it to `true`. Why do we need to enable `debug_mode`? Sometimes you will be greeted by an error. And this error is due to a race condition where the theme is trying to access the `lightdm` object even though it doesn't exist *yet*. Debug mode will allow you to `right-click` and `reload` the greeter just like a webpage.

		```
		#edit /etc/lightdm/lightdm-webkit2-greeter.conf
		```

		Find `webkit_theme` then set its value to `glorious`. Find `debug_mode` then set it to true. If you encountered an error, right-click then reload.

#### Reboot then Login

We're almost there! You can now login to you system with all the configuration we've done so far. 

```
$ reboot
```

#### Install and Configure Network Manager (Optional)

As of now, we're using `iwd` if we're using wireless connection and `dhcpcd` if we're on wired connection. Of course, if you're a minimalist, these tools are enough. But if you want a GUI network manager like `network-manager`:

1. Install network manager and its utilities.

	```
	# pacman -S networkmanager network-manager-applet dhclient modemmanager usb_modeswitch mobile-broadband-provider-info
	```

2. Disable `iwd` or `dhcpcd`.

	```
	# If using iwd
	# Disconnect from connection
	# iwctl station wlan0 disconnect
	# systemctl stop iwd
	# systemctl disable iwd

	# If using dhcpcd
	# systemctl stop dhcpcd
	# systemctl disable dhcpcd
	```

3. Enable Network Manager service.

	```
	# If using dhcpcd
	# systemctl enable NetworkManger
	# systemctl start NetworkManger
	```

	Sometimes you have to `reboot` if you cannot connect using the NetworkManger.

Optional:

Run `network-manager-applet` to have the network manager applet in you system tray. 

#### Enable MAC randomization

MAC randomization can be used for increased privacy by not disclosing your real MAC address to the network. 

+ Randomization for iwd

	1. Create and edit `/etc/iwd/main.conf`. Then add the following lines:

		```
		[General]
		AddressRandomization=once
		AddressRandomizationRange=nic
		```

+ Randomization for network-manager

	1. Install macchanger.

		```
		# pacman -S macchanger
		```

	2. Create `30-mac-randomization.conf` in your `/etc/NetworkManager/conf.d/`. Add this:

		```
		[device-mac-randomization]
		# "yes" is already the default for scanning
		wifi.scan-rand-mac-address=yes

		[connection-mac-randomization]
		ethernet.cloned-mac-address=random
		wifi.cloned-mac-address=stable
		```

#### Bluetooth Connection


1. Install the `bluez` package, providing the Bluetooth protocol stack.
2. Install the `bluez-utils` package, providing the `bluetoothctl` utility
3. The generic Bluetooth driver is the `btusb` Kernel module. Check whether that module is loaded. If it's not, then [load the module](https://wiki.archlinux.org/index.php/Kernel_module#Manual_module_handling).
4. Start/enable `bluetooth.service`.

```
# Install
# pacman -S bluez bluez-utils
# Start/Enable
# systemctl start bluetooth.service
# systemctl enable bluetooth.service
```

#### Bluetooth GUI

We'll use `blueman` for this. Blueman is a full featured Bluetooth manager written in GTK.

1. Install blueman.

	```
	# pacman -S blueman
	```

	Be sure to enable the Bluetooth daemon and start Blueman with `blueman-applet`. A graphical settings panel can be launched with `blueman-manager`.

2. Disable auto-power-on 

	Blueman automatically enables Bluetooth adapter () when certain events (on boot, laptop lid is opened, ...) occur. This can be disabled with the `auto-power-on` in org.blueman.plugins.powermanager: 

	```
	$ gsettings set org.blueman.plugins.powermanager auto-power-on false
	```

#### Authentication Managers

Polkit is used for controlling system-wide privileges. It provides an organized way for non-privileged processes to communicate with privileged ones. In contrast to systems such as sudo, it does not grant root permission to an entire process, but rather allows a finer level of control of centralized system policy.

Note that you don't need to bother with this if you installed a desktop environment earlier. I will install this because I'm using `awesomewm`, remember?

1. Install `polkit-kde-agent`/`lxqt-policykit`/`polkit-gnome`, and `gnome-keyring`.

	I'm using Qt apps, so I'll install lxqt-policykit. Note that you only need one authentication manager and gnome-keyring.

	```
	# pacman -S polkit polkit-kde-agent gnome-keyring
	```

2. Run it

	For lxqt-policykit, run:

	```
	$ /usr/bin/lxqt-policykit-agent
	```

	For polkit-kde-agent, run:

	```
	$ /usr/lib/polkit-kde-authentication-agent-1
	```

	For polkit-gnome:

	```
	$ /usr/lib/polkit-gnome/polkit-gnome-authentication-agent-1
	```

#### Better Power Management

1. Install and start/enable `tlp`:

	```
	# pacman -S tlp
	# systemctl enable tlp.service
	```

2. Install `upower`, `acpid` and `acpi_call`:

	Install:

	```
	# pacman -S acpid acpi_call upower
	```


	Enable acpid

	```
	# systemctl enable acpid.service
	```

#### Better Fan Control for Thinkpad

1. We'll use/install `thinkfan` here.

	```
	$ yay -S thinkfan
	```

	Note that the thinkfan package installs `/usr/lib/modprobe.d/thinkpad_acpi.conf`, which contains:

	```
	options thinkpad_acpi fan_control=1
	```

	So fan control is enabled by default. Alternatively, you can enable fan control as follows: 

	```
	# echo "options thinkpad_acpi fan_control=1" > /etc/modprobe.d/thinkfan.conf
	```

2. Now, load the module: 

	```
	# modprobe thinkpad_acpi
	# cat /proc/acpi/ibm/fan
	```

	You should see that the fan level is "auto" by default, but you can echo a level command to the same file to control the fan speed manually. The thinkfan daemon will do this automatically.

3. Open `/etc/thinkfan.conf`. Then use the following configuration: 

	```
	tp_fan /proc/acpi/ibm/fan
	hwmon /sys/class/thermal/thermal_zone0/temp

	(0, 0,  60)
	(1, 53, 65)
	(2, 55, 66)
	(3, 57, 68)
	(4, 61, 70)
	(5, 64, 71)
	(7, 68, 32767)
	```

4. Finally, enable the thinkfan systemd service: 

	```
	# systemctl enable thinkfan.service
	```

#### Firewall

We'll use `Uncomplicated Firewall` or `ufw` for short.

1. Install the `ufw` package. Start and enable `ufw.service` to make it available at boot. Note that this will not work if `iptables.service` is also enabled (and same for its ipv6 counterpart). 

	```
	# pacman -S ufw
	# systemctl start ufw.service
	# systemctl enable ufw.service
	```

2. Configuration

	Here's some basic configuration. A very simplistic configuration which will deny all by default, allow any protocol from inside a 192.168.0.1-192.168.0.255 LAN, and allow incoming Deluge and rate limited SSH traffic from anywhere: 

	```
	# ufw default deny
	# ufw allow from 192.168.0.0/24
	# ufw allow Deluge
	# ufw limit ssh
	```

3. The next line is only needed once the first time you install the package: 

	```
	# ufw enable
	```

Adding other applications. The PKG comes with some defaults based on the default ports of many common daemons and programs. Inspect the options by looking in the `/etc/ufw/applications.d` directory or by listing them in the program itself: 

```
# ufw app list
```

#### And that's it! We covered all the "must-have" part! Now, it's time for basic ricing. :)

### Basic Ricing

My [setups](https://github.com/manilarome/the-glorious-dotfiles) are mostly transparent and blurred, that's why I mostly use Qt5 apps. Still, we have to use and rice GTK apps because not everything has an alternative Qt version.
Let's start with installing some GTK and Qt system settings and and its themes.

#### GTK3 Theming

If we want to change our system font, GTK3, icon and cursor theme. We need a settings application. So we need to install that.

1. Install `lxappearance-gtk3`.

	```
	# pacman -S lxappearance-gtk3
	```

2. After that we need to install a `GTK3` theme.
	
	I love dark theme so let's install a dark GTK3 theme. I'll use the Orchis theme

	```
	$ yay -S orchis-theme-git
	```

3. Set your GTK3 theme in lxappearance.
	
	```
	$ lxappearance
	```

#### Qt5 Theming

1. Install `qt5ct` and `qt5-styleplugins`. We need these tools to [configure Qt5 apps under environments other than KDE Plasma](https://wiki.archlinux.org/index.php/Qt#Configuration_of_Qt5_apps_under_environments_other_than_KDE_Plasma). `qt5ct` provides a Qt5 QPA independent of the desktop environment and a configuration utility.

2. Set the environment variable `QT_QPA_PLATFORMTHEME="qt5ct"` so that the settings are picked up by Qt applications. You can set the environment variable in your `~/.xprofile` and `/etc/environment` for global settings.

3. Log out, so the environment variable can take effect. Log in. 

4. Install `kvantum-qt5`. Kvantum manager is an SVG-based theme engine for Qt5. This will be the main tool that will provide us the full blur effect and those sweet eyecandies.

5. Install a kvantum theme. I recommend the [Inverse-dark](https://store.kde.org/p/1365482/) theme, because it's clean and modern looking. An edited version is in my [dotfiles](https://github.com/manilarome/the-glorious-dotfiles/tree/master/config/Kvantum/Inverse-dark).

6. Open `kvantummanager` to set your desired kvantum theme. You can configure the active theme to make some changes like making the dolphin view transparent, disable tooltip shadows, etc., etc.

6. Run `qt5ct` again. Change the `style` to `kvantum` to use the kvantum theme. Hit `OK`.

7. Open a Qt5 app. Enjoy.


#### Install an Icon theme

A part of basic ricing is improving the icon theme.

1. Install an icon theme. My main icon theme is [`Tela`](https://github.com/vinceliuice/Tela-icon-theme), but not all the icons are covered by it so I'm also using [`Papirus`](https://github.com/PapirusDevelopmentTeam/papirus-icon-theme) as *inherits*.

	Install Papirus and Tela icon theme

	```
	$ yay -S papirus-icon-theme tela-icon-theme
	```

2. Add `Papirus` as inherits of `Tela`.

	```
	$ sudoedit /usr/share/icons/Tela/index.theme
	```

	Find `Inherits` then add `Papirus`.

	```
	Inherits=Papirus,Papirus-Dark,hicolor
	```

3. Set your theme in `lxappearance` and `qt5ct`.

#### Font Installation

Let's be honest, font rendering in Linux *is not that* good by default. So let's make them great again! Let's start with installing some beautiful fonts.

Basic fonts

```
# pacman -S ttf-dejavu ttf-liberation noto-fonts noto-fonts-emoji
```

Inter font as system font

```
# pacman -S inter-font ttf-roboto
```

#### Improve Font Rendering

Make your system fonts great again! Improve your fonts for system-wide usage without installing a patched font library packages like the `Infinality`.

1. If you haven't already, install these fonts:

	```
	# pacman -S ttf-dejavu ttf-liberation noto-fonts inter-font
	```

2. Enable font presets by creating symbolic links:

	```
	# ln -s /etc/fonts/conf.avail/70-no-bitmaps.conf /etc/fonts/conf.d
	# ln -s /etc/fonts/conf.avail/10-sub-pixel-rgb.conf /etc/fonts/conf.d
	# ln -s /etc/fonts/conf.avail/11-lcdfilter-default.conf /etc/fonts/conf.d
	```

	The above will disable embedded bitmap for all fonts, enable sub-pixel RGB rendering, and enable the LCD filter which is designed to reduce colour fringing when subpixel rendering is used.

	For font consistency, all applications should be set to use the serif, sans-serif, and monospace aliases, which are mapped to particular fonts by fontconfig.

3. Create `/etc/fonts/local.conf`, then add:

	```xml
	<?xml version="1.0"?>
	<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
	<fontconfig>
			<match>
					<edit mode="prepend" name="family">
						<string>Inter</string>
					</edit>
			</match>
			<match target="pattern">
					<test qual="any" name="family">
							<string>serif</string>
					</test>
					<edit name="family" mode="assign" binding="same">
							<string>Noto Serif</string>
					</edit>
			</match>
			<match target="pattern">
					<test qual="any" name="family">
						<string>sans-serif</string></test>
					<edit name="family" mode="assign" binding="same">
							<string>Noto Sans</string>
					</edit>
			</match>
			<match target="pattern">
					<test qual="any" name="family">
						<string>monospace</string>
					</test>
					<edit name="family" mode="assign" binding="same">
							<string>Noto Mono</string>
					</edit>
			</match>
	</fontconfig>
	```

4. Set your font settings to match above in your DE system settings. Open `qt5ct` and `lxappearance` then set the font to your preference.

You can also follow this [guide](https://jichu4n.com/posts/how-to-set-default-fonts-and-font-aliases-on-linux/).

#### Install a Cursor theme

Let's make the cursor theme more modern looking!

[`Vimix Cursor`](https://github.com/vinceliuice/Vimix-cursors) is a beautiful theme! It can be found in the AUR.

1. Install the theme.

	```
	$ yay -S vimix-cursors --removemake --noconfirm
	```

2. Set the theme in your `qt5ct` and `lxappearance`.

#### Improve Terminal Experience with FISH

`fish` is better than `zsh`, in my honest opinion. Why? The defaults are great! No need to install a package or a plugin to have, for example autocompletion, etc. In it's snappy af! A highly recommended shell!

0. Before starting, we may want to see what shell is currently being used: 

	```
	$ echo $SHELL
	```

1. Not using FISH? Then install the `fish` package.

	```
	# pacman -S fish
	```

2. Make sure that fish has been installed correctly by running the following in a terminal: 

	```
	$ fish
	```

3. Set `fish` as interactive shell. We will still use `bash` as the default shell. Not setting fish as system wide or user default allows the current Bash scripts to run on startup. It ensures the current user's environment variables are unchanged and are exported to fish which then runs as a Bash child. Add this on your `~/.bashrc`.

	```
	if [[ $(ps --no-header --pid=$PPID --format=cmd) != "fish" ]]
	then
		exec fish
	fi
	```

4. Install `pkgfile` to enable `Command not found` hook.

	```
	# pacman -S pkgfile
	# Update pkgfile databse
	# pkgfile --update
	```

5. Update autosuggestion database by running:
	
	```
	$ fish_update_completions
	```

#### Install `[oh-my-fish](https://github.com/oh-my-fish/oh-my-fish)`

After rebooting, let's now improve the `fish` experience.

1. Install oh-my-fish.

	```
	$ curl -L https://get.oh-my.fish | fish
	```

2. Install a prompt theme. There's a bunch of themes available that can be found [here](https://github.com/oh-my-fish/oh-my-fish/blob/master/docs/Themes.md). Speaking of themes, I created my own and you can find it [here](https://github.com/manilarome/fishblocks). Install the theme you want, then move on. Installing a theme from `omf` is easy, just run `omf install THEMENAME` and it will do its job.

3. Install some useful plugins.
	
	```
	$ omf install archlinux bang-bang cd colorman sudope vcs
	```

	colorman plugin need to be source:

	```
	$ echo "source ~/.local/share/omf/pkg/colorman/init.fish" >> ~/.config/fish/config.fish
	```
	
4. Of course, you probably need a nerd font or something if the prompt is "broken". MesloLGS NF is recommended. You can download and install it from the [AUR](ttf-meslo-nerd-font-powerlevel10k) or manually [here](https://github.com/romkatv/powerlevel10k#meslo-nerd-font-patched-for-powerlevel10k).

#### Improve Terminal Experience with ZSH

Improve your terminal experience! Let's replace `bash` with `zsh`. `Zsh` is a powerful shell that operates as both an interactive shell and as a scripting language interpreter. 

0. Before starting, we may want to see what shell is currently being used: 

	```
	$ echo $SHELL
	```

1. Not using ZSH? Then install the `zsh` package. For additional completion definitions, install the `zsh-completions` package as well. 

	```
	# pacman -S zsh zsh-completions
	```

2. Make sure that Zsh has been installed correctly by running the following in a terminal: 

	```
	$ zsh
	```

	You should now see *zsh-newuser-install*, which will walk you through some basic configuration. If you want to skip this, press <kbd>q</kbd>. If you did not see it, you can invoke it manually with: 

	```
	$ autoload -Uz zsh-newuser-install
	$ zsh-newuser-install -f
	```

	**Note:**  
	Make sure your terminal's size is at least 72×15 otherwise *zsh-newuser-install* will not run.

3. Change your `$SHELL` from whatever you're using right now to `ZSH`.
	
	User:

	```
	$ chsh -s $(which zsh)
	```

	System-wide:

	```
	# chsh -s $(which zsh)
	```

	It's better if you `reboot` your system.


#### Install `oh-my-zsh`

After rebooting, let's now improve the `zsh` prompt.

1. Install `oh-my-zsh`:

	```
	$ sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
	```

	After this open a terminal then a guide will walk you through some basic configuration.

#### Improve Terminal Prompt using PowerLevel10k

1. Download the recommended font manually [here](https://github.com/romkatv/powerlevel10k#meslo-nerd-font-patched-for-powerlevel10k). Or install it from AUR:

	```
	$ yay -Syu ttf-meslo-nerd-font-powerlevel10k
	```


2. If you downloaded the font manually, copy the downloaded font to the right font directory.

	User only. If the folder doesn't exist, create it.

	```
	$ cp FONTNAME.TTF "${HOME}"/.local/share/fonts/TTF/
	```

	System-wide:

	```
	# cp FONTNAME.TTF /usr/share/fonts/TTF/
	```

3. Rebuild font information cache files

	```
	$ fc-cache
	```

4. Change your terminal font to `MesloLGS NF`.

5. Install the `Powerlevel10k` theme.

	```
	$ git clone --depth=1 https://github.com/romkatv/powerlevel10k.git $ZSH_CUSTOM/themes/powerlevel10k
	```

6. Open `~/.zshrc` with your editor

	```
	$ $EDITOR $HOME/.zshrc
	```

	Find the line `ZSH_THEME=robbyrussell` it's not easy to miss. Then change it to `ZSH_THEME=powerlevel10k/powerlevel10k`.

7. Open a terminal. There should be some instructions/dialog there that will greet you and will guide you to theme your Powerlevel10k prompt.

More info about Powerlevel10k [here](https://github.com/romkatv/powerlevel10k).

#### Use `gtk3-classic` a patched GTK3

One of the things I don't like is using GTK3 apps that have a CSD or client-side decoration with a window manager. It removes the unified look because some apps have CSDs and some don't. That's why I'm using a patched version of `GTK3` called [`gtk3-mushrooms`](https://github.com/krumelmonster/gtk3-mushrooms). `gtk3-mushrooms` is a set of patches for GTK3 library that makes it better.

Install `gtk3-classic` a package based on `gtk3-mushrooms` that contains additional patches that will replace remove `gtk3`'s CSDs.

```
$ yay -S gtk3-classic
```

Notable changes after installing `gtk3-classic`:

+ CSDs are totally disabled by default.
+ Typeahead feature is restored.
+ Message dialogs have traditional appearance with left-aligned texts and right-aligned buttons.
+ Scrollbars are always visible. You can enable invisible scrollbars by `GTK_OVERLAY_SCROLLING=1` environment variable.
+ Labels are wrapped similarly to GTK2. This patch fixes too wide windows in applications improperly ported from GTK2.
+ Any many more!

So the real question is why did the `GNOME` team keeps removing the best features and adding things nobody ask for? Just kidding!

#### We're done ricing!

### Install daily applications

#### Media

**`vlc`** as video player  
**`mpd`** as music server  
**`mpc`** as minimalist command line interface to `mpd`  
**`ncmpcpp`** as fully featured `mpd` client using ncurses  
**`feh`** as image viewer  
**`perl-image-exiftool`** as reader and rewriter of EXIF informations that supports raw files  
**`simplescreenrecorder`** as screen recorder  
**`pulseeffects`** as equalizer  
**`pavucontrol-qt`** as pulseaudio mixer

#### Network tools

**`aircrack-ng`** as deauth program  
**`fluxion`** as  MiTM program  
**`create_ap`** as NATed/Bridged Software Access Point

#### Terminal eyecandies

**`neofetch`** as a CLI system information tool  
**`cava`** as audio visualizer  
**`pipes.sh`** as animated pipes terminal screensaver  
**`cmatrix`** as matrix-like terminal screensaver  

#### Media Editors

**`gimp`** as image manipulation program  
**`inkscape`** as vector graphics editor  
**`ffmpeg`** as video converter  
**`imagemagick`** as image viewing/manipulation program  

#### Basic tools

**`maim`** as screenshot tool  
**`mupdf`** as pdf viewer  
**`redshift`** as the blue light filter  
**`arandr`** as xrandr's face  
**`mugshot`** as personal user details updater  
**`pamac-aur`** as frontend for libalpm  
**`toilet`** as ascii generator tool  
**`dconf-editor`** as dconf editor   
**`tranmission-qt`** as BitTorrent client
**`htop`** as interactive process viewer

#### Hotel

**`trivago`** as hotel
