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

### MaaS 설치
- 참고 동영상: https://www.youtube.com/watch?v=ojTTgrtl-RU
