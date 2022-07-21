# Ubuntu 22.04 on ZFS with Regolith Desktop

My current working environment installation notes\
If you do not want ZFS then you can install 22.04 desktop on your way; then proceed to [install Regolith](#install-regolith).

## ZFS root installation
### Option 1. zfs-installer
This is the method I ended up doing.
It is tricky since Saverio Miroddi's excellent [zfs-installer](https://github.com/64kramsystem/zfs-installer) does not support Ubuntu 22.04 (yet?) only 20.04.\
I tested it and it stopped on the very first dialog; haven't investigate it in more detail.

Instead I spined up a [supported live image](https://releases.ubuntu.com/20.04/) and run the script with that and performed dist-upgrade to the next LTS release 22.04.

Tweaks I made:
- set bpool to 5GB (I struggled with it getting full when it was on the defaul 2GB)
- set swap to 5GB (default 2)
- set space at the end of the disk to 12GB (leaving it on 0 resulted in error previously)
- add `rpool/ROOT/var/lib/docker` and `rpool/ROOT/kvm` to default dataset list

## Option 2. Follow openzfs wiki
Thanks to OpenZFS team there is [an excellent howto on installing 22.04 to ZFS root](https://openzfs.github.io/openzfs-docs/Getting%20Started/Ubuntu/Ubuntu%2022.04%20Root%20on%20ZFS.html).
It is not that straightforward but explains everyt step in detail.

## Install zsys
Seems like ZFS support in Ubuntu and zsys in particular is abandoned in Ubuntu but it is still installable and usable in 22.04 so go ahead and install it:

```sh
sudo apt install zsys
```

It will create restore points on every `apt` operation so you can reboot to a previous state if something goes wrong by choosing a snapshot from **History for Ubuntu 22.04 LTS** in grub menu.


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

Reboot and enjoy LightDM login and debloated system.

## Additional packages

### From package manager

```sh
sudo snap install spotify chromium dracula-gtk-theme
sudo apt install git git-lfs gitk curl pass stow screen tmux fonts-powerline zsh exa fd-find ripgrep fzf bat autojump maim geany
```

Set Chromium as default browser\
Extensions:
 - ublock origin
 - I don't care about cookies
 - https everywhere
 - dracula


### Manual installs

#### neovim
Get "stable" **.deb** from github releases\
https://github.com/neovim/neovim/releases/tag/stable

#### Alacritty
There is a snap package but it failed to run when I tried so I ended up installing it manually.\
https://github.com/alacritty/alacritty/blob/master/INSTALL.md

Set it as default xterm:
`sudo update-alternatives --install /usr/bin/x-terminal-emulator x-terminal-emulator $(which alacritty) 50`

#### lsd aka. ls deluxe
Get **.deb** from github releases\
https://github.com/Peltoche/lsd/releases

exa is already installed but lsd seems more like a drop-in replacement for ls so I ended up using it via an [ls alias](https://github.com/lehoczkics/dotfiles/blob/master/zsh/.zshrc#L150).

#### delta
Get **.deb** from github releases\
https://github.com/dandavison/delta/releases

#### Patched fonts

Download and install the following font sets. They are required for p10k ZSH theme.

[Regular](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Regular.ttf),
[Bold](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold.ttf), 
[Italic](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Italic.ttf),
[Bold+Italic](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold%20Italic.ttf)

Set your xterm to use **MesloLGS NF**, [example for Alacritty here](https://github.com/lehoczkics/dotfiles/blob/master/alacritty/.config/alacritty/alacritty.yml#L10)

#### ZSH environment

Install ohmyzsh:\
`sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`

Install p10k theme:\
`git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k`

Install zsh extras:\
`git clone https://github.com/zsh-users/zsh-autosuggestions.git $ZSH_CUSTOM/plugins/zsh-autosuggestions`
`git clone https://github.com/zsh-users/zsh-syntax-highlighting.git $ZSH_CUSTOM/plugins/zsh-syntax-highlighting`

## Optional: Install dotfiles
To get the whole thing working, proceed to get [dotfiles](https://github.com/lehoczkics/dotfiles); or at least [the zsh folder](https://github.com/lehoczkics/dotfiles/tree/master/zsh)

## References
https://medium.com/@satriajanaka09/setup-zsh-oh-my-zsh-powerlevel10k-on-ubuntu-20-04-c4a4052508fd

