# Install KVM Windows 10 to Focal-based vserver

Assume virtio and Win10 ISO-s are already placed in `/tank/kvm/iso`.
If not then get them:
```
cd /tank/kvm/iso/
wget https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso
```

## Option 1: ZVOL as virtual storage (traditional)

Replace WINDOWS1 with actual hostname and adjust memory and CPU numbers accordingly:

```
zfs create tank/kvm/WINDOWS1
zfs create -V 60G -s -b 128k tank/kvm/WINDOWS1/vda

virt-install \
 --name WINDOWS1 \
 --memory 8192 \
 --vcpus 4 \
 --cpu host \
 --video cirrus \
 --features hyperv_relaxed=on,hyperv_spinlocks=on,hyperv_vapic=on \
 --clock hypervclock_present=yes \
 --disk /dev/zvol/tank/kvm/WINDOWS1/vda,bus=virtio \
 --disk /tank/kvm/iso/Win_Pro_2004.ISO,device=cdrom,bus=sata \
 --disk /tank/kvm/iso/virtio-win.iso,device=cdrom,bus=sata \
 --boot cdrom \
 --network bridge=br-eth0 \
 --graphics vnc,listen=0.0.0.0 \
 --noautoconsole \
 --os-type=windows \
 --os-variant=win10
```

Start **virt-manager** on your machine; add the server as new connection and begin Windows installation.\
Add `virtio` as storage driver from the iso when prompted by the installer.\
Do not forget to install KVM drivers from the virtio CD when the installation is completed.\
Remove CDROM devices from the VM config when you do not need them any more. 

## Option 2: Create QEMU virtual disk and use as storage (more flexible)

Replace WINDOWS2 with actual hostname and adjust memory and CPU numbers accordingly:

```
zfs create tank/kvm/WINDOWS2
zfs set mountpoint=/tank/kvm/WINDOWS2 
zfs set recordsize=64k tank/kvm/WINDOWS2
qemu-img create -f qcow2 -ocluster_size=64K /tank/kvm/WINDOWS2/WINDOWS2.qcow2 60G

virt-install \
 --name WINDOWS2 \
 --memory 8192 \
 --vcpus 4 \
 --cpu host \
 --video cirrus \
 --features hyperv_relaxed=on,hyperv_spinlocks=on,hyperv_vapic=on \
 --clock hypervclock_present=yes \
 --disk /tank/kvm/WINDOWS2/WINDOWS2.qcow2,format=qcow2,bus=virtio \
 --disk /tank/kvm/iso/Win_Pro_2004.ISO,device=cdrom,bus=sata \
 --disk /tank/kvm/iso/virtio-win.iso,device=cdrom,bus=sata \
 --boot cdrom \
 --network bridge=br-eth0 \
 --graphics vnc,listen=0.0.0.0 \
 --noautoconsole \
 --os-type=windows \
 --os-variant=win10
```

References:

 https://jrs-s.net/2018/03/13/zvol-vs-qcow2-with-kvm/
 https://www.reddit.com/r/zfs/comments/bz5ya2/zfs_kvm_ntfs_benchmarking_various_record_sizes/

