## Extras

The sole purpose of this notes is to help me remember my workflow. Here are some of the things I always do.

### Shell

Friendship ended with `zsh`. `fish` is my new best friend.


1. Install `fish`:

	```
	# pacman -S fish
	```

2. Set `fish` as interactive shell. We will still use `bash` as the default shell. Not setting fish as system wide or user default allows the current Bash scripts to run on startup. It ensures the current user's environment variables are unchanged and are exported to fish which then runs as a Bash child. Add this on your `~/.bashrc`.

	```
	if grep -qxv 'fish' /proc/$PPID/comm && [[ ${SHLVL} == [1,2] ]]
	then
	        shopt -q login_shell && LOGIN_OPTION='--login' || LOGIN_OPTION=''
	        exec fish $LOGIN_OPTION
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
	$ curl https://raw.githubusercontent.com/oh-my-fish/oh-my-fish/master/bin/install | fish
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

### Theming

#### Themes:
Colors: PlasmaXDark
App Style: Darkly
	- Everything 75% transparent except sidebar and toolbar which are 70%
Plasma Style: WhiteSur-dark
Window Decor: Darkly
Icons: Colloid
Pointer: Colloid
Boot Splash: Arch 10

#### Font:
	- Inter
	- MesloLGS NF

#### Desktop Effects:
	- Better Blur DX
	- Geometry Change
	- Rounded Corners(Formerly ShapeCorners)

#### Panels

##### Top Panel:
	- Kmenu
	- Window Title
	- Global Menu
	- Clock
	- Status Bar

##### Dock:
	- Plasma Colorizer
	- Tahoe Launcher
	- Icon-Only Task Manager
	- Separator
	- Virtual Desktop Bar
	- Folders
	- Plasmusic
	- Color Picker

#### Virtual Desktops
	- Tools
	- Internet
	- Gaming
	- Media
	- Workspace
