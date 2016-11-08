# Test installation & Lauch an Instance

## Test Installation
- Check components
```
$ . admin-openrc
$ openstack service list
+----------------------------------+----------+----------+
| ID                               | Name     | Type     |
+----------------------------------+----------+----------+
| 3441781bc4d24d889d6455d566f5f0da | nova     | compute  |
| 633c4a07572c44e3a4e0e1e94107c8f5 | neutron  | network  |
| 968d3c6d16fe4f019c9772829f4e0930 | keystone | identity |
| f7d1211bfe494c4dba3ac7635094014b | glance   | image    |
+----------------------------------+----------+----------+

$ openstack endpoint list
+----------------------------------+-----------+--------------+--------------+---------+-----------+-------------------------------------------+
| ID                               | Region    | Service Name | Service Type | Enabled | Interface | URL                                       |
+----------------------------------+-----------+--------------+--------------+---------+-----------+-------------------------------------------+
| 1bf9e2508c9a4341a782dbdc04f1659d | RegionOne | neutron      | network      | True    | admin     | http://controller:9696                    |
| 2318893b528c43be87b19164407443aa | RegionOne | nova         | compute      | True    | admin     | http://controller:8774/v2.1/%(tenant_id)s |
| 2802b2010d3444edb348f17da8934cc6 | RegionOne | nova         | compute      | True    | public    | http://controller:8774/v2.1/%(tenant_id)s |
| 39b04865039947468ebb6edd6b2c434c | RegionOne | keystone     | identity     | True    | internal  | http://controller:5000/v3                 |
| 431d826fe27c41ac949467be71573311 | RegionOne | keystone     | identity     | True    | admin     | http://controller:35357/v3                |
| 45f6de7a767b409db041f8853dcf9039 | RegionOne | neutron      | network      | True    | public    | http://controller:9696                    |
| 66e1735c609944d7b168f7388823d536 | RegionOne | neutron      | network      | True    | internal  | http://controller:9696                    |
| 78ed7bc764b84422a6b8e4b12b5bcf19 | RegionOne | nova         | compute      | True    | internal  | http://controller:8774/v2.1/%(tenant_id)s |
| 829d50b46eb44513a6b05be761b8841f | RegionOne | glance       | image        | True    | internal  | http://controller:9292                    |
| a81e172bae464760b4dc0cf16f4bcb79 | RegionOne | keystone     | identity     | True    | public    | http://controller:5000/v3                 |
| d8567e7403de48a980f5da9b759f843b | RegionOne | glance       | image        | True    | public    | http://controller:9292                    |
| efd67816447f4bfc826904b3a466e9c5 | RegionOne | glance       | image        | True    | admin     | http://controller:9292                    |
+----------------------------------+-----------+--------------+--------------+---------+-----------+-------------------------------------------+

$ openstack image list
+--------------------------------------+--------------+--------+
| ID                                   | Name         | Status |
+--------------------------------------+--------------+--------+
| fbcd3aa4-521e-479c-9ad5-f8dae0130348 | ubuntu 14.04 | active |
| 50e86a69-2e39-4348-af32-cd5a42ade9f0 | cirros       | active |
+--------------------------------------+--------------+--------+

$ openstack user list
+----------------------------------+---------+
| ID                               | Name    |
+----------------------------------+---------+
| 1b67bbc6480f4b77870412f70389c092 | admin   |
| 2af759b0655947e7904bf6a6f133b85b | glance  |
| 392bcca0df014c3ca4495f890dc69e57 | demo    |
| 5bf8a034a3b644c482fbe9b15b3e2a71 | nova    |
| af8a1163d80c437da320491f3c13c4ce | neutron |
+----------------------------------+---------+
```

## Lauch an instance
### Create virtual networks
#### Provider network
- Overview

![network1-overview](http://docs.openstack.org/mitaka/install-guide-ubuntu/_images/network1-overview.png)

- Connectivity

![network1-connectivity](http://docs.openstack.org/mitaka/install-guide-ubuntu/_images/network1-connectivity.png)

#### Self-service network 
- Overview

![network2-overview](http://docs.openstack.org/mitaka/install-guide-ubuntu/_images/network2-overview.png)

- Connectivity

![network2-connectivity](http://docs.openstack.org/mitaka/install-guide-ubuntu/_images/network2-connectivity.png)


### Create an instance
```
$ . admin-openrc
$ wget http://cloud-images.ubuntu.com/trusty/current/trusty-server-cloudimg-amd64-disk1.img
$ openstack image create "ubuntu 14.04" \
  --file trusty-server-cloudimg-amd64-disk1.img \
  --disk-format qcow2 --container-format bare \
  --public
$ openstack image list
+--------------------------------------+--------------+--------+
| ID                                   | Name         | Status |
+--------------------------------------+--------------+--------+
| fbcd3aa4-521e-479c-9ad5-f8dae0130348 | ubuntu 14.04 | active |
| 50e86a69-2e39-4348-af32-cd5a42ade9f0 | cirros       | active |
+--------------------------------------+--------------+--------+
```
