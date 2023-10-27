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

'''
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
'''

https://developers.redhat.com/blog/2020/08/18/iptables-the-two-variants-and-their-relationship-with-nftables#

'''
sudo systemctl status libvirtd
sudo systemctl start libvirtd
sudo systemctl enable libvirtd
'''
