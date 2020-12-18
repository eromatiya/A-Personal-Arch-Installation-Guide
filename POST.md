## POST INSTALLATION

In post installation we will be using a lot of `sudo`. I'm not responsible if you broke your newly installed system. Remember that this guide is *for* ***future me***.

### Connect to the internet

We will be downloading stuff so we need an internet connection! So set-up your connection first!

### Check for Updates

It's recommended to check for updates first before installing anything so:

```
# pacman -Syu
```

### Display Server and Protocol

We need to install a display server, a protocol or both. Normally, your desktop environment or window manager of choice will automatically install these as a dependency. But for this guide's sake we will install `X` server:

```
# pacman -S xorg-server xorg-xrdb xorg-xinit xorg-xrandr xorg-xev xorg-xdpyinfo xorg-xprop
```

If you're planning to use a window manager like `awesome`, `bspwm` or `i3`, you should install X. While if you're planning to use `sway`, then wayland it is. If `GNOME`, you can install both. Again, your environment of choice will automatically install these as its dependencies.

#### Video Drivers

After installing the graphical server, we need to install the video drivers. I'm only using an integrated intel graphics card. **Sobs.** So an intel driver is what I need.


```
# pacman -S xf86-video-intel vulkan-intel vulkan-icd-loader libva-intel-driver
```

Add your (kernel) graphics driver to your initramfs. For example, if using `intel` add `i915`: 

```
# sudoedit /etc/mkinitcpio.conf
```

Then add `i915` to the `MODULES`:

```
MODULES=(i915 ...)
```

#### Audio Drivers

```
# pacman -S alsa-utils pulseaudio-alsa pulseaudio-bluetooth pulseaudio pavucontrol
```

#### File System Tools

File system tools

```
# pacman -S unrar unzip p7zip unarchiver gvfs-mtp libmtp ntfs-3g \
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

#### Missing Kernel Modules

If you noticed, there's a warning message while running `mkinitcpio -p linux`, fix this by installing these firmwares:

```
$ yay -S wd719x-firmware aic94xx-firmware --removemake --noconfirm
# mkinitcpio -p linux
```


#### Desktop Environment and Window Manager

Install your preferred desktop environment or window manager. 

I'm an `awesome` and `KDE Plasma` guy, but right now I am using `Plasma`. So in this guide, I'll include a guide to set-up both `Plasma` and `Awesome`.

+ KDE Plasma

	- Install the `plasma-meta` meta-package or the `plasma` group. For differences between `plasma-meta` and `plasma` reference Package group. Alternatively, for a more minimal Plasma installation, install the `plasma-desktop` package. Although I always install `plasma-desktop`, `plasma-meta` and some other programs such as `libappindicator-gtk3`, `libappindicator-gtk2`, `packagekit-qt5`, and etc.

		```
		# pacman -S plasma-desktop plasma-meta
		```

	- KDE Plasma provides a global menu, to have a better integration with `GTK` programs, install `appmenu-gtk-module`:

		```
		# pacman -S appmenu-gtk-module
		```

	- If some programs like `Discord` has a blurry icon in the system tray, install the libappindicators:

		```
		# pacman -S libappindicator-gtk3 libappindicator-gtk2
		```

	- `Discover` is the Plasma's app store, it will be automatically installed by installing the `plasma-meta` package. If it doesn't show any applications, install `packagekit-qt5`:

		```
		# pacman -S packagekit-qt5
		```

	- Xorg is dying and nobody wants to maintain it anymore, while "Wayland is the future". I agree with this, although wayland needs to mature a little bit more to replace X completely. So yeah, I also want a Wayland session to test things out:

		```
		# pacman -S qt5-wayland plasma-wayland-session
		```

	- Some of the plasmoids uses `qdbus`, so also intall `qt5-tools` that will provide it:

		```
		# pacman -S qt5-tools
		```

	- I need a dock and I will be using `latte-dock` from the AUR:

		```
		$ yay -S latte-dock-git
		```

+ Awesome Window Manager

	- I always use the latest version of `awesomewm` because the devs are doing an amazing job by maintaining this treasure of a program and I also want to get the latest fixes and features ASAP (one of the reason I use arch btw). So install it from the AUR:

		```
		$ yay -S awesome-git --noconfirm --removemake
		```

	- Of course, just a window manager is not enough to get the experience I want. So I need to install a compositor and some utilities to achieve it.

		- I will be using `light` as the backlight control tool:

			```
			$ yay -S light-git
			```

		- Picom as the compositor:

			```
			$ yay -S picom-git --noconfirm --removemake
			```

		- Rofi as the application launcher:

			```
			# pacman -S rofi
			```

		- Authentication Managers

			Polkit is used for controlling system-wide privileges. It provides an organized way for non-privileged processes to communicate with privileged ones. In contrast to systems such as sudo, it does not grant root permission to an entire process, but rather allows a finer level of control of centralized system policy.

			+ Install `polkit-kde-agent`/`lxqt-policykit`/`polkit-gnome`, and `gnome-keyring`:

				I'm using Qt apps with awesome, so I'll install lxqt-policykit for UI consistency. Note that you only need one authentication manager and gnome-keyring.

				```
				# pacman -S polkit lxqt-policykit gnome-keyring
				```

			+ Run it:

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

#### Terminal Emulator

After installing an environment, we need a terminal emulator. Every linux user's first partner. Note that we're still on the `TTY`.

+ KDE Plasma

	`Konsole` is the best terminal emulator for KDE Plasma due to its integration to the environment. While `yakuake` will be our drop-down terminal.

	```
	# pacman -S konsole yakuake
	```

+ Awesome Window Manager

	For me, `kitty` is the best terminal emulator for my awesome wm setups as it's easy to configure and *fast*. I will also install `xterm` as a backup emulator.
	
	```
	# pacman -S kitty xterm
	```

#### GTK

GTK, or the GIMP Toolkit, is a multi-platform toolkit for creating graphical user interfaces.

If not yet installed:

```
# pacman -S gtk3
```

Install GTK engines

```
# pacman -S gtk-engine-murrine gtk-engines gnome-theme-extra
```

#### File Managers

KDE's `dolphin` is the best file manager in the Linux world, imho. So I always use this no matter what environment I'm using. We will also install `ranger`, a CLI-based file manager.

```
# pacman -S dolphin dolphin-plugins kde-cli-tools ranger
```

To generate thumbnails, I'll also install these:

+ kdegraphics-thumbnailers: Image files and PDFs
+ kimageformats: Gimp `.xcf` files
+ qt5-imageformats : `.webp`, `.tiff`, `.tga`, `.jp2` files
+ kdesdk-thumbnailers: Plugins for the thumbnailing system
+ ffmpegthumbs: Video files (based on ffmpeg)
+ raw-thumbnailer: `.raw` files
+ taglib : Audio files

```
# pacman -S kdegraphics-thumbnailers kimageformats qt5-imageformats kdesdk-thumbnailers \
ffmpegthumbs raw-thumbnailer taglib
```

Enable preview showing of required file type in *Settings > Configure Dolphin... > General > Previews*.

There's a lot more thumbnail generators that can be found from the AUR (like a generator to create a thumbnail for `APK` files), but I don't really use them.


#### GUI-based Text Editors

`vim` is my text editor of choice, but sublime Text 3 is my go-to GUI text editor as it's lighter than the ~~bloated~~ chromium-based counterparts like `atom` and `vscode`.

```
$ yay -S sublime-text-dev
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

If you're using KDE Plasma, you don't need to do these:

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

#### Microcode

Processor manufacturers release stability and security updates to the processor microcode. These updates provide bug fixes that can be critical to the stability of your system. Without them, you may experience spurious crashes or unexpected system halts that can be difficult to track down. 

Install microcode.

For AMD processors:

```
# pacman -S amd-ucode
```

For Intel processors:

```
# pacman -S intel-ucode
```

If your Arch installation is on a removable drive that needs to have microcode for both manufacturer processors, install both packages. 

Load  microcode. For `systemd-boot`, use the `initrd` option to load the microcode, **before** the initial ramdisk, as follows:

```
# sudoedit /boot/loader/entries/entry.conf
```

```
title   Arch Linux
linux   /vmlinuz-linux
initrd  /CPU_MANUFACTURER-ucode.img
initrd  /initramfs-linux.img
...
```

Change `CPU_MANUFACTURER` with either `amd` or `intel` depending on your processor.


#### Plymouth

Plymouth provides a flicker-free graphical boot process. In short, a splash screen.

1. Install plymouth.

	```
	$ yay -S plymouth
	```

2. Add `plymouth` to the HOOKS array in mkinitcpio.conf. It must be added **after** `base` and `udev`/`systemd` for it to work: 

	```
	# sudoedit /etc/mkinitcpio.conf
	```

	- Unencrypted partition

		Put `plymouth` after base and udev:

		```
		HOOKS=(base udev plymouth ...)
		```

	- Encrypted partition **and** `systemd`-based initramfs.

		Again, you need a `systemd`-based initramfs for the `plymouth` HOOK to work.

		Put `sd-plymouth` after the `base` and `systemd` hooks:

		```
		HOOKS=(base systemd sd-plymouth ...)
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

	The `plasma-meta` package will include and install `sddm`. So there's no need to install one. Enable it by:

	If you installed and using `plymouth`, enable this to have a smooth transition:

	```
	# systemctl enable sddm-plymouth
	```

	If not:

	```
	# systemctl enable sddm
	```	

+ Awesome Window Manager

	I'm using `lightdm` with `lightdm-webkit2-greeter`, so this guide will cover that. 

	- First, install it:

		```
		# pacman -S lightdm lightdm-webkit2-greeter
		```

	- To enable graphical login, enable the appropriate systemd service. For example, for Lightdn, enable `lightdm.service`. Just because we're using plymouth, we will enable `lightdm-plymouth.service` to have a smooth transition from plymouth to lightdm.

		```
		# systemctl enable lightdm-plymouth
		```

	- Install a theme. I create my own lightdm-webkit2 theme and it's called [glorious](https://github.com/manilarome/lightdm-webkit2-theme-glorious).

		```
		$ yay -S lightdm-webkit2-theme-glorious
		```

	- Configure the lightdm to use lightdm-webkit2-greeter by:

		+ Set lightdm greeter session to webkit2.

			```
			# sudoedit /etc/lightdm/lightdm.conf
			```

			Find `greeter-session` under the `[Seat:*]` section, uncomment it, then set its value to `lightdm-webkit2-greeter`.

		+ Set it as the lightdm-webkit2 theme then enable `debug_mode` by setting it to `true`. Why do we need to enable `debug_mode`? Sometimes you will be greeted by an error. And this error is due to a race condition where the theme is trying to access the `lightdm` object even though it doesn't exist *yet*. Debug mode will allow you to `right-click` and `reload` the greeter just like a webpage.

			```
			# sudoedit /etc/lightdm/lightdm-webkit2-greeter.conf
			```

			Find `webkit_theme` then set its value to `glorious`. Find `debug_mode` then set it to true. If you encountered an error, right-click then reload.

#### Silent boot

This is for who prefer to limit the verbosity of their system to a strict minimum, either for aesthetics or other reasons. For me, it's aesthetics. 

Edit boot loader kernel parameters:

```
# sudoedit /boot/loader/entries/arch.conf
```

Add these parameters `(options ... loglevel=3 vga=current rd.udev.log_priority=3 fbcon=nodefer ...)` in the `options`:

```
options quiet loglevel=3 vga=current rd.udev.log_priority=3 fbcon=nodefer
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

#### Fan Control for Thinkpad

**For thinkpad users**, install `thinkfan` here.

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

Now, load the module: 

```
# modprobe thinkpad_acpi
# cat /proc/acpi/ibm/fan
```

You should see that the fan level is "auto" by default, but you can echo a level command to the same file to control the fan speed manually. The thinkfan daemon will do this automatically.

**For Lenovo x230 users**:

Open or create `/etc/thinkfan.conf`. Then use the following configuration: 

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
("level full-speed", 63, 32767)
```

To find the best thinkfan configuration for you, search it on the internet. I found mine on the ArchWiki. Maybe you can find yours there too.

**Make sure to have a configuration file before enabling the thinkfan service!** 

```
# systemctl enable thinkfan.service
```

#### Enable MAC randomization

MAC randomization can be used for increased privacy by not disclosing your real MAC address to the network. 

+ Randomization for iwd

	Create and edit `/etc/iwd/main.conf`. Then add the following lines:

	```
	[General]
	AddressRandomization=once
	AddressRandomizationRange=nic
	```

+ Randomization for network-manager

	- Install macchanger.

		```
		# pacman -S macchanger
		```

	- Create `30-mac-randomization.conf` in your `/etc/NetworkManager/conf.d/`. Add this:

		```
		[device-mac-randomization]
		# "yes" is already the default for scanning
		wifi.scan-rand-mac-address=yes

		[connection-mac-randomization]
		ethernet.cloned-mac-address=random
		wifi.cloned-mac-address=stable
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

