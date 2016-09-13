# MAAS

### MAAS Package Repositories
```
sudo add-apt-repository ppa:maas/stable
sudo apt-get update
```

### Installing MAAS from the command line 
- Installing in a Single Node MAAS
```
sudo apt-get install maas -y
sudo apt-get install maas-dhcp maas-dns -y
```

### Post-Install tasks
- Create a superuser account
```
sudo maas createadmin --username=root --email=MYEMAIL@EXAMPLE.COM
```
- Log in on the server: http://localhost/MAAS
- Import the boot images  


### VMWare Workstation/Fustion 
- [How to configure MAAS to be able to boot virtual machines via VMWare type](http://askubuntu.com/questions/663771/how-to-configure-maas-to-be-able-to-boot-virtual-machines-via-vmware-type)
- [Creating a Shared Virtual Machine in Fusion](https://pubs.vmware.com/fusion-8/index.jsp?topic=%2Fcom.vmware.fusion.using.doc%2FGUID-30FCA4B3-D9FD-40AF-8817-F0902AE6D758.html)
  - For shared vm, save in "/Users/Shared" folder instead of default "/Documents/Virtual Machines"
