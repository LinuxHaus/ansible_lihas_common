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
