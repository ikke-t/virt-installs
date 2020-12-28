# virt-installs

Collection of my virt-install scripts to quickly spawn up some VMs I often
need. This is very personal thing, but feel welcome to use them if you find
them useful. I used to use Vagrant, but as it doesn't support libvirt, I gave
up and rather use virt-install nowadays.

I use these on my RHEL8 laptop, but they should work anywhere.

If you want to do more complicated setups, here is an example how I create VMs
for [OpenShift clusters using libvirt through
Ansible](https://github.com/ikke-t/ocp-libvirt-infra-ansibles).

# Connecting to VM

I use ssh to connect to VM's, like ```ssh cloud-user@ip```.

I use virsh to control VMs on command line. And
[virt-manager](https://virt-manager.org/) or
[cockpit](https://cockpit-project.org/) for GUI.

To quickly connect to new VM by use: ```virsh console rhel8```.

# Setting up credentials

I drop some dummy password for root, and ssh-key for cloud-user. Change them to
yours, they are in kickstart files. See variables:

* **rootpw**: Put your encrypted password here. E.g: ```openssl passwd -6 foobar```
* **sshkey**: Your SSH publi key, so you can ssh to machine after it boots.

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

# RHEL8-Edge

You need to have your own rpm-ostree built. See
[instructions how to build one](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/composing_installing_and_managing_rhel_for_edge_images/index).
Once you have built, downlaoded and extracted it, you can host it e.g.
with lighttpd container:

```
podman run -ti --name lighttpd --rm -v "~/rhel-iot/:/var/www/localhost/htdocs:Z" -p 8080:80 sebp/lighttpd
```

Modify the local kickstart file to have your passwords, and your host's IP
address in the url, and start the install from extracted RHEL DVD repo:

```
virt-install --name rhel-edge \
  --description 'RHEL 8 Edge' \
  --memory 4096 \
  --vcpus 4 \
  --disk ~/VirtualMachines/rhel-edge.qcow2,size=8 \
  --os-variant rhel8.3 \
  --location http://download-node-02.eng.bos.redhat.com/rhel-8/rel-eng/RHEL-8/latest-RHEL-8/compose/BaseOS/x86_64/os/ \
  --initrd-inject=/home/ikke/rhel-iot/ks.cfg \
  --extra-args="ks=file:/ks.cfg console=tty0 console=ttyS0,115200n8"
```

After install of edge I typically switch it into virbr0 network. I noticed
It doesn't seem to be able to route to podman container in user namespace
from virtual machine in libvirt default network virbr0. If you run the both as
root, perhaps it would work with option ```--network bridge=virbr0```.

