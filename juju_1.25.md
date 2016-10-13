# Deploy a charm with Juju to MaaS

## Consideration
- Use different MaaS API key for each Juju environment.

## Env
- Juju 1.25 (Stable version is is 1.25.6)
- 14.04 LTS 

## MaaS UI config. for Juju
- Generate MaaS key 
  - MaaS UI > Account > Generate MaaS Key 
- Add SSH key
  - MaaS UI > Account > Add SSH Key 
  - copy ~/.ssh/id_rsa.pub

## Installing Juju and configuring it to work with MAAS
- Installing Juju
```
sudo apt-get update
sudo apt-get install juju-core -y
sudo apt-get install juju-quickstart juju-deployer charm-tools -y
```
- Configuring Juju to work with MAAS
  - Generate an SSH key
```
ssh-keygen -t rsa -b 2048
```
  - Edit the Juju environments.yaml file
```
juju generate-config
vi .juju/environments.yaml
 default: maas 
 maas:
    type: maas
    maas-server: 'http://172.16.100.1/MAAS/'
    maas-oauth: '--MAAS API key string--'
    authorized-keys-path: ~/.ssh/authorized_keys
juju switch maas 

```
  - Environment testing
```
juju quickstart
```
  - Delete maas environment
```
juju destroy-environment maas
```


## Reference
- Getting Started with Juju 1.25: https://jujucharms.com/docs/1.25/getting-started
- Deploying Production Grade OpenStack with MAAS, Juju and Landscape: https://help.ubuntu.com/lts/clouddocs/en/Intro.html

