# Personal Debian setup

This document is a step-by-step process (still WIP) to reproduce the setup I like on a fresh Debian install. As such, it is unlikely to fit your needs and preferences and is mainly aimed for me.

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

* `python3-pip`
* `vim-nox`
* `syncthing`
* `kitty`
* `restic`
* `keepassxc`
* `qemu-system`
* `fzf`
* `ufw` (then run sudo ufw enable)
* `zsh`
* `zsh-autosuggestions`
* `zsh-syntax-highlighting`
* `imagemagick`
* `libimage-exiftool-perl`
* `ffmpeg`
* `mpv`
* `sct`
* `qutebrowser`
* `higan`
* `tldr` (then run tldr -u)
* bullseye-backports: `pipx`

Those should already be present but better check:

* `ruby`
* `gnome-tweaks`
* `git`
* `gpg`

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

## Set autostart scripts

## Configure custom shortcuts

## Configure gnome tweaks and extensions

Reduce keyboard repeat delay

## Sync main directories with syncthing

## Restic config and restore

## Restore dotfiles

## Restore SSH and GPG keys

You know where to find your keys. Once you got them, restore them and set them as default with the following commands:

## Clone some repositories

vim-cursed
amulet
blog

## Install vim plugins
