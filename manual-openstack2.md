## 5. Compute Service

### Overview
- Use OpenStack Compute to host and manage cloud computing systems.
- OpenStack Compute interacts with
  - OpenStack Identity for authentication;
  - OpenStack Image service for disk and server images; and
  - OpenStack dashboard for the user and administrative interface.
- OpenStack Compute consists of the following areas and their components:
  - Controller node
    - nova-api service : Accepts and responds to end user compute API calls.
    - nova-conductor module : Mediates interactions between the nova-compute service and the database.
    - nova-consoleauth daemon : Authorizes tokens for users that console proxies (nova-novncproxy) provide.
    - nova-novncproxy daemon : Provides a proxy for accessing running instances through a VNC connection. Supports browser-based novnc clients.
    - nova-scheduler service : Takes a virtual machine instance request from the queue and determines on which compute server host it runs.
    - The queue
    - SQL database
  - Compute nodes
    - nova-compute service : A worker daemon that creates and terminates virtual machine instances through hypervisor APIs.
  - Etc
    - nova-api-metadata service: Used when you run in multi-host mode with nova-network installations.
    - nova-network worker daemon : Similar to the nova-compute service
    - nova-cert module : A server daemon that serves the Nova Cert service for X509 certificates.
    - nova-spicehtml5proxy daemon : Provides a proxy for accessing running instances through a SPICE connection. Supports browser-based HTML5 client.
    - nova-xvpvncproxy daemon : Provides a proxy for accessing running instances through a VNC connection. Supports an OpenStack-specific Java client.
    - nova-cert daemon : x509 certificates.
    - nova client : Enables users to submit commands as a tenant administrator or end user.
    
### Install and configure controller node
- Prerequisites : NOVA_DBPASS, NOVA_PASS
```
$ echo $NOVA_DBPASS

$ mysql -u root -p
CREATE DATABASE nova_api;
CREATE DATABASE nova;
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY 'NOVA_DBPASS';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'NOVA_DBPASS';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'NOVA_DBPASS';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'NOVA_DBPASS';

$ echo $NOVA_PASS
$ . admin-openrc  
$ openstack user create --domain default --password $NOVA_PASS nova
$ openstack role add --project service --user nova admin
$ openstack service create --name nova --description "OpenStack Compute" compute
$ openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1/%\(tenant_id\)s
$ openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1/%\(tenant_id\)s  
$ openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1/%\(tenant_id\)s  
```
- Install and configure components: NOVA_DBPASS, RABBIT_PASS, NOVA_PASS, MANAGEMENT_INTERFACE_IP_ADDRESS
```
# apt-get install nova-api nova-conductor nova-consoleauth nova-novncproxy nova-scheduler -y
# echo $NOVA_DBPASS $RABBIT_PASS $NOVA_PASS
# sudo vi /etc/nova/nova.conf
[DEFAULT]
enabled_apis = osapi_compute,metadata
rpc_backend = rabbit
auth_strategy = keystone
my_ip = MANAGEMENT_INTERFACE_IP_ADDRESS 
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[api_database]
connection = mysql+pymysql://nova:NOVA_DBPASS@controller/nova_api

[database]
connection = mysql+pymysql://nova:NOVA_DBPASS@controller/nova

[oslo_messaging_rabbit]
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = RABBIT_PASS

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = NOVA_PASS

[vnc]
vncserver_listen = $my_ip
vncserver_proxyclient_address = $my_ip

[glance]
api_servers = http://controller:9292

[oslo_concurrency]
lock_path = /var/lib/nova/tmp

# su -s /bin/sh -c "nova-manage api_db sync" nova
# su -s /bin/sh -c "nova-manage db sync" nova
```

- Finalize installation
```
# service nova-api restart
# service nova-consoleauth restart
# service nova-scheduler restart
# service nova-conductor restart
# service nova-novncproxy restart
# . admin-openrc
# openstack compute service list
+----+------------------+------------+----------+---------+-------+----------------------------+
| Id | Binary           | Host       | Zone     | Status  | State | Updated At                 |
+----+------------------+------------+----------+---------+-------+----------------------------+
|  4 | nova-consoleauth | controller | internal | enabled | up    | 2016-11-07T08:23:25.000000 |
|  5 | nova-scheduler   | controller | internal | enabled | up    | 2016-11-07T08:23:24.000000 |
|  6 | nova-conductor   | controller | internal | enabled | up    | 2016-11-07T08:23:26.000000 |
+----+------------------+------------+----------+---------+-------+----------------------------+

```

### Install and configure a compute node
- Install and configure components :  MANAGEMENT_INTERFACE_IP_ADDRESS, RABBIT_PASS, NOVA_PASS
```
# apt-get install nova-compute -y

# sudo vi /etc/nova/nova.conf 
[DEFAULT]
rpc_backend = rabbit
auth_strategy = keystone
my_ip = MANAGEMENT_INTERFACE_IP_ADDRESS
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[oslo_messaging_rabbit]
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = RABBIT_PASS

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = NOVA_PASS

[vnc]
enabled = True
vncserver_listen = 0.0.0.0
vncserver_proxyclient_address = $my_ip
novncproxy_base_url = http://controller:6080/vnc_auto.html

[glance]
api_servers = http://controller:9292

[oslo_concurrency]
lock_path = /var/lib/nova/tmp
```
- Finalize installation
```
$ egrep -c '(vmx|svm)' /proc/cpuinfo
## If this command returns a value of one or greater, your compute node supports hardware acceleration which typically requires no additional configuration.
## If this command returns a value of zero, your compute node does not support hardware acceleration and you must configure libvirt to use QEMU instead of KVM.  

# service nova-compute restart
```
### Verify operation
```
$ . admin-openrc
$ openstack compute service list
+----+------------------+------------+----------+---------+-------+----------------------------+
| Id | Binary           | Host       | Zone     | Status  | State | Updated At                 |
+----+------------------+------------+----------+---------+-------+----------------------------+
|  4 | nova-consoleauth | controller | internal | enabled | up    | 2016-11-07T08:38:15.000000 |
|  5 | nova-scheduler   | controller | internal | enabled | up    | 2016-11-07T08:38:15.000000 |
|  6 | nova-conductor   | controller | internal | enabled | up    | 2016-11-07T08:38:16.000000 |
|  7 | nova-compute     | compute1   | nova     | enabled | up    | 2016-11-07T08:38:14.000000 |
+----+------------------+------------+----------+---------+-------+----------------------------+
```

## 6. Networking service

### Overview
- Components
  - neutron-server : Accepts and routes API requests to the appropriate OpenStack Networking plug-in for action
  - plug-ins and agents : Plugs and unplugs ports, creates networks or subnets, and provides IP addressing.
  - Messing queue 

### Install and configure controller node
- Prerequisites : NEUTRON_DBPASS, NEUTRON_PASS
```
$ echo $NEUTRON_DBPASS, $NEUTRON_PASS
$ mysql -u root -p
CREATE DATABASE neutron;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'NEUTRON_DBPASS';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'NEUTRON_DBPASS';

$ . admin-openrc
$ openstack user create --domain default --password $NEUTRON_PASS neutron
$ openstack role add --project service --user neutron admin
$ openstack service create --name neutron --description "OpenStack Networking" network
$ openstack endpoint create --region RegionOne network public http://controller:9696
$ openstack endpoint create --region RegionOne network internal http://controller:9696  
$ openstack endpoint create --region RegionOne network admin http://controller:9696  
```
- Configure networking options
  - Apply Networking Option 2: Self-service networks
  - Install the components
```
# apt-get install neutron-server neutron-plugin-ml2 \
  neutron-linuxbridge-agent neutron-l3-agent neutron-dhcp-agent \
  neutron-metadata-agent -y
```
  - Configure the server component: NEUTRON_DBPASS, RABBIT_PASS, NEUTRON_PASS, NOVA_PASS
```
# echo $NEUTRON_DBPASS, $RABBIT_PASS, $NEUTRON_PASS, $NOVA_PASS
# vi /etc/neutron/neutron.conf
[DEFAULT]
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = True
rpc_backend = rabbit
auth_strategy = keystone
notify_nova_on_port_status_changes = True
notify_nova_on_port_data_changes = True

[database]
connection = mysql+pymysql://neutron:NEUTRON_DBPASS@controller/neutron

[oslo_messaging_rabbit]
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = RABBIT_PASS

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = NEUTRON_PASS

[nova]
auth_url = http://controller:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = NOVA_PASS
```
  - Configure the Modular Layer 2 (ML2) plug-in
```
# vi /etc/neutron/plugins/ml2/ml2_conf.ini
[ml2]
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan
mechanism_drivers = linuxbridge,l2population
extension_drivers = port_security

[ml2_type_flat]
flat_networks = provider

[ml2_type_vxlan]
vni_ranges = 1:1000

[securitygroup]
enable_ipset = True
```
  - Configure the Linux bridge agent: PROVIDER_INTERFACE_NAME(public interface), OVERLAY_INTERFACE_IP_ADDRESS(management IP address)
```
# vi /etc/neutron/plugins/ml2/linuxbridge_agent.ini
[linux_bridge]
physical_interface_mappings = provider:PROVIDER_INTERFACE_NAME

[vxlan]
enable_vxlan = True
local_ip = OVERLAY_INTERFACE_IP_ADDRESS
l2_population = True

[securitygroup]
enable_security_group = True
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```
  - Configure the layer-3 agent
```
# vi /etc/neutron/l3_agent.ini
[DEFAULT]
interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
external_network_bridge =
```
  - Configure the DHCP agent
```
# vi /etc/neutron/dhcp_agent.ini
[DEFAULT]
interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = True
```


- Configure the metadata agent : METADATA_SECRET
```
# echo $METADATA_SECRET

# vi /etc/neutron/metadata_agent.ini
[DEFAULT]
nova_metadata_ip = controller
metadata_proxy_shared_secret = METADATA_SECRET
```
- Configure Compute to use Networking : NEUTRON_PASS, METADATA_SECRET
```
# echo $NEUTRON_PASS, $METADATA_SECRET

# vi /etc/nova/nova.conf
[neutron]
url = http://controller:9696
auth_url = http://controller:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = NEUTRON_PASS

service_metadata_proxy = True
metadata_proxy_shared_secret = METADATA_SECRET
```
- Finalize installation
```
# su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
  --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
# service nova-api restart  
# service neutron-server restart
# service neutron-linuxbridge-agent restart
# service neutron-dhcp-agent restart
# service neutron-metadata-agent restart
```

### Install and configure compute node
- Install the components
```
# apt-get install neutron-linuxbridge-agent -y
```
- Configure the common component : RABBIT_PASS, NEUTRON_PASS
```
# echo $RABBIT_PASS, $NEUTRON_PASS
# vi /etc/neutron/neutron.conf

[DEFAULT]
rpc_backend = rabbit
auth_strategy = keystone

[oslo_messaging_rabbit]
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = RABBIT_PASS

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = NEUTRON_PASS
```
- Configure networking options 
  - Apply Networking Option 2: Self-service networks
  - Configure the Linux bridge agent : PROVIDER_INTERFACE_NAME (eth1), OVERLAY_INTERFACE_IP_ADDRESS (10.0.0.31)
```
# vi /etc/neutron/plugins/ml2/linuxbridge_agent.ini
[linux_bridge]
physical_interface_mappings = provider:PROVIDER_INTERFACE_NAME

[vxlan]
enable_vxlan = True
local_ip = OVERLAY_INTERFACE_IP_ADDRESS
l2_population = True

[securitygroup]
enable_security_group = True
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```
- Configure Compute to use Networking : NEUTRON_PASS
```
# echo $NEUTRON_PASS

# vi /etc/nova/nova.conf
[neutron]
url = http://controller:9696
auth_url = http://controller:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = NEUTRON_PASS

```
- Finalize installation
```
# service nova-compute restart
# service neutron-linuxbridge-agent restart
```

### Verify operation on Controller 
```
$ . admin-openrc
$ neutron ext-list
+---------------------------+-----------------------------------------------+
| alias                     | name                                          |
+---------------------------+-----------------------------------------------+
| default-subnetpools       | Default Subnetpools                           |
| network-ip-availability   | Network IP Availability                       |
| network_availability_zone | Network Availability Zone                     |
| auto-allocated-topology   | Auto Allocated Topology Services              |
| ext-gw-mode               | Neutron L3 Configurable external gateway mode |
| binding                   | Port Binding                                  |
| agent                     | agent                                         |
| subnet_allocation         | Subnet Allocation                             |
| l3_agent_scheduler        | L3 Agent Scheduler                            |
| tag                       | Tag support                                   |
| external-net              | Neutron external network                      |
| net-mtu                   | Network MTU                                   |
| availability_zone         | Availability Zone                             |
| quotas                    | Quota management support                      |
| l3-ha                     | HA Router extension                           |
| provider                  | Provider Network                              |
| multi-provider            | Multi Provider Network                        |
| address-scope             | Address scope                                 |
| extraroute                | Neutron Extra Route                           |
| timestamp_core            | Time Stamp Fields addition for core resources |
| router                    | Neutron L3 Router                             |
| extra_dhcp_opt            | Neutron Extra DHCP opts                       |
| dns-integration           | DNS Integration                               |
| security-group            | security-group                                |
| dhcp_agent_scheduler      | DHCP Agent Scheduler                          |
| router_availability_zone  | Router Availability Zone                      |
| rbac-policies             | RBAC Policies                                 |
| standard-attr-description | standard-attr-description                     |
| port-security             | Port Security                                 |
| allowed-address-pairs     | Allowed Address Pairs                         |
| dvr                       | Distributed Virtual Router                    |
+---------------------------+-----------------------------------------------+

$ neutron agent-list
+--------------------------------------+--------------------+------------+-------------------+-------+----------------+---------------------------+
| id                                   | agent_type         | host       | availability_zone | alive | admin_state_up | binary                    |
+--------------------------------------+--------------------+------------+-------------------+-------+----------------+---------------------------+
| 392ccb72-9b24-4bab-b4f5-92ee84914705 | Metadata agent     | controller |                   | :-)   | True           | neutron-metadata-agent    |
| 6c075cba-865d-4826-a935-294c0283587c | DHCP agent         | controller | nova              | :-)   | True           | neutron-dhcp-agent        |
| 7025f635-e75d-45a1-b600-b05dab356216 | Linux bridge agent | compute1   |                   | :-)   | True           | neutron-linuxbridge-agent |
| a3853133-3b07-4257-80e7-5485278bf2af | L3 agent           | controller | nova              | :-)   | True           | neutron-l3-agent          |
| e05cad72-2b37-44dc-a683-5213ded611d4 | Linux bridge agent | controller |                   | :-)   | True           | neutron-linuxbridge-agent |
+--------------------------------------+--------------------+------------+-------------------+-------+----------------+---------------------------+
```

### Launch an instance

- Create virtual networks on Controller node
  - Create provider network : START_IP_ADDRESS, END_IP_ADDRESS, DNS_RESOLVER, PROVIDER_NETWORK_GATEWAY, PROVIDER_NETWORK_CIDR 
```
$ . admin-openrc
$ neutron net-create --shared --provider:physical_network provider \
  --provider:network_type flat provider
$ neutron subnet-create --name provider \
  --allocation-pool start=START_IP_ADDRESS,end=END_IP_ADDRESS \
  --dns-nameserver DNS_RESOLVER --gateway PROVIDER_NETWORK_GATEWAY \
  provider PROVIDER_NETWORK_CIDR

(case1) public ip
$ neutron subnet-create --name provider \
  --allocation-pool start=203.0.113.101,end=203.0.113.250 \
  --dns-nameserver 8.8.4.4 --gateway 203.0.113.1 \
  provider 203.0.113.0/24

(case2) private ip
$ neutron subnet-create --name provider \
  --allocation-pool start=192.168.10.101,end=192.168.10.250 \
  --dns-nameserver 8.8.8.8 --gateway 192.168.10.1 \
  provider 192.168.10.0/24    
```
  - Create Self-service network
    - Create the self-service network
```
$ . demo-openrc
$ neutron net-create selfservice
$ neutron subnet-create --name selfservice \
  --dns-nameserver DNS_RESOLVER --gateway SELFSERVICE_NETWORK_GATEWAY \
  selfservice SELFSERVICE_NETWORK_CIDR
  
(case1) 
$ neutron subnet-create --name selfservice \
  --dns-nameserver 8.8.8.8 --gateway 172.16.1.1 \
  selfservice 172.16.1.0/24
```
    - Create a router
```  
$ . admin-openrc
$ neutron net-update provider --router:external 

$ . demo-openrc
$ neutron router-create router
$ neutron router-interface-add router selfservice
$ neutron router-gateway-set router provider
```    
    - Verify operation
```
$ . admin-openrc
$ ip netns
$ neutron router-port-list router
$ ping -c 4 ROUTER_IP
```
- Create m1.nano flavor
- Generate a key pair
- Add security group rules
- Launch an instance
- Block Storage
- Orchestration
- Shared File Systems
