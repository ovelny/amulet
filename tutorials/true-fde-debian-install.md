# True FDE with Debian install

This guide provides step by step instructions for a FDE install on Debian, *including* the /boot partition which is usually left unencrypted. It has been tested with Debian bullseye.

There are ways to avoid typing the passphrase twice at boot, but I personally don't care. See the references section if you're interesting in setting this up.

Be aware that the first passphrase prompt will ignore your keyboard layout and use qwerty: if you can't type your passphrase with this, you're in trouble.

## Debian install

While booting up with debian ISO, select graphical install. The first part is pretty straightforward: select your region, keyboard layout etc until you reach the partition screen. From there, select manual partitioning.

We will partition our system in the simplest way: a main partition and a /boot partition, nothing else. Setting up a swap partition is easy if you want to (see references if interested).

Select your device, confirm and then follow the next steps:

* Create /boot partition:
	* Select `pri/log - FREE SPACE`
	* Choose "create a new partition"
	* Pick an appropriate size for a /boot partition: 512 MB is a bit overkill but ensures that you won't ever have to resize it
	* Choose "Primary"
	* Choose "Beginning"
	* Change the mount point to "/boot"
	* Choose "done setting up the partition"
* Create main partition:
	* Select again `pri/log - FREE SPACE`
	* Choose "create a new partition"
	* Since this our last and main partition, select all available space with "max"
	* Choose "primary"
	* Change "use as: Ext4..." to "physical volume for encryption"
	* Choose "Erase data: yes"
	* Choose "done setting up the partition"
* Encrypt the main partition:
  * Choose "configure encrypted volumes"
	* Choose "yes"
  * Choose "create encrypted volumes"
  * Choose your main partition and press "continue"
  * Choose "finish"
  * Choose "yes"
  * Pick a good encryption passphrase
  * Choose your disk under the "Encrypted volume..." section
  * Change the mount point to "/" (root file system)
  * Choose "done setting up the partition"
* Choose "finish partitioning and write changes to disk"
* A warning about swap will appear, simply ignore it and choose "no" to continue
* Choose "yes"

The rest of the installation will prompt you about additional packages you might want to install, your preferred DE etc. All of this is up to you, proceed until the end.

Finally, choose "yes" when prompted to install GRUB to your primary drive, and choose your device.

## Encrypting the /boot partition

Reboot and login as your normally would on your fresh system.

Then follow those steps:

* Launch a terminal and use `su`
* Run `lsblk` and find your encrypted device (which should be just above \*\_crypt partition)
* Run the following command according to your device `sudo cryptsetup luksDump /dev/<something>`
* Confirm that the line `Version: 2` is present and that `0: luks2` is present in `Keyslots`

To encrypt /boot, we need to convert the LUKS2 device to LUKS1. Reboot the system and press `e` at the GRUB menu, add `break=mount` to the end at the linux line for the kernel and press `F10`. You will drop into a initramfs shell.

Then run the following commands, one by one:

```bash
cryptsetup luksConvertKey --pbkdf pbkdf2 /dev/<something>
cryptsetup convert --type luks1 /dev/<something>
cryptsetup luksDump /dev/<something>
```

The last command should now show the line `Version: 1` and `Key Slot 1: ENABLED` (while the others are disabled, 0 to 7).

Continue with boot by typing `exit` and login.

Open a terminal and run the following commands:

```bash
su
mount -o remount,ro /boot
cp -axT /boot /boot.tmp
umount /boot
rmdir /boot
mv -T /boot.tmp /boot
```

Next, edit `/etc/fstab` and comment out the `/boot` mountpoint entry:

```bash
# UUID=... /boot <bla bla bla...>
```

We're almost there. Run the following:

```bash
echo "GRUB_ENABLE_CRYPTODISK=y" >> /etc/default/grub
sudo update-grub
sudo grub-install /dev/sda
```

Reboot and you should now need to enter your passphrase before entering GRUB. All done.

## Bonus: managing the old /boot partition and SSD disk tweak

You'll notice that the initial /boot partition is still there, and you can mount it with `disks` or any similar tool. What of it?

You could always keep it: it doesn't take a lot of space and comparing the original partition with the copy made in the /boot directory could be useful. You could even keep a sha256sum of the original kernel and compare it with the directory one with a script after booting up.

If you don't mind those checks, you can also ditch the partition. For this, you'll need to boot on a live USB drive with any flavor of Linux, run gparted (for instance), delete the partition and use that free space with your main / partition.

Once that choice is made, keeping it or not, there is one last edit you can do in `/etc/default/grub` if you have a SSD disk. Boot up normally and run `ls /dev/mapper` to identify your encrypted partition's name (for instance, sda2_crypt).

Then edit the `GRUB_CMDLINE_LINUX=""` line in `/etc/default/grub` and replace it with the following, making sure to use the correct partition:

```bash
GRUB_CMDLINE_LINUX="cryptdevice=/dev/sda2:sda2_crypt:allow-discards"
```

## References

* https://www.unixsheikh.com/tutorials/real-full-disk-encryption-using-grub-on-debian-linux-for-bios.html
* https://www.dwarmstrong.org/fde-debian/
* https://xo.tc/setting-up-full-disk-encryption-on-debian-jessie.html
