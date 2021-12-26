# Install physical server with Ubuntu Focal

This howto meant to be a collection of copy-pastable commands which can be used to create a general purpose server with the following:
* Two SSD-s with the following partition layout; feel free to alter the sizes:
  1. 512MB FAT32 to hold the boot loader (EFI System Partition)
  1. 40GB RAID member - to be used as swap
  1. the rest of the space as another RAID member - to be used as the root file system
* btrfs file system with subvolumes for snapshots
* Ubuntu Focal Server as the system
* Optonally create ZFS pool from anonter two SSDs in mirror.

## Start live system, enable ssh

### Preparations
Boot a Live ISO image in UEFI mode. My choise was [grml](https://grml.org/).
* In BIOS Boot settings, choose **UEFI boot** mode.

### Setup live environment
#### Option 1: with grml ISO
In the GRUB menu, press **E** to Edit the first option and add the following to the line begins with **linux**: `hostname=[YOUR HOSTNAME] ssh=YourAw3somePassw0rd`\
Then press **Ctrl** + **X** to boot
This will set the hostname and start SSH with the above password. Awsum, ha?

#### Option 2: other live ISO
When the live system is up, open Terminal and type:
```
sudo -i
passwd
apt install openssh-server
echo "PermitRootLogin yes" >> /etc/ssh/sshd_config
systemctl restart sshd
hostnamectl set-hostname [your hostname]
```

Now you can continue installation from your own machine by ssh-ing to the machine with **root** user and the password you set.

## Installation 

Log in to the live environment via SSH and start copypasting from here.\
Beware: this article has no brain; use your own!

`ssh root@[your hostname]`

### Preparations

Detect disks:
`lsblk`

Assume **sda** and **sdb** are the disks which should hold the system.

Install tools into live system:
```
apt-get update && apt-get -y install mdadm grub-efi-amd64 vim debootstrap arch-install-scripts
```
### Format disks

```
sgdisk -Z /dev/sda
sgdisk -Z /dev/sdb
sgdisk -n 1:0:+512M -t 1:ef00 -c 1:EFI /dev/sda
sgdisk -n 2:0:+40G -t 2:fd00 -c 2:"RAID_swap" /dev/sda
sgdisk -n 3:0:0 -t 3:fd00 -c 3:"RAID_btrfs" /dev/sda
sgdisk /dev/sda -R /dev/sdb -G
mkfs.fat -F32 -n EFI /dev/sda1
```

Create RAID1 arrays: (issue commands separately)
```
mdadm --create /dev/md0 --level=1 --raid-disks=2 /dev/sd[ab]2
mdadm --create /dev/md1 --level=1 --raid-disks=2 /dev/sd[ab]3
```
Check your work: 
```watch sudo cat /proc/mdstat```

It is advised to wait for the RAID sync to finish before continuing.<br>

Create swap and filesystem on top of RAID arrays:
```
sgdisk -Z /dev/md0
sgdisk -Z /dev/md1
sgdisk -N 1 -t 1:8200 -c 1:"swap" /dev/md0
sgdisk -N 1 -t 1:8300 -c 1:"system" /dev/md1
```

Activate swap:
```
mkswap -L swap /dev/md0
swapon -L swap
```

### BTRFS setup

Create BTRFS file system with subvolumes and EFI partition:\
Feel free to use a different subvol layout or mount options.\
Unfortunately `@` as the root subvol is mandatory for **apt-btrfs-snapshot** to work but others are free to create:
```
o_btrfs=defaults,x-mount.mkdir

subvols=('@' '@home' '@log' '@tmp')
mountpoints=('/' '/home' '/var/log' '/tmp')

mkfs.btrfs --force --label=system /dev/md1
mount -t btrfs LABEL=system /mnt
for mysubv in ${subvols[@]}; do btrfs subvolume create /mnt/${mysubv}; done

umount -R /mnt

for i in $(seq 1 ${#subvols} ); do mount -t btrfs -o subvol=${subvols[i]},$o_btrfs LABEL=system /mnt/${mountpoints[i]}; done

mkdir -p /mnt/boot/efi
mount -o $o_btrfs LABEL=EFI /mnt/boot/efi
```

### Install base system

```
MIRROR="http://hu.archive.ubuntu.com/ubuntu"
debootstrap --arch amd64 focal /mnt $MIRROR
```
This should took a while, take a break.

### Initial setup
Chroot to the new install and make initial config: (issue commands separately)
```
genfstab -L -p /mnt > /mnt/etc/fstab
arch-chroot /mnt
passwd                         # set temporary password for root
dpkg-reconfigure locales       # choose en_US.UTF-8 UTF-8 and make it default locale
dpkg-reconfigure tzdata        # Europe -> Budapest
dpkg-reconfigure console-setup # defaults will do
```

### Install basic packages
Add local ubuntu repos to `/etc/apt/sources.list`:
```
cat << 'EOF' > /etc/apt/sources.list;
deb http://hu.archive.ubuntu.com/ubuntu focal main restricted universe multiverse
deb http://hu.archive.ubuntu.com/ubuntu focal-updates main restricted universe multiverse
deb http://hu.archive.ubuntu.com/ubuntu focal-backports main restricted universe multiverse
deb http://hu.archive.ubuntu.com/ubuntu focal-security main restricted universe multiverse
EOF
```

Update system and install basic packages:
```
apt update && apt upgrade -y && apt install -y apt-btrfs-snapshot python3-distutils ubuntu-server man-db manpages-posix dnsutils zfsutils-linux tasksel linux-image-generic grub-efi-amd64 efibootmgr btrfs-progs vim mdadm openssh-server
```
This should took a while, take a break.

### Install boot loader and clone it to 2nd disk
```
update-grub # not sure it is needed here as well but causes no harm

grub-install --boot-directory=/boot --bootloader-id=Ubuntu --target=x86_64-efi --efi-directory=/boot/efi --recheck
sed -i 's/quiet splash/nomodeset/g' /etc/default/grub
update-grub

umount /dev/sda1
dd if=/dev/sda1 of=/dev/sdb1
```

Insert second disk to EFI menu:
```
efibootmgr -c -g -d /dev/sdb -p 1 -L "Ubuntu #2" -l '\EFI\Ubuntu\grubx64.efi'
```

Allow root login on SSH: 
```
echo "PermitRootLogin yes" >> /etc/ssh/sshd_config
```

Exit chroot, shut down live system, remove the live ISO and boot the new install!

### Optional: create ZFS pool from another SSD pair
Assume **sdc** and **sdd** are the data disks, check with `lsblk`:

```
zpool create -o ashift=12 tank mirror sdc sdd -f
zpool export tank
rm /dev/disk/by-id/wwn*
zpool import tank -d /dev/disk/by-id

# options stolen from Ubuntu factory install
zfs set xattr=sa compression=lz4 dnodesize=auto acltype=posixacl atime=off canmount=off mountpoint=none tank

zfs create tank/lxd
zfs create tank/kvm
zfs create -o mountpoint=/tank/kvm/iso tank/kvm/iso
```

## Profit

------

References:

https://askubuntu.com/questions/660023/how-to-install-ubuntu-14-04-16-04-64-bit-with-a-dual-boot-raid-1-partition-on-an
