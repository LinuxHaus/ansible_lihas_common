# ansible_lihas_common
Do basic setup common to all our installations

## Requirements

To run solo: `ansible-galaxy install -r requirements.yml`

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
    * X: string
    * IP: ip-address
    * []: list of names
* X.config.network
    * if present, overwrite /etc/network/interfaces, supports bridges, vlans and interfaces as such
## Variables example
```
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
``` 
