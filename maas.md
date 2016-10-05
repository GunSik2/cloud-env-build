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
maas maas machines add-chassis chassis_type=vmware username=gun  password=choiGun7 protocol='https' hostname=192.168.10.11
```

### Trouble shooting
- cdrom error during apt-get update
```
sudo vi /etc/apt/sources.list
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
