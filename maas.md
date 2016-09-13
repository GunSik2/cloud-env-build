# MAAS

- MAAS Package Repositories
```
sudo add-apt-repository ppa:maas/stable
sudo apt-get update
```

- Installing MAAS from the command line 
  - Installing in a Single Node MAAS
```
sudo apt-get install maas -y
sudo apt-get install maas-dhcp maas-dns -y
```
  - Installing MAAS in a LXC container
    - Install LXD and ZFS
```
sudo apt-get install lxd zfsutils-linux
sudo modprobe zfs
sudo lxd init
```
    - Create a LXC profile for MAAS
```
lxc profile copy default maas
// bind the NIC inside the container (eth0) against the bridge on the physical host (br0)
lxc profile device set maas eth0 parent br0  
lxc profile edit maas
config:
  raw.lxc: |-
    lxc.cgroup.devices.allow = c 10:237 rwm
    lxc.aa_profile = unconfined
    lxc.cgroup.devices.allow = b 7:* rwm
  security.privileged: "true"
for i in `seq 0 7`; do lxc profile device add maas loop$i unix-block path=/dev/loop$i; done
```
    - Launch LXD container
```
lxc launch -p maas ubuntu:16.04 xenial-maas
```
    - Install MAAS
```
lxc exec xenial-maas bash
```
- Post-Install tasks
  - Create a superuser account
```
sudo maas createadmin --username=root --email=MYEMAIL@EXAMPLE.COM
```
  - Log in on the server
  - Import the boot images  
