# Kali on LXC
Prerequisites: running Proxmox ve

## Get CT template
Navigate [here](https://uk.lxd.images.canonical.com/images/kali/current/amd64/default/) then choose latest folder and download `rootfs.tar.xz`.\
Then upload it to Proxmox storage as **CT Template**.

## Deploy CT
Give it suuficient resources.
Do not forget to untick "Unprivileged"!
Do not start it just yet.

## Edit lxc config
In Proxmox CLI, look for the config file by container No. in `/etc/pve/lxc` and edit it.\
Append the following to the file:

```sh
lxc.cgroup.devices.allow: c 10:200 rwm
lxc.hook.autodev: sh -c "modprobe tun; cd ${LXC_ROOTFS_MOUNT}/dev; mkdir net; mknod net/tun c 10 200; chmod 0666 net/tun"
```

This will allow you to create a tunnel interface and run OpenVPN.

## Install packages

Start the CT; update packages and install basic stuff:

```sh
apt update
apt full-upgrade
apt install kali-linux-headless seclists kali-tools-windows-resources
```

To be continued...

