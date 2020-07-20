# Virt-install to launch server based VM
## Install Ubuntu
Create an VM first
```
virt-install \
--name ubuntu1804 \
--ram 4096 \
--disk path=/var/lib/libvirt/images/ubuntu1804.img,size=30 \
--vcpus 2 \
--os-type linux \
--os-variant ubuntu18.04 \
--network bridge=br0 \
--graphics none \
--console pty,target_type=serial \
--location 'http://jp.archive.ubuntu.com/ubuntu/dists/bionic/main/installer-amd64/' \
--extra-args 'console=ttyS0,115200n8 serial'
```
Then it will take some time to install this machine

After that, shutdown the VM
```
virsh shutdown ubuntu1804

# mount guest's disk and enable a service like follows
guestmount -d ubuntu1804 -i /mnt
ln -s /mnt/lib/systemd/system/getty@.service /mnt/etc/systemd/system/getty.target.wants/getty@ttyS0.service
umount /mnt
# start guest again, if it's possible to connect to the guest's console, it's OK all
virsh start ubuntu1804 --console
```
After that you  will see:

```
Ubuntu 18.04 LTS ubuntu ttyS0

ubuntu login:
```

If you do not do this step before, after installation, you will not see any console output after the vm reboot.

## Network
First, check the default network is started and autostarted as the [link](https://blog.programster.org/kvm-missing-default-network)
If using virt-install and do not do anything, you may triagger the default network set, which is virbr0 network with
NAT rules set.
In yaml, you will see:
```
    <interface type='network'>
      <mac address='52:54:00:0b:a6:f3'/>
      <source network='default'/>
      <model type='e1000'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x0'/>
    </interface>
```

If you want to bridge to specify network card and join the network on your host, pls try to edit domain and add xml
as below:
```
    <interface type='direct'>
      <mac address='52:54:00:27:e4:f1'/>
      <source dev='eno1' mode='bridge'/>
      <model type='e1000'/>
    </interface>
```
While the model type e1000 is for x86, in Arm64 you need to change to another type.


