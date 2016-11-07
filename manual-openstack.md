# Manual installation

## Reference
- http://docs.openstack.org/mitaka/install-guide-ubuntu/environment-networking.html

## Environment
### Host networking
- Controller
  - eth0: 10.0.0.11  (management nw)
  - eth1: 192.168.10.24 (provider nw)
- Compute1 
  - eth0: 10.0.0.31  (management nw)  
  - eth1: 192.168.10.25 (provider nw)
- Controller Config
```
# vi /etc/network/interfaces
auto eth0
iface eth0 inet static
      address 10.0.0.11
      netmask 255.255.255.0
      broadcast 10.0.0.255
auto eth1
iface eth1 inet dhcp

# vi /etc/hosts
10.0.0.11       controller
10.0.0.31       compute1
```
- Test
  - Controller node
```
# ping -c 4 openstack.org
# ping -c 4 compute1
```
  - Compute node
```
# ping -c 4 openstack.org
# ping -c 4 controller
```
### NTP 
### OpenStack packages on All nodes
```
# apt-get install software-properties-common
# add-apt-repository cloud-archive:mitaka
# apt-get update && apt-get dist-upgrade
# apt-get install python-openstackclient
```
### Controller nodes
- SQL database
```
# apt-get install mariadb-server python-pymysql
# vi /etc/mysql/conf.d/openstack.cnf
[mysqld]
...
bind-address = 10.0.0.11
default-storage-engine = innodb
innodb_file_per_table
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8

# service mysql restart
# mysql_secure_installation
```
- Message queue : RABBIT_PASS
```
# apt-get install rabbitmq-server
# rabbitmqctl add_user openstack $RABBIT_PASS
# rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```
- Memcached
```
# apt-get install memcached python-memcache
# vi /etc/memcached.conf
-l 10.0.0.11  # 127.0.0.1 to controller
# service memcached restart
```

