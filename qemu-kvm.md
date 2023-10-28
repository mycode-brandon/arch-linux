https://www.youtube.com/watch?v=HfNKpT2jo7U

Hardware -> KVM (in kernel) -> qemu (emulator) -> vm/vm/vm

-> libvirt (providves api to qemu and kvm) ->

user/client tools:

virsh : command line tools for libvirt
virt-manager : gui alternative to virsh
virt-install : helper tool for new vm guests / part of virt-manager
virt-viewer : UI for interacting wiht VM's via vnc/spice / part of virt-manager

support tools:

dnsmasq : dns/dhcp server
dhclient : dhcp resolution
ebtables : setting up NAT
bridge-utils : bridge interfaces
openbsd-netcat : enables remote management over ssh

```
sudo pacman -Sy \
qemu \
dhclient \
openbsd-netcat \
virt-viewer \
libvirt \
dnsmasq \
dmidecode \
ebtables \
virt-install \
virt-manager \
bridge-utils
```

https://developers.redhat.com/blog/2020/08/18/iptables-the-two-variants-and-their-relationship-with-nftables#

```
sudo systemctl status libvirtd
sudo systemctl start libvirtd
sudo systemctl enable libvirtd
```

Permissions

qemu:///system
qemu:///user 

```
mkdir ~/.config/libvirt
sudo cp -v /etc/libvirt/libvirt.conf ~/.config/libvirt/
sudo chown brandon:wheel ~/.config/libvirt/libvirt.conf
```

edit `libvirt.conf`

`vim ~/.config/libvirt/libvirt.conf`

uncomment out `uri_default = "qemu:///system"`

`cat /etc/group` to list all groups and check to make sure libvirt group is available.

add user to libvirt group

`usermod -aG libvirt brandon` or `sudo gpasswd -a brandon libvirt`

douhble check with `getent group libvirt` to make sure user is there.

download iso and double check with `sha256sum <isoname>`

