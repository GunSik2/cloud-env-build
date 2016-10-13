# Installing OpenStack with Juju

## Deploying OpenStack with Juju
- Configure Juju
```
vi ~/juju-config.yaml
keystone:
  admin-password: 'openstack'
nova-cloud-controller:
  network-manager: Neutron
neutron-gateway:
  ext-port: eth2
  bridge-mappings: 'external:br-ens33'
  os-data-network: 10.10.20.0/24
  instance-mtu: 1400
neutron-api:
  network-device-mtu: 1400
  # Always make sure you enable security groups
  neutron-security-groups: true
  overlay-network-type: vxlan
rabbitmq-server:
cinder-api:
  enabled-services: api,scheduler
cinder-volume:
  enabled-services: volume
  # Adjust this to match the block device on your volume host
  block-device: fd0
glance:
heat:
mysql:
openstack-dashboard:
  webroot: /
nova-compute:
  manage-neutron-plugin-legacy-mode: false
  virt-type: kvm
neutron-openvswitch:
  os-data-network: 10.10.20.0/24
```
- Initialising Juju
```
CONFIG=~/juju-config.yaml

juju deploy --config=$CONFIG mysql --to lxd:11 --debug
juju deploy --config=$CONFIG rabbitmq-server --to controller01:lxc:1
sleep 120s

juju deploy --config=$CONFIG keystone --to lxc:1
juju add-relation keystone:shared-db mysql:shared-db
```
- Examples
```
juju deploy mysql --to 23       (deploy to machine 23)
juju deploy mysql --to 24/lxd/3 (deploy to lxd container 3 on machine 24)
juju deploy mysql --to lxd:25   (deploy to a new lxd container on machine 25)
juju deploy mysql --to lxd      (deploy to a new lxd container on a new machine)
juju deploy mysql --to zone=us-east-1a    (provider-dependent; deploy to a specific AZ)
juju deploy mysql --to host.maas    (deploy to a specific MAAS node)
juju deploy mysql -n 5 --constraints mem=8G   (deploy 5 units to machines with at least 8 GB of memory)
juju deploy haproxy -n 2 --constraints spaces=dmz,^cms,^database    (deploy 2 units to machines that are part of the 'dmz' space but not of the 'cmd' or the 'database' spaces)
```


## Reference
- https://help.ubuntu.com/lts/clouddocs/en/Installing-OpenStack.html
- https://jujucharms.com/docs/2.0/charms-deploying
- https://jujucharms.com/u/openstack-charmers-next/openstack-base
- https://www.hastexo.com/resources/hints-and-kinks/ubuntu-openstack-juju-4-nodes/
- https://wiki.ubuntu.com/OpenStack/OpenStackCharms/ReleaseNotes1604
