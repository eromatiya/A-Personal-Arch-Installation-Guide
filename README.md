# A Personal Arch Installation Guide


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

#### Verify the boot mode

If UEFI mode is enabled on an UEFI motherboard, Archiso will boot Arch Linux accordingly via systemd-boot. To verify this, list the efivars directory:  

```bash
$ ls /sys/firmware/efi/efivars
```

#### Check the drives

The most common main drive is **sda**.

```bash
$ lsblk
```

#### Let’s clean up our main drive to create new partitions for our installation

```bash
$ gdisk /dev/sda
```

Press <kbd>x</kbd> to enter **expert mode**. Then press <kbd>z</kbd> to *zap* our drive. Then hit <kbd>y</kbd> when prompted about wiping out GPT and blanking out MBR.  
	
**NOTE:**  
Remember that pressing <kbd>x</kbd> button will put you to *expert mode* and <kbd>z</kbd> will *zap* or wipe-out the drive. **Think carefully before pressing <kbd>Y</kbd>! This CAN'T be undone!**

#### Now let’s create our partitions

```bash
$ cgdisk
```

Enter the device filename of your main drive, e.g. `/dev/sda`. Just press <kbd>Return</kbd> when warned about damaged GPT.

Now we should be presented with our main drive showing the partition number, partition size, partition type, and partition name. If you see list of partitions, delete all those first.

#### Let’s create our boot partition

+   Hit New from the options at the bottom.
+   Just hit enter to select the default option for the first sector.
+   Now the partion size - Arch wiki recommends 200-300 MB for the boot + size. Let’s make it 500MiB or 1GB in case we need to add more OS to our machine. I’m gonna assign mine with 1024MiB. Hit enter.
+   Set GUID to **EF00**. Hit enter.
+   Set name to boot. Hit enter.
+   Now you should see the new partition in the partitions list with a partition type of EFI System and a partition name of boot. You will also notice there is 1007KB above the created partition. That is the MBR. Don’t worry about that and just leave it there.

#### Create the swap partition

+   Hit New again from the options at the bottom of partition list.
+   Just hit enter to select the default option for the first sector.
+   For the swap partition size, it is advisable to have 1.5 times the size of your RAM. I have 8GB of RAM so I’m gonna put 12GiB for the partition size. Hit enter.
+   Set GUID to **8200**. Hit enter.
+   Set name to swap. Hit enter.

#### Create the root partition

+   Hit New again.
+   Hit enter to select the default option for the first sector.
+   Hit enter again to input your root size.
+   Also hit enter for the GUID to select default.
+   Then set name of the partition to root.


#### Create the home partition

+   Hit New again.
+   Hit enter to select the default option for the first sector.
+   Hit enter again to use the remainder of the disk.
+   Also hit enter for the GUID to select default.
+   Then set name of the partition to home.

Lastly, hit **Write** at the bottom of the patitions list to *write the changes* to the disk. Type `yes` to *confirm* the write command. Now we are done partitioning the disk. Hit **Quit** *to exit cgdisk*.

### FORMATTING PARTITIONS

#### Let’s see the new partition list

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

#### Format the boot partition as FAT32

```bash
$ mkfs.fat -F32 /dev/sda1
```  

#### Enable the swap partition

```bash
$ mkswap /dev/sda2  
$ swapon /dev/sda2
```  

#### Format home and root partition as ext4

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

```bash
$ systemctl start dhcpcd@enp3s0
```

If you are on a laptop, you can connect to a wireless access point using wifi-menu -o <wireless_interface>.

```bash
# Enable netctl systemd service
$ systemctl start netctl  

# Search and Connect to wifi using your wifi interface
$ wifi-menu wlp7s0
```  

Ping google to make sure we are online:

```bash
$ ping -c 3 8.8.8.8
``` 

If you receive Unknown host or Destination host unreachable response, means you are not online yet. Review your network configuration and redo the steps above.

After setting up the drive and connecting to the internet, we are now ready to install Arch Linux to our system.

First we need to mount the partitions that we created earlier to their appropriate mount points.

### Mounting partitions

#### Mount the `root` partition:  

```bash
$ mount /dev/sda3 /mnt
```  

If `/mnt` directory doesn't exist create it.**

```bash
$ mkdir /mnt
```  

#### Create and mount the `boot` partition  

```bash
$ mkdir /mnt/boot  
$ mount /dev/sda1 /mnt/boot
```  

#### Create and mount the `home` partion  

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

```bash
$ mkinitcpio -p linux
```

### Wireless Connections

For wireless connections, install iw, wpa_supplicant, and (for wifi-menu) dialog. **This is needed if you want to use a wi-fi connection after the next reboot!**

```bash
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

**pacman Easter Eggs**

While you're at it, **you can add colors and PAC-MAN in pacman!**

Open `/etc/pacman.conf`, then find `# Misc options`. After that, **below** that, add:

```
# Uncomment Color
Color

# Add ILoveCandy
ILoveCandy
```

### Passwords and Users

Set the root password:  

```bash
$ passwd
```

Add a new user with username *MYUSERNAME*. Of course, change *MYUSERNAME*, with your own:  

```bash
$ useradd -m -g users -G wheel,storage,power,video,audio,rfkill -s /bin/bash MYUSERNAME
```  

Set password the password for user *MYUSERNAME*:  

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
**Note:**  
If your `root` is not in `/dev/sda3`, make sure to change it.

Save and exit.

### Exit chroot and reboot:  

```bash
$ exit
$ reboot
```

## POST INSTALLATION

In post installation we will be using a lot of `sudo`. I'm not responsible if you broke your newly installed system. Remember that this guide is *for* **future me**.

#### Install X server

After booting up, you will be greeted by TTY. Still, there's no sign of GUI everywhere. So we need to install it. But before that, we need an internet connection! Follow this [guide](#connect-to-internet). Make sure to open the guide in a new tab, otherwise this page will scroll up instead and I think you don't want that.

Now that you have an internet connection, you need to install a graphical server - `xorg` or `wayland`. I'm using `AwesomeWM` as my window manager and it only runs on `X`, sooo let's install `X`.

```bash
$ sudo pacman -S xorg-server xorg-xrdb xorg-xinit xorg-xrandr xorg-xev xorg-xdpyinfo xorg-xprop
```

#### Install video drivers

After installing the graphical server, we need to install the video drivers. I'm using an integrated intel graphics card.

```bash
$ sudo pacman -S xf86-video-intel vulkan-intel lib32-vulkan-intel vulkan-icd-loader lib32-vulkan-icd-loader lib32-mesa
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

#### Install Input Drivers

I'm using a touchpad so install a driver for it.

```bash
$ sudo pacman -S xf86-input-synaptics
```

#### Install audio drivers

```bash
$ sudo pacman -S alsa-utils pulseaudio-alsa pulseaudio-bluetooth pulseaudio pavucontrol
```

#### Install GTK3 and GTK2

GTK, or the GIMP Toolkit, is a multi-platform toolkit for creating graphical user interfaces.

1. Install GTK

	```bash
	$ sudo pacman -S gtk3 gtk2
	```

2. Install GTK engines

	```bash
	$ sudo pacman -S gtk-engine-murrine gtk-engines
	```

#### Install file system tools and file manager

```bash
# File system tools
$ sudo pacman -S unrar unzip p7zip gvfs-mtp libmtp ntfs-3g android-udev ffmpegthumbnailer mtpfs xdg-user-dirs

# File managers
$ sudo pacman -S dolphin dolphin-plugins kde-cli-tools ranger
```

`xdg-user-dirs` is a tool to help manage "well known" user directories like the desktop folder and the music folder so generate XDG user directories by:

```bash
$ xdg-user-dirs-update
```

#### Terminal Emulator

We need a terminal emulator. Every linux user's first partner.

```bash
$ sudo pacman -S kitty xterm
```

#### Install GUI and CLI web browser

```bash
$ sudo pacman -S firefox w3m
```

#### Install git

Git is the version control system (VCS) designed and developed by Linus Torvalds, the creator of the Linux kernel. Git is now used to maintain AUR packages, as well as many other projects, including sources for the Linux kernel. 

```bash
$ sudo pacman -S git
```

#### Install an AUR helper

`yay` will be our AUR helper. `go` and `base-devel` are a dependency of `yay`, so make sure you have it first. To install `yay`:

```bash
# Install the go package
$ sudo pacman -S go
# Clone the repo
$ git clone https://aur.archlinux.org/yay.git
$ cd yay
# Compile, build and install
$ makepkg -sri
```

#### Install missing kernel modules.

```bash
$ yay -S wd719x-firmware aic94xx-firmware --removemake --noconfirm
$ sudo mkinitcpio -p linux
```

#### Backlight Control

We'll use `light` for this because this is a better version of `xorg-xbacklight`.

```bash
$ yay -S light-git
```

#### Text Editors

Install an editor.

```bash
# Editor for CLI
$ sudo pacman -S vim

# EDITOR for GUI
$ yay -S sublime-text-dev
```

#### GUI Environment

Install your desktop environment or window manager. 

I'm using a window manager and it is `AwesomeWM`.

```bash
$ yay -S awesome-git --noconfirm --removemake
```

If I will be using a desktop environment, it will be `KDE Plasma`.

Install the `plasma-meta` meta-package or the `plasma` group. For differences between `plasma-meta` and `plasma` reference Package group. Alternatively, for a more minimal Plasma installation, install the `plasma-desktop` package.

```bash
# Let's start with a minimal Plasma enviroment
$ sudo pacman -S plasma-desktop
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

1. Install plymouth.

	```bash
	# Install
	$ yay -S plymouth-git

	# If you also use GDM, you should install the gdm-plymouth, which compiles gdm with plymouth support. 
	$ yay -S gdm-plymouth
	```

2. Add `plymouth` to the HOOKS array in mkinitcpio.conf. It must be added after base and udev for it to work: 

	```bash
	$ sudoedit /etc/mkinitcpio.conf
	```

	Put `plymouth` after base and udev:

	```
	HOOKS=(base udev plymouth ...)
	```

3. Set the theme.

	```bash
	# List all the installed plymouth theme
	$ sudo plymouth-set-default-theme -l

	# Select a theme then rebuild  initrd
	$ sudo plymouth-set-default-theme -R theme_name
	```

4. You now need to append `splash` in the kernel parameters. See [Silent Boot](#silent-boot).

#### Silent boot

This is for who prefer to limit the verbosity of their system to a strict minimum, either for aesthetics or other reasons. For me, it's aesthetics. 

1. Edit boot loader kernel parameters:

	```bash
	$ sudoedit /boot/loader/entries/arch.conf
	```

2. Add these parameters `(options ... loglevel=3 vga=current rd.udev.log_priority=3 ...)` in the `options`:

	```
	options quiet splash loglevel=3 vga=current rd.udev.log_priority=3
	```

#### Microcode

Processor manufacturers release stability and security updates to the processor microcode. These updates provide bug fixes that can be critical to the stability of your system. Without them, you may experience spurious crashes or unexpected system halts that can be difficult to track down. 

1. Install microcode.

	```bash
	# For AMD processors
	$ sudo pacman -S amd-ucode

	# For intel processors
	$ sudo pacman -S intel-ucode
	```

	If your Arch installation is on a removable drive that needs to have microcode for both manufacturer processors, install both packages. 


2. Load  microcode. For systemd-boot, use the `initrd` option to load the microcode, before the initial ramdisk, as follows:

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

	Change `cpu_manufacturer-ucode` with either `amd` or `intel` depending on your processor.

#### Display Manager

A display manager, or login manager, is typically a graphical user interface that is displayed at the end of the boot process in place of the default shell.

There's a ton of display manager out there. I'm using `sddm` so this guide will cover that. 

1. First, install it:

	```bash
	$ sudo pacman -S sddm
	```

2. To enable graphical login, enable the appropriate systemd service. For example, for SDDM, enable `sddm.service`. Just because we're using plymouth, we will enable `sddm-plymouth.service` to have a plymouth support.

	```bash
	$ sudo systemctl enable sddm-plymouth
	```

3. Install a theme. I'm using the `Sugar Candy` theme.

	```bash
	$ yay -S sddm-theme-sugar-candy-git
	```

4. Configure the theme by:

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

1. Install network manager and its utilities.

	```bash
	$ sudo pacman -S networkmanager network-manager-applet dhclient modemmanager usb_modeswitch mobile-broadband-provider-info
	```

2. Disable netctl or dhcpcd.

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

3. Enable Network Manager service.

	```bash
	# If using dhcpcd
	$ sudo systemctl enable NetworkManger
	$ sudo systemctl start NetworkManger
	```

	Sometimes you have to `reboot` if you cannot connect using the NetworkManger.

Optional:

Run `network-manager-applet` to have the network manager applet in you system tray. 

#### Enable MAC randomization

MAC randomization can be used for increased privacy by not disclosing your real MAC address to the network. 

1. Install macchanger.

	```bash
	$ sudo pacman -S macchanger
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

```bash
# Install
$ sudo pacman -S bluez bluez-utils
# Start/Enable
$ sudo systemctl start bluetooth.service
$ sudo systemctl enable bluetooth.service
```

#### Bluetooth GUI

We'll use `blueman` for this. Blueman is a full featured Bluetooth manager written in GTK.

1. Install blueman.

	```bash
	$ sudo pacman -S blueman
	```

	Be sure to enable the Bluetooth daemon and start Blueman with `blueman-applet`. A graphical settings panel can be launched with `blueman-manager`.

2. Disable auto-power-on 

	Blueman automatically enables Bluetooth adapter () when certain events (on boot, laptop lid is opened, ...) occur. This can be disabled with the `auto-power-on` in org.blueman.plugins.powermanager: 

	```bash
	$ gsettings set org.blueman.plugins.powermanager auto-power-on false
	```

#### Authentication Managers

Polkit is used for controlling system-wide privileges. It provides an organized way for non-privileged processes to communicate with privileged ones. In contrast to systems such as sudo, it does not grant root permission to an entire process, but rather allows a finer level of control of centralized system policy. 

1. Install `polkit`, `polkit-kde-agent`/`polkit-gnome`, and `gnome-keyring`.

``` bash
# I'm using Qt apps so I'll install polkit-kde-agent
$ sudo pacman -S polkit polkit-kde-agent gnome-keyring
# For polkit-kde-agent, run:
$ /usr/lib/polkit-kde-authentication-agent-1
# For polkit-gnome
$ /usr/lib/polkit-gnome/polkit-gnome-authentication-agent-1
```

#### Better Power Management

1. Install and start/enable `TLP`:

	```bash
	$ sudo pacman -S tlp
	$ sudo systemctl enable tlp.service
	```

2. Install `upower`, `acpid` and `acpi_call`:

	```bash
	# Install
	$ sudo pacman -S acpid acpi_call upower
	# Enable acpid
	$ sudo systemctl enable acpid.service
	```

#### Better Fan Control for Thinkpad

1. We'll use/install `thinkfan` here.

	```bash
	$ yay -S thinkfan
	```

	Note that the thinkfan package installs `/usr/lib/modprobe.d/thinkpad_acpi.conf`, which contains:

	```
	options thinkpad_acpi fan_control=1
	```

	So fan control is enabled by default. Alternatively, you can enable fan control as follows: 

	```bash
	$ echo "options thinkpad_acpi fan_control=1" > /etc/modprobe.d/thinkfan.conf
	```

2. Now, load the module: 

	```bash
	$ sudo modprobe thinkpad_acpi
	$ sudo cat /proc/acpi/ibm/fan
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

	```bash
	$ systemctl enable thinkfan.service
	```

#### Firewall

We'll use `Uncomplicated Firewall` or `ufw` for short.

1. Install the `ufw` package. Start and enable `ufw.service` to make it available at boot. Note that this will not work if `iptables.service` is also enabled (and same for its ipv6 counterpart). 

	```bash
	$ sudo pacman -S ufw
	$ sudo systemctl start ufw.service
	$ sudo systemctl enable ufw.service
	```

2. Configuration

	Here's some basic configuration. A very simplistic configuration which will deny all by default, allow any protocol from inside a 192.168.0.1-192.168.0.255 LAN, and allow incoming Deluge and rate limited SSH traffic from anywhere: 

	```bash
	$ sudo ufw default deny
	$ sudo ufw allow from 192.168.0.0/24
	$ sudo ufw allow Deluge
	$ sudo ufw limit ssh
	```

3. The next line is only needed once the first time you install the package: 

	```bash
	$ sudo ufw enable
	```

Adding other applications. The PKG comes with some defaults based on the default ports of many common daemons and programs. Inspect the options by looking in the `/etc/ufw/applications.d` directory or by listing them in the program itself: 

```bash
$ sudo ufw app list
```

#### Real-Time Performance Monitoring Tool

`netdata` will be the man for this job. `netdata` is a system for distributed real-time performance and health monitoring. netdata is created by the group that also created FireHOL and FireQOS. 

1. Install and start/enable NetData

	```bash
	$ sudo pacman -S netdata
	$ sudo systemctl start netdata
	$ sudo systemctl enable netdata
	```

You can find all these settings, with their default values, by accessing the URL `http://localhost:19999/netdata.conf`.

Access NetData via a Web browser by accessing <http://localhost:19999>.


#### And that's it! We covered all the must-have part! Now, it's time for basic ricing. :)

### Basic Ricing

My [setups](https://github.com/manilarome/the-glorious-dotfiles) are mostly transparent and blurred, that's why I mostly use Qt5 apps. Still, we have to use and rice GTK apps because not everything has an alternative Qt version.
Let's start with installing some GTK and Qt system settings and and its themes.

#### GTK3 Theming

If we want to change our system font, GTK3, icon and cursor theme. We need a settings application. So we need to install that.

1. Install `lxappearance-gtk3`.

	```bash
	$ sudo pacman -S lxappearance-gtk3
	```

2. After that we need to install a `GTK3` theme.
	
	I love dark theme so let's install a dark GTK3 theme.

	```bash
	# Material theme
	$ sudo pacman -S material-gtk-theme
	# Equilux theme
	$ yay -S  equilux-theme
	```

3. Set your GTK3 theme in lxappearance.
	
	```bash
	$ lxappearance
	```

#### Qt5 Theming

1. Install `qt5ct` and `qt5-styleplugins`. We need these tools to [configure Qt5 apps under environments other than KDE Plasma](https://wiki.archlinux.org/index.php/Qt#Configuration_of_Qt5_apps_under_environments_other_than_KDE_Plasma). `qt5ct` provides a Qt5 QPA independent of the desktop environment and a configuration utility.

	```bash
	$ sudo pacman -S qt5ct qt5-styleplugins
	```

2. Set the environment variable `QT_QPA_PLATFORMTHEME="qt5ct"` so that the settings are picked up by Qt applications. You can set the environment variable in your `~/.xprofile` or something.

3. Log out, so the environment variable can take effect. Log in. 

4. Install `kvantum-qt5`. Kvantum manager is an SVG-based theme engine for Qt5. This will be the main tool that will provide us the full blur effect and those sweet eyecandies.

	```bash
	$ sudo pacman -S kvantum-qt5
	```

5. Install a kvantum theme. I recommend the [Inverse-dark](https://store.kde.org/p/1365482/) theme, because it's clean and modern looking. An edited version is in my [dotfiles](https://github.com/manilarome/the-glorious-dotfiles/tree/master/config/Kvantum/Inverse-dark).

6. Open `kvantummanager` to set your desired kvantum theme. You can configure the active theme to make some changes like making the dolphin view transparent, disable tooltip shadows, etc., etc.

6. Run `qt5ct` again. Change the `style` to `kvantum` to use the kvantum theme. Hit `OK`.

7. Open a Qt5 app. Enjoy.


#### Install an Icon theme

A part of basic ricing is improving the icon theme.

1. Install an icon theme. My main icon theme is [`Tela`](https://github.com/vinceliuice/Tela-icon-theme), but not all the icons are covered by it so I'm also using [`Papirus`](https://github.com/PapirusDevelopmentTeam/papirus-icon-theme) as *inherits*.

	```bash
	# Install Papirus Icon
	$ sudo pacman -S papirus-icon-theme
	# Install Tela Icon
	$ yay -S tela-icon-theme-git
	```

2. Add `Papirus` as inherits of `Tela`.

	```bash
	$ sudoedit /usr/share/icons/Tela/index.theme
	```

	Find `Inherits` then add `Papirus`.

	```
	Inherits=Papirus,Papirus-Dark,hicolor
	```

3. Set your theme in `lxappearance` and `qt5ct`.

#### Font Installation

Let's be honest, font rendering in Linux *is not that* good by default. So let's make them great again! Let's start with installing some beautiful fonts.

```bash
# Basic fonts
$ sudo pacman -S ttf-dejavu ttf-liberation noto-fonts noto-fonts-emoji
# macOS fonts. I just love macOS Fonts
$ yay -S otf-san-francisco-pro otf-sfmono-patched 
```

#### Improve Font Rendering

Make your system fonts great again! Improve your fonts for system-wide usage without installing a patched font library packages like the `Infinality`.

1. If you haven't already, install these fonts:

	```bash
	$ sudo pacman -S ttf-dejavu ttf-liberation noto-fonts
	```

2. Enable font presets by creating symbolic links:

	```bash
	$ sudo ln -s /etc/fonts/conf.avail/70-no-bitmaps.conf /etc/fonts/conf.d
	$ sudo ln -s /etc/fonts/conf.avail/10-sub-pixel-rgb.conf /etc/fonts/conf.d
	$ sudo ln -s /etc/fonts/conf.avail/11-lcdfilter-default.conf /etc/fonts/conf.d
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

4. Set your font settings to match above in your DE system settings. Open `qt5ct` and `lxappearance` then set the font to your preference.

You can also follow this [guide](https://jichu4n.com/posts/how-to-set-default-fonts-and-font-aliases-on-linux/).

#### Install a Cursor theme

Let's make the cursor theme more modern looking!

[`Vimix Cursor`](https://github.com/vinceliuice/Vimix-cursors) is a beautiful theme! It can be found in the AUR.

1. Install the theme.

	```bash
	$ yay -S vimix-cursors
	```

2. Set the theme in your `qt5ct` and `lxappearance`.

#### Improve Terminal Experience

Improve your terminal experience! Let's replace `bash` with `zsh`. `Zsh` is a powerful shell that operates as both an interactive shell and as a scripting language interpreter. 

0. Before starting, we may want to see what shell is currently being used: 

	```bash
	$ echo $SHELL
	```

1. Not using ZSH? Then install the `zsh` package. For additional completion definitions, install the `zsh-completions` package as well. 

	```bash
	$ sudo pacman -S zsh zsh-completions
	```

2. Make sure that Zsh has been installed correctly by running the following in a terminal: 

	```bash
	$ zsh
	```

	You should now see *zsh-newuser-install*, which will walk you through some basic configuration. If you want to skip this, press <kbd>q</kbd>. If you did not see it, you can invoke it manually with: 

	```bash
	$ autoload -Uz zsh-newuser-install
	$ zsh-newuser-install -f
	```

	**Note:**  
	Make sure your terminal's size is at least 72×15 otherwise *zsh-newuser-install* will not run.

3. Change your `$SHELL` from whatever you're using right now to `ZSH`.

	```bash
	# User
	$ chsh -s $(which zsh)

	# System-wide
	$ sudo chsh -s $(which zsh)
	```

	It's better if you `reboot` your system.


#### Install `oh-my-zsh`

After rebooting, let's now improve the `zsh` prompt.

1. Install `oh-my-zsh`:

	```zsh
	# via curl
	$ sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
	# via wget
	$ sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
	```

	After this open a terminal then a guide will walk you through some basic configuration.

#### Improve Terminal Prompt using PowerLevel10k

1. Download the recommended font [here](https://github.com/romkatv/powerlevel10k#meslo-nerd-font-patched-for-powerlevel10k).


2. Copy the downloaded font to the right font directory.

	```zsh
	# User only. If the folder doesn't exist, create it.
	cp FONTNAME.TTF "${HOME}"/.local/share/fonts/TTF/

	# System-wide
	cp FONTNAME.TTF /usr/share/fonts/TTF/
	```

3. Rebuild font information cache files

	```zsh
	# User wide
	$ fc-cache
	# System-wide
	$ sudo fc-cache
	```

4. Change your terminal font to `MesloLGS NF`.

5. Install the `Powerlevel10k` theme.

	```zsh
	$ git clone --depth=1 https://github.com/romkatv/powerlevel10k.git $ZSH_CUSTOM/themes/powerlevel10k
	```

6. Open `~/.zshrc` with your editor

	```zsh
	$ $EDITOR $HOME/.zshrc
	```

	Find the line `ZSH_THEME=robbyrussell` it's not easy to miss. Then change it to `ZSH_THEME=powerlevel10k/powerlevel10k`.

7. Open a terminal. There should be some instructions/dialog there that will greet you and will guide you to theme your Powerlevel10k prompt.

More info about Powerlevel10k [here](https://github.com/romkatv/powerlevel10k).

#### Use `gtk3-mushrooms` a patched GTK3

One of the things I don't like is using GTK3 apps that have a CSD or client-side decoration with a window manager. It removes the unified look because some apps have CSDs and some don't. That's why I'm using a patched version of `GTK3` called [`gtk3-mushrooms`](https://github.com/krumelmonster/gtk3-mushrooms). `gtk3-mushrooms` is a set of patches for GTK3 library that makes it better.

Install `gtk3-mushrooms` and replace the vanilla `gtk3`.

```zsh
$ yay -S gtk3-mushrooms
```

Notable changes after installing `gtk3-mushrooms`:

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

#### Hotel

**`trivago`** as hotel
