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
  --size 500G -o /var/lib/libvirt/images/ocp-bastion-server.qcow2 \
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

## Install Vagrant
```
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
sudo yum -y install vagrant
```
