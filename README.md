
# A PERSONAL Arch Installation Guide


#### This is a personal guide so if you're lost and just found this guide from somewhere, I recommend you to read the official [`wiki`](https://wiki.archlinux.org/index.php/Installation_guide)! This guide will focus on systemd-boot and UEFI setups, so we will not use `grub`. 

Arch changed a lot of things in packaging like removing a lot of packages in base group. I will mostly forgot those yet again, so I need this guide for myself.

I'm using GPT and systemd-boot in my installation. My system is `Lenovo X230`, so this guide will not cover the installation of proprietary drivers. **🖕, Nvidia** - Linus Torvalds.

### Set the keyboard layout
The default console keymap is US. Available layouts can be listed with:

```bash
$ ls /usr/share/kbd/keymaps/**/*.map.gz
```

To modify the layout, append a corresponding file name to loadkeys, omitting path and file extension. For example, to set a US keyboard layout:  

```bash
$ loadkeys us
```


### Update the system clock
Use timedatectl to ensure the system clock is accurate:

```bash
$ timedatectl set-ntp true
```

### Disk Partitioning
**Verify the boot mode**
If UEFI mode is enabled on an UEFI motherboard, Archiso will boot Arch Linux accordingly via systemd-boot. To verify this, list the efivars directory:  

```bash
$ ls /sys/firmware/efi/efivars
```

**Check the drives**
The most common main drive is **sda**.

```bash
$ lsblk
```

**Let’s clean up our main drive to create new partitions for our installation**  

```bash
$ gdisk /dev/sda
```

	Press <kbd>x</kbd> to enter **expert mode**. Then press <kbd>z</kbd> to *zap* our drive. Then hit <kbd>y</kbd> when prompted about wiping out GPT and blanking out MBR.  
	
**NOTE:**  
Remember that pressing <kbd>x</kbd> button will put you to *expert mode* and <kbd>z</kbd> will *zap* or wipe-out the drive. **Think carefully before pressing <kbd>Y</kbd> you can't undo this!**

**Now let’s create our partitions**  

```bash
$ cgdisk
```
Enter the device filename of your main drive, e.g. `/dev/sda`. Just press <kbd>Return</kbd> when warned about damaged GPT.

Now we should be presented with our main drive showing the partition number, partition size, partition type, and partition name. If you see list of partitions, delete all those first.

**Let’s create our boot partition**  

	 + Hit New from the options at the bottom.
	 + Just hit enter to select the default option for the first sector.
	 + Now the partion size - Arch wiki recommends 200-300 MB for the boot + size. Let’s make it 500MiB or 1GB in case we need to add more OS to our machine. I’m gonna assign mine with 1024MiB. Hit enter.
	 + Set GUID to **EF00**. Hit enter.
	 + Set name to boot. Hit enter.
	 + Now you should see the new partition in the partitions list with a partition type of EFI System and a partition name of boot. You will also notice there is 1007KB above the created partition. That is the MBR. Don’t worry about that and just leave it there.

**Create the swap partition**

	 + Hit New again from the options at the bottom of partition list.
	 + Just hit enter to select the default option for the first sector.
	 + For the swap partition size, it is advisable to have 1.5 times the size of your RAM. I have 8GB of RAM so I’m gonna put 12GiB for the partition size. Hit enter.
	 + Set GUID to **8200**. Hit enter.
	 + Set name to swap. Hit enter.

**Create the root partition**

	 + Hit New again.
	 + Hit enter to select the default option for the first sector.
	 + Hit enter again to input your root size.
	 + Also hit enter for the GUID to select default.
	 + Then set name of the partition to root.


**Create the home partition**

	 + Hit New again.
	 + Hit enter to select the default option for the first sector.
	 + Hit enter again to use the remainder of the disk.
	 + Also hit enter for the GUID to select default.
	 + Then set name of the partition to home.

Lastly, hit **Write** at the bottom of the patitions list to *write the changes* to the disk. Type `yes` to *confirm* the write command. Now we are done partitioning the disk. Hit **Quit** *to exit cgdisk*.


### FORMATTING PARTITIONS

**Let’s see the new partition list**

```bash
$ lsblk
```

You should see something like this:

| NAME | MAJ:MIN | RM | SIZE | RO | TYPE | MOUNTPOINT |
| --- | --- | --- | --- | --- | --- | --- |
| sda | 8:0 | 0 | 477G | 0 | disk |   |
| sda1 | 8:1 | 0 | 1 | 0 | part | /boot |
| sda2 | 8:2 | 0 | 1 | 0 | part | [SWAP] |
| sda3 | 8:3 | 0 | 175G | 0 | part | / |
| sda4 | 8:4 | 0 | 300G | 0 | part | /home  |

**`sda`** is the hard drive  
**`sda1`** is the boot partition  
**`sda2`** is the swap partition  
**`sda3`** is the home partition  
**`sda4`** is the root partition  

**Format the boot partition as FAT32**  

```bash
$ mkfs.fat -F32 /dev/sda1
```  

**Enable the swap partition**  

```bash
$ mkswap /dev/sda2  
$ swapon /dev/sda2
```  

**Format home and root partition as ext4**  

```bash
$ mkfs.ext4 /dev/sda3  
$ mkfs.ext4 /dev/sda4
```  

### Connect to internet

We need to make sure that we are connected to the internet to be able to install Arch Linux `base` and `linux` packages. Let’s see the names of our interfaces.

```bash
$ ip link
```

You should see something like this:
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
		link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s30: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN mode DEFAULT group default qlen 1000
		link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff
3: wlp7s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DORMANT group default qlen 1000
		link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff permaddr 00:00:00:00:00:00
```

**`enp0s30`** is the wired interface  
**`wlp7s0`** is the wireless interface  

If you are on a wired connection, you can enable your wired interface by systemctl start dhcpcd@<interface>.  

```
$ systemctl start dhcpcd@enp3s0
```

If you are on a laptop, you can connect to a wireless access point using wifi-menu -o <wireless_interface>.

```
$ systemctl enable netctl  
$ wifi-menu wlp7s0
```  

Ping google to make sure we are online:

```
$ ping -c 3 google.com
``` 

If you receive Unknown host or Destination host unreachable response, means you are not online yet. Review your network configuration and redo the steps above.

After setting up the drive and connecting to the internet, we are now ready to install Arch Linux to our system.

First we need to mount the partitions that we created earlier to their appropriate mount points.

### Mounting partitions

Mount the **`root`** partition:  

```
$ mount /dev/sda3 /mnt
```  

**If `/mnt` directory doesn't exist create it.**

```bash
$ mkdir /mnt
```  

Create and mount the **`boot`** partition  

```bash
$ mkdir /mnt/boot  
$ mount /dev/sda1 /mnt/boot
```  

Create and mount the **`home`** partion  

```bash
$ mkdir /mnt/home
$ mount /dev/sda4 /mnt/home
``` 

We don’t need to mount **`swap`** since it is already enabled.  

### Installing the base and linux packages

Now let’s go ahead and install **`base`**, **`linux`** and **`base-devel`** packages into our system. The kernel is not included in the base group including other application like editors. Even the lovely **`nano`** doesn't survived the snap.  
So let's also install the packages detached from the base group.  

```bash
$ pacstrap /mnt base linux linux-firmware base-devel less logrotate man-db man-pages which dhcpcd netctl inetutils jfsutils mdadm perl reiserfsprogs sysfsutils systemd-sysvcompat texinfo usbutils xfsprogs s-nail nano iputils
```

It's your decision if you want to all install the packages.

Just hit enter/yes for all the prompts that come up. Wait for Arch to finish installing the base, linux and base-devel and other packages.  


### Generating the fstab

```bash
$ genfstab -U /mnt >> /mnt/etc/fstab
```  


### Chroot time

Now, chroot to the newly installed system  

```bash
$ arch-chroot /mnt /bin/bash
```

### Configure the time

A selection of timezones can be found under `/usr/share/zoneinfo/`. Since I am in the Philippines, I will be using `/usr/share/zoneinfo/Asia/Manila`. Select the appropriate timezone for your country:  

```bash
$ ln -s /usr/share/zoneinfo/Asia/Manila /etc/localtime
```

Run hwclock to generate `/etc/adjtime`: 

```bash
$ hwclock --systohc
```

### Localization

The Locale defines which language the system uses, and other regional considerations such as currency denomination, numerology, and character sets. Possible values are listed in `/etc/locale.gen`. Uncomment `en_US.UTF-8`, as well as other needed localisations.

**Uncomment** `en_US.UTF-8 UTF-8` and other needed locales in `/etc/locale.gen`, **save**, and generate them with:  

```bash
$ locale-gen
```

Create the locale.conf file, and set the LANG variable accordingly:  

```bash
$ echo 'LANG=en_US.UTF-8' > /etc/locale.conf
```

If you set the keyboard layout, make the changes persistent in vconsole.conf:  

```bash
echo 'KEYMAP=us' > /etc/vconsole.conf
```


### Network configuration

Create the hostname file:  

```bash
$ echo 'MYHOSTNAME' > /etc/hostname
```

Open/Create hosts file to add matching entries to hosts:  

```bash
nano /etc/hosts
```  

Add this:  

```bash
127.0.0.1    localhost  
::1          localhost  
127.0.1.1    MYHOSTNAME.localdomain	  MYHOSTNAME  
```

**NOTE:**  
If the system has a permanent IP address, it should be used instead of 127.0.1.1.

### Initramfs  

Creating a new initramfs is usually not required, because mkinitcpio was run on installation of the kernel package with pacstrap.
For LVM, system encryption or RAID, modify mkinitcpio.conf and recreate the initramfs image:  

```
$ mkinitcpio -p linux
```

### Wireless Connections

For wireless connections, install iw, wpa_supplicant, and (for wifi-menu) dialog. This is needed if you want to use a wi-fi connection after the next reboot!

```
$ pacman -S iw wpa_supplicant dialog
```

### Adding Repositories

Enable multilib and AUR repositories in `/etc/pacman.conf:`  

```bash
$ nano /etc/pacman.conf
```

Uncomment multilib (remove # from the beginning of the lines):  

```
[multilib]
Include = /etc/pacman.d/mirrorlist
```

Add the following lines at the end of `/etc/pacman.conf` to enable AUR repo:  

```
[archlinuxfr]
SigLevel = Never
Server = http://repo.archlinux.fr/$arch
```

**Pacman Easter Eggs**

While you're at it, **you can add colors and PAC-MAN in pacman!**

Open `/etc/pacman.conf`, then find `# Misc options`. After that, **below** that, add:

```
Color
ILoveCandy
```

### Passwords and Users

Set the root password:  

```bash
$ passwd
```

Add a new user with username *MYUSERNAME*:  

```bash
$ useradd -m -g users -G wheel,storage,power,video,audio,rfkill -s /bin/bash MYUSERNAME
```  

Set password the password for user MYUSERNAME:  

```bash
$ passwd MYUSERNAME
```


### Add the new user to sudoers:

If you want a root power by using the command `sudo`, you should grant your self with that power.

```bash
$ EDITOR=nano visudo
```

Uncomment the line (Remove #):

```bash
# %wheel ALL=(ALL) ALL
```

### Install the boot loader:  

Yeah, this is where we install the bootloader. We'll be using **`systemd-boot`**, so no need for `grub2`.

```bash
$ bootctl install
```

Create a boot entry:  

```bash
$ nano /boot/loader/entries/arch.conf
```

Add these lines:

```bash
title Arch Linux  
linux /vmlinuz-linux  
initrd  /initramfs-linux.img  
options root=/dev/sda3 rw
```

Save and exit.

### Exit chroot and reboot:  

```bash
$ exit
$ reboot
```


## POST INSTALLATION

#### Install X server

After booting up, you will be greeted by TTY. Still, there's no sign of GUI everywhere. So we need to install it. But before that, we need an internet connection! Follow this [guide](#connect-to-internet). Make sure to open the guide in a new tab, otherwise this page will scroll up instead and I think you don't want that.

Now that you have an internet connection, you need to install a graphical server - `xorg` or `wayland`. I'm using `AwesomeWM` as my window manager and it only runs on `X`, sooo let's install `X`.

```bash
$ sudo pacman -S xorg-server xorg-xrdb xorg-xinit xorg-xrandr xorg-xev xorg-xdpyinfo xorg-xprop
```

#### Install video drivers

After installing the graphical server, we need to install the video drivers. I'm using an integrated intel graphics card.

```bash
$ sudo pacman -S xf86-video-intel vulkan-intel vulkan-intel lib32-vulkan-intel vulkan-icd-loader lib32-vulkan-icd-loader lib32-mesa
```

Add your (kernel) graphics driver to your initramfs. For example, if using intel add **`i915`**: 

```bash
# We will use sudoedit to edit text files with higher privileges
# This will not work(probably) if you don't set your $EDITOR
# You can also use `sudo nano` for simplicity's sake
$ sudoedit /etc/mkinitcpio.conf
```

Then add `i915` to your initramfs:

```
MODULES=(i915 ...)
```

#### Install audio drivers

```bash
$ sudo pacman -S alsa-utils pulseaudio-alsa pulseaudio-bluetooth pulseaudio pavucontrol pulseeffects
```

#### Install file system tools and file manager

```bash
# File system tools
$ sudo pacman -S unrar unzip p7zip gvfs-mtp libmtp ntfs-3g android-udev ffmpegthumbnailer mtpfs 
# File managers
$ sudo pacman -S dolphin ranger
```

#### Install GUI and CLI web browser

```bash
$ sudo pacman -S firefox w3m
```

#### Terminal Emulator

Of course, we need a terminal emulator

```bash
$ sudo pacman -S kitty xterm
```

#### Install git

```bash
$ sudo pacman -S git
```

#### Install an AUR helper

`yay` will be our AUR helper. `go` and `base-devel` is a dependency of `yay`, so make sure you have it first. To install `yay`:

```bash
# Install the go package
$ sudo pacman -S go
# Clone the repo
$ git clone https://aur.archlinux.org/yay.git
$ cd yay
# Compile, build and install
$ makepkg -sri
```

#### GUI Environment

Install your desktop environment or window manager. I'm using a window manager and it is `awesomewm`.

```bash
$ yay -S awesome-git --noconfirm --removemake
```

#### Install a compositor

I will be using `picom-tryone-git` for AUR.

```bash
$ yay -S picom-tryone-git --noconfirm --removemake
```

#### Install an application launcher. 

We will use `rofi-git`.

```bash
$ yay -S rofi-git --noconfirm --removemake
```

#### Plymouth

Plymouth provides a flicker-free graphical boot process.

```bash
# Install
$ yay -S plymouth-git
# If you also use GDM, you should install the gdm-plymouth, which compiles gdm with plymouth support. 
$ yay -S gdm-plymouth
```

Add `plymouth` to the HOOKS array in mkinitcpio.conf. It must be added after base and udev for it to work: 

```bash
$ sudoedit /etc/mkinitcpio.conf
```

Put `plymouth` after base and udev:

```
HOOKS=(base udev plymouth ...)
```

Set the theme.

```bash
# List all the installed plymouth theme
$ sudo plymouth-set-default-theme -l
# Select a theme then rebuild  initrd
$ sudo plymouth-set-default-theme -R theme_name
```

You now need to append `splash` in the kernel parameters. See [Silent Boot](#silent-boot).

#### Install missing kernel modules.

```bash
$ yay -S wd719x-firmware aic94xx-firmware --removemake --noconfirm
$ sudo mkinitcpio -p linux
```

#### Silent boot

This is for who prefer to limit the verbosity of their system to a strict minimum, either for aesthetics or other reasons. For me, it's aesthetics. 

Edit boot loader kernel parameters:

```bash
$ sudoedit /boot/loader/entries/arch.conf
```

Add these lines at the bottom of the file then save:

```
options quiet splash loglevel=3 vga=current rd.udev.log_priority=3
```

#### Microcode

Processor manufacturers release stability and security updates to the processor microcode. These updates provide bug fixes that can be critical to the stability of your system. Without them, you may experience spurious crashes or unexpected system halts that can be difficult to track down. 

```bash
# For AMD processors
$ sudo pacman -S amd-ucode
# For intel processors
$ sudo pacman -S intel-ucode
```

If your Arch installation is on a removable drive that needs to have microcode for both manufacturer processors, install both packages. 

Load  microcode. For systemd-boot, use the `initrd` option to load the microcode, before the initial ramdisk, as follows:

```bash
$ sudoedit /boot/loader/entries/entry.conf
```

```
title   Arch Linux
linux   /vmlinuz-linux
initrd  /cpu_manufacturer-ucode.img
initrd  /initramfs-linux.img
...
```

Change `cpu_manufacturer-ucode` with either `amd` or `intel` depending on your processors.

#### Display Manager

A display manager, or login manager, is typically a graphical user interface that is displayed at the end of the boot process in place of the default shell.

There's a ton of display manager out there. I'm using sddm so this guide will cover that. First, install it:

```bash
$ sudo pacman -S sddm
```

To enable graphical login, enable the appropriate systemd service. For example, for SDDM, enable `sddm.service`. Just because we're using plymouth, we will enable `sddm-plymouth.service` to have a plymouth support.

Install a theme. I'm using the `Sugar Candy` theme.

```bash
$ yay -S sddm-theme-sugar-candy-git
```

Configure the theme by:

```bash
$ sudoedit /usr/share/sddm/themes/Sugar-Candy/theme.conf
```

#### Reboot then Login

We're almost there! You can now login to you system with all the configuration we've done so far. 

```bash
$ reboot
```

#### Install and Configure Network Manager

As of now, we're using `netctl` if we're using wireless connection and `dhcpcd` if we're on wired connection. So it's time to install a GUI to connect to the internet.

```bash
$ sudo pacman -S networkmanager network-manager-applet dhclient modemmanager usb_modeswitch mobile-broadband-provider-info
```

Disable netctl or dhcpcd.

```bash
# If using netctl
# Stop current connection
$ sudo netctl stop profile_name
$ sudo netctl disable profile_name
$ sudo systemctl stop netctl
$ sudo systemctl disable netctl

# If using dhcpcd
$ sudo systemctl stop dhcpcd
$ sudo systemctl disable dhcpcd
```

Enable Network Manager service.

```bash
# If using dhcpcd
$ sudo systemctl enable NetworkManger
$ sudo systemctl start NetworkManger
```

Sometimes you have to reboot if you cannot connect using the NetworkManger.

Optional:

Run `network-manager-applet` to have the network manager applet in you system tray. 

Enable MAC randomization. MAC randomization can be used for increased privacy by not disclosing your real MAC address to the network. 

Install macchanger.

```bash
$ sudo pacman -S macchanger
```

Create `30-mac-randomization.conf` in your `/etc/NetworkManager/conf.d/`. Add this:

```
[device-mac-randomization]
# "yes" is already the default for scanning
wifi.scan-rand-mac-address=yes

[connection-mac-randomization]
ethernet.cloned-mac-address=random
wifi.cloned-mac-address=stable
```


#### Font Installation


#### Improve Font Rendering

Make your system fonts great again! Improve your fonts for system-wide usage without installing a patched font library packages like the `Infinality`.

Install these fonts, for example:

```bash
$ sudo pacman -S ttf-dejavu ttf-liberation noto-fonts
```

Enable font presets by creating symbolic links:

```bash
$ sudo ln -s /etc/fonts/conf.avail/70-no-bitmaps.conf /etc/fonts/conf.d
$ sudo ln -s /etc/fonts/conf.avail/10-sub-pixel-rgb.conf /etc/fonts/conf.d
$ sudo ln -s /etc/fonts/conf.avail/11-lcdfilter-default.conf /etc/fonts/conf.d
```

The above will disable embedded bitmap for all fonts, enable sub-pixel RGB rendering, and enable the LCD filter which is designed to reduce colour fringing when subpixel rendering is used.

For font consistency, all applications should be set to use the serif, sans-serif, and monospace aliases, which are mapped to particular fonts by fontconfig.

Create `/etc/fonts/local.conf`, then add:

```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
		<match>
				<edit mode="prepend" name="family">
					<string>SF Pro Text</string>
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

Set your font settings to match above in your DE system settings.



| Network tools |
| --- |
| aircrack-ng |

| Maintenance |
| --- |
| tlp |
| ufw |
| netdata |
| thinkfan |
| acpid |
| acpi_call |

| Font |
| --- |
| noto-fonts-emoji |

| Bluetooth |
| --- |
| bluez |
| bluez-utils |
| blueman |

| Editors |
| --- |
| vim |
| sub1ime-text |

| terminal eyecandies |
| --- |
| neofetch |

| tools |
| --- |
| imagemagick |
| dconf-editor |
| maim |
| mupdf |
| redshift |
| arandr |

| settings |
| --- |
| lxappearance |
| qt5ct |
| qt5-styleplugins |

| vbox |
| --- |
| virtualbox |
| virtualbox-host-modules-arch |
| virtualbox-guest-iso |
| linux-headers |
| virtualbox-ext-oracle |

| media |
| --- |
| vlc |
| mpd |
ffmpeg
| mpc |
| ncmpcpp |
| perl-image-exiftool |
| feh |
| simplescreenrecorder |


| Media editors |
| --- |
| gimp |
| inkscape |


| more aur apps |
| --- |
| picom-tryone-git |
| cava-git |
| awesome-git |
| mugshot |
| flat-remix |
| flat-remix-gtk |
| pamac-aur |
| gtk3-nocsd-git |
| gtk3-mushrooms |
| rxvt-unicode-pixbuf |
| rofi-git |
| glxinfo |
| toilet |
| korla-icon-theme-git |
| pkhex-bin |
| oomox-git |
| ttf-vista-fonts |
