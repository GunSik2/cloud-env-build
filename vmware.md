# VMWare 구성하기

## PC 환경 설정
- Enalbe Intel VT-x 
  - Advanced -> CPU Configuration -> Intel Virtualization Technology in the BIOS

- Windows Terminal 환경 설치
  - (옵션1) Windows 10 ubuntu 환경 구성 
    - Windows Aniversery 설치 : https://blogs.windows.com/windowsexperience/2016/08/02/how-to-get-the-windows-10-anniversary-update/#7ZCmt746OqrZKDYx.97
    - Update & Security > For Developers. Activate the “Developer Mode” 
    - Control Panel > “Programs” >  “Turn Windows Features On or Off” > Windows Subsystem for Linux (Beta) : enable

  - (옵션2) cygwin 환경 구성
    - cygwin 설치 : https://cygwin.com/install.html

- 윈도우 원격 접속 설정
  - "제어판\시스템 및 보안" > 시스템 하위 원격 액세스 허용 클릭
  - 원격 데스크톱 옵션 중 "이 컴퓨터에 대한 원격 연결 허용(L) 선택 및 "네트워크 수준 인증.. 허용" 옵션 Disable

## VMWare 환경 구성
- VMWare workstation 설치 : http://www.vmware.com/products/workstation/workstation-evaluation.html
- 네트워크 구성
  - VMNET0 - Bridge mode, 192.168.1.0/24
  - VMNET2 – Host-only mode, 10.10.1.0/24
  - VMNET3 – Host-only mode, 10.10.2.0/24
- VM 생성
  - Deploy node : HDD 20G, Ram 2G, Cpu 1x2 (Virtualization support), NIC eth0 vmnet2, eth1 bridge
  - Controller node : 


## Experiment
- MAAS: nat mode for portability
- Add port forwarding
  - VMWare > Virtaul Network Editor 
  - VMnet8 > NAT Settings > Add
    - 12822 : 192.168.42.128:22
    - 12880 : 192.168.42.128:80
  - Apply  
- Add required repository
```
mount /dev/cdrom /media/cdrom
sudo apt-get install python-software-properties
sudo add-apt-repository ppa:juju/stable
sudo add-apt-repository ppa:maas/stable
sudo add-apt-repository ppa:cloud-installer/stable
sudo apt-get update
```
- Install MAAS
```
sudo apt-get install maas
sudo maas-region-admin createadmin
```
- Login to the MAAS UI at http://<maas.ip>/MAAS/
