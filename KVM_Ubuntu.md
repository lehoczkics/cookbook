# Install Ubuntu KVM to ZFS ZVOL onto Ubuntu 20.04 server

See [Focal server install howto](focal-server.md) for how to install the physical machine itself.
<br>
Make sure Focal Server installer ISO is present at `/tank/kvm/iso/focalserver.iso`\
In this examlpe 2 vCPUs, 16GB RAM and 300GB disk is used; feel free to use different values.

## Create ZVOL

```
zfs create tank/kvm/SERVERNAME
zfs create -V 300G -s -b 128k tank/kvm/SERVERNAME/vda
```

## Create VM

```
virt-install --name SERVERNAME --memory 16384 --vcpus 2 \
 --disk /dev/zvol/tank/kvm/SERVERNAME/vda \
 --disk /tank/kvm/iso/focalserver.iso,device=cdrom,bus=sata \
 --network bridge=br-eth0 \
 --noautoconsole --graphics=vnc \
 --os-variant ubuntu20.04 --boot cdrom
```

Open VM console and proceed with installation.\
"Guided" partitioning is good enough; untick LVM since that is fultile in this case.\
Set `ubuntu` as default user and add some password to it; enable SSH.\
Get network interface MAC address and add it to DHCP.
<br>
When the installation is complete, remove the virtual CD from the VM config and reboot.

## Postinstall steps

Disable "persistent interface naming" via kernel parameters:\
edit `/etc/default/grub` and add parameters: `GRUB_CMDLINE_LINUX_DEFAULT="biosdevname=0 net.ifnames=0"`\
then perform `update-grub`
<br>

Delete default network config and add simple "eth0 with DHCP":
```
rm /etc/netplan/*

cat <<EOF > /etc/netplan/10-interfaces.yaml;
network:
        version: 2
        renderer: networkd
        ethernets:
                eth0:
                        dhcp4: true
EOF
```

Reboot for get eth0 working. 

