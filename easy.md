






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
