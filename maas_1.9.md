# MAAS Installation

### Linux Installation
- Download ubuntu Server : http://releases.ubuntu.com/14.04/
- Run VMWare & create a new ubuntu server using the downloaded image
  - config network to use Host-only, NAT
```

auto ens32  # Host-only
iface ens32 inet static
        address 10.10.20.11
        netmask 255.255.255.0
        network 10.10.20.0
        broadcast 10.10.20.255
        dns-nameserver 10.10.20.11
        dns-search maas

auto ens33   # NAT
iface ens33 inet dhcp
        up route add -net 192.168.10.0 netmask 255.255.255.0 gw 192.168.20.2
```

### Installing MAAS from the command line 
- Install SSH
```
sudo apt update
sudo apt install ssh
```

- Installing in a Single Node MAAS
```
sudo apt install software-properties-common -y
sudo apt-add-repository -y ppa:maas/stable 
sudo apt update

sudo apt-get install maas -y
sudo dpkg -s maas | grep Version
Version: 2.0.0+bzr5189-0ubuntu1~16.04.1
```
- Installing in a Distributed Node MAAS
  - One node : region controller
```
sudo apt install maas-region-controller
```
  - Second node : rack controller
```
sudo apt install maas-rack-controller
sudo maas-rack register
```

### Post-Install tasks
- Create a superuser account
```
sudo maas createadmin 
```
- Log in on the server: http://localhost/MAAS
- 
- Import the boot images : Images > Apply changes
- Create DHCP : Networks > Click VLAN for "10.10.20.0/24" > Provide DHCP > Remove Gateway IP > Provide DHCP  
- Create maas user's ssh key 
```
ssh-keygen -f ~/.ssh/id_rsa -N ''
cat ~/.ssh/id_rsa.pub
```
- Register public key to MaaS : MAAS > Settings > Add SSH key
- Set maas rack controller to Host-only ip (http://10.10.10.10/MAAS)
```
sudo dpkg-reconfigure maas-rack-controller
```

### MaaS Node Management in VMWare
- Create nodes
  - The first/default node should be the Host-only network
  - A node should have a NAT for accessing public network
  - Nodes: bootstrap01, controller01, neutron01, compute01, compute02 
  - Boot & shutdown all nodes
- Instal Pyvmomi from source for accessing VMWare
```
sudo apt install python3-pip -y
sudo apt-get install git -y
git clone https://github.com/vmware/pyvmomi; cd pyvmomi
sudo python3 setup.py install
```
- Add new nodes to MaaS 
```
maas login maas http://10.10.20.11/MAAS <maas-key>
maas maas machines add-chassis chassis_type=vmware username=username  password=password protocol='https+unverified' hostname=192.168.10.11
```
- Commission the imported nodes
  - UI > Nodes > Check nodes > Take action - Commission
- Change nodes configuration
  - UI > Nodes > Click a node (Ready State) > Set interfaces (IP Address as DHCP) 
- Deploy the commissioned nodes
  - UI > Nodes > Check nodes > Take action - Deploy
- Change DNS server
```
sudo vi /etc/resolv.conf
nameserver 10.10.20.11  # Host-only (MAAS-Net)
nameserver 192.168.20.2  # NAT DNS
```
- Access to the nodes
```
ssh -i bootstrap01.maas
```

### MaaS cli
```
maas maas nodes list
maas maas nodes accept-all

maas maas node release <system-id>
maas maas node delete <system-id>
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
- MaaS 1.9 : http://maas.ubuntu.com/docs1.9
