# Rebuilding My Homelab: Creating a Functional Enterprise Platform and Learning Environment
## Why I Decided to Rebuild

Over time my homelab had become what many homelabs naturally evolve into: a collection of virtual machines, experiments, and partially completed projects. While it was useful for testing individual technologies, it no longer reflected how infrastructure is managed in a real-world environment.

I decided to rebuild the lab from scratch with two primary goals:

 - Provide a set of services I actually use day-to-day.
 - Create an environment where I can safely learn and experiment with new technologies.

Rather than simply deploying applications, I wanted to build a platform that mimics the layers found in a small enterprise environment. This means establishing proper identity management, storage, networking, DNS, automation and centralised management before introducing Kubernetes and cloud-native applications.

The result will be a lab that is both functional and educational.

## Design Principles

The rebuild is based around several principles:

### Functionality First

Everything deployed should serve a useful purpose.

Examples include:

 - Centralised authentication
 - DNS services
 - Shared storage
 - Documentation systems
 - Source control
 - Monitoring

If a service provides no practical value, it isn't a priority.

### Enterprise Concepts

The objective is not to build a collection of containers but to understand how infrastructure components interact.

Areas of focus include:

 - Identity management
 - DNS
 - Kerberos
 - Storage
 - Automation
 - Kubernetes
 - Monitoring
 - Security

### Learn Through Usage

The best way to learn a technology is to depend on it.

For example:
```
If DNS breaks,
the lab breaks.
```

```
If FreeIPA breaks,
authentication breaks.
```

This creates practical troubleshooting scenarios that are difficult to simulate in isolated tutorials.

## Architecture

The environment will be running on a Proxmox host.

```
Proxmox
│
├── rhel-admin-01.lab.local
├── rhel-storage-01.lab.local
├── ipa01.lab.local
│
├── k8s-control-01.lab.local
├── k8s-worker-01.lab.local
└── k8s-worker-02.lab.local
```

The build is intentionally layered.

Infrastructure services are deployed first.

Applications come later.

## Phase 1: Administration Server

### rhel-admin-01.lab.local

This was the first server built.

Its role is to become the management workstation for the entire platform.

#### Responsibilities
```
SSH Management
Git
Ansible
Podman
General Administration
Future Kubernetes Management
```

#### Specifications
```
2 vCPU
4 GB RAM
40 GB Disk
RHEL 10
```

#### Outcome

This machine provides a central place from which every other system will eventually be managed.

## Phase 2: Storage Server
### rhel-storage-01.lab.local

The second server built was dedicated storage.

Instead of simply adding a large filesystem, I wanted to incorporate RHCSA storage concepts.

#### Specifications
2 vCPU
8 GB RAM

40 GB OS Disk
200 GB Data Disk

#### Technologies Implemented
```
LVM
XFS
NFS
SELinux
Firewalld
```

#### Storage Layout

The storage disk was converted into an LVM volume group:
```
vg_storage
│
├── lv_apps
├── lv_media
├── lv_backups
└── Free Space
```

Logical volumes were formatted using XFS and mounted under:
```
/srv/apps
/srv/media
/srv/backups
```

#### NFS Exports

The server now exports storage to the rest of the environment.
```
/srv/apps
/srv/media
/srv/backups
```

These shares will eventually become Persistent Volumes for Kubernetes workloads.

## Phase 3: Identity Management
### ipa01.lab.local

The third server built was FreeIPA.

This server represents a major shift from a collection of standalone Linux systems to a centrally managed environment.

#### Specifications

```
2 vCPU
8 GB RAM
60 GB Disk
Rocky Linux 10
```

#### Services Provided

```
LDAP
Kerberos
DNS
Certificate Authority
Host Management
```

### DNS Architecture

A dedicated DNS zone was created:

```
lab.local
```

With records including:

```
ipa01.lab.local
rhel-admin-01.lab.local
rhel-storage-01.lab.local
```

Initially host resolution relied on:

```
/etc/hosts
```

However, these records are being moved into FreeIPA DNS.

Resolution is to be centrally managed.

## Key Lessons Learned

Several useful lessons emerged during the build process.

### DNS Comes Earlier Than Expected

Before beginning, I assumed DNS could wait until later.

The reality is that DNS affects almost everything:

```
FreeIPA
Certificates
Kerberos
Kubernetes
Monitoring
```

Introducing DNS early should simplify future integration.

### Build Foundations Before Kubernetes

There is a temptation to deploy Kubernetes immediately.

Resisting that temptation has been valuable.

By the time Kubernetes is introduced, the environment will already have:

```
Identity Management
Storage
DNS
Authentication
```

### Functional Infrastructure Is More Valuable

Many homelabs focus entirely on application deployment.

For this rebuild, the focus is infrastructure first.

Operating and troubleshooting:

```
FreeIPA
NFS
DNS
Kerberos
```

has already provided more practical learning than deploying another application container.

## Current Status

Completed:

```
✅ rhel-admin-01
✅ rhel-storage-01
✅ LVM Storage
✅ XFS Filesystems
✅ NFS Shares
✅ FreeIPA
✅ Kerberos
✅ LDAP
✅ Internal DNS
✅ Internal Certificate Authority
```

## Next Steps

The next phase is integrating the existing servers with FreeIPA.

### Join Existing Servers

Join:

```
rhel-admin-01
rhel-storage-01
```

to the FreeIPA domain.

This will provide:

```
Central Authentication
Kerberos SSO
Host Management
DNS Integration
```

### Build Kubernetes

Deploy:

```
k8s-control-01.lab.local
k8s-worker-01.lab.local
k8s-worker-02.lab.local
```

using Rocky Linux.

Planned configuration:

```
4 vCPU
8 GB RAM
60 GB Disk
```

The cluster will be built using:

```
kubeadm
containerd
```

to align with CKA objectives.

### Platform Services

Once Kubernetes is operational the plan is to deploy:

```
ArgoCD
Prometheus
Grafana
Loki
```

Followed by:

```
Gitea
Vaultwarden
WikiJS
Nextcloud
```

## Conclusion

The rebuild began as an attempt to tidy up an ageing homelab, but I think has already evolved into something much more useful.

Instead of focusing solely on applications, the project now centres around building and operating the supporting infrastructure that modern platforms depend upon.

By establishing storage, identity management and DNS first, the future Kubernetes environment will be built on a realistic and functional foundation, providing both practical services and a valuable learning platform for RHCSA, Kubernetes and platform engineering concepts.

Although manually building DNS and identity services is not how most modern cloud-native platforms are deployed today, the exercise will provide a much deeper understanding of the foundational technologies that Kubernetes depends on.

By configuring FreeIPA, DNS zones and network services from the ground up, it becomes easier to understand how Kubernetes uses DNS internally for service discovery, cluster communication and application routing, making troubleshooting and platform operations far more intuitive.
