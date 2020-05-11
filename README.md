# virt-installs

Collection of my virt-install scripts to quickly spawn up some VMs I often
need. This is very personal thing, but feel welcome to use them if you find
them useful. I used to use Vagrant, but as it doesn't support libvirt, I gave
up and rather use virt-install nowadays.

I use these on my RHEL8 laptop, but they should work anywhere.

If you want to do more complicated setups, here is an example how I create VMs
for [OpenShift clusters using libvirt through
Ansible](https://github.com/ikke-t/ocp-libvirt-infra-ansibles).

# Fedora

## If I want to try beta:

```
virt-install --name Fedora32 \
--description 'Fedora 32' \
--ram 4096 \
--vcpus 2 \
--disk path=~/VirtualMachines/f32.qcow2,size=20 \
--os-type linux \
--os-variant fedora30 \
--network bridge=virbr0 \
--graphics vnc,listen=127.0.0.1,port=5901 \
--cdrom ~/VirtualMachines/Fedora-Workstation-Live-x86_64-32_Beta-1.2.iso \
--noautoconsole
```

## Stable:

```
virt-install --name Fedora32 \
--description 'Fedora 32' \
--ram 4096 \
--vcpus 2 \
--disk path=~/VirtualMachines/f32.qcow2,size=20 \
--os-type linux \
--os-variant fedora30 \
--network bridge=virbr0 \
--graphics vnc,listen=127.0.0.1,port=5901 \
-l http://ftp.funet.fi/pub/linux/mirrors/fedora/linux/releases/32/Server/x86_64/os/ \
-x "ks=https://raw.githubusercontent.com/ikke-t/virt-installs/master/ks-f31.cfg" \
--noautoconsole
```

## With serial console:

```
virt-install --name Fedora32 \
--description 'Fedora 32' \
--ram 4096 \
--vcpus 2 \
--disk path=~/VirtualMachines/f32.qcow2,size=20 \
--os-type linux \
--os-variant fedora30 \
--network bridge=virbr0 \
--graphics none \
--serial pty \
--console pty,target.type=virtio \
-l http://ftp.funet.fi/pub/linux/mirrors/fedora/linux/releases/32/Server/x86_64/os/ \
-x "ks=https://raw.githubusercontent.com/ikke-t/virt-installs/master/ks-f31.cfg console=tty0 console=ttyS0,115200n8"
```

# CentOS 8

```
virt-install --name centos8 \
--description 'CentOS 8' \
--ram 4096 \
--vcpus 2 \
--disk path=~/VirtualMachines/centos8.qcow2,size=20 \
--os-type linux \
--os-variant rhel8-unknown \
--network bridge=virbr0 \
--graphics none \
--serial pty \
--console pty,target.type=virtio \
-l http://ftp.funet.fi/pub/linux/mirrors/centos/8.1.1911/BaseOS/x86_64/kickstart/ \
-x "https://github.com/ikke-t/virt-installs/raw/master/ks-rhel8.cfg console=tty0 console=ttyS0,115200n8"
```

# RHEL8

You need to have your own repo mirror. For example you could download the
install DVD and mount it somewhere as loop device, and share the rpm contents
with lighttpd container.

```
virt-install --name rhel8 \
--description 'RHEL 8' \
--ram 4096 \
--vcpus 2 \
--disk path=~/VirtualMachines/rhel8.qcow2,size=20 \
--os-type linux \
--os-variant rhel8-unknown \
--network bridge=virbr0 \
--graphics none \
--serial pty \
--console pty,target.type=virtio \
-l http://download-node-02.eng.bos.redhat.com/rhel-8/rel-eng/RHEL-8/latest-RHEL-8/compose/BaseOS/x86_64/os/ \
-x "ks=https://github.com/ikke-t/virt-installs/raw/master/ks-rhel8.cfg console=tty0 console=ttyS0,115200n8"
```


