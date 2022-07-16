# Ubuntu 22.04 on ZFS with Regolith Desktop

My current working environment installation notes
If you do not want ZFS then you can install 22.04 desktop your way then proceed to [install Regolith](#install-regolith).

## ZFS root installation
### Option 1. zfs-installer
This is the method I ended up doing.
It is tricky since Saverio Miroddi's excellent [zfs-installer](https://github.com/64kramsystem/zfs-installer) does not support Ubuntu 22.04 (yet?) only 20.04.\
I tested it and it stopped on the very first dialog; haven't investigate it in more detail.

Instead I spined up a [supported live image](https://releases.ubuntu.com/20.04/) and run the script with that and performed dist-upgrade to the next LTS release 22.04.

## Option 2. Follow openzfs wiki
Thanks to OpenZFS team there is [an excellent howto on installing 22.04 to ZFS root](https://openzfs.github.io/openzfs-docs/Getting%20Started/Ubuntu/Ubuntu%2022.04%20Root%20on%20ZFS.html).
It is not that straightforward but explains everyt step in detail.

## Install Regolith
Assuming you have a working Ubuntu 22.04 desktop, you can install Regolith on top of it.
Just follow the steps on https://regolith-desktop.com/ main page, I copy it here for reference:

Add repo key:

```sh
wget -qO - https://regolith-desktop.io/regolith.key | \
gpg --dearmor | sudo tee /usr/share/keyrings/regolith-archive-keyring.gpg > /dev/null
```

Add PPA repo:

```sh
echo deb "[arch=amd64 signed-by=/usr/share/keyrings/regolith-archive-keyring.gpg] \
https://regolith-desktop.io/release-ubuntu-jammy-amd64 jammy main" | \
sudo tee /etc/apt/sources.list.d/regolith.list
```

Install Regolith:

```sh
sudo apt update
sudo apt install regolith-desktop 
```

Reboot and choose Regolith session at login.

Additional Regolith packages:

```sh
sudo apt install regolith-lightdm-config regolith-look-* i3xrocks-focused-window-name i3xrocks-rofication i3xrocks-info i3xrocks-app-launcher i3xrocks-memory i3xrocks-battery
```

Remove unneeded packages:

```sh
sudo apt remove gdm3 gnome-shell evolution-data-server*
sudo apt autoremove
```

Reboot and enjoy LightDM. 


