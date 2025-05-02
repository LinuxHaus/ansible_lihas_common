# ansible_lihas_common
Do basic setup common to all our installations

* set locales
* set timezone
* setup additional repositories
* disable obsolete services on lxc containers
* remove 127.0.0.1-entry from /etc/hosts as this collides e.g. with proxmox-ve
* can setup /etc/host entries
* can setup /etc/network/interfaces
* can setup additional routes via lihas-routes.service
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
## Tags
selectivly run only parts:
* variables: source variables, always use thes when using tags
* extrepo: install/use extrepo
* groups: create groups
* users: create users
* network: do /etc/network/interfaces
* routes: do /etc/systemd/system/lihas-routes.service
* templates: copy extra templates
* templatesperms: copy extra templates with permissions

## Variables
* lihas_network_interface_source_default:
    * add `source /etc/network/interfaces.d/*`, default true
* X.config.hosts.IP[]
    * X: string, use host or groupname
    * IP: ip-address
    * []: list of names
* X.config.network
    * if present, overwrite /etc/network/interfaces, supports bridges, vlans and interfaces as such
* X.config.network.interfaces_rename.{}:
    * rename interface in key to value
* X.config.network.interfaces_extra.[]:
    * add lines at the end of /etc/network/interfaces
* X.config.network.interfaces.{}.auto
* X.config.network.interfaces.{}.config_type
* X.config.network.interfaces.{}.config6_type
* X.config.network.interfaces.{}.address
* X.config.network.interfaces.{}.address6
* X.config.network.interfaces.{}.pointopoint
* X.config.network.interfaces.{}.gateway
* X.config.network.interfaces.{}.gateway6
* X.config.network.interfaces.{}.hwaddress
* X.config.network.interfaces.{}.mtu
* X.config.network.interfaces.{}.metric
* X.config.network.interfaces.{}.dns-nameserver
* X.config.network.interfaces.{}.preup6
* X.config.network.interfaces.{}.up6
* X.config.network.interfaces.{}.down6
* X.config.network.interfaces.{}.postdown6
* X.config.network.interfaces.{}.preup
* X.config.network.interfaces.{}.up
* X.config.network.interfaces.{}.down
* X.config.network.interfaces.{}.postdown
* X.config.network.interfaces.{}.extra: []
* X.config.network.interfaces.{}.comment:
* X.config.network.bridges.{}
* X.config.network.bridges.{}.bridge_type
* X.config.network.bridges.{}.slaves: []
* X.config.network.bridges.{}.stp
* X.config.network.bridges.{}.fd
* X.config.network.bridges.{}.maxwait
* X.config.network.bridges.{}.vlan_aware
* X.config.network.vlans.{}
* X.config.network.vlans.{}.rawdevice
* X.config.network.vlans.{}
* X.config.network.bond.{}.slaves: []
* X.config.network.bond.{}.miimon:
* X.config.network.bond.{}.mode:
* X.config.network.bond.{}.primary:
* X.config.routes.'network/cidr'.gateway
* X.config.routes.'network/cidr'.metric
* lihas_enable_repo: boolen
    * enable lihas extrepo, default false
* X.config.extrepo: []
    * extrepo Repositories to enable
* X.config.software_repository.[].repo:
    * debian repositories to enable
* X.config.fileswithpermissions.[].files: []
    * files to copy
* X.config.fileswithpermissions.[].directories: []
    * directories to create
* X.config.fileswithpermissions.[].perm.owner:
* X.config.fileswithpermissions.[].perm.group:
* X.config.fileswithpermissions.[].perm.mode:
    * owner, group, mode (octal or quoted)
* X.config.fileswithpermissions.[].unsafe_writes:
    * allow unsafe writes in ansible.builtin.templates
* X.config.fileswithpermissions.[].commandi.[]:
    * commands to run after changes
* X.config.fileswithpermissions.[].binary:
    * true: copy as binary file, false: copy as template
* X.config.users."username".uid: 1234
* X.config.users."username".comment: "Karl Koch"
* X.config.users."username".group: primarygroup
* X.config.users."username".groups: group1,group2
* X.config.users."username".home: /home/user
* X.config.users."username".shell: /bin/bash
* X.config.users."username".ssh_authorized_keys: [ 'ssh-rsa abcd demo' ]
* X.config.users."username".sudo: false
    * sudo without password
* X.config.users."username".sudo_selfservice: false
    * sudo without password to use passwd, smbpasswd on own user
* lihas_users_filter: []
    * only create these users
* TODO: lihas_users_filter_exclusive: false
    * remove existing users that would be created without a `lihas_users_filter`

## Variables example
```
locales_default_environment_locale: de_DE.UTF-8
locales_locales_to_be_generated: de_DE.UTF-8 UTF-8, en_US.UTF-8 UTF-8
tzdata_areas: Europe
tzdata_zones_europe: Berlin
XY:
  config:
    groups:
      "groupname":
	gid: 1234
    users:
      "username":
        uid: 1234
        comment: "Karl Koch"
        group: primarygroup
        groups: group1,group2
        home: /home/user
        shell: /bin/bash
        ssh_authorized_keys: [ 'ssh-rsa abcd demo' ]
    hosts:
      "192.168.0.1":
        - example.com
        - examle
    network:
      interfaces_rename:
	enp3s0: wan
      bridges:
        vmbr9999:
          bridge_type: linux # linux or ovs
          slaves:
            - eth1
      interfaces:
        auto: allow-auto
        comment: Comment
        vmbr9999:
          config_type: static # static, manual, dhcp
          address: 10.10.255.2/24
          up:
            - "ip route add 10.0.0.0/8 via 10.10.255.254 || true"
	  extra:
            - "anything"
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
      bond:
        bond0:
          slaves:
            - eth0
            - eth1
          miimon: 100
          mode: active-backup
          primary: eth0

    routes:
      10.0.0.0/8:
        gateway: 192.168.4.1
        metric: 100
    software:
      debian:
        - lldpd
    software_repository:
      repo:
        - "deb https://cloud.r-project.org/bin/linux/ubuntu {{ ansible_distribution_release }}-cran40/"
    systemd:
      service:
        enabled: []
    files: []
```
