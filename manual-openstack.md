# Manual installation

## Reference
- http://docs.openstack.org/mitaka/install-guide-ubuntu/environment-networking.html

## 1. Overview
### Node & Service Deployment
- Node1 - Controller : Horizon, Neutron, Keystone, Glance, Heat
- Node2 - Compute1 : Nova, Swift, Cinder
### Networking
- Self-service networks
  - provider network : public network
  - management network : private network

## 2. Environment
### Security
- Create password file & copy it to all nodes
```
echo "ADMIN_PASS=$(openssl rand -hex 10)
CEILOMETER_DBPASS=$(openssl rand -hex 10)
CEILOMETER_PASS=$(openssl rand -hex 10)
CINDER_DBPASS=$(openssl rand -hex 10)
CINDER_PASS=$(openssl rand -hex 10)
DASH_DBPASS=$(openssl rand -hex 10)
DEMO_PASS=$(openssl rand -hex 10)
GLANCE_DBPASS=$(openssl rand -hex 10)
GLANCE_PASS=$(openssl rand -hex 10)
HEAT_DBPASS=$(openssl rand -hex 10)
HEAT_DOMAIN_PASS=$(openssl rand -hex 10)
HEAT_PASS=$(openssl rand -hex 10)
KEYSTONE_DBPASS=$(openssl rand -hex 10)
NEUTRON_DBPASS=$(openssl rand -hex 10)
NEUTRON_PASS=$(openssl rand -hex 10)
NOVA_DBPASS=$(openssl rand -hex 10)
NOVA_PASS=$(openssl rand -hex 10)
RABBIT_PASS=$(openssl rand -hex 10)
SWIFT_PASS=$(openssl rand -hex 10)
ADMIN_TOKEN=$(opensll rand -hex 10)" > password.sh

. password.sh
```
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
# apt-get install software-properties-common -y
# add-apt-repository cloud-archive:mitaka
# apt-get update && apt-get dist-upgrade -y
# apt-get install python-openstackclient -y
```
### Controller nodes
- SQL database
```
# apt-get install mariadb-server python-pymysql -y
# vi /etc/mysql/conf.d/openstack.cnf
[mysqld]
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
# echo $RABBIT_PASS
# apt-get install rabbitmq-server -y
# rabbitmqctl add_user openstack $RABBIT_PASS
# rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```
- Memcached
```
# apt-get install memcached python-memcache -y
# vi /etc/memcached.conf
-l 10.0.0.11  # 127.0.0.1 to controller
# service memcached restart
```

## 3. Identity Service on Controller node
### Overview
The Identity service contains these components:
- Server : A centralized server provides authentication and authorization services using a RESTful interface
- Drivers : Drivers or a service back end (for example, SQL databases or LDAP servers).
- Modules : Modules intercept service requests, extract user credentials, and send them to the centralized server for authorization

### Install and configure
- Prerequisites : KEYSTONE_DBPASS
```
$ echo $KEYSTONE_DBPASS
$ mysql -u root -p 
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'KEYSTONE_DBPASS';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'KEYSTONE_DBPASS'
```

- Install and configure components : ADMIN_TOKEN, KEYSTONE_DBPASS
```
# echo "manual" > /etc/init/keystone.override
# apt-get install keystone apache2 libapache2-mod-wsgi -y
# echo $ADMIN_TOKEN $KEYSTONE_DBPASS

# vi /etc/keystone/keystone.conf
[DEFAULT]
admin_token = ADMIN_TOKEN
[database]
connection = mysql+pymysql://keystone:KEYSTONE_DBPASS@controller/keystone
[token]
provider = fernet

# su -s /bin/sh -c "keystone-manage db_sync" keystone
# keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
```
- Configure the Apache HTTP server
```
# vi /etc/apache2/sites-available/wsgi-keystone.conf
ServerName controller
Listen 5000
Listen 35357

<VirtualHost *:5000>
    WSGIDaemonProcess keystone-public processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-public
    WSGIScriptAlias / /usr/bin/keystone-wsgi-public
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    ErrorLogFormat "%{cu}t %M"
    ErrorLog /var/log/apache2/keystone.log
    CustomLog /var/log/apache2/keystone_access.log combined

    <Directory /usr/bin>
        Require all granted
    </Directory>
</VirtualHost>

<VirtualHost *:35357>
    WSGIDaemonProcess keystone-admin processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-admin
    WSGIScriptAlias / /usr/bin/keystone-wsgi-admin
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    ErrorLogFormat "%{cu}t %M"
    ErrorLog /var/log/apache2/keystone.log
    CustomLog /var/log/apache2/keystone_access.log combined

    <Directory /usr/bin>
        Require all granted
    </Directory>
</VirtualHost>

# ln -s /etc/apache2/sites-available/wsgi-keystone.conf /etc/apache2/sites-enabled
```
- Finalize the installation
```
# service apache2 restart
# rm -f /var/lib/keystone/keystone.db
```
### Create the service entity and API endpoints
- Prerequisites
```
$ export OS_TOKEN=$ADMIN_TOKEN
$ export OS_URL=http://controller:35357/v3
$ export OS_IDENTITY_API_VERSION=3
```
- Create the service entity and API endpoints
```
$ openstack service create --name keystone --description "OpenStack Identity" identity
$ openstack endpoint create --region RegionOne identity public http://controller:5000/v3
$ openstack endpoint create --region RegionOne identity internal http://controller:5000/v3
$ openstack endpoint create --region RegionOne identity admin http://controller:35357/v3  
```

- Create a domain, projects, users, and roles
```
$ openstack domain create --description "Default Domain" default

$ openstack project create --domain default --description "Admin Project" admin
$ openstack user create --domain default  --password $ADMIN_PASS admin
$ openstack role create admin
$ openstack role add --project admin --user admin admin

$ openstack project create --domain default --description "Service Project" service
$ openstack project create --domain default --description "Demo Project" demo
$ openstack user create --domain default --password $DEMO_PASS demo
$ openstack role create user
$ openstack role add --project demo --user demo user
```
### Verify operation
```
$ unset OS_TOKEN OS_URL
$ openstack --os-auth-url http://controller:35357/v3 \
  --os-project-domain-name default --os-user-domain-name default \
  --os-project-name admin --os-username admin token issue
$ openstack --os-auth-url http://controller:5000/v3 \
  --os-project-domain-name default --os-user-domain-name default \
  --os-project-name demo --os-username demo token issue
```
### Create OpenStack client environment scripts
- Creating the scripts 
```
$ echo $ADMIN_PASS $DEMO_PASS

$ vi admin-openrc
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS
export OS_AUTH_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2

$ vi demo-openrc
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=DEMO_PASS
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```
Using the scripts
```
$ . admin-openrc
$ openstack token issue
```

## 4. Image Service on Controller node
### Overview
- The Image service includes the following components:
  - glance-api : Accepts Image API calls for image discovery, retrieval, and storage.
  - glance-registry : Stores, processes, and retrieves metadata about images. (A private internal service)
  - Database : Stores image metadata
  - Storage repository for image files : Supported including normal file systems, Object Storage, RADOS block devices, HTTP, and Amazon S3
  - Metadata definition service : A common API to define custom metadata

### Install and configure
- Prerequisites : GLANCE_DBPASS
```
$ echo $GLANCE_DBPASS

$ mysql -u root -p
CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'GLANCE_DBPASS';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'GLANCE_DBPASS';
  
  
$ . admin-openrc  
$ openstack user create --domain default --password $GLANCE_PASS glance
$ openstack role add --project service --user glance admin
$ openstack service create --name glance --description "OpenStack Image" image
$ openstack endpoint create --region RegionOne image public http://controller:9292
$ openstack endpoint create --region RegionOne image internal http://controller:9292
$ openstack endpoint create --region RegionOne image admin http://controller:9292
```
- Install and configure components
```
# apt-get install glance -y

# echo $GLANCE_DBPASS $GLANCE_PASS
# vi /etc/glance/glance-api.conf
[database]
connection = mysql+pymysql://glance:GLANCE_DBPASS@controller/glance
[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = glance
password = GLANCE_PASS
[paste_deploy]
flavor = keystone
[keystone_authtoken]
# Comment out or remove any other options in the [keystone_authtoken] section
[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/

# echo $GLANCE_DBPASS $GLANCE_PASS
# vi /etc/glance/glance-registry.conf
[database]
connection = mysql+pymysql://glance:GLANCE_DBPASS@controller/glance
[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = glance
password = GLANCE_PASS
[paste_deploy]
flavor = keystone

# su -s /bin/sh -c "glance-manage db_sync" glance
```
- Finalize installation
```
# service glance-registry restart
# service glance-api restart
```
### Verify operation

