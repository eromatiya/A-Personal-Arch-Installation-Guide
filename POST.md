## POST INSTALLATION

In post installation we will be using a lot of `sudo`. I'm not responsible if you broke your newly installed system. Remember that this guide is *for* ***future me***.

### Connect to the internet

We will be downloading stuff so we need an internet connection! So set-up your connection first!

### Check for Updates

It's recommended to check for updates first before installing anything so:

```
# pacman -Syu
```

### Drivers

#### Video Drivers

For AMD:
```
# pacman -S mesa lib32-mesa vulkan-radeon lib32-vulkan-radeon libva-utils
```

Add your (kernel) graphics driver to your initramfs. For example, if using `intel` add `i915` or if using `amd` add `amdgpu`:

```
# sudoedit /etc/mkinitcpio.conf
```

Then add `amdgpu` to the `MODULES`:

```
MODULES=(amdgpu ...)
```

#### Audio Drivers

Usually don't have to install these if you are going to install a desktop environment like KDE. It will automatically install what is needed.

```
# pacman -S pipewire lib32-pipewire wireplumber pipewire-audio pipewire-pulse pipewire-jack pipewire-session-manager
```

#### File System Tools

File system tools

```
# pacman -S unrar unzip p7zip unarchiver gvfs-mtp libmtp \
android-udev mtpfs xdg-user-dirs
```

`xdg-user-dirs` is a tool to help manage "well known" user directories like the desktop folder and the music folder. This will be automatically run on your next log in. Though you can manually generate XDG user directories by:

```
# xdg-user-dirs-update
```

#### Git

If you didn't include `git` on `pacstrap` earlier, it's time to install it now. This tool will come in handy later:

```
# pacman -S git
```

#### AUR Helper

The "later" is now, old man. We will now install an AUR helper, `yay`.

Clone `yay-bin` from the AUR using `git`.

```
$ git clone https://aur.archlinux.org/yay-bin.git
$ cd yay-bin/
$ makepkg -sri
```

#### Desktop Environment and Window Manager

Install your preferred desktop environment or window manager.

I'm a `KDE Plasma` guy, so in this guide, I'll include a guide to set-up `Plasma`.

+ KDE Plasma

	- Install the `plasma` then uninstall `discover`.

		```
		# pacman -S plasma
		# pacman -R discover
		```

	- KDE Plasma provides a global menu, to have a better integration with `GTK` programs, install `appmenu-gtk-module`:

		```
		# pacman -S appmenu-gtk-module
		```

	- If some programs like `Discord` has a blurry icon in the system tray, install the libappindicators:

		```
		# pacman -S libappindicator
		```

  - To make consistent file dialog under KDE Plasma, install:

		```
		# pacman -S xdg-desktop-portal xdg-desktop-portal-kde
		```

        Set the environment variable `GTK_USE_PORTAL=1` and `GDK_DEBUG=portals` in `/etc/environment`.

#### Terminal Emulator

After installing an environment, we need a terminal emulator. Every linux user's first partner. Note that we're still on the `TTY`.

+ KDE Plasma

	`Konsole` is the best terminal emulator for KDE Plasma due to its integration to the environment. While `yakuake` will be our drop-down terminal.

	```
	# pacman -S konsole
	```

#### File Managers

KDE's `dolphin` is the best file manager in the Linux world, imho. So I always use this no matter what environment I'm using. We will also install `ranger`, a CLI-based file manager.

```
# pacman -S dolphin dolphin-plugins kio-admin kde-cli-tools ranger
```

To generate thumbnails, I'll also install these:

+ kdegraphics-thumbnailers: Image files and PDFs
+ kimageformats: Gimp `.xcf` files
+ qt6-imageformats : `.webp`, `.tiff`, `.tga`, `.jp2` files
+ kdesdk-thumbnailers: Plugins for the thumbnailing system
+ ffmpegthumbs: Video files (based on ffmpeg)
+ taglib : Audio files
+ icoutils : *.ico, *.cur files and embedded *.exe icons
+ libappimage (with kio-extras also installed): embedded *.AppImage icons

```
# pacman -S kdegraphics-thumbnailers kimageformats qt6-imageformats kdesdk-thumbnailers \
ffmpegthumbs taglib icoutils libappimage
```

Enable preview showing of required file type in *Settings > Configure Dolphin... > General > Previews*.

There's a lot more thumbnail generators that can be found from the AUR (like a generator to create a thumbnail for `APK` files), but I don't really use them.

#### GUI-based Text Editors

`vim` is my text editor of choice, but sublime Text 4 is my go-to GUI text editor as it's lighter than the chromium-based counterparts like `atom` and `vscode`.

```
$ yay -S sublime-text-4
```

Note that Sublime is not "free" and needs a license.

#### Web browsers

`Firefox` and `w3m` are my trusty web browsers.

```
# pacman -S firefox w3m
```

I also install `google-chrome` as my back up web browser:

```
$ yay -S google-chrome
```

#### Bluetooth

If you're using KDE Plasma, skip this step:

```
# pacman -S bluez
```

But make sure to install `bluez-utils` and enable bluetooth manually:

```
# pacman -S bluez-utils
# systemctl enable bluetooth.service
```

The generic Bluetooth driver is the `btusb` Kernel module. Check whether that module is loaded. If it's not, then [load it](https://wiki.archlinux.org/index.php/Kernel_module#Manual_module_handling).

If you're not using Plasma, install `blueman`. The best GTK bluetooth manager.

```
# pacman -S blueman
```

Blueman automatically enables Bluetooth adapter when certain events (on boot, laptop lid is opened, etc.) occur. This can be disabled with the `auto-power-on` in `org.blueman.plugins.powermanager`:

```
$ gsettings set org.blueman.plugins.powermanager auto-power-on false
```

#### Plymouth

Plymouth provides a flicker-free graphical boot process. In short, a splash screen.

1. Install plymouth.

	```
	# pacman -S plymouth
	```

2. Add `plymouth` to the HOOKS array in mkinitcpio.conf. It must be added **after** `base` and `udev`/`systemd` for it to work:

	```
	# sudoedit /etc/mkinitcpio.conf
	```

	Add `plymouth` after base and systemd:

	```
	HOOKS=(base systemd plymouth ...)
	```

3. Set the theme.

	List all the installed plymouth theme:

	```
	# plymouth-set-default-theme -l
	```

	I have my own theme btw, and you can found it on the AUR. It is called `arch10`. So install it:

	```
	$ yay -S plymouth-theme-arch10
	```

	Choose a theme by running the command below, then it will rebuild the initramfs image:

	```
	# plymouth-set-default-theme -R arch10
	```

	Replace `arch10` with your choice.

4. You now need to append `splash` in the kernel parameters in your boot entry options.

	```
	# sudoedit /boot/loader/enable/arch.conf
	```

	Example:

	```
	title Arch Linux
	linux /vmlinuz-linux
	initrd /intel-ucode.img
	initrd /initramfs-linux.img
	options rd.luks.name=/DEV/SDA2/UUID/HERE=volume root=/dev/mapper/volume-root rw
	options quiet splash fbcon=nodefer
	```

#### Display Manager

A display manager, or login manager, is typically a graphical user interface that is displayed at the end of the boot process in place of the default shell.

+ KDE Plasma

	```
	# pacman -S plasma-login-manager
	# systemctl enable plasmalogin.service
	```

#### Silent boot

This is for who prefer to limit the verbosity of their system to a strict minimum, either for aesthetics or other reasons. For me, it's aesthetics.

Edit boot loader kernel parameters:

```
# sudoedit /boot/loader/entries/arch.conf
```

On a new line add these parameters:

```
options quiet splash loglevel=3 rd.udev.log_priority=3 rd.systemd.show_status=auto consoleblank=0
```

If you're using `plymouth` and its `splash` kernel parameter, put `splash` after the `quiet` parameter.

If you have an encrypted partition, `quiet` and `fbcon=nodefer` is enough. For example:

```
options quiet splash fbcon=nodefer
```

Remove `splash` if you're not using `plymouth`.

#### Install and Configure Network Manager

As of now, we're using `iwd` if we're using wireless connection and `dhcpcd` if we're on wired connection.

**Network Manager is recommended if you're on a Plasma environment**. So make sure to enable it and disable the other networking tools. While if you're using awesome, you can just continue using `iwd` and `dhcpcd`. I mean it's your choice.

+ Install network manager and its utilities.

	`plasma-meta` will include and install networkmanager. But just to be safe:

	```
	# pacman -S networkmanager network-manager-applet dhclient modemmanager usb_modeswitch mobile-broadband-provider-info
	```

+ Disable `iwd` or `dhcpcd`.

	- `iwd`

		```
		# iwctl station wlan0 disconnect
		# systemctl disable --now iwd
		```

	- `dhcpcd`

		```
		# systemctl disable --now dhcpcd
		```

+ Enable Network Manager service.

	```
	# systemctl enable --now NetworkManger
	```

#### Reboot then Login

The system's fully functional! You can now login to you system with all the configuration we've done so far.

```
$ reboot
```

## Extras

#### Power Management for Laptops

TLP brings you the benefits of advanced power management for Linux without the need to understand every technical detail. **This is for laptops.**

Install and enable it now:

```
# pacman -S tlp
# systemctl enable --now tlp.service
```

Install `upower`, `acpid` and `acpi_call`:

```
# pacman -S acpid acpi_call upower
```

Enable acpid

```
# systemctl enable acpid.service
```

#### Firewall

We'll use `Uncomplicated Firewall` or `ufw` for short.

1. Install the `ufw` package. Start and enable `ufw.service` to make it available at boot. Note that this will not work if `iptables.service` is also enabled (and same for its ipv6 counterpart).

	```
	# pacman -S ufw
	```

2. Configuration

	Here's some basic configuration. A very simplistic configuration which will deny all by default, allow any protocol from inside a 192.168.0.1-192.168.0.255 LAN, and allow incoming Deluge and rate limited SSH traffic from anywhere:

	```
	# ufw default deny
	# ufw allow from 192.168.0.0/24
	# ufw allow Transmission
	# ufw limit ssh
	```

	If you are going to use KDE Connect, make sure to configure UFW for it.

3. The next line is only needed once the first time you install the package:

	```
	# ufw enable
	# systemctl enable --now ufw.service
	```

Adding other applications. The PKG comes with some defaults based on the default ports of many common daemons and programs. Inspect the options by looking in the `/etc/ufw/applications.d` directory or by listing them in the program itself:

```
# ufw app list
```

#### Fonts

Improve fonts.

Install these fonts. Inter will be my system font no matter what the environment.

```
# pacman -S ttf-dejavu ttf-liberation noto-fonts noto-fonts-emoji inter-font ttf-roboto
```

Additional fonts to support Asian characters

```
# pacman -S noto-fonts-cjk noto-fonts-extra
```

Enable font presets by creating symbolic links:

```
# ln -s /etc/fonts/conf.avail/70-no-bitmaps.conf /etc/fonts/conf.d
# ln -s /etc/fonts/conf.avail/10-sub-pixel-rgb.conf /etc/fonts/conf.d
# ln -s /etc/fonts/conf.avail/11-lcdfilter-default.conf /etc/fonts/conf.d
```

The above will disable embedded bitmap for all fonts, enable sub-pixel RGB rendering, and enable the LCD filter which is designed to reduce colour fringing when subpixel rendering is used.

For font consistency, all applications should be set to use the `serif`, `sans-serif`, and `monospace` aliases, which are mapped to particular fonts by fontconfig.

Create `/etc/fonts/local.conf`, then add:

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
			<string>sans-serif</string>
		</test>
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

Update and set your font of choice on settings.


#### Change storage ioscheduler

```
sudoedit /etc/udev/rules.d/60-ioschedulers.rules
```

```
# HDD
ACTION=="add|change", KERNEL=="sd[a-z]*", ATTR{queue/rotational}=="1", \
    ATTR{queue/scheduler}="bfq"

# SSD
ACTION=="add|change", KERNEL=="sd[a-z]*|mmcblk[0-9]*", ATTR{queue/rotational}=="0", \
    ATTR{queue/scheduler}="mq-deadline"

# NVMe SSD
ACTION=="add|change", KERNEL=="nvme[0-9]*", ATTR{queue/rotational}=="0", \
    ATTR{queue/scheduler}="none"
```


```
$  udevadm control --reload-rules
$  udevadm trigger

$  cat /sys/block/nvme0n1/queue/scheduler # check output
$  cat /sys/block/sda/queue/scheduler
```


#### Enable SSD TRIM

To keep SSDs fast and maintain cell health enable `fstrim.timer`:

```
# systemctl enable --now fstrim.timer
```
