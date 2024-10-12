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
sudo virsh net-autostart default
sudo virsh net-start default
sudo virsh net-list
```

## Let's create a VM using KVM
```
wget https://cloud-images.ubuntu.com/releases/24.04/release-20240423/ubuntu-24.04-server-cloudimg-amd64.img
```

Configure ubuntu password
```
cat >user-data.txt <<EOF
#cloud-config
password:  Root@123
chpasswd: { expire: False }
ssh_pwauth: True
EOF
```

```
qemu-img create -b ubuntu-24.04-server-cloudimg-amd64.img -F qcow2 -f qcow2 ubuntu-vm-disk.qcow2 500G

virt-install --name master-1 \
  --virt-type kvm --memory 131072 --vcpus 8 \
  --boot hd,menu=on \
  --disk path=master-1.qcow2,device=disk \
  --disk path=user-data.img,format=raw \
  --graphics none \
  --os-type Linux --os-variant ubuntu24.04
```
