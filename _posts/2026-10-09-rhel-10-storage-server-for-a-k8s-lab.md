# Building an RHCSA Storage Server on RHEL 10 for a Kubernetes Homelab

This guide walks through the creation of a dedicated storage server that will later provide NFS-backed storage to a Kubernetes cluster. The configuration also several RHCSA objectives, including:

- LVM administration
- XFS filesystem management
- Persistent mounting
- NFS exports
- Firewalld configuration
- SELinux configuration

## Lab Architecture

```
Proxmox
│
├── rhel-admin-01.lab.local
│
├── rhel-storage-01.lab.local
│ ├── LVM
│ ├── XFS
│ ├── NFS
│ └── SELinux
│
├── k8s-control-01.lab.local
├── k8s-worker-01.lab.local
└── k8s-worker-02.lab.local
```

## VM Configuration
### Proxmox Specification

```
Name: rhel-storage-01.lab.local

vCPU: 2
RAM: 8GB

Disk 1: 40GB
Bus: SCSI 0
Purpose: Operating System

Disk 2: 200GB
Bus: SCSI 1
Purpose: Storage
```

## Install RHEL 10
```
Hostname:
rhel-storage-01.lab.local
 
IP Address:
10.2.0.211
 
Gateway:
10.2.0.1

Minimal install
```

### Initial package installation

First update the system
```
sudo dnf update -y
```

Then install required pakages
```
sudo dnf install -y \
vim \
bash-completion \
qemu-guest-agent \
lvm2 \
xfsprogs \
nfs-utils
```

### Verify Storage Disk
```
lsblk
```

Should display;
```
sda   40G
└─ OS

sdb  200G
```

Confirm with
```
sudo fdisk -l /dev/sdb
```

Which should show something like;
```
Disk /dev/sdb: 200 GiB
```

### Create Physical Volume

Then initialise the disk for LVM:
```
sudo pvcreate /dev/sdb
```

Verify using 
```
sudo pvs
```

### Create Volume Group

Create a volume group
```
sudo vgcreate vg_storage /dev/sdb
```

Verify using 
```
sudo vgs
```

Should display similar to 
```
vg_storage
200G
```

### Create Logical Volumes

Create three logical volumes.

Applications:
```
sudo lvcreate -L 50G -n lv_apps vg_storage
```

Media:
```
sudo lvcreate -L 50G -n lv_media vg_storage
```

Backups:
```
sudo lvcreate -L 50G -n lv_backups vg_storage
```

Verify using
```
sudo lvs
```

Should display the following;
```
lv_apps
lv_media
lv_backups
```

Leaving 50GB free for other potential projects.

### Create XFS Filesystems

Format each logical volume.

Applications:
```
sudo mkfs.xfs /dev/vg_storage/lv_apps
```

Applications:
```
sudo mkfs.xfs /dev/vg_storage/lv_media
```

Backups:
```
sudo mkfs.xfs /dev/vg_storage/lv_backups
```

Verify using;
```
sudo blkid /dev/vg_storage/*
```

Should display;
```
TYPE="xfs"
```

### Next Create Mount Points

Create directories:
```
sudo mkdir -p /srv/apps
sudo mkdir -p /srv/media
sudo mkdir -p /srv/backups
```

Verify using;
```
ls -ld /srv/*
```

### Test Mounts

Mount manually first.
```
sudo mount /dev/vg_storage/lv_apps /srv/apps
sudo mount /dev/vg_storage/lv_media /srv/media
sudo mount /dev/vg_storage/lv_backups /srv/backups
```

Check using;
```
df -h
```

Should show the mounted file shares;
```
/srv/apps
/srv/media
/srv/backups
```

### Configure Persistent Mounts

Edit the fstab file;
```
sudo vi /etc/fstab
```

Then add;
```
/dev/vg_storage/lv_apps     /srv/apps      xfs defaults 0 0
/dev/vg_storage/lv_media    /srv/media     xfs defaults 0 0
/dev/vg_storage/lv_backups  /srv/backups   xfs defaults 0 0
```

Then reload;
```
sudo mount -a
```

### Configure NFS

Install NFS utilities
```
sudo dnf install -y nfs-utils
```

Enable the NFS server:
```
sudo systemctl enable --now nfs-server
```

#### Configure Exports

Edit:
```
sudo vi /etc/exports
```

Then add;
```
/srv/apps     10.2.0.0/24(rw,sync,no_subtree_check)
/srv/media    10.2.0.0/24(rw,sync,no_subtree_check)
/srv/backups  10.2.0.0/24(rw,sync,no_subtree_check)
```

Then apply;
```
sudo exportfs -ra
```

Verify using;
```
sudo exportfs -v
```

### Configure Firewalld

This will allow NFS services;
```
sudo firewall-cmd --permanent --add-service=nfs
sudo firewall-cmd --permanent --add-service=rpc-bind
sudo firewall-cmd --permanent --add-service=mountd
```

Then reload the service;
```
sudo firewall-cmd --reload
```

Verify using;
```
sudo firewall-cmd --list-services
```

Should display;
```
ssh nfs rpc-bind mountd
```

### Configure SELinux

Confirm SELinux is enforcing;
```
getenforce
```

Should show;
```
Enforcing
```

Allow NFS exports;
```
sudo setsebool -P nfs_export_all_rw 1
```

Verify using;
```
getsebool nfs_export_all_rw
```

Should display;
```
nfs_export_all_rw --> on
```

