## Install KVM in Oracle Linux v9.4
```
sudo dnf install qemu-kvm -y
sudo dnf install libvirt libvirt-client -y
sudo dnf install libguestfs-tools virt-top virt-install virt-manager libguestfs-tools-c guestfs-tools -y
sudo systemctl enable --now libvirtd
sudo systemctl status libvirtd
virsh list --all
sudo dnf install cockpit cockpit-machines
sudo systemctl enable --now cockpit.socket
sudo systemctl status cockpit.socket
sudo firewall-cmd --add-service=cockpit --permanent
sudo firewall-cmd --reload
```

## Download Ubuntu 24.04 cloud image
```
wget https://cloud-images.ubuntu.com/releases/24.04/release-20240423/ubuntu-24.04-server-cloudimg-amd64.img
```

## Let's create a network
```
<network>
  <name>openshift4</name>
  <forward mode='nat'>
    <nat>
      <port start='1024' end='65535'/>
    </nat>
  </forward>
  <bridge name='openshift4' stp='on' delay='0'/>
  <domain name='openshift4'/>
  <ip address='192.168.100.1' netmask='255.255.255.0'>
  </ip>
</network>  
```
```
sudo virsh net-define --file virt-net.xml
sudo virsh net-autostart k8s
sudo virsh net-start k8s
sudo virsh net-list
```

## Let's create a VM using KVM
```
wget https://cloud-images.ubuntu.com/releases/24.04/release-20240423/ubuntu-24.04-server-cloudimg-amd64.img
```

### Create master-1 VM Live Image
```
qemu-img create -b ubuntu-24.04-server-cloudimg-amd64.img -F qcow2 -f qcow2 master-1.qcow2 500G
qemu-img resize master-1.qcow2 500G
mv master-1.qcow2 /var/lib/libvirt/images/master-1.qcow2

virt-customize --add /var/lib/libvirt/images/master-1.qcow2 --root-password password:Root@123

virt-customize -a /var/lib/libvirt/images/master-1.qcow2 --install net-tools,network-manager,vim,git,iputils-ping

sudo virt-install --name master-1 \
--memory 131072 \
--vcpus 8 \
--disk /var/lib/libvirt/images/master-1.qcow2,device=disk,bus=virtio \
--os-variant ubuntu24.04 \
--virt-type kvm --graphics none \
--network network=k8s,model=virtio \
--noautoconsole \
--import
```

Configure IP address for the VM
```
nmcli con add type ethernet con-name enp1s0 ifname enp1s0 \
  connection.autoconnect yes ipv4.method manual \
  ipv4.address 192.168.10.2/24 ipv4.gateway 192.168.10.1 \
  ipv4.dns 8.8.8.8

ping -c 2 8.8.8.8
ping -c 2 google.com
```

### Create master-2 VM Live Image
```
qemu-img create -b ubuntu-24.04-server-cloudimg-amd64.img -F qcow2 -f qcow2 master-2.qcow2 500G
qemu-img resize master-2.qcow2 500G
mv master-2.qcow2 /var/lib/libvirt/images/master-2.qcow2

virt-customize --add /var/lib/libvirt/images/master-2.qcow2 --root-password password:Root@123

virt-customize -a /var/lib/libvirt/images/master-2.qcow2 --install net-tools,network-manager,vim,git,iputils-ping

sudo virt-install --name master-2 \
--memory 131072 \
--vcpus 8 \
--disk /var/lib/libvirt/images/master-2.qcow2,device=disk,bus=virtio \
--os-variant ubuntu24.04 \
--virt-type kvm --graphics none \
--network network=k8s,model=virtio \
--noautoconsole \
--import
```

Test if the VM internet connectivity is working
```
nmcli con add type ethernet con-name enp1s0 ifname enp1s0 \
  connection.autoconnect yes ipv4.method manual \
  ipv4.address 192.168.10.3/24 ipv4.gateway 192.168.10.1 \
  ipv4.dns 8.8.8.8

ping -c 2 8.8.8.8
ping -c 2 google.com
```

### Create master-3 VM Live Image
```
qemu-img create -b ubuntu-24.04-server-cloudimg-amd64.img -F qcow2 -f qcow2 master-3.qcow2 500G
qemu-img resize master-3.qcow2 500G
mv master-3.qcow2 /var/lib/libvirt/images/master-3.qcow2

virt-customize --add /var/lib/libvirt/images/master-3.qcow2 --root-password password:Root@123

virt-customize -a /var/lib/libvirt/images/master-3.qcow2 --install net-tools,network-manager,vim,git,iputils-ping

sudo virt-install --name master-3 \
--memory 131072 \
--vcpus 8 \
--disk /var/lib/libvirt/images/master-3.qcow2,device=disk,bus=virtio \
--os-variant ubuntu24.04 \
--virt-type kvm --graphics none \
--network network=k8s,model=virtio \
--noautoconsole \
--import
```

Test if the VM internet connectivity is working
```
nmcli con add type ethernet con-name enp1s0 ifname enp1s0 \
  connection.autoconnect yes ipv4.method manual \
  ipv4.address 192.168.10.4/24 ipv4.gateway 192.168.10.1 \
  ipv4.dns 8.8.8.8

ping -c 2 8.8.8.8
ping -c 2 google.com
```
