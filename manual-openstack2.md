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

