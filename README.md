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

## Let's create a VM using KVM
```
sudo virt-builder ubuntu-20.04  --format qcow2 \
  --size 500G -o /var/lib/libvirt/images/master-1.qcow2 \
  --root-password password:rps@123

sudo virt-install \
  --name ocp-bastion-server \
  --ram 131072 \
  --vcpus 8 \
  --disk path=/var/lib/libvirt/images/master-1.qcow2 \
  --os-variant ubuntu20.04 \
  --network bridge=default \
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
