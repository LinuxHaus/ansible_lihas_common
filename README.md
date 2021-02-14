# ansible_lihas_common
Do basic setup common to all our installations

* set locales
* set timezone
* setup additional repositories
* disable obsolete services on lxc containers
* remove 127.0.0.1-entry from /etc/hosts as this collides e.g. with proxmox-ve
* can setup /etc/host entries
* can setup /etc/network/interfaces
* installs aptitude etckeeper extrepo needrestart rsync screen etckeeper tzdata vim
* can install extra software listed in %.config.software.debian[]
* can copy files/templates listed in %.config.files[]

## Requirements

To run solo:
```
ansible-galaxy install -r requirements.yml
ansible-playbook -i localhost, common.yml
```

## Dependencies

* lihas_variables

## Example Playbook

```
---
- hosts: '*'
  role: lihas_common
...
```
## Variables
* X.config.hosts.IP[]
    * X: string, use host or groupname
    * IP: ip-address
    * []: list of names
* X.config.network
    * if present, overwrite /etc/network/interfaces, supports bridges, vlans and interfaces as such
## Variables example
```
locales_default_environment_locale: de_DE.UTF-8
locales_locales_to_be_generated:
  - de_DE.UTF-8 UTF-8
  - en_US.UTF-8 UTF-8
tzdata_areas: Europe
tzdata_zones_europe: Berlin
XY:
  config:
    hosts:
      "192.168.0.1":
        - example.com
        - examle
    network:
      bridges:
        vmbr9999:
          bridge_type: linux # linux or ovs
          slaves:
            - eth1
      interfaces:
        vmbr9999:
          config_type: static # static, manual, dhcp
          address: 10.10.255.2/24
          up:
            - "ip route add 10.0.0.0/8 via 10.10.255.254 || true"
        eth0:
          config_type: static # static, manual, dhcp
          address: 10.10.0.242/24
          metric: 254
          dns-nameserver: 10.10.1.1
        vlan101:
          config_type: manual
      vlans:
        vlan101:
          rawdevice: eth2
          vlanid: '101'
    software:
      debian:
        - lldpd
    files: []
```
