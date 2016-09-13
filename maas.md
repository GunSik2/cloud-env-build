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
- Log in on the server
- Import the boot images  
