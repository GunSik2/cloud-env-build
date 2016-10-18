# Deploy a charm with Juju to MaaS

## Consideration
- Use different MaaS API key for each Juju environment.

## Env
- Juju 2.0
- 16.04 LTS (Xenial)
- LXD 
- ZFS 

## MaaS UI config. for Juju
- Generate MaaS key 
  - MaaS UI > Account > Generate MaaS Key 
- Add SSH key
  - MaaS UI > Account > Add SSH Key 
  - copy ~/.ssh/id_rsa.pub

## Run Juju in LXD local environment
- Install
```
sudo apt-add-repository -y ppa:juju/devel
sudo apt update
sudo apt install juju lxd zfsutils-linux
newgrp lxd
sudo lxd init
: Size in GB of the new loop device (1GB minimum) [default=10GB]: 20
: Do you want to configure the LXD bridge (yes/no) [default=yes]? yes
: interface name: default (lxdbr0)
: IPv4 subnet: yes
: IPv4 setting: default (eg.10.255.0.1, 24, 10.255.0.2 (DHCP), 10.255.0.254 (Last DHCP), 250 (Max number))
: NAT the IPv4: yes
: Ipv6 subnet: no
```

- Create juju lxd controller 
```
juju status
juju bootstrap localhost localhost
juju status
juju gui
juju show-controller --show-password

juju list-controllers 
juju whoami
```

- Deploy a charm
```
juju deploy wiki-simple
juju status
```

- 
- Delete a local controller
```
- juju destroy-controller maas-localhost --destroy-all-models
```

## Run Juju in MaaS cloud environment
- Install
```
sudo apt-add-repository -y ppa:juju/devel
sudo apt update
sudo apt install juju 
newgrp lxd
```
- Create a yaml
```
vi ~/.juju/maas-clouds.yaml
clouds:
   maas:
      type: maas
      auth-types: [oauth1]
      endpoint: http://10.10.20.11/MAAS
```
- Create juju cloud controller
```
juju add-cloud maas ~/.juju/maas-clouds.yaml
juju list-clouds
juju add-credential maas
> Enter credential name: maas
> maas-oauth: <MaaS API Key>

juju bootstrap maas maas --to bootstrap01.maas  --debug
juju list-controllers 
```

## Add MaaS Machines & Deploy jujucharms

- Add MaaS machine
```
juju add-machine controller01.maas
juju add-machine neutron01.maas
juju add-machine compute01.maas
juju add-machine compute02.maas
```
- Deploy a charm
```
juju deploy juju-gui --to bootstrap01.maas:lxc:0
juju get juju-gui

juju deploy wiki-simple
juju status
```

## LXC commands
```
lxc launch ubuntu:

```
## Reference
- [Getting Started with Juju 2.0](https://jujucharms.com/docs/stable/getting-started)
- [Using a MAAS cloud](https://jujucharms.com/docs/2.0/clouds-maas)
- [Juju Quick Start](https://maas.ubuntu.com/docs/juju-quick-start.html)
- [LXC commands](https://insights.ubuntu.com/2016/03/22/lxd-2-0-your-first-lxd-container/)
