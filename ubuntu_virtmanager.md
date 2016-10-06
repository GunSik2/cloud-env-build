# ubuntu openstack on VMWare guest

### 범위
- VMWare 에서 ubuntu Despktop 을 설치
- ubuntu Desktop 에 VirtManager 설치
- ubuntu Desktop 에 MaaS 설치

### Desktop 설치
```
sudo aptitude install --without-recommends ubuntu-desktop
```

### VirtManager 설치
- 가상화 지원 하드웨어 체크 
  - 0 이 아닌 경우 HW 가상화 지원 
```
egrep -c '(svm|vmx)' /proc/cpuinfo 
```

- Virtualizaion Tool 설치
```
sudo apt install qemu-kvm libvirt-bin bridge-utils virt-manager -y
sudo adduser $USER libvirtd
sudo su - $USER
```

- VM 목록
```
virsh -c qemu:///system list
```

- VM 생성
```
wget http://releases.ubuntu.com/14.04/ubuntu-14.04.5-server-amd64.iso
```
- GUI 실행
```
virt-manager -c qemu:///system
```
- Create Network 
  - Edit > Create Network > Virtual Networks tab  
  - Network : maas - NAT
- Create VM for MaaS Controller
  - File > New VirtualMachine
  - VM : maas, using maas network

### MaaS 설치
- Install MaaS on MaaS Controller
```
sudo apt install software-properties-common -y
sudo add-apt-repository ppa:maas/stable
sudo apt-get update

sudo apt-get install maas -y
sudo apt-get install maas-dhcp maas-dns -y

sudo maas-region-admin createadmin
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
```
  - copy public key and paste it on "MAAS > Settings > Add SSH key"
  
  

### Utility 
```
sudo apt install byobu
sudo apt install iptraf
sudo apt install nethogs

```

#### Reference
- 동영상: https://www.youtube.com/watch?v=ojTTgrtl-RU
- [Introduction: Deploying OpenStack on MAAS 1.9+ with Juju](http://blog.naydenov.net/2015/11/deploying-openstack-on-maas-1-9-with-juju-network-setup/)
