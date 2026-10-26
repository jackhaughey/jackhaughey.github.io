# Using Ansible to Verify K8s installtion

## Containerd
```
ansible kubernetes \
  -i ansible/inventory.yml \
  -a "systemctl is-active containerd"
```

## Systemd Cgroups
```
ansible kubernetes \
  -i ansible/inventory.yml \
  -a "grep SystemdCgroup /etc/containerd/config.toml"
```

## Kublet
```
ansible kubernetes \
  -i ansible/inventory.yml \
  -a "systemctl is-enabled kubelet"
```


