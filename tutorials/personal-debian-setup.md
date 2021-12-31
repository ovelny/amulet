# Personal Debian setup

This document is a step-by-step process to reproduce the setup I like for a fresh Debian install. As such, it is unlikely to fit your needs and preferences and is mainly aimed for me.

## Installing Debian stable with FDE

[Follow the detailed guide here.](https://github.com/ovelny/amulet/blob/main/tutorials/true-fde-debian-install.md)

## Add current user to sudo group

Your default user might not be part of the sudo group at first:

* Be root: `su -`
* Run `usermod -aG sudo <username>` and `exit`
* Reboot
* Run `sudo whoami` and `groups` to ensure your user is added

## Enable Debian backports

Debian backports are useful to get the latest packages for non-critical tools. To install it, follow the instructions on https://backports.debian.org/Instructions/ which are simply:

* Add the following lines in /etc/apt/sources.list:
  * `deb http://deb.debian.org/debian bullseye-backports main contrib non-free` (check that bullseye is indeed your current version)
* Run `sudo apt update` and `sudo apt upgrade`

To search / install a package from the backports, just add the flag `-t bullseye-backports` to your usual `apt` commands:

* `sudo apt install -t bullseye-backports <package-name>`
* `apt search -t bullseye-backports <package-name>`

In case your `sources.list` file gets messed up, refer to the examples here: https://wiki.debian.org/SourcesList#Example_sources.list 

## Install base packages

Let's now install base packages for this host: unless specified otherwise, those can be installed from `stable`.

* python3-pip
* vim-nox
* syncthing
* kitty
* restic
* keepassxc
* qemu-system
* fzf
* ufw (then run sudo ufw enable)
* zsh
* zsh-autosuggestions
* zsh-syntax-highlighting
* imagemagick
* libimage-exiftool-perl
* ffmpeg
* mpv
* sct
* qutebrowser
* higan
* age
* htop
* tree
* tmux
* curl
* pulseeffects
* rsync
* ncdu
* wipe
* tldr (then run tldr -u)
* bullseye-backports: pipx

```bash
sudo apt install python3-pip vim-nox syncthing kitty restic keepassxc qemu-system fzf ufw zsh zsh-autosuggestions zsh-syntax-highlighting imagemagick libimage-exiftool-perl ffmpeg mpv sct qutebrowser higan age htop tree tmux curl pulseeffects rsync ncdu wipe tldr pipx/bullseye-backports && sudo ufw enable && tldr -u
```

Those packages should already be present but check anyway:

* ruby
* gnome-tweaks
* git
* gpg
* chrome-gnome-shell

## Ruby dependencies

```bash
sudo gem install nanoc fastimage exifr redcarpet rouge nokogiri rest-client builder adsf puma
```

## Python packages with pipx

```bash
for pkg in "glances" "yt-dlp" "frida-tools" "objection" "pex"; do pipx install "$pkg"; done
```

## Other packages fetched elsewhere

Those programs are to be found manually on the interwebs, installation is straightforward:

* Mullvad
* Steam
* Discord
* Librewolf (see last section)

## Change default programs

Set browser to librewolf and music + video to mpv. That's it.

## Change shell to zsh

```bash
chsh -s $(which zsh)
```

Restart shell and check with `echo $ZSH_VERSION` that it worked.

## Restore dotfiles

Start by restoring dotfiles: we'll have to launch syncthing manually once, before the autostart file takes over after this step.

Run `syncthing` (remove default folder too) and start syncing the `~/.dotfiles` directory. In advanced options, choose to ignore syncing permissions.

Then sync the `~/bin` in the same way and run `chmod +x ~/bin/dot`. Also sync the `~/sync` directory while you're at it.

Run `~/bin/dot install` and reboot. The updated zsh theme and automatic syncthing startup should be proof enough that your dotfiles are restored.

## Configure custom shortcuts

* Set launch calculator to super + c
* Set text editor to super + t
* Set launch terminal to ctrl + alt + t
* Set switch applications to disabled
* Set switch windows to alt + tab
* Set save a screenshot to pictures to super + print
* Set lock screen to ctrl + alt + l
* Set maximize window to ctrl + alt + y

## Configure gnome tweaks and extensions

Go to https://extensions.gnome.org/ (with firefox), install the extension and install the following:

* https://extensions.gnome.org/extension/949/bottompanel/
* https://extensions.gnome.org/extension/1128/hide-activities-button/
* https://extensions.gnome.org/extension/1110/hide-clock/
* https://extensions.gnome.org/extension/723/pixel-saver/

Enable them all except hide clock and check results.

Next: 

* Go to gnome tweaks > appearance > applications and select adwaita dark.
* Go to gnome tweaks > fonts and select hinting: full and antialiasing: subpixel.
* Go to accessibility > repeat keys and reduce repeat delay (roughly by half).

## Restore SSH and GPG keys

You know where to find your keys.

For SSH:

* Place your keys in `~/.ssh`
* `chmod 600 ~/.ssh/id_rsa`
* `chmod 600 ~/.ssh/id_rsa.pub`
* `ssh-add`

For GPG:

* Save secret & public keys in a temporary location
* `gpg --import <mygpgkey_pub.gpg>`
* `gpg --allow-secret-key-import --import <mygpgkey_sec.gpg>`
* Run `gpg --edit-key ovelny@protonmail.com`
* Run `trust` in gpg prompt and select ultimate trust (5)
* Run `gpg --list-secret-keys` to check trust has changed
* Delete keys in temporary location

## Restic config and restore

Restic's "key" is already present in `~/.dotfiles`, and its symlink can be found in `~/.config/restic/restic_key.gpg` after running the previous steps.

You can use that key to decrypt restic's repo with the following flag:

```bash
restic --password-command "gpg --decrypt --default-recipient-self /home/ovelny/.config/restic/restic_key.gpg" <rest-of-the-command>...
```

Ensure that your restic repository is intact by running `restic --password-command "<...>" -r /<repo-path> check`.

Since we're aiming for a fresh install, we're not going to restore the entire /home directory. Only the following should be restored:

* `~/Documents`
* `~/flashcards`
* `~/ovelwiki`
* `~/qemu`
* `~/Videos`
* `~/roms`

The rest is either already present thanks to syncthing or located in the second drive — at the time of writing this document.

The second drive — whatever its location is — can be restored entirely with the exception of the `Steam` directory. That one can be kept away unless you need to restore a save someday.

Use the following to restore a specific file or directory with restic, which should also work nicely for the entire 2nd drive:

```bash

# Grab the ID of the snapshot you wanna use
restic --password-command "gpg --decrypt --default-recipient-self /home/ovelny/.config/restic/restic_key.gpg" -r /<path-of-restic-repo> snapshots 

# Restore a directory or file
restic --password-command "gpg --decrypt --default-recipient-self /home/ovelny/.config/restic/restic_key.gpg" -r /<path-of-restic-repo> restore <ID> --target /<where-to-restore-directory> --include /<path-of-directory-to-restore> 
```

Keep in mind however that the absolute path is restored, not only the directory's content. This is how restic works and there is currently no way around it (except for mounting restic's repo somewhere, but I don't like that). Just `mv` the files accordingly and remove the emptied absolute path.

## Symlink Music directory

Your music directory is currently located on your secondary hard drive: after restic restore, delete `~/Music` and use a symlink to said drive's directory.

```bash
ln -s /<second-drive-path>/Music /home/ovelny/Music
```

## Sync main directories with syncthing

The following directories should be synced with syncthing:

* `~/.dotfiles` (already done)
* `~/bin` (already done)
* `~/ovelwiki`
* `~/Music/music-essentials` (can be synced to `~/Music` previous symlink even if syncthing might complain)
* `~/Pictures`
* `~/sync` (already done)

For those that are not synced already, the most pratical way is to delete the default directory if it already exists (like `~/Pictures`). That way you're sure you won't sync anything the other way around, from your new machine to the "original" device — a bit brutal but feels safer.

For all directories, make sure to disable syncing permissions in advanced options.

## Clone some repositories

```bash
mkdir ~/code && cd ~/code
git clone git@github.com:ovelny/vim-cursed.git
git clone git@github.com:ovelny/amulet.git
git clone git@github.com:ovelny/vessel.git
cd vessel && mkdir output && cd output
git clone git@github.com:ovelny/ovelny.github.io.git
```

## Set up .vimrc and install vim plugins

```bash
# Symlink vim config
ln -s /home/ovelny/code/vim-cursed/.vimrc /home/ovelny/.vimrc

# vim-plug installation
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

Next, run vim and execute `:PlugInstall` to install all plugins listed in `~/.vimrc`.

Finally, the YouCompleteMe plugin requires a few more steps for a full installation: https://github.com/ycm-core/YouCompleteMe#linux-64-bit

Pylint and black are also required with the previous config:

```bash
python3 -m pip install --user pylint
python3 -m pip install --user black
```

## Configure pulseeffects

Enable equalizer and select gstreamer_dance in presets.

## Configure qutebrowser certs

You know where to find your certs.

```bash
sudo apt install libnss3-tools

certutil -d "sql:$HOME/.pki/nssdb" -A -i ~/Downloads/<your-burp-cert> -n "Burp Suite CA" -t C,,

# Repeat last command for zaproxy cert and delete them from their temporary location
```

## Setup daily backups

The `backup_moon` script is already present in `~/bin`. Ensure that it is executable with `chmod +x ~/bin/backup_moon` and add the following line with `crontab -e`:

```bash
0 4 * * * /home/ovelny/bin/backup_moon
```

You should also check if all the paths in `backup_moon` are still the same with your new install, and change them accordingly.

## Configure librewolf

* Add librewolf's repository and install it with these instructions: https://librewolf.net/installation/debian/

* Install the following extensions:
	* privacy badger (disable showing count of trackers)
	* bot sentinel
	* fraidycat (add all the sites you follow by importing fraidycat.json in ~/sync)
	* wappalyzer (hide count of technologies found)
	* buster captcha solver
	* youtube nonstop
	* foxyproxy standard
	* gnome shell integration
	* firefox multi-account containers (if compatible)
	* sponsorblock for youtube (disable display of buttons)
	* vimium-ff (change scroll speed to 70px and disable smooth scrolling)
	* violentmonkey (hide scripts running count)

### Import and configure userscripts with violentmonkey

* Install the following scripts:
	* https://greasyfork.org/en/scripts/423851-simple-youtube-age-restriction-bypass
	* https://greasyfork.org/en/scripts/412178-youtube-dismiss-sign-in
	* https://gist.github.com/ovelny/0c08956715f574ec155d257af15b9777

### Foxyproxy configuration

* Set the following proxy settings:
	* city_ruins_zaproxy:
	  * proxy type: HTTP
		* IP address: 127.0.0.1
		* Port 8081
		* socks version: 5
		* socks proxy? empty
	* city_ruins_burpsuite:
	  * proxy type: HTTP
		* IP address: 127.0.0.1
		* Port 8080
		* socks version: 5
		* socks proxy? empty

### Librewolf settings

* Enable compact mode:
	* go to about:config and toggle browser.compactmode.show to true
	* go to more tools > customize toolbars > density and enable compact mode
* Enable dark theme:
	* go to about:addons > themes > enable dark theme
* Customize main settings:
	* go to about:preferences
	* check "open previous windows and tabs"
	* check "always check if librewolf is your default browser" and enable as default
	* in "home" tab, uncheck homepage content > web search (and everything else if checked)
	* in "privacy and security" tab:
		* uncheck "delete cookie and site data when LibreWolf is closed" in "Cookies and Site Data"
		* select "remember history" in history section
		* in certificates > view certificates, import burpsuite and zaproxy certificates
		* allow certificates for websites only, not emails
	* in "librewolf" tab:
		* check "extensions & themes auto-update"
		* uncheck "unable ipv6"
		* check "restore previous session"
		* uncheck "enable linux style copy/paste":
			* uncheck middle mouse paste in details
		* uncheck "advanced css styling"
		* uncheck "peer connections"
		* uncheck "screensharing"
		* uncheck "geolocation data"
		* check "hide canvas request popups"
		* uncheck "serve DRM-restricted content"
		* uncheck "window letterboxing"
		* check "disable asm.js"
		* uncheck "disable web assembly"
		* uncheck "enable google safe browsing"
		* uncheck "allow google safe browsing to view your downloads"
		* uncheck "allow installation of other language packs"

### Librewolf customization

* In more tools > customize toolbar:
	* select compact mode (enabled with special flag shown earlier)
	* in toolbars > bookmarks toolbar, select "never show"
	* uncheck title bar
	* hide all extensions (in overflow menu) except:
		* ublock
		* vimium
		* foxyproxy
		* wappalyzer
		* (in this order, left to right)
	* remove downloads icon in toolbar
	* add "forget" icon between extensions and overflow icon

## All done

Congrats, you have a system you can call home now. Grab a coffee and enjoy.
