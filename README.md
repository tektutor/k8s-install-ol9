## Install KVM in Oracle Linux v9.4
```
sudo dnf install -y oraclelinux-release-el9
sudo dnf config-manager --enable ol9_kvm_utils
sudo dnf update
sudo dnf group install "Virtualization Host"
sudo dnf install qemu-kvm virt-install virt-viewer
```

Configure libvirt service
```
for drv in qemu network nodedev nwfilter secret storage interface; 
  do
   sudo systemctl enable virt${drv}d.service
   sudo systemctl enable virt${drv}d{,-ro,-admin}.socket;
   sudo systemctl start virt${drv}d{,-ro,-admin}.socket; 
  done

sudo systemctl enable virtproxyd.service
sudo systemctl enable virtproxyd-tls.socket
sudo systemctl start virtproxyd-tls.socket

sudo systemctl list-units --type=socket virt*
```

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
wget https://cloud-images.ubuntu.com/daily/server/noble/current/noble-server-cloudimg-amd64.img
```

## Clone kubespray
```
su -
cd ~
git clone https://github.com/kubernetes-sigs/kubespray.git
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
sudo virsh net-autostart openshift4
sudo virsh net-start openshift4
sudo virsh net-list
```

## Let's create a VM using KVM
```
sudo virt-builder fedora-39  --format qcow2 \
  --size 500G -o /var/lib/libvirt/images/master-1.qcow2 \
  --root-password password:Root@123

sudo virt-install \
  --name master-1 \
  --ram 131072 \
  --vcpus 8 \
  --disk path=/var/lib/libvirt/images/master-1.qcow2 \
  --os-variant rhel8.0 \
  --network bridge=openshift4 \
  --graphics none \
  --serial pty \
  --console pty \
  --boot hd \
  --import
```

Connect to master-1 vm
```
nmcli con add type ethernet con-name enp1s0 ifname enp1s0 \
  connection.autoconnect yes ipv4.method manual \
  ipv4.address 192.168.100.254/24 ipv4.gateway 192.168.100.1 \
  ipv4.dns 8.8.8.8

ping -c 2 8.8.8.8
ping -c 2 google.com

sudo dnf -y upgrade
sudo dnf -y install git vim wget curl bash-completion tree tar libselinux-python3 firewalld ansible
sudo reboot
```

```
sudo virt-builder fedora-39  --format qcow2 \
  --size 500G -o /var/lib/libvirt/images/master-2.qcow2 \
  --root-password password:Root@123

sudo virt-install \
  --name master-2 \
  --ram 131072 \
  --vcpus 8 \
  --disk path=/var/lib/libvirt/images/master-2.qcow2 \
  --os-variant rhel8.0 \
  --network bridge=openshift4 \
  --graphics none \
  --serial pty \
  --console pty \
  --boot hd \
  --import
```

Connect to master-2 vm
```
nmcli con add type ethernet con-name enp1s0 ifname enp1s0 \
  connection.autoconnect yes ipv4.method manual \
  ipv4.address 192.168.100.253/24 ipv4.gateway 192.168.100.1 \
  ipv4.dns 8.8.8.8

ping -c 2 8.8.8.8
ping -c 2 google.com

sudo dnf -y upgrade
sudo dnf -y install git vim wget curl bash-completion tree tar libselinux-python3 firewalld ansible
sudo reboot
```

Master-3 VM
```
sudo virt-builder fedora-39  --format qcow2 \
  --size 500G -o /var/lib/libvirt/images/master-3.qcow2 \
  --root-password password:Root@123

sudo virt-install \
  --name master-3 \
  --ram 131072 \
  --vcpus 8 \
  --disk path=/var/lib/libvirt/images/master-3.qcow2 \
  --os-variant rhel8.0 \
  --network bridge=openshift4 \
  --graphics none \
  --serial pty \
  --console pty \
  --boot hd \
  --import
```

Connect to master-3 vm
```
nmcli con add type ethernet con-name enp1s0 ifname enp1s0 \
  connection.autoconnect yes ipv4.method manual \
  ipv4.address 192.168.100.252/24 ipv4.gateway 192.168.100.1 \
  ipv4.dns 8.8.8.8

ping -c 2 8.8.8.8
ping -c 2 google.com

sudo dnf -y upgrade
sudo dnf -y install git vim wget curl bash-completion tree tar libselinux-python3 firewalld ansible
sudo reboot
```

Worker-1 VM
```
sudo virt-builder fedora-39  --format qcow2 \
  --size 500G -o /var/lib/libvirt/images/worker-1.qcow2 \
  --root-password password:Root@123

sudo virt-install \
  --name worker-1 \
  --ram 131072 \
  --vcpus 8 \
  --disk path=/var/lib/libvirt/images/worker-1.qcow2 \
  --os-variant rhel8.0 \
  --network bridge=openshift4 \
  --graphics none \
  --serial pty \
  --console pty \
  --boot hd \
  --import
```

Connect to worker-1 vm
```
nmcli con add type ethernet con-name enp1s0 ifname enp1s0 \
  connection.autoconnect yes ipv4.method manual \
  ipv4.address 192.168.100.251/24 ipv4.gateway 192.168.100.1 \
  ipv4.dns 8.8.8.8

ping -c 2 8.8.8.8
ping -c 2 google.com

sudo dnf -y upgrade
sudo dnf -y install git vim wget curl bash-completion tree tar libselinux-python3 firewalld ansible
sudo reboot
```

Worker-2 VM
```
sudo virt-builder fedora-39  --format qcow2 \
  --size 500G -o /var/lib/libvirt/images/worker-2.qcow2 \
  --root-password password:Root@123

sudo virt-install \
  --name worker-2 \
  --ram 131072 \
  --vcpus 8 \
  --disk path=/var/lib/libvirt/images/worker-2.qcow2 \
  --os-variant rhel8.0 \
  --network bridge=openshift4 \
  --graphics none \
  --serial pty \
  --console pty \
  --boot hd \
  --import
```

Connect to worker-2 vm
```
nmcli con add type ethernet con-name enp1s0 ifname enp1s0 \
  connection.autoconnect yes ipv4.method manual \
  ipv4.address 192.168.100.250/24 ipv4.gateway 192.168.100.1 \
  ipv4.dns 8.8.8.8

ping -c 2 8.8.8.8
ping -c 2 google.com

sudo dnf -y upgrade
sudo dnf -y install git vim wget curl bash-completion tree tar libselinux-python3 firewalld ansible
sudo reboot
```

Worker-3 VM
```
sudo virt-builder fedora-39  --format qcow2 \
  --size 500G -o /var/lib/libvirt/images/worker-3.qcow2 \
  --root-password password:Root@123

sudo virt-install \
  --name worker-3 \
  --ram 131072 \
  --vcpus 8 \
  --disk path=/var/lib/libvirt/images/worker-3.qcow2 \
  --os-variant rhel8.0 \
  --network bridge=openshift4 \
  --graphics none \
  --serial pty \
  --console pty \
  --boot hd \
  --import
```

Connect to worker-3 vm
```
nmcli con add type ethernet con-name enp1s0 ifname enp1s0 \
  connection.autoconnect yes ipv4.method manual \
  ipv4.address 192.168.100.249/24 ipv4.gateway 192.168.100.1 \
  ipv4.dns 8.8.8.8

ping -c 2 8.8.8.8
ping -c 2 google.com

sudo dnf -y upgrade
sudo dnf -y install git vim wget curl bash-completion tree tar libselinux-python3 firewalld ansible
sudo reboot
```


## Clone Kubespray
```
git clone https://github.com/kubernetes-sigs/kubespray.git
```
