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
virt-install --name master-1 --memory 131072 --vcpus 8 \
--disk size=500 --location noble-server-clouding-amd64.img --os-variant ubuntu24.04

virt-install \
--name master-1 \
--ram 131072 \
--disk path=/var/lib/libvirt/images/noble-server-clouding-amd64.img,size=500 \
--vcpus 8 \
--os-variant ubuntu24.04 \
--network bridge=br0 \
--graphics none \
--console pty,target_type=serial \
--location /root/Downloads/ubuntu-24.04-live-server-amd64.iso,kernel=casper/vmlinuz,initrd=casper/initrd \
--extra-args 'console=ttyS0,115200n8'
```
