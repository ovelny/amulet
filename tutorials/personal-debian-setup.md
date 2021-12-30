# Personal Debian setup

This document is a step-by-step process (still WIP) to reproduce the setup I like for a fresh Debian install. As such, it is unlikely to fit your needs and preferences and is mainly aimed for me.

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
* Librewolf

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

Then sync the `~/bin` in the same way and run `chmod +x ~/bin/dot`.

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

## Sync main directories with syncthing

The following directories should be synced with syncthing:

* TBA

## Restic config and restore

Restic's "key" is already present in `~/.dotfiles`, and its symlink can be found in `~/.config/restic/restic_key.gpg` after running the previous steps.

You can use that key to decrypt restic's repo with the following flag:

```bash
restic --password-command "gpg --decrypt --default-recipient-self /home/ovelny/.config/restic/restic_key.gpg" <rest-of-the-command>...
```

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
ln -s /home/<user>/code/vim-cursed/.vimrc /home/<user>/

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

## Configure librewolf
