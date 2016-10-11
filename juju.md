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

## Run Juju in local environment
- Install
```
sudo apt-add-repository -y ppa:juju/devel
sudo apt update
sudo apt install juju zfsutils-linux
newgrp lxd
sudo lxd init
: Size in GB of the new loop device (1GB minimum) [default=10GB]: 20
: Do you want to configure the LXD bridge (yes/no) [default=yes]? yes
: interface name: lxdbr0
: IPv4 subnet: yes
: IPv4 setting: 10.255.0.1, 24, 10.255.0.2 (DHCP), 10.255.0.254 (Last DHCP), 250 (Max number)
: NAT the IPv4: yes
: Ipv6 subnet: no

```

- Create a controller 
```
juju status
juju bootstrap lxd-test localhost
juju list-controllers 
juju whoami
```

- Deploy a charm
```
juju deploy mysql
juju status
```

## Juju MaaS cloud environment
- Create a yaml
```
vi ~/.juju/maas-clouds.yaml
clouds:
   devmaas:
      type: maas
      auth-types: [oauth1]
      endpoint: http://172.18.42.10/MAAS
```
- Create juju cloud controller
```
juju add-cloud devmaas ~/.juju/maas-clouds.yaml
juju list-clouds
juju add-credential devmaas
> maas-oauth: <MaaS API Key>

juju bootstrap devmaas-controller devdmaas
```


## Reference
- [Getting Started with Juju 2.0](https://jujucharms.com/docs/stable/getting-started)
- [Using a MAAS cloud](https://jujucharms.com/docs/2.0/clouds-maas)
- [Juju Quick Start](https://maas.ubuntu.com/docs/juju-quick-start.html)
