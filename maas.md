# MAAS Installation

### Linux Installation
- Download ubuntu Server : http://releases.ubuntu.com/14.04/
- Run VMWare & create a new ubuntu server using the downloaded image
  - config netwrork to use bridge mode

### MAAS Package Repositories
```
sudo apt install software-properties-common -y
sudo add-apt-repository ppa:maas/stable
sudo apt-get update
```

### Installing MAAS from the command line 
- Installing in a Single Node MAAS
```
sudo apt-get install maas -y
sudo apt-get install maas-dhcp maas-dns -y
```
### Installing Pyvmomi from source
```
sudo apt-get install git
sudo apt install python3-pip
git clone https://github.com/vmware/pyvmomi; cd pyvmomi
sudo python3 setup.py install

# sudo apt-get install python-pip -y
# sudo pip3 install --upgrade pyvmomi -y
```

### Post-Install tasks
- Create a superuser account
```
sudo maas createadmin --username=root --email=MYEMAIL@EXAMPLE.COM
```
- Log in on the server: http://localhost/MAAS
- Import the boot images  
- Create ssh
```
sudo mkdir /home/maas
sudo chown maas:maas /home/maas

sudo chsh -s /bin/bash maas
sudo su - maas
ssh-keygen -f ~/.ssh/id_rsa -N ''
copy public key and paste it MAAS > Settings > Add SSH key
```

### Create node

### Import node
- Create chesis
```
maas login maas http://http://192.168.10.18/MAAS Mhmm68ZJfQsZvGKzcQ:wHF5Bp5vSMDmvU5dWp:hWcn7UbkbMbdw4CYe4r5fNQMkZkUbF4S
maas maas machines add-chassis chassis_type=vmware username=username  password=password protocol='https+unverified' hostname=192.168.10.11
```

### Trouble shooting
- rack controller 오류 발생 시 재 등록
```
sudo dpkg-reconfigure maas-rack-controller
```
- cdrom error during apt-get update
```
sudo vi /etc/apt/sources.list
```
- connection error from vm to host
  - config: Bridge & Host only (ens33, ens38)
  - added NAT to solve the problem (ens39)
  
```
route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         192.168.10.1    0.0.0.0         UG    0      0        0 ens33
10.10.10.0      *               255.255.255.0   U     0      0        0 ens38
192.168.10.0    *               255.255.255.0   U     0      0        0 ens33
192.168.253.0   *               255.255.255.0   U     0      0        0 ens39
```
  - use NAT gw as default gw
```
sudo route add default gw 192.168.253.2 
route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         192.168.253.2   0.0.0.0         UG    0      0        0 ens39
default         192.168.10.1    0.0.0.0         UG    0      0        0 ens33
10.10.10.0      *               255.255.255.0   U     0      0        0 ens38
192.168.10.0    *               255.255.255.0   U     0      0        0 ens33
192.168.253.0   *               255.255.255.0   U     0      0        0 ens39

sudo route del default gw 192.168.10.1
route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         192.168.253.2   0.0.0.0         UG    0      0        0 ens39
10.10.10.0      *               255.255.255.0   U     0      0        0 ens38
192.168.10.0    *               255.255.255.0   U     0      0        0 ens33
192.168.253.0   *               255.255.255.0   U     0      0        0 ens39

```
  - use NAT gw as host net
```
sudo route add -net 192.168.10.0 netmask 255.255.255.0 gw 192.168.253.2 dev ens39
route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         192.168.253.2   0.0.0.0         UG    0      0        0 ens39
10.10.10.0      *               255.255.255.0   U     0      0        0 ens38
192.168.10.0    192.168.253.2   255.255.255.0   UG    0      0        0 ens39
192.168.10.0    *               255.255.255.0   U     0      0        0 ens33
192.168.253.0   *               255.255.255.0   U     0      0        0 ens39
```
  - 호스트에서 Guest 접속을 위해 bridge ip 대신 nat ip를 사용하여 접속
  - 요약 
```
sudo route add default gw 192.168.42.2     # NAT
sudo route del default gw 192.168.88.1     # Bridge
sudo route add -net 192.168.88.0/24 gateway 192.168.42.2 dev ens33  # Bridge -> NAT dev
```

### VMWare Workstation/Fustion 
- [How to configure MAAS to be able to boot virtual machines via VMWare type](http://askubuntu.com/questions/663771/how-to-configure-maas-to-be-able-to-boot-virtual-machines-via-vmware-type)
- [Creating a Shared Virtual Machine in Fusion](https://pubs.vmware.com/fusion-8/index.jsp?topic=%2Fcom.vmware.fusion.using.doc%2FGUID-30FCA4B3-D9FD-40AF-8817-F0902AE6D758.html)
  - For shared vm, save in "/Users/Shared" folder instead of default "/Documents/Virtual Machines"

### Reference
- Get OpenStack Autopilot : http://www.ubuntu.com/download/cloud
- VMWare network config: http://blog.daum.net/_blog/BlogTypeView.do?blogid=0gub1&articleno=14&_bloghome_menu=recenttext
- Ubunut vmware-tools : https://help.ubuntu.com/community/VMware/Tools
- https://maas.ubuntu.com/docs2.0/install.html
- http://certification-static.canonical.com/docs/MAAS_Advanced_NUC_Installation_And_Configuration.pdf
- https://oxynets.wordpress.com/2015/09/21/ubuntu-server-how-to-manage-the-virtual-machines-created-via-vmware-workstation-on-maas-with-ubuntu-14-04-3lts-server-edition/
