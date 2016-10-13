# Deploy a charm with Juju to MaaS

## Consideration
- Use different MaaS API key for each Juju environment.

## Env
- Juju 1.25 (Stable version is is 1.25.6)
- 16.04 LTS (Xenial)


## MaaS UI config. for Juju
- Generate MaaS key 
  - MaaS UI > Account > Generate MaaS Key 
- Add SSH key
  - MaaS UI > Account > Add SSH Key 
  - copy ~/.ssh/id_rsa.pub

## Run Juju in local environment
- Go to bootstrap01
```
ssh ubuntu@bootstrap01.maas
```
- Install
```
sudo apt-add-repository -y ppa:juju/devel
sudo apt update
sudo apt install juju zfsutils-linux -y
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

- Create a local controller 
```
juju status
juju bootstrap ubuntu-localhost localhost
juju list-controllers 
juju whoami
```

- Deploy a charm
```
juju deploy wiki-simple
juju status
```

## Run Juju in cloud environment (MaaS)
- Create a yaml
```
vi ~/.juju/maas-clouds.yaml
clouds:
   maas:
      type: maas
      auth-types: [oauth1]
      endpoint: http://172.18.42.10/MAAS
```
- Create juju cloud controller
```
juju add-cloud maas ~/.juju/maas-clouds.yaml
juju list-clouds
juju add-credential maas
> maas-oauth: <MaaS API Key>

juju bootstrap maas-controller maas
juju list-controllers 
```

- Deploy a charm
```
juju deploy wiki-simple
juju status
```


## Reference
- Getting Started with Juju 1.25: https://jujucharms.com/docs/1.25/getting-started
- Juju version: https://jujucharms.com/docs/2.0/reference-releases
- Using a MAAS cloud: https://jujucharms.com/docs/2.0/clouds-maas
- Install Openstack: https://help.ubuntu.com/lts/clouddocs/en/Installing-OpenStack.html
- Juju Quick Start: https://maas.ubuntu.com/docs/juju-quick-start.html

