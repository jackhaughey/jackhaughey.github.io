# Using an Ansible Config File

This configuration means that the inventory file no longer needs to be specified.

```
$ cat ansible.cfg 

[defaults]
inventory = ansible/inventory.yml
```
