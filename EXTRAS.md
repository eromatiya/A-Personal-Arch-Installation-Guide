## Extras

The sole purpose of this notes is to help me remember my workflow. Here are some of the things I always do.

### Theming

- Icon themes
	+ papirus-icon-theme
	+ tela-icon-theme
	+ tela-circle-icon-theme

- GTK Theme
	+ whitesur-dark
	+ orchis theme

- Qt5 Theme
	+ `qt5ct` with kvantum

- Cursor Theme
	- vimix-cursors

### Shell

Friendship ended with `zsh`. `fish` is my new best friend.


1. Install `fish`:

	```
	# pacman -S fish
	```

2. Set `fish` as interactive shell. We will still use `bash` as the default shell. Not setting fish as system wide or user default allows the current Bash scripts to run on startup. It ensures the current user's environment variables are unchanged and are exported to fish which then runs as a Bash child. Add this on your `~/.bashrc`.

	```
	if [[ $(ps --no-header --pid=$PPID --format=cmd) != "fish" ]]
	then
		exec fish
	fi
	```

3. Install `pkgfile` to enable `Command not found` hook. This is optional.

	```
	# pacman -S pkgfile
	# Update pkgfile databse
	# pkgfile --update
	```

4. Update autosuggestion database by running:
	
	```
	$ fish_update_completions
	```

5. Install oh-my-fish.

	```
	$ curl -L https://get.oh-my.fish | fish
	```

6. Install a prompt theme. There's a bunch of themes available that can be found [here](https://github.com/oh-my-fish/oh-my-fish/blob/master/docs/Themes.md). Speaking of themes, I created my own and you can find it [here](https://github.com/manilarome/fishblocks). Install the theme you want, then move on. Installing a theme from `omf` is easy, just run `omf install THEMENAME` and it will do its job.


7. Install some useful plugins.
	
	```
	$ omf install archlinux bang-bang cd colorman sudope vcs
	```
	
	`colorman` plugin need to be source:

	```
	$ echo "source ~/.local/share/omf/pkg/colorman/init.fish" >> ~/.config/fish/config.fish
	```
	
### Removing GTK3-CSDs

I don't like CSDs on a non GNOME environment. Having them removes the unified look of the desktop. Replacing GTK3 with `gtk3-classic` will fix this problem.

```
$ yay -S gtk3-classic
```

Notable changes after installing `gtk3-classic`:

- CSDs are totally disabled by default. (Xorg only)
- Typeahead feature is restored.
- Message dialogs have traditional appearance with left-aligned texts and right-aligned buttons.
- Scrollbars are always visible. You can enable invisible scrollbars by `GTK_OVERLAY_SCROLLING=1` environment variable.
- Labels are wrapped similarly to GTK2. This patch fixes too wide windows in applications improperly ported from GTK2.
- Any many more!


## Daily Apps

- Media
	+ `vlc`: media player
	+ `mpd`, `mpc`, `ncmpcpp`: CLI music player
	+ `feh` or `gwenview`: image viewer
	+ `simplescreenrecorder`: screen recorder
	+ `pulseaudio-equalizer-ladspa`: equalizer
	+ `pavucontrol-qt`: pulseaudio mixer

- Terminal Emulator Eyecandies
	+ `neofetch`: CLI information tool
	+ `cava`: audio visualizer
	+ `pipes.sh`: animated pipes screensaver
	+ `cmatrix`: matrix screesaver

- Media and Graphics Editing Tools
	+ `gimp`: image manipulation program
	+ `inkscape`: vector graphics editor
	+ `ffmpeg`: video converter
	+ `imagemagick`: image viewing/manipulation program

- Misc
	+ `maim` or `spectacle`: screenshot tool
	+ `mupdf`: as pdf viewer
	+ `foliate`: ebook reader
	+ `redshift`: blue light filter
	+ `mugshot`: personal user details updater
	+ `pamac-aur`: gui package manager
	+ `arandr`: xrandr front-end
	+ `transmission-qt`: Bittorent client
	+ `htop`: CLI interactive process viewer
	+ `scrcpy`: android viewing tool
	+ `partitionamanager`: partition manager
	+ `aircrack-ng`: network tool
	+ `fluxion`: network tool
	+ `create_ap`: NATed/Bridged Software Access Point, basically hotspot creator

- Hotel
	+ `trivago`
